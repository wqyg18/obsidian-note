首先将输入句子（如 "The cat sat on the mat"）经过分词器（tokenizer）处理，转换成一系列 token（如 `["The", "cat", "sat", "on", "the", "mat"]`，或更细粒度的子词），每个 token 会被映射为一个对应的 token ID。接着，这些 ID 通过一个 embedding 层转换为向量表示（即词向量），最终加上位置编码后送入 Transformer 网络进行后续处理。

```python
Input: ["Hello", "world", "!"]

X = [
  [0.2, -0.1, 0.5, 1.0],  # "Hello"
  [0.0,  0.4, 0.3, 0.9],  # "world"
  [-0.2, 0.1, 0.6, 0.7]   # "!"
]
```

Q、K、V 的每一行确实对应一个 token（词/子词）的向量表示