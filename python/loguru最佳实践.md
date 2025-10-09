非常好的问题，这其实是 **Loguru 的常见架构设计点**，关系到日志系统的全局一致性和灵活性。  
我来帮你分析两种方式的区别，并给出**推荐做法**。

---

## 🧩 两种方案对比

### ✅ 方案一：在单独模块中定义并配置全局 `logger`

```python
# app/core/logging.py
from loguru import logger
import sys
from pathlib import Path

LOG_DIR = Path("logs")
LOG_DIR.mkdir(exist_ok=True)

logger.remove()  # 移除默认的控制台输出
logger.add(sys.stderr, level="INFO")
logger.add(LOG_DIR / "app.log", rotation="10 MB", retention="7 days", level="DEBUG")

__all__ = ["logger"]
```

然后在其他地方直接使用：

```python
# app/main.py
from app.core.logging import logger

logger.info("Application started")
```

**特点：**

- ✅ 全局唯一 `logger` 实例（推荐）
    
- ✅ 不需要重复调用 `setup_logging()`
    
- ✅ 配置只加载一次（在模块导入时执行）
    
- ⚠️ 如果你想在运行时动态改变配置（如切换日志路径、级别等），就不太方便
    

这种方式是**绝大多数项目的推荐方案**（包括 FastAPI、Flask、Django 等项目结构中）。

---

### ⚙️ 方案二：定义一个 `setup_logging()` 函数

```python
# app/core/logging.py
from loguru import logger
import sys
from pathlib import Path

def setup_logging(log_level="INFO"):
    logger.remove()
    logger.add(sys.stderr, level=log_level)
    logger.add("logs/app.log", rotation="10 MB", retention="7 days", level="DEBUG")
    return logger
```

然后在应用启动时调用：

```python
# app/main.py
from app.core.logging import setup_logging

logger = setup_logging("DEBUG")
logger.info("Application started")
```

**特点：**

- ✅ 灵活，可根据不同环境（开发/生产）动态设置日志配置
    
- ⚠️ 必须确保所有模块都在调用 `setup_logging()` **之后**导入 `logger`
    
- ⚠️ 不小心的话，可能会创建多个配置不同的 `logger`（造成日志重复或丢失）
    

这种方式适合于：

- 需要在**启动阶段动态控制日志配置**（比如 CLI 工具或多进程程序）
    
- 或者在**测试环境中想使用临时日志配置**
    

---

## 🧠 推荐最佳实践（结合两者优点）

在实际工程中，通常采用**方案一 + 初始化函数**的折中方案：

```python
# app/core/logging.py
from loguru import logger
import sys
from pathlib import Path

def setup_logging(level="INFO", log_file="logs/app.log"):
    logger.remove()
    logger.add(sys.stderr, level=level)
    logger.add(Path(log_file), rotation="10 MB", retention="7 days", level="DEBUG")

# 默认加载一次基础配置
setup_logging()

__all__ = ["logger", "setup_logging"]
```

然后在应用入口（比如 FastAPI 的 `main.py`）中：

```python
from app.core.logging import setup_logging, logger

setup_logging(level="DEBUG")  # 若需要重新配置
logger.info("App started")
```

这样：

- 平时只 `from app.core.logging import logger` 即可直接用；
    
- 若有特殊需求（例如测试或生产环境），仍可以调用 `setup_logging()` 重新配置；
    
- 全局 `logger` 始终是同一个对象，不会重复创建。
    

---

## 🧭 总结建议

|场景|推荐方案|
|---|---|
|一般 Web 项目（FastAPI, Flask 等）|✅ 方案一（全局单例 logger）|
|CLI 工具、脚本类项目|⚙️ 方案二（通过 setup_logging 灵活配置）|
|混合场景 / 有多环境配置|💡 折中方案（全局 logger + 可选 setup_logging）|

---

是否希望我帮你基于 FastAPI 项目结构，写一个标准的 `logging.py`（包含彩色控制台、文件日志、异常捕获整合的版本）？这会是生产可用的模板。