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

单层 Transformer decoder

1. 层的顺序在 `forward` 函数里面通过代码执行顺序执行。input_layernorm → self_attn → post_attention_layernorm → mlp
2. 多头展开 → 各自计算 → o_proj 融合 → 送给 FFN

todo: 残差 ffn

**Qwen3Attention**

forward 流程：

- hidden_states
- qkv_proj (融合线性层，一次算出 Q/K/V)
- split → q, k, v
- view (reshape 成多头形状: [seq_len, num_heads, head_dim])
- q_norm / k_norm (QK 归一化，仅在无 bias 时)
- rotary_emb (RoPE 旋转位置编码，注入位置信息)
- attn (注意力计算，含 KV Cache 管理)
- o_proj (输出投影，映射回 hidden_size)
output

todo: attn

## Layers

xx