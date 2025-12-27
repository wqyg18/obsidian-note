## 一、SchedulerOutput 中真正“关键”的几个参数（极简版）

> **一句话**：  
> SchedulerOutput = **这一轮 GPU 要新增多少 token + 如何为这些 token 分配 KV 空间**

### 1️⃣ `scheduled_new_reqs`

**含义**：  
👉 **第一次进入 GPU 的请求**

**作用**：

- 初始化 request 的 KV cache
    
- 建立 req_id → KV block 映射
    
- 发送 prompt / sampling / LoRA / MM 信息（只发一次）
    

---

### 2️⃣ `scheduled_cached_reqs`

**含义**：  
👉 **已经在 GPU 上存在的请求，继续算**

**作用**：

- 只发送“增量信息”
    
- 告诉 GPU：这些 req 继续 decode / spec decode
    
- 不重复传 prompt 等大数据
    

---

### 3️⃣ `num_scheduled_tokens` ⭐⭐⭐

```python
req_id -> 本轮新增 token 数
```

**含义**：

- 精确描述：**每个 request 在这一轮要算几个 token**
    

**这是 token-centric batching 的核心字段**

---

### 4️⃣ `total_num_scheduled_tokens` ⭐⭐⭐⭐⭐

```python
sum(num_scheduled_tokens.values())
```

**含义**：  
👉 **这一轮 GPU 的真实 batch size（token 数）**

**为什么必须由 Scheduler 算好**：

- PagedAttention 需要提前规划 KV block
    
- kernel launch 前必须知道 batch shape
    
- 多 GPU / 多 worker 必须完全一致
    
- 避免 GPU 内部重复 / 不一致计算
    

---

### 5️⃣ `finished_req_ids`

**含义**：  
👉 **这一轮结束的请求**

**作用**：

- 释放 KV cache / encoder cache
    
- 防止显存泄漏和 OOM
    

---

## 二、核心设计思想（一句话）

> **vLLM 的 batching 是「token-centric」，不是「sequence-centric」**

- 不关心最长序列
    
- 只关心：**这一轮有多少新 token 要算**
    

---

## 三、ASCII 模型（强烈建议你记这个）

### Scheduler → Worker 的真实关系

```
               Scheduler (CPU)
        ┌──────────────────────────┐
        │ scheduled_new_reqs       │
        │ scheduled_cached_reqs    │
        │                          │
        │ num_scheduled_tokens     │
        │   A -> 1                 │
        │   B -> 4                 │
        │   C -> 1                 │
        │                          │
        │ total_num_scheduled_tokens = 6
        │                          │
        │ finished_req_ids = {D}   │
        └──────────────┬───────────┘
                       │
                       ▼
               Worker / GPU
        ┌─────────────────────────┐
        │ 预分配 6 个 KV slots     │
        │ 规划 KV block 地址       │
        │                           │
        │ 执行 attention / decode  │
        │ 写入 6 个新 token        │
        │                           │
        │ 释放 req D 的 KV cache   │
        └─────────────────────────┘
```

---

## 四、一个最小但“完全真实”的例子

### 当前状态

|req|已有 token|本轮|
|---|---|---|
|A|100|decode 1|
|B|50|spec decode 4|
|C|10|decode 1|
|D|20|结束|

---

### SchedulerOutput 关键字段

```python
num_scheduled_tokens = {
    "A": 1,
    "B": 4,
    "C": 1,
}

total_num_scheduled_tokens = 6

finished_req_ids = {"D"}
```

---

### GPU 实际看到的是：

```text
[ A_token_101,
  B_token_51,
  B_token_52,
  B_token_53,
  B_token_54,
  C_token_11 ]
```

👉 **6 个 token = batch size = kernel 的 batch 维度**

没有 padding  
没有“最长序列”  
只有 **flat token batch + KV 映射**
