
# Visual Studio：Solution 与 Project 的关系笔记

## 1. 基本概念

- 一个 **Solution（.sln）** 可以包含多个 **Project**
- Solution 更像是一个“项目容器 / 工作区”
- Project 才是真正的代码单元（类库、控制台程序、WPF 等）

---

## 2. 示例结构说明

假设有两个项目：

- **ProjectA**
  - 类型：.NET Framework 类库（Class Library）
  - 输出：DLL
  - 作用：提供公共功能

- **ProjectB**
  - 类型：Console / WPF / WinForms 等可执行程序
  - 作用：引用并使用 ProjectA

推荐的目录结构为：

```

MySolution/  
├── MySolution.sln  
├── ProjectA/  
└── ProjectB/

```

> 注意：  
> 创建第一个项目时，**不推荐**选择“Solution 和 Project 在同一路径下”，  
> 否则后续添加多个 Project 时目录结构会比较混乱。

---

## 3. 创建多个 Project 的流程

1. 创建第一个项目（ProjectA）
   - 生成新的 Solution
   - 选择合适的解决方案目录结构

2. 创建第二个项目（ProjectB）
   - 选择 **“添加到现有解决方案”**
   - 让 ProjectB 和 ProjectA 处于同一个 .sln 下

---

## 4. 项目之间的引用方式（重点）

在 **ProjectB** 中引用 **ProjectA**：

1. 右键 ProjectB
2. 选择 **“添加” → “项目引用”**
3. 勾选 ProjectA
4. 确认

这样做的好处：
- 自动处理依赖关系
- 支持调试、重构、跳转代码
- 不需要手动拷贝 DLL

---

## 5. 启动项目设置

- 类库（Class Library）**不能直接运行**
- 需要：
  - 右键 ProjectB
  - 选择 **“设为启动项目”**

然后直接运行 Solution 即可 🎉

---

## 6. 总结一句话

> **Solution 管理多个 Project，  
> Project 之间通过“项目引用”协作，  
> 可执行项目作为启动入口。**

bong！完成 🚀