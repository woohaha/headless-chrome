# Docker Chrome 远程调试环境

## 项目目的

提供一个容器化的 Chrome 浏览器环境，支持：
- 通过 Web GUI (KasmVNC) 访问 Chrome 桌面
- 通过 Chrome DevTools Protocol (CDP) 进行远程调试
- 让宿主机的 MCP chrome-devtools 工具连接容器内的 Chrome

## 架构

```
宿主机:9222 → Docker(9222:9223) → 容器:9223(Python转发器) → 容器:9222(Chrome)
```

### 为什么需要端口转发器？

Chrome 143+ 忽略 `--remote-debugging-address=0.0.0.0` 参数，只监听 `127.0.0.1`。
使用 Python 脚本将 `0.0.0.0:9223` 转发到 `127.0.0.1:9222`，使外部可访问。

## 项目结构

```
.
├── docker-compose.yml      # Docker Compose 配置
├── .mcp.json               # MCP 服务器配置（chrome-devtools 连接参数）
├── .claude/
│   └── settings.local.json # Claude Code 本地权限配置
└── config/                 # Chrome 容器持久化配置（挂载到 /config）
    ├── port-forward.py     # Python 端口转发脚本
    ├── chrome-debug-profile/  # Chrome 用户数据目录（启用远程调试需要）
    └── .config/
        └── openbox/
            └── autostart   # 容器启动时自动运行的脚本
```

## 端口说明

| 宿主机端口 | 容器端口 | 用途 |
|-----------|---------|------|
| 3000 | 3000 | KasmVNC Web GUI (HTTP) |
| 3001 | 3001 | KasmVNC Web GUI (HTTPS) |
| 9222 | 9223 | Chrome DevTools Protocol |

## 使用方法

### 启动容器
```bash
docker compose up -d
```

### 访问 Chrome GUI
浏览器打开 `http://localhost:3000`

### 验证远程调试
```bash
curl http://localhost:9222/json/version
curl http://localhost:9222/json/list
```

### MCP chrome-devtools 连接
配置已在 `.mcp.json` 中设置，重启 Claude Code 后自动连接。

## 关键配置文件

### config/.config/openbox/autostart
容器内 Chrome 启动脚本，包含：
- 启动端口转发器
- 启动 Chrome 并启用远程调试 (`--remote-debugging-port=9222`)
- 使用独立的用户数据目录 (`--user-data-dir=/config/chrome-debug-profile`)

### config/port-forward.py
Python TCP 端口转发器：`0.0.0.0:9223` → `127.0.0.1:9222`
