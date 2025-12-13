> 适用场景：
> 
> - 网络受限（无法稳定 git clone / pip install）
>     
> - 需要 **环境强一致、结果可复现**
>     
> - 算法 / 科研 / GPU / 集群环境
>     
> - 本地开发 + 容器运行
>     

---

## 一、整体设计思想

本方案遵循 **「环境与代码解耦」** 的核心原则：

- **Dockerfile 只负责环境**（系统、Python、CUDA、依赖）
    
- **业务代码始终由宿主机维护**
    
- **开发阶段使用 bind mount**，避免频繁构建镜像
    
- **发布阶段通过 Dockerfile + COPY 固化代码**
    

这样可以同时满足：

- ✅ 环境可复现
    
- ✅ 开发效率高
    
- ✅ 最终镜像干净、可追溯
    
- ✅ 不依赖外部网络
    

---

## 二、目录结构推荐

```text
project-root/
├── docker/
│   ├── Dockerfile.base        # 基础环境镜像
│   └── Dockerfile.release     # 发布镜像（包含代码）
├── requirements.txt           # Python 依赖（可选）
├── curvenet/                  # 项目代码
│   ├── train.py
│   ├── model/
│   └── utils/
└── README.md
```

---

## 三、步骤 1：构建基础环境镜像（Base Image）

### 目标

- 固化 **系统 + Python + CUDA + 第三方依赖**
    
- 尽量做到 **一次构建，长期复用**
    

### 示例：`Dockerfile.base`

```dockerfile
FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive
ENV LANG=C.UTF-8

# 1. 系统依赖
RUN apt-get update && apt-get install -y \
    build-essential \
    wget \
    curl \
    ca-certificates \
    python3 \
    python3-pip \
    python3-dev \
    && rm -rf /var/lib/apt/lists/*

# 2. Python 依赖（可选）
COPY requirements.txt /tmp/requirements.txt
RUN pip3 install --no-cache-dir -r /tmp/requirements.txt

WORKDIR /workspace
```

### 构建基础镜像

```bash
docker build -f docker/Dockerfile.base -t curvenet:base .
```

📌 **注意**：

- 该镜像 **不包含任何业务代码**
    
- 可以在多台机器 / 多个项目中复用
    

---

## 四、步骤 2：宿主机进行项目开发

### 原则

- 所有代码编辑 **只在宿主机完成**
    
- 使用 VSCode / PyCharm / Vim 等熟悉工具
    
- 项目目录即真实代码源
    

```bash
/home/username/Projects/curvenet
```

---

## 五、步骤 3：开发阶段使用 bind mount 运行容器

### 目标

- 在 **容器环境中运行代码**
    
- 同时 **实时同步宿主机代码修改**
    

### 启动开发容器

```bash
docker run -it --rm \
  -v /home/username/Projects/curvenet:/workspace/curvenet \
  curvenet:base \
  bash
```

进入容器后：

```bash
cd /workspace/curvenet
python3 train.py
```

### 这一阶段的特点

- ✅ 不需要 docker build
    
- ✅ 不需要 docker cp
    
- ✅ 不需要 docker commit
    
- ✅ 修改代码立刻生效
    

📌 **这是整个流程中最频繁使用的阶段**。

---

## 六、步骤 4：发布阶段 —— 构建最终镜像（Release Image）

当你满足以下条件时，进入发布阶段：

- 算法逻辑已稳定
    
- 实验结果可复现
    
- 希望将 **代码 + 环境整体固化**
    

---

### 4.1 发布镜像 Dockerfile

创建 `Dockerfile.release`：

```dockerfile
FROM curvenet:base

# 拷贝项目代码
COPY curvenet /workspace/curvenet

WORKDIR /workspace/curvenet

CMD ["python3", "train.py"]
```

---

### 4.2 构建最终镜像

```bash
docker build -f docker/Dockerfile.release -t curvenet:release .
```

---

### 4.3 使用发布镜像运行

```bash
docker run --rm curvenet:release
```

📌 此时：

- 镜像 **完全自包含**
    
- 不依赖宿主机代码目录
    
- 可直接拷贝 / 导出 / 分发
    

---

## 七、为什么不推荐 `docker cp + docker commit`

|问题|说明|
|---|---|
|不可复现|无法通过 Dockerfile 重建|
|不透明|不清楚代码如何进入镜像|
|不利维护|多人 / 多机难以协作|
|镜像膨胀|易包含缓存、调试残留|

📌 `docker commit` **仅适合作为临时调试或紧急备份手段**。

---

## 八、流程总结（一图胜千言）

```text
        Dockerfile.base
              ↓
       curvenet:base 镜像
              ↓
   bind mount 开发调试阶段
              ↓
        Dockerfile.release
              ↓
      curvenet:release 镜像
```

---

## 九、最佳实践建议

- Base Image：
    
    - 尽量稳定、少改动
        
    - 可打 tag（如 cuda12-py310）
        
- Release Image：
    
    - 一次实验 / 一次版本
        
    - 与论文 / 实验编号对应
        
- 永远不要把：
    
    - 代码修改
        
    - 实验逻辑
        
    - 临时文件
        
    
    混进 base image
    

---

> **一句话总结：**
> 
> 👉 用 Dockerfile 管住环境  
> 👉 用 bind mount 提高开发效率  
> 👉 用 COPY + build 固化最终成果