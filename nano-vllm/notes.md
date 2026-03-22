# Nano-vLLM

```txt
.
├── bench.py
├── example.py
├── LICENSE
├── nanovllm
│   ├── config.py
│   ├── engine/
│   ├── __init__.py
│   ├── layers/
│   ├── llm.py
│   ├── models/
│   ├── __pycache__
│   ├── sampling_params.py
│   └── utils/
├── pyproject.toml
└── README.md
```

以下分为 `Engine`、`Layers`、`Models` 几个部分介绍

## Engine

xxx

## Models

以 `Qwen3-0.6B` 为例

**Qwen3ForCausalLM**

在 `models/qwen3.py` 下面定义了两个类 `Qwen3Model`、`Qwen3ForCausalLM`。这两个类都继承了 `nn.Module`。`Qwen3Model` 是 Transformer 主体，内部包含 embedding、decoder layer、norm layer 等，将输入的 token 变成 hidden state。（hidden state 是一个语义向量，维度是 hidden_size）

`Qwen3Model` 后面可以接不同的 “头” 来完成不同任务（因果生成、分类、序列标注等）

`Qwen3ForCausalLM` 在 `Qwen3Model` 基础上增加 `lm_head`。`lm_head` 本质是一个线性层，权重矩阵是 `[vocab_size, hidden_size]`。总之，`lm_head` 的权重矩阵的每一行是一个词的 “特征向量”，做矩阵乘是在算 hidden_state 和每个词的相似度，产生 logits 用于后续的采样

```txt
logits = hidden_states @ lm_head.weight.T

[1, hidden_size] × [hidden_size, vocab_size] → [1, vocab_size]
```

`Qwen3ForCausalLM` 类定义了：

```py
packed_modules_mapping = {
    "q_proj": ("qkv_proj", "q"),
    "k_proj": ("qkv_proj", "k"),
    "v_proj": ("qkv_proj", "v"),
    "gate_proj": ("gate_up_proj", 0),
    "up_proj": ("gate_up_proj", 1),
}

def __init__(self, config: Qwen3Config) -> None

def forward(self, input_ids: torch.Tensor, positions: torch.Tensor) -> torch.Tensor

def compute_logits(self, hidden_states: torch.Tensor) -> torch.Tensor
```

1. 将 HuggingFace 格式的权重转换成 vLLM 需要的融合形式。HF 实现将 Attn 和 MLP 的权重分开，Attn 有三个独立矩阵 `q_proj`/`k_proj`/`v_proj`，MLP 是两个独立矩阵 `gate_proj`/`up_proj`。为了提升推理性能，这里定义 dict，将 QKV、GateUp 矩阵拼起来。以 QKV 为例，未融合时需要 3 次 GEMM，融合后需要 1 次 GEMM
2. `__init__` 函数。两个核心组件 `Qwen3Model`、`lm_head`。以及一个特殊配置 `tie_word_embeddings`，如果开启，`lm_head` 和 `embed_tokens` 共享一份权重，可以节省显存
3. `forward` 函数。输入 token，输出 hidden_state
4. `compute_logits` 函数。将 hidden_state 转为 logits

**Qwen3DecoderLayer**

单层 Transformer decoder 实现

1. 搭建 transformer decoder 时，在 `__init__` 函数中定义各层模块， 在 `forward` 方法中显式指定调用顺序。
2. 这里 decoder 的实现和原始 Transformer 不同，在 attn 前有 pre norm，这样训练更稳定，梯度通过残差直接回传，不会被 norm 层卡住，更容易收敛。
input_layernorm → self_attn → post_attention_layernorm → mlp

**Qwen3Attention**

decoder layer 中的注意力层。包含三步：

1. 计算 qkv 向量，以及 rope 位置编码。qkv_proj → split → reshape → QK Norm → RoPE
2. 注意力计算。`attn(Q, K, V)`
3. 多头输出结果投影，映射到 `[seq_len, hidden_size]`。多头展开 → 各自计算 → o_proj 融合 → 送给 FFN

`__init__` 函数：

1. TP 分头计算。将 head 分到 `tp_size` 个 GPU 上计算，
2. 核心组件，qkv 融合矩阵，o 投影矩阵，RoPE，attn 计算。这里采用 GQA 优化 KV Cache 占用
3. 可选的 QK Norm。没有 `qkv_bias` 时，用 RMSNorm 来稳定 QK 的数值范围，可以防止注意力分数爆炸

`forward` 函数：

1. qkv 融合计算。融合的 qkv 投影矩阵的形状 `qkv_proj` 是 `[hidden_size, (num_heads + 2 * num_kv_heads) * head_dim]`。embeddings 和融合的 qkv 投影矩阵相乘后，结果是 `[seq_len, (num_heads + 2 * num_kv_heads) * head_dim]`。
2. split 上述结果，将 qkv 分开。在列上切分 `num_heads*head_dim | num_kv_heads*head_dim | num_kv_heads*head_dim`
3. 将一个长向量拆成多个 head。以 q 为例，`[seq_len, num_heads * head_dim] -> [seq_len, num_heads, head_dim]`。`head_dim` 是每个 head 的独立工作维度
4. 多头输出投影。单个 head 输出 `[seq_len, head_dim]`，head 数量是 `num_heads`（Q 的数量），因此最后多头输出 concat 后是 `[seq_len, head_dim * num_heads]`，`o_proj` 形状是 `[hidden_size, hidden_size]`，注意这里 `hidden_size=head_dim * num_heads`。GQA 中，kv 头数量只影响 kv 存储和计算过程，不影响输出的头数

**Qwen3MLP**

`Qwen3MLP` 是 Transformer 中的 FFN 模块，即 MLP 层

1. 功能：对特征进行 “升维线性变换 -> 非线性激活 -> 降维线性变换” 的加工，增强模型表达能力。以 ReLU 激活为例，$FFN(X)=ReLU(XW_1)W_2$$
2. 流程：
   - 升维：输入 `hidden_state`，输出 `2 * intermediate_state`，分别用于 gate 和 up
   - 激活：使用 SwiGLU
   - 降维：降维，从 `intermediate_state` 到 `hidden_size`

**总结**

整个流程是：

```text
input_ids 
   ↓
Embedding (vocab_size → hidden_size)
   ↓
[ Pre-LN Residual Block ] × num_hidden_layers
   ├── x = x + Attn( RMSNorm(x) )
   ├── x = x + MLP( RMSNorm(x) )   # MLP: SwiGLU + 升维降维
   ↓
Final RMSNorm
   ↓
LM Head (hidden_size → vocab_size)  # compute_logits 调用
   ↓
logits → (optional) softmax → probabilities
```

## Layers

xx