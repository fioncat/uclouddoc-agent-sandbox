# 频率限制
<subtitle>了解平台 API 及沙箱操作的访问阈值，确保您的应用平稳运行。</subtitle>

## 配置环境

在使用 SDK 之前，请确保已配置 `AGENTBOX_API_KEY` 环境变量。

?> 您可以在 [控制台 API 密钥页面](https://console.ucloud.cn/modelverse/experience/api-keys) 获取您的密钥。

```bash
export AGENTBOX_API_KEY=your_api_key
```

为了保障服务的稳定性与公平共享，UCloud Sandbox 平台对各类型的 API 调用及资源使用设置了频率限制。

---

## 达到限制时的表现

当您的调用超过设定的阈值时，平台会采取以下保护措施：

*   **HTTP 协议**：直接调用 REST API 或沙箱端口时，将返回 `429 Too Many Requests`，表示已达到配额限制。
*   **Python SDK**：方法调用将抛出 `RateLimitException` 异常。

?> 如遇到频率限制问题，请联系我们团队获取配额调整支持。

## 最佳实践

!> **如何应对限制？**
1.  **指数退避**：在捕获到 `RateLimitException` 时，建议采用指数退避算法（Exponential Backoff）重试。
2.  **及时销毁**：不再使用的沙箱请务必显式调用 `.kill()`，以释放并发配额。
3.  **批量合并**：尽量合并文件读写或相关命令执行，减少短时间内频繁的 API 往返。