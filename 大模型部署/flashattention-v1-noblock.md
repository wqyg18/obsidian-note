# FlashAttention 中的 Online Softmax 推导笔记

## 1. Attention 的基本定义

给定 Query、Key、Value：

- $$Q \in \mathbb{R}^{n \times d_k}$$
    
- $$K \in \mathbb{R}^{n \times d_k}$$
    
- $$V \in \mathbb{R}^{n \times d_v}$$
    

> 注意：本文中省略常见的缩放因子  
> $$\frac{1}{\sqrt{d_k}}$$  
> 不影响推导结构。

---

## 2. Softmax Attention 的标准形式

### 2.1 相似度分数（标量）

第 $i$ 个 query 与第 $j$ 个 key 的打分为：

$$  
s_{ij} = q_i \cdot k_j \in \mathbb{R}  
$$

---

### 2.2 注意力权重

对每一行 $i$，softmax 定义为：

$$  
\alpha_{ij}  
= \frac{e^{s_{ij}}}{\sum\limits_{t} e^{s_{it}}}  
$$

---

### 2.3 输出向量

第 $i$ 个输出是 value 的加权和：

$$  
\vec o_i  
= \sum_j \alpha_{ij} \vec v_j  
\quad \in \mathbb{R}^{d_v}  
$$

---

## 3. 数值稳定的 Softmax 重写

### 3.1 行最大值（标量）

为避免指数溢出，引入行最大值：

$$  
m_i = \max_j s_{ij} \in \mathbb{R}  
$$

---

### 3.2 稳定形式的 softmax

$$  
\alpha_{ij}  
= \frac{e^{s_{ij} - m_i}}{\sum\limits_t e^{s_{it} - m_i}}  
$$

---

### 3.3 输出向量的分子 / 分母形式

将 softmax 代入输出定义：

$$  
\vec o_i  
= \frac{  
\sum\limits_j e^{s_{ij} - m_i} \vec v_j  
}{  
\sum\limits_t e^{s_{it} - m_i}  
}  
$$

这里要明确：

- **分子是向量**
    
- **分母是标量**
    
- 最终输出仍是向量
    

---

## 4. Online / Streaming Softmax 的目标

我们希望在 **逐个 $j$ 访问 $(s_{ij}, \vec v_j)$** 的情况下计算：

$$  
\vec o_i  
= \frac{  
\sum_j e^{s_{ij}} \vec v_j  
}{  
\sum_j e^{s_{ij}}  
}  
$$

但不能提前知道全局最大值 $m_i$，因此需要：

- 在线维护最大值
    
- 在线维护分母
    
- 在线维护分子（向量）
    

---

## 5. 引入在线累积变量

对每个固定的 $i$，维护以下量：

- 当前最大值（标量）
    

$$  
m_i  
$$

- softmax 分母的累积（标量）
    

$$  
\ell_i  
= \sum_{t \le j} e^{s_{it} - m_i}  
$$

- softmax 分子的累积（向量）
    

$$  
\vec O_i  
= \sum_{t \le j} e^{s_{it} - m_i} \vec v_t  
$$

### 初始化（尚未看到任何 $j$）

$$  
m_i = -\infty, \quad  
\ell_i = 0, \quad  
\vec O_i = \vec 0  
$$

---

## 6. 关键问题：最大值变化时如何修正历史累积？

当处理新的位置 $j$ 时：

### 6.1 更新最大值

$$  
m_i' = \max(m_i, s_{ij})  
$$

---

### 6.2 统一指数基准

之前累积的是：

$$  
e^{s_{it} - m_i}  
$$

现在要改成：

$$  
e^{s_{it} - m_i'}  
= e^{s_{it} - m_i} \cdot e^{m_i - m_i'}  
$$

也就是说：

> **所有旧的累积项都要乘一个统一的 rescale 因子**

$$  
\exp(m_i - m_i')  
$$

---

## 7. Online Softmax 的递推公式（核心）

### 7.1 分母递推（标量）

# $$  
\ell_i'

\ell_i \cdot e^{m_i - m_i'}  
+  
e^{s_{ij} - m_i'}  
$$

---

### 7.2 分子递推（向量）

# $$  
\vec O_i'

\vec O_i \cdot e^{m_i - m_i'}  
+  
e^{s_{ij} - m_i'} \vec v_j  
$$

说明：

- $e^{m_i - m_i'}$ 是**标量**
    
- 乘到 $\vec O_i$ 上表示对整个向量的缩放
    
- 向量维度始终保持一致
    

---

## 8. 最终输出

当所有 $j$ 都处理完后：

$$  
\boxed{  
\vec o_i = \frac{\vec O_i}{\ell_i}  
}  
$$

这与标准 softmax attention 的定义 **完全等价**。

---

## 9. 推导正确性的核心直觉

1. **Softmax 只依赖相对大小**  
    减去最大值不改变结果
    
2. **最大值变化只引入统一缩放因子**  
    可以一次性修正历史累积
    
3. **分子和分母同步 rescale**  
    比例关系保持不变
    
4. **整个过程是严格等价的数学重写**  
    不是近似，也不是启发式
    

---

## 10. 总结（一句话版）

> FlashAttention 的核心不是「新 attention 公式」，  
> 而是把
> 
> $$  
> \frac{\sum_j e^{s_{ij}} \vec v_j}{\sum_j e^{s_{ij}}}  
> $$
> 
> 重写成 **可在线维护最大值、分母、分子向量的严格等价形式**。

---
