## falsh-attn
`(2, num_blocks, block_size, num_kv_heads, head_size)`


- `NHD`: the last 3 dimensions are organized as `(seq_len, num_heads, head_dim)`.
    
- `HND`: the last 3 dimensions are organized as `(num_heads, seq_len, head_dim)`.

NHD (2, num_blocks, block_size, num_kv_heads, head_size) 自然顺序 
HND (2, num_blocks, num_kv_heads, block_size, head_size) 某些 kernel 更高效


## 逻辑shape vs 物理layout
可能是不同的, 因为可能存储的时候某些按照某些维度来更加便捷, 但是可能不是很利于人类读取

