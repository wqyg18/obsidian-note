[Search \| DeepWiki](https://deepwiki.com/search/mpclient_b623a3fd-dca3-4579-a2a0-66393c1ef372?mode=fast)

![image.png](https://cdn.jsdelivr.net/gh/wqyg18/MyPicGo@main/img/20251226173015605.png)


![image.png](https://cdn.jsdelivr.net/gh/wqyg18/MyPicGo@main/img/20251226173026995.png)




## input_sockets 的作用与 ROUTER / DEALER 握手机制

### 1. 角色定义
* **EngineCoreClient**: 使用 **ZMQ ROUTER** socket，作为中心调度与控制端（Control Plane 中枢）。
* **EngineCore**: 使用 **ZMQ DEALER** socket，作为被调度的推理执行进程（GPU Worker）。

---

### 2. 启动与握手顺序
1.  **建立连接**: EngineCore 进程启动，创建 DEALER socket，并 `connect` 到 EngineCoreClient 的 ROUTER 地址。
2.  **身份声明**: EngineCore (DEALER) **必须主动发送一条初始消息**（通常是空帧 `b""`）。
3.  **身份学习**: ROUTER 在收到该消息时，**首次获知**该 DEALER 的 `identity`。
4.  **映射建立**: ROUTER 内部建立 `identity -> connection` 的路由映射表。
5.  **双向就绪**: 从这一刻开始，ROUTER 才能够向该 EngineCore (DEALER) 发送任何后续控制消息。

> **关键点**： **DEALER 必须先发消息**，否则 ROUTER 无法寻址目标 DEALER。

---

### 3. ROUTER / DEALER 的协议语义
* **接收结构**: ROUTER 接收的消息结构为：`[identity][payload...]`。
* **身份分配**: `identity` 由 ZMQ 在 DEALER 第一次发送消息时分配并暴露给 ROUTER。
* **显式寻址**: ROUTER 发送时必须指定目标：`send(identity, payload)`。
* **寻址约束**: 在 `identity` 尚未被学习之前，ROUTER 无法向该 DEALER 发送任何数据。

---

### 4. input_sockets 的职责
`input_sockets` 构成了 EngineCoreClient → EngineCore 的 **控制通道 (Control Plane)**，主要承载以下请求类型：
* **ADD**: 新增推理请求。
* **ABORT**: 中断 / 取消请求。
* **UTILITY**: 状态检查、控制类请求。

**通道特性**：
* **可寻址性**: 基于 per-EngineCore identity 实现精确控制。
* **异步性**: 无 REQ/REP 状态机约束，无需等待应答即可发送下一条。
* **解耦性**: 顺序与业务解耦，由上层调度器保证逻辑顺序。

---

### 5. 设计动机（工程视角）
* **中心化调度**: ROUTER 放在 Client 侧，便于承担“多 EngineCore 统一调度与路由”的角色。
* **非阻塞通信**: DEALER 放在 Worker 侧，使 EngineCore 不受一问一答协议限制，可异步接收指令。
* **路由自发现**: 初始空消息并非业务数据，而是协议层所需的 **路由建表信号**。