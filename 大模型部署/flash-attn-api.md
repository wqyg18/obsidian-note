```python
flash_attn_func,
flash_attn_kvpacked_func,
flash_attn_qkvpacked_func,
flash_attn_varlen_func,
flash_attn_varlen_kvpacked_func,
flash_attn_varlen_qkvpacked_func,
flash_attn_with_kvcache,
```

共有这几个API, 其中`flash_attn_with_kvcache`是唯一的带有`kvcache`的`api`
`kvpacked`代表`kv`一起传, `qkvpacked`代表`qkv`一起传

```python
flash_attn_func(q,k,v,...),
flash_attn_kvpacked_func(q,kv,...),
flash_attn_qkvpacked_func(qkv,...),

flash_attn_varlen_func(q,k,v,...,block_table,...),
flash_attn_varlen_kvpacked_func(q,kv,...),
flash_attn_varlen_qkvpacked_func(qkv,...),

flash_attn_with_kvcache(q,k_cache,v_cache,...,block_table,...),
```

目前仅需要看
基于 `nanovllm`的话: 
`flash_attn_func`, 普通的`attention`, 未使用

---

`flash_attn_varlen_func`, 用于`prefill`阶段
`flash_attn_with_kvcache`, 用于`decode`阶段

这两个都支持`block_table`, 即支持`paged attention`


```python
                    FlashAttention (blockwise, IO-aware)

HBM
================================================================================
Q = [Q1, Q2, ..., Qt]
K = [K1, K2, ..., Ks]
V = [V1, V2, ..., Vs]
O = [O1, O2, ..., Ot]
# no full S, no full P stored in HBM
================================================================================


For one query block Qi:

      load Qi
HBM -----------> SRAM
                  ---------------------------------------------
                  Qi
                  mi   (running row max)
                  li   (running row sum / denominator)
                  Oi   (running output)
                  ---------------------------------------------

Then scan all K/V blocks:

      load Kj, Vj
HBM -----------> SRAM
                  ---------------------------------------------
                  Qi, Kj, Vj
                  Sij = Qi @ Kj^T
                  local softmax stats: mij, lij
                  online update: mi, li
                  update output: Oi <- Oi + block contribution
                  ---------------------------------------------

repeat for j = 1 ... s

final write-back:
SRAM -----------> HBM
        Oi
```