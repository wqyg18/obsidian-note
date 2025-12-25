# 大语言模型（LLM）推理笔记：Prefill, Decode 与 KV Cache

## 1. 基础定义与特征提取 (Embedding)

输入一段 token 序列：$[t_1, t_2, \dots, t_N]$

将所有 token 转换为 Embedding 矩阵：
$$X = \begin{bmatrix} x_1 \\ \vdots \\ x_N \end{bmatrix}, \quad x_i \in \mathbb{R}^d, \quad X \in \mathbb{R}^{N \times d}$$

其中：
* $N$：序列长度 (Sequence Length)
* $d$：模型隐藏层维度 (Embedding Dimension)

---

## 2. 注意力机制 (Attention Mechanism)

通过模型权重矩阵 $W_Q, W_K, W_V \in \mathbb{R}^{d \times d}$ 计算得到 $Q, K, V$ 矩阵：

$$Q = XW_Q, \quad K = XW_K, \quad V = XW_V$$
$$Q, K, V \in \mathbb{R}^{N \times d}$$

**Attention 标准公式：**
$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V$$

---

## 3. 推理阶段 (Inference Stages)

大模型的生成是一个自回归 (Autoregressive) 的过程。

* **输入：** "今天吃饭了吗？" (假设长度为 $N$) $\rightarrow$ **输出：** "我" (第 $N+1$ 个 token)
* **下一步输入：** "今天吃饭了吗？我" $\rightarrow$ **输出：** "不" (第 $N+2$ 个 token)
* **循环直到：** 输出 `EOS` (End of String) 标志。

### A. Prefill 阶段 (预填充)
* **任务：** 处理用户输入的全部 $N$ 个 token。
* **过程：** 并行计算前 $N$ 个 token 的 $Q, K, V$ 矩阵。
* **结果：** 输出第一个生成 token ($t_{N+1}$)，并将计算出的 $K$ 和 $V$ 存入 **KV Cache**。

### B. Decode 阶段 (解码)
* **任务：** 逐个生成后续 token。
* **过程：** 利用 **KV Cache** 避免重复计算。

在计算第 $N+1$ 个步（即生成 $t_{N+2}$）时：
1.  **输入：** 仅为上一步生成的单个 token $t_{N+1}$。
2.  **特征提取：** 得到 $x_{n+1}$。
3.  **计算当前的 QKV：**
    * $q_{n+1} = x_{n+1}W_Q$
    * $k_{n+1} = x_{n+1}W_K$
    * $v_{n+1} = x_{n+1}W_V$
4.  **结合 Cache 进行注意力计算：**
    将新的 $k_{n+1}, v_{n+1}$ 拼接（Append）到已有的缓存 $K, V$ 中：
    $$\text{Attention}(q_{n+1}, [K_{cache}; k_{n+1}], [V_{cache}; v_{n+1}]) \Rightarrow t_{n+2}$$

---

## 4. 总结：KV Cache 的必要性
* **Prefill 阶段**是计算密集型（Compute-bound），一次性填满缓存。
* **Decode 阶段**是访存密集型（Memory-bound），通过缓存 $K$ 和 $V$，使得每一轮预测只需要计算当前 token 的 $QKV$，极大地减少了冗余计算量。