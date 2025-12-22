# 构建日志

<subtitle>实时获取模板构建日志并自定义日志处理逻辑。</subtitle>

?> **前置条件**：请先完成 [API Key 配置](/agent-sandbox/docs/product/01-prerequisites.md)

构建过程中会通过 `on_build_logs` 回调实时推送构建日志。您可以使用默认的日志处理器，也可以自定义处理逻辑。

?> 构建入口与状态轮询请参阅：[构建模板](/agent-sandbox/docs/sdk/template/09-build.md)

## 默认日志处理器

SDK 提供 `default_build_logger` 函数，可按级别过滤并输出日志：

```python
from ucloud_sandbox import Template, default_build_logger

Template.build(
    template,
    alias="my-template",
    on_build_logs=default_build_logger(
        min_level="info",  # 要显示的最低日志级别
    )
)
```

## 自定义日志处理器

您可以提供一个回调函数来完全自定义日志的处理逻辑：

```python
import sys
from ucloud_sandbox import Template

# 简单日志记录
on_build_logs=lambda log_entry: print(log_entry)

# 自定义格式化
def custom_logger(log_entry):
    time = log_entry.timestamp.isoformat()
    print(f"[{time}] {log_entry.level.upper()}: {log_entry.message}")

Template.build(template, alias="my-template", on_build_logs=custom_logger)

# 按日志级别过滤
def error_logger(log_entry):
    if log_entry.level in ["error", "warn"]:
        print(f"ERROR/WARNING: {log_entry}", file=sys.stderr)

Template.build(template, alias="my-template", on_build_logs=error_logger)
```

`on_build_logs` 回调接收具有以下属性的结构化 `LogEntry` 对象：

```python
from typing import Literal
from dataclasses import dataclass
from datetime import datetime

LogEntryLevel = Literal["debug", "info", "warn", "error"]

@dataclass
class LogEntry:
    timestamp: datetime
    level: LogEntryLevel
    message: str

    def __str__(self) -> str:  # 返回格式化的日志字符串
        pass


# 指示构建过程的开始
@dataclass
class LogEntryStart(LogEntry):
    level: LogEntryLevel = "debug"

# 指示构建过程的结束
@dataclass
class LogEntryEnd(LogEntry):
    level: LogEntryLevel = "debug"
```

回调函数接收的 `log_entry` 对象包含多种类型。除了通用的 `LogEntry`，还有 `LogEntryStart` 和 `LogEntryEnd`，它们分别标志着构建流程的开始和结束（默认级别为 `debug`）。您可以利用类型检查来处理这些特殊事件：

```python
from ucloud_sandbox import LogEntryStart, LogEntryEnd

if isinstance(log_entry, LogEntryStart):
    # 构建开始
    return

if isinstance(log_entry, LogEntryEnd):
    # 构建结束
    return
```
