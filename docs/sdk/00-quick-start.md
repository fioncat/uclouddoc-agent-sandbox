# SDK 快速开始
<subtitle>几分钟内完成安装并创建您的第一个沙箱。</subtitle>

本指南将带您完成 UCloud Sandbox SDK 的快速入门，包括 CLI 安装、创建第一个沙箱以及构建自定义模板。

---

## 1. 安装 CLI

确保您的系统中已安装 Node.js 环境。

```bash
npm i -g @ucloud-sdks/ucloud-sandbox-cli
```

安装完成后，可以通过以下命令验证是否成功：

```bash
ucloud-sandbox-cli --help
```

> [!TIP]
> 更多 CLI 功能请参阅 [CLI 完整指南](/agent-sandbox/docs/cli/cli.md)。

---

## 2. 配置环境

在使用 SDK 之前，请确保已配置 `AGENTBOX_API_KEY` 环境变量。

?> 您可以在 [控制台 API 密钥页面](https://console.ucloud.cn/modelverse/experience/api-keys) 获取您的秘钥。

```bash
export AGENTBOX_API_KEY=your_api_key
```

---

## 3. 创建第一个沙箱

使用 Python SDK 快速创建沙箱：

```python
from ucloud_sandbox import Sandbox

# 创建沙箱并设置存活时间为 60 秒
sandbox = Sandbox.create(timeout=60)

# 获取沙箱详细信息
info = sandbox.get_info()
print(info)

# 使用完毕后立即销毁
sandbox.kill()
```

!> 注意：超时的沙箱将由系统自动回收并清理。建议在业务流程结束时手动调用 `kill()` 方法释放资源。

> [!NOTE]
> 更多沙箱生命周期管理请参阅 [沙箱生命周期](/agent-sandbox/docs/sdk/sandbox/01-lifecycle.md)。

---

## 4. 构建自定义模板

模板（Template）是沙箱的蓝图，允许您预装软件、配置环境变量、预置文件。

### 方式一：使用 CLI（推荐）

```bash
# 初始化模板项目
ucloud-sandbox-cli template init

# 构建模板
ucloud-sandbox-cli template build --name my-agent-env
```

### 方式二：使用 Python SDK

**编写模板定义：**

```python
from ucloud_sandbox import Template, wait_for_timeout

template = (
    Template()
    .from_base_image()  # 使用官方预置的基础镜像
    .set_envs({
        "APP_VERSION": "1.0.0",
        "DEBUG": "true"
    })
    .set_start_cmd("echo 'Environment is ready'", wait_for_timeout(5_000))
)
```

**构建并发布：**

```python
from ucloud_sandbox import Template, default_build_logger

Template.build(
    template,
    alias="my-agent-env",
    cpu_count=2,
    memory_mb=1024,
    on_build_logs=default_build_logger(),
)
```

### 使用自定义模板

```python
from ucloud_sandbox import Sandbox

# 使用模板别名创建沙箱
sbx = Sandbox.create(template="my-agent-env")

# 检查环境变量
result = sbx.commands.run("echo $APP_VERSION")
print(f"Version: {result.stdout}")  # 输出: Version: 1.0.0
```

> [!TIP]
> 模板别名是您全局唯一的标识符。更多模板功能请参阅 [模板完整指南](/agent-sandbox/docs/sdk/template/01-quick-start.md)。

---

## 下一步

- [沙箱生命周期管理](/agent-sandbox/docs/sdk/sandbox/01-lifecycle.md) - 了解超时设置与运行监控
- [模板工作原理](/agent-sandbox/docs/sdk/template/02-how-it-works.md) - 深入理解模板机制
- [提升启动速度](/agent-sandbox/docs/sdk/template/04-cache.md) - 优化沙箱启动性能
