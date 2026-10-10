# 学习通（超星）MCP 接入 Operit

## 三个社区 MCP 地址

- Xuexitong-mcp（原 chaoxing-mcp）：https://github.com/DanMo661/Xuexitong-mcp
- easystudy-mcp：https://github.com/Cindorolla95/easystudy-mcp
- chaoxing-mcp（han-maker-wp）：https://lobehub.com/zh/mcp/han-maker-wp-chaoxing-mcp

> 感谢以上开源作者的无私分享，本教程只是站在他们肩膀上的接入笔记。
>
> 本教程改自 **第一个 MCP（Xuexitong-mcp）** 的实战经验，步骤与它一一对应。

---

## 正文：怎么把学习通 MCP 接进 Operit

**为什么需要折腾**
Operit 在 Android 上只能稳定拉起 `npx (node)`，拉不动 `python3`。
所以官方 stdio 方式 `"command": "python3"` 一直起不来 ——
症状是 `server_status.json` 里该服务 `toolsCachedTime` 永远是 0。

**解法**：把 stdio 的 Python MCP 改造成 **HTTP（streamable-http）远程服务**，
再用 Operit 的 `httpStream` 端点连接。以下以 **Xuexitong-mcp**（第一个 MCP）为样本。

### 1. 加 HTTP 入口

```python
import xuexitong_mcp.server as s
mcp = s.mcp
app = mcp.streamable_http_app()
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="127.0.0.1", port=18002, log_level="warning")
```

### 2. 启动 + 验证

```bash
nohup venv/bin/python xuexitong_http.py > /tmp/xxt_http.log 2>&1 &
curl -s -o /dev/null -w '%{http_code}' -X POST http://127.0.0.1:18002/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream'   # 期望 200
```

### 3. 改 mcp_config.json

- `mcpServers.xuexitong` → `"url": "http://127.0.0.1:18002/mcp"`
- `pluginMetadata.xuexitong` → `type: remote`、`connectionType: httpStream`、`endpoint` 同上

### 4. 重启并验证

`operit_editor:restart_mcp_with_logs` 重启（期望 3 success）→ `ping_mcp` 确认 16 个工具全拉到。

### 排错

| 现象 | 原因 | 处理 |
|---|---|---|
| toolsCachedTime 一直 0 | 拉不起 python3 | 改 HTTP 远程 |
| curl 返回 400 | 请求头不够 | 补 Accept: application/json, text/event-stream |
| 能连但没工具 | endpoint 错 / 服务没起 | 确认监听 + URL 带 /mcp |
| 登录风控 | 高频调用 | 别批量，间隔调用 |

---

教程作者：**Vexis & Slux**