vllm中, 具体路径: `vllm/model_executor/layers/linear.py`中存在很多的`linear`
这些就是线性层, 也就是LLM中常用的, 比如`gate_up`, `down`, `qkv_proj`, `o_proj`等等

这些`linear`就是普通的层, 但是,是实现了各种并行的层, 不同的层对应不同的场景, 使用不同的策略
[Basic Model - vLLM](https://docs.vllm.ai/en/stable/contributing/model/basic/#computation-code)
例如

- [`ReplicatedLinear`](https://docs.vllm.ai/en/stable/api/vllm/model_executor/layers/linear/#vllm.model_executor.layers.linear.ReplicatedLinear "ReplicatedLinear"): Replicates the inputs and weights across multiple GPUs. No memory saving.
- [`RowParallelLinear`](https://docs.vllm.ai/en/stable/api/vllm/model_executor/layers/linear/#vllm.model_executor.layers.linear.RowParallelLinear "RowParallelLinear"): The input tensor is partitioned along the hidden dimension. The weight matrix is partitioned along the rows (input dimension). An _all-reduce_ operation is performed after the matrix multiplication to reduce the results. Typically used for the second FFN layer and the output linear transformation of the attention layer.
- [`ColumnParallelLinear`](https://docs.vllm.ai/en/stable/api/vllm/model_executor/layers/linear/#vllm.model_executor.layers.linear.ColumnParallelLinear "ColumnParallelLinear"): The input tensor is replicated. The weight matrix is partitioned along the columns (output dimension). The result is partitioned along the column dimension. Typically used for the first FFN layer and the separated QKV transformation of the attention layer in the original Transformer.
- [`MergedColumnParallelLinear`](https://docs.vllm.ai/en/stable/api/vllm/model_executor/layers/linear/#vllm.model_executor.layers.linear.MergedColumnParallelLinear "MergedColumnParallelLinear"): Column-parallel linear that merges multiple [`ColumnParallelLinear`](https://docs.vllm.ai/en/stable/api/vllm/model_executor/layers/linear/#vllm.model_executor.layers.linear.ColumnParallelLinear "ColumnParallelLinear") operators. Typically used for the first FFN layer with weighted activation functions (e.g., SiLU). This class handles the sharded weight loading logic of multiple weight matrices.
- [`QKVParallelLinear`](https://docs.vllm.ai/en/stable/api/vllm/model_executor/layers/linear/#vllm.model_executor.layers.linear.QKVParallelLinear "QKVParallelLinear"): Parallel linear layer for the query, key, and value projections of the multi-head and grouped-query attention mechanisms. When number of key/value heads are less than the world size, this class replicates the key/value heads properly. This class handles the weight loading and replication of the weight matrices.

例如:

| 层            | 对应linear                   |
| ------------ | -------------------------- |
| gate_up_proj | MergedColumnParallelLinear |
| down_proj    | RowParallelLinear          |
| qkv_proj     | QKVParallelLinear          |
| o_proj       | RowParallelLinear(         |

定义自己的模型需要:
+ MLP
+ Attention
+ DecoderLayer
+ Model
+ ModelForCausalLM

```powershell
每张 GPU 都有完整 x
        │
        ▼
MergedColumnParallelLinear(gate_up_proj)
        │
        ├── GPU0: gate_0, up_0
        └── GPU1: gate_1, up_1
        │
        ▼
silu(gate_i) * up_i       # 本地做，不通信
        │
        ▼
RowParallelLinear(down_proj)
        │
        ├── GPU0: partial_out_0
        └── GPU1: partial_out_1
        │
        ▼
all_reduce SUM
        │
        ▼
每张 GPU 得到完整 hidden_states
```

[Site Unreachable](https://zhuanlan.zhihu.com/p/1893400554597750507)
[Title Unavailable \| Site Unreachable](https://medium.com/@zdj0712/dive-into-tensor-parallelism-building-columnparallellinear-and-rowparallellinear-from-scratch-cf68ce7332d8)
这篇文章中的图片很直观

![image.png](https://cdn.jsdelivr.net/gh/wqyg18/MyPicGo@main/img/20260501231448659.png)
![image.png](https://cdn.jsdelivr.net/gh/wqyg18/MyPicGo@main/img/20260501231455057.png)
![image.png](https://cdn.jsdelivr.net/gh/wqyg18/MyPicGo@main/img/20260501231437349.png)

用qwen3的MLP来举例子, 

```python
gate_up = gate_up_proj(x)  # 列并行
gate,up = gate_up.chunk(2, dim=-1)

# activate函数在GPU本地计算, 不需要并行
hidden = silu(gate) * up
out = down_proj(hidden) # 行并行
```
```powershell
输入 x
  |
  |  MergedColumnParallelLinear
  |  对应图里的 Column-wise
  v
每张 GPU 得到自己的 gate_up shard
  |
  |  SiluAndMul
  |  每张 GPU 本地做:
  |      gate_i, up_i = chunk(gate_up_i)
  |      hidden_i = silu(gate_i) * up_i
  v
每张 GPU 得到自己的 hidden shard
  |
  |  RowParallelLinear
  |  对应图里的 Row-wise
  v
每张 GPU 计算 partial output
  |
  |  all-reduce sum
  v
每张 GPU 得到完整 out
```

## 并行策略

这些不同的linear, 就是执行的TP(tensor parallel)

对于不同的layer, 比如某k个 hidden layer, 他们之间使用PP(Pipeline Parallel)


Qwen3 dense 结构
```powershell
x
│
├─ residual = x
│
├─ input_layernorm / RMSNorm
│
├─ SelfAttention
│     ├─ q_proj
│     ├─ k_proj
│     ├─ v_proj
│     ├─ q_norm / k_norm
│     ├─ RoPE 加到 Q/K 上
│     ├─ causal attention / GQA / KV cache
│     └─ o_proj
│
├─ x = residual + attention_out
│
├─ residual = x
│
├─ post_attention_layernorm / RMSNorm
│
├─ MLP / SwiGLU
│     ├─ gate_up_proj: hidden_size -> 2 * intermediate_size
│     ├─ split:
│     │     gate, up
│     ├─ SiLU(gate) * up
│     └─ down_proj: intermediate_size -> hidden_size
│
└─ x = residual + mlp_out
```
