```python
qkv_proj -> split -> q_norm/k_norm -> rotary_emb(positions, q, k) -> Attention(q,k,v)
```
