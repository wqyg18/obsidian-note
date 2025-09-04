pytorch2.7+版本需要glibc2.18+

对于glibc2.17-版本:
```toml
dependencies = [
  "torch==2.6.0",
  "torchvision>=0.21.0",
  "torchaudio==2.6.0",
]

[tool.uv.sources]
torch = [
    { index = "pytorch-cu118" },
]
torchvision = [
    { index = "pytorch-cu118" },
]
torchaudio = [
	{ index = "pytorch-cu118" },
]

[[tool.uv.index]]
name = "pytorch-cu118"
url = "https://download.pytorch.org/whl/cu118"
explicit = true
```
