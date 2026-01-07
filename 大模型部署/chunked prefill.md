---

# Chunked Prefill：统一理解笔记（含数学形式）

---

## 0. 核心一句话（先记住）

> Chunked prefill = 将标准 prefill 的 query 维度按块（chunk）拆分，
> 
> 每一块独立做一次 causal self-attention，
> 
> 目标只是构建 KV cache，而不一定生成 token。

---

## 1. 自回归 Transformer 的统一数学定义（背景）

$attention(Q, K\_cache, V\_cache)$

- $Q.shape = [Nq, d]$
    
- $Nq = 1 \rightarrow$ decode-style path
    
- $Nq > 1 \rightarrow$ prefill-style path
    

给定 token 序列：

$$[x_1, x_2, \dots, x_n]$$

Embedding 后：

$$X = [x_1, \dots, x_n] \in \mathbb{R}^{n \times d}$$

线性映射：

$$Q = XW_Q,\quad K = XW_K,\quad V = XW_V$$

Causal self-attention 定义为：

$$\mathrm{Attn}(Q,K,V) = \mathrm{softmax}\left( \frac{QK^T}{\sqrt{d}} - M \right)V$$

其中：

- softmax **逐行**进行。
    
- 输出是 **hidden states**，不是 token。
    

---

## 2. 标准 Prefill（作为基准）

### 数学形式

$$Q \in \mathbb{R}^{n \times d},\quad K \in \mathbb{R}^{n \times d},\quad V \in \mathbb{R}^{n \times d}$$

Attention 输出：

$$H = [h_1, \dots, h_n] \in \mathbb{R}^{n \times d}$$

### 语义

- 计算 **所有位置的 hidden state**。
    
- 写入 **所有位置的 KV cache**。
    
- **token 不是必需产物**：
    
    - 推理时通常只关心最后一个 $h_n$。
        
    - 甚至可以完全不走 LM head。
        

---

## 3. Decode（仅用于对照）

### 数学形式

在已有 cache 的前提下，仅计算一个新位置：

$$q_{n+1} \in \mathbb{R}^{1 \times d}$$

$$h_{n+1} = \mathrm{Attn} \bigl( q_{n+1}, [K_{\le n}; k_{n+1}], [V_{\le n}; v_{n+1}] \bigr)$$

随后：

$$h_{n+1} \xrightarrow{\text{LM head}} \text{logits} \rightarrow \text{token}$$

### 关键点

- 只算 **1 个 query**。
    
- **必须**走 LM head。
    
- 目标是生成 token。
    

---

## 4. Chunked Prefill 的严格数学定义（重点）

### 4.1 分块定义

设：

- prompt 长度为 $n$
    
- chunk size 为 $c$
    

将序列划分为：

$$X = \begin{bmatrix} X^{(1)} \\ X^{(2)} \\ \vdots \\ X^{(N)} \end{bmatrix}, \quad X^{(i)} \in \mathbb{R}^{c_i \times d}$$

其中 $c_i = c$（最后一块可能更小）。

---

### 4.2 第 $i$ 个 chunk 的计算

#### Query（矩阵）

$$Q^{(i)} = X^{(i)} W_Q \in \mathbb{R}^{c_i \times d}$$

**含义：** 这一 chunk 内 **每一个新位置** 都有一个 query。

---

#### Key / Value（历史 + 当前）

$$K^{(\le i)} = \begin{bmatrix} X^{(1)} \\ \vdots \\ X^{(i)} \end{bmatrix} W_K = [K_{\text{cache}}; K^{(i)}]$$

$$V^{(\le i)} = [V_{\text{cache}}; V^{(i)}]$$

- 旧 KV：直接从 cache 读。
    
- 当前 chunk 的 KV：现算并写入 cache。
    

---

#### Attention（核心公式）

$$\boxed{ H^{(i)} = \mathrm{softmax} \left( \frac{Q^{(i)} (K^{(\le i)})^T}{\sqrt d} - M^{(i)} \right) V^{(\le i)} }$$

输出：

$$H^{(i)} = [h_{(i-1)c+1}, \dots, h_{ic}] \in \mathbb{R}^{c_i \times d}$$

---

### 4.3 计算结果的使用方式（非常关键）

- **一定会做的：** 写入 KV cache（这是 prefill 的核心目标）。
    
- **不一定会做的：** 对 $h_{ic}$ 走 LM head 并生成 token。
    

> **chunked prefill 的本质目标不是生成 token，而是补齐 KV。**

---

## 5. Chunked Prefill 与 Prefill / Decode 的关系

### 5.1 数学等价性

将所有 chunk 的输出拼接：

$$H = \begin{bmatrix} H^{(1)} \\ H^{(2)} \\ \vdots \\ H^{(N)} \end{bmatrix}$$

则：

$$H = \mathrm{Attn}(Q,K,V)$$

> **chunked prefill 在数学结果上与完整 prefill 完全一致（忽略数值误差）。**

---

### 5.2 特例关系

- **Decode = chunked prefill 的极限情况**：当 $c = 1$ 时。
    

---

## 6. 工程语义总结（非常重要）

- Chunked prefill **不是新模型**。
    
- 不改变 attention 的数学定义。
    
- 只是改变：
    
    - **一次算多少个 query**。
        
    - **KV cache 的构建节奏**。
        

---

## 7. 一句话终极总结（可直接背）

> Chunked prefill 是一种执行策略：
> 
> 将标准 prefill 的多 query attention 拆成多次小规模 attention；
> 
> 每次用当前 chunk 的 $Q$（矩阵）attend 到“历史 + 当前”的 KV；
> 
> 目标只是构建 KV cache，token 生成不是必须步骤。

---

如果你愿意，下一步我可以帮你把这份笔记：

- **直接映射到 vLLM / PagedAttention 的 kernel 输入张量描述**
    
- 或压缩成 **3 条“面试级”核心结论句**。