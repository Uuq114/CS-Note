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

以 `Qwen3-0.6B` 为例，`config.json` 定义了架构是 `Qwen3ForCausalLM`

在`models/qwen3.py`下面定义了：

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

首先是`packed_modules_mapping`，



## Layers

xx