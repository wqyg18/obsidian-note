trae 配置
```json
{

  "mcpServers": {

    "serena": {

      "command": "uvx",

      "args": [

        "--from",

        "git+https://github.com/oraios/serena",

        "serena",

        "start-mcp-server"

      ]

    }

  }

}
```

好的，下面我为你系统地介绍一下 **Serena MCP** 的所有工具，按照类别整理框架，先提供整体结构，再详细讲解每一个工具，并指出其中哪些是按流程串联起来、常用于实际操作流程中的工具。

---

## 一、整体框架结构

我们可以把 Serena MCP 工具分为以下几个主要类别：

1. **项目与Onboarding 工具**  
    — 与项目初始化、激活、启动上下文记忆相关的工具。
    
2. **代码探索与语义检索工具**  
    — 用于查找符号、引用、目录结构等，实现对代码的理解。
    
3. **代码编辑与修改工具**  
    — 对代码进行插入、替换、删除、符号体替换等操作。
    
4. **Shell 执行与交互工具**  
    — 在项目环境中运行命令或脚本，帮助调试或触发测试。
    
5. **记忆（Memory）管理工具**  
    — 管理上下文记忆，为后续操作提供支持。
    
6. **“思考”辅助工具（Think 工具）**  
    — 帮助评估任务状态、分析信息，推动流程进展。
    

---

## 二、详细工具介绍

以下是各类别对应的工具，按功能依次说明：

### 1. 项目与 Onboarding 工具

这些工具通常在使用开始时按流程串联使用：

- **activate_project**：激活并加载某个项目，使其成为当前上下文。
    
- **onboarding**：执行项目入门流程，初始化设置并创建记忆结构。
    
- **check_onboarding_performed**：检查 onboarding 是否已完成。
    
- **prepare_for_new_conversation**：为新的对话准备上下文状态。
    

_这些工具组成常见的“项目上车（onboard）”流程，LLM 通常会依次调用它们来准备操作环境。_

---

### 2. 代码探索与语义检索工具

- **find_symbol**：在代码库中根据名称路径查找符号（类、方法等），支持过滤和限定路径。
    
- **find_referencing_symbols**：找到引用某个符号的位置，包括上下文片段等元信息。
    
- **get_symbols_overview**：获取项目或目录的顶级符号概览，帮助了解结构。
    
- **find_file** / **list_dir**：浏览目录结构、查找文件。
    

_这些工具使代理能够像 IDE 一样“看得懂”代码结构，在操作之前了解上下文。_

---

### 3. 代码编辑与修改工具

- **insert_after_symbol** / **insert_before_symbol**：在某个符号之后或之前插入代码块。
    
- **replace_symbol_body**：替换符号的整个主体，实现精确编辑。
    
- **replace_regex**：对代码进行正则替换操作。
    

_编辑流程中，LLM 常常先用探索工具定位位置，再调用这些工具对代码进行增删改。_

---

### 4. Shell 执行与交互工具

- **execute_shell_command**：在代码环境中执行如 `pytest` 等 shell 命令，并获取结果。
    

_这是一个流程节点：编辑之后往往需要运行测试或脚本，验证修改效果。_

---

### 5. 记忆管理工具

- **write_memory** / **read_memory**：写入或读取上下文记忆。
    
- **list_memories** / **delete_memory**：列出记忆条目或删除旧记忆。
    

_这些工具支持 Serena 的记忆能力，使得对话或操作具备“记住项目历史”的能力，提升智能和连贯性。_

---

### 6. “思考”辅助工具

- **think_about_collected_information** / **think_about_task_adherence** / **think_about_whether_you_are_done**：让模型反思当前信息、任务是否符合预期、是否已完成。
    

_这些工具常嵌入在流程节点之间，帮助模型自主判断该执行下一步什么，例如判断是否需要继续编辑或可以提交。_

---

## 工具按流程串联示例（典型操作流程）

1. **Onboarding 阶段**  
    调用 `onboarding` → `activate_project` → `check_onboarding_performed`，设定项目上下文。
    
2. **代码探索阶段**  
    使用 `get_symbols_overview` → 定位目标符号 `find_symbol` → 查看引用 `find_referencing_symbols`。
    
3. **编辑实施阶段**  
    定位后执行 `insert_after_symbol` 或 `replace_symbol_body` / `replace_regex`。
    
4. **验证阶段**  
    使用 `execute_shell_command` 运行测试，获取反馈。
    
5. **反思或状态判断**  
    调用 `think_about_collected_information` / `think_about_whether_you_are_done` 评估是否继续。
    
6. **记忆存储阶段**  
    如果需要保留结果，使用 `write_memory`，后续可读取 `read_memory`。
    

---

## 三、总结一览表

|类别|工具列表|流程作用简介|
|---|---|---|
|项目 Onboarding|`activate_project`, `onboarding`, `check_onboarding_performed`, `prepare_for_new_conversation`|初始化项目上下文|
|代码探索工具|`find_symbol`, `find_referencing_symbols`, `get_symbols_overview`, `find_file`, `list_dir`|定位代码结构与符号|
|编辑与修改|`insert_after_symbol`, `insert_before_symbol`, `replace_symbol_body`, `replace_regex`|精准修改代码|
|Shell 交互工具|`execute_shell_command`|运行命令/测试验证|
|记忆管理|`write_memory`, `read_memory`, `list_memories`, `delete_memory`|存储与调用上下文记忆|
|思考辅助工具|`think_about_collected_information`, `…_task_adherence`, `…_whether_you_are_done`|自主判断任务进展|

---

## 四、使用建议

- 如果你在一个流程里使用 Serena MCP，可以按照上表结构组织 prompt 或指令，合理串联每一步。
    
- 你还可以根据项目需求，定制 `project.yml`，开启或禁用特定工具（例如只读模式、关闭 shell 执行等）
    

---

希望这个结构清晰、分类明确、一目了然地帮助你理解 Serena MCP 的工具体系和流程串联方式！如果你对某个具体工具使用场景或参数细节感兴趣，也欢迎进一步询问。