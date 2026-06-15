# Jarvis

**Windows / macOS 本地 AI 智能体** — Rust 单二进制、每平台一个安装包。  
Web 控制台、飞书 / Telegram / **Discord**、内置工具、标准 Skills、MCP、定时任务、无头浏览器。

**English → [README.md](README.md)**

---

## 产品定位

| | |
|---|---|
| **实现** | **Rust** 原生编译为单一可执行文件 `JarvisAgent`，不捆绑 Node.js / Python 运行时。 |
| **分发** | **仅安装包** — Windows NSIS `.exe`、macOS `.pkg`，双平台同一思路：装一次，配置进固定用户目录。 |
| **扩展** | 标准 **Skills**（`SKILL.md`）、**MCP**（stdio 服务端 + 外部 MCP 客户端）、**内置工具**、**定时任务**、**无头浏览器**。 |

### 单一安装包 — 优缺点

| 优点 | 缺点 |
|------|------|
| 配置、记忆、Skills、日志路径固定，便于备份与 Web UI 发现 | 不是解压即用的「绿色便携」目录 |
| 开始菜单 / 应用程序快捷方式、卸载入口（Windows）、系统级目录（macOS） | 各平台/架构需单独构建（Windows x64、macOS arm64/x86_64） |
| 减少「放错目录 / 找不到 config」类问题 | 升级需下载新安装包（开发机可用维护脚本） |
| 与常见桌面软件分发方式一致 | 无法在 Releases 上直接发 ZIP 拷到 U 盘 |

Releases **不提供**便携 ZIP / tar.gz，**仅提供安装包**。

---

## 下载

本仓库仅发布 **编译后的安装包**，不含源码。

| 平台 | 安装包 |
|------|--------|
| **Windows (x64)** | `Jarvis-Setup-*-x64.exe` — NSIS 向导 |
| **macOS (Apple Silicon)** | `Jarvis-Setup-*-macos-aarch64.pkg` |


---

## 快速安装

### Windows

1. 下载并运行 **Jarvis-Setup-*-x64.exe**。
2. 数据目录：`%LOCALAPPDATA%\Jarvis\`（含 `config.toml`、`memory/`、`skills/`、`logs/`）。
3. 启动网关（开始菜单或 `JarvisAgent.exe --gateway`）。
4. 打开 **http://127.0.0.1:8080/webui/** → **配置** 填写 API Key。

### macOS

1. 下载 **Jarvis-Setup-*-macos-*.pkg**，双击安装。
2. 在 **应用程序** 中打开 **Jarvis**（自动启动 gateway）。
3. 浏览器 **http://127.0.0.1:8080/webui/** → **配置**。

**配置文件在哪？以 Web 控制台「配置」页标题旁显示的路径为准**（即当前进程实际读写的文件）。

| 安装 / 启动方式 | 典型路径 |
|-----------------|----------|
| `.pkg` 安装后双击 **Jarvis.app** | `~/Library/Application Support/Jarvis/config.toml` |
| 命令行 `JarvisAgent --gateway`（无 `--config`） | 同上（首次启动会自动创建目录） |
| 仓库内 `cargo run` 或 SSH 部署 | 仓库根目录 `config.toml`（若存在则优先于用户目录） |
| 显式指定 | `--config /path/to/config.toml` 或环境变量 `JARVIS_CONFIG` |

`~` 即 `/Users/你的用户名`。Finder 中按 **⌘⇧G**，粘贴 Web 上显示的完整路径即可打开目录。  
**注意**：`Library` 为隐藏文件夹；若从未成功启动过 Jarvis，用户数据目录可能尚未创建。

---

## 功能概览

### 内置工具

Agent 可调用以下工具（受 **allowed dirs** 与敏感操作 **yes/no** 授权约束）：

| 工具 | 说明 |
|------|------|
| **filesystem** | 在允许目录内读、写、列目录、glob |
| **shell** | 执行白名单命令；高风险操作需授权 |
| **grep** | 文件内容搜索 |
| **web_search** | 联网搜索（需配置） |
| **git** | 允许仓库内的 Git 操作 |
| **screenshot** | 截屏（需系统权限） |
| **disk_report** | 磁盘占用与缓存提示 |
| **system_report** | CPU / 内存 / 进程概览 |
| **headless** | 抓取或爬取网页（静态 HTTP 或 Chromium） |
| **code** | 代码读写/搜索/符号定位/重命名（Tree-sitter AST，类 Cursor Agent） |
| **document** | PDF / Word / Excel / PPT 解析与编辑 |
| **skill** | 加载与路由 **Skills** |
| **scheduler** | 创建与管理 **定时任务** |
| **project_scan** | 项目结构扫描 |
| **mcp_list_servers** / **mcp_invoke** | 列出并调用已配置的 **MCP** 服务 |

### Skills（标准 `SKILL.md`）

- 在配置的 **skills 目录** 放置技能（默认用户目录 `skills/` 或内置 `skills/`）。
- 每个技能为含 **`SKILL.md`**（YAML frontmatter + 说明）的文件夹，与常见 Agent Skill 约定兼容。
- 可选 **`scripts/`** — Python / JavaScript 脚本自动注册为工具。
- 在 **Web UI → Skills** 管理路径与重载。

### MCP（Model Context Protocol）

- **MCP 服务端** — 供 Cursor 等宿主调用 Jarvis 工具：
  ```bash
  JarvisAgent --mcp --config path/to/config.toml
  ```
- **MCP 客户端** — 在 `config.toml` 配置外部 MCP 服务；Agent 通过 `mcp_list_servers` / `mcp_invoke` 调用。

### 定时任务

- 内置 **scheduler** 工具与网关后台调度。
- 可在 Web UI 或由 Agent 创建类 cron 任务；配置通道后可向飞书 / Telegram / **Discord** / Web 推送结果。

### 代码与文档工具（v0.0.6+）

- **`code`** — 统一编码工具：读/写/编辑/搜索、工程树概览、**符号定义/引用**、跨文件重命名；Rust / TS / Python 等由 Tree-sitter 解析。
- **`document`** — 提取或编辑 **PDF、DOCX、XLSX、PPTX**（文本替换、表格读取、Markdown 转 Office 等）。

### 无头浏览器

- **静态抓取** — 无浏览器环境的快速 HTTP/HTML 解析。
- **Chromium** — 本机已安装 Chrome/Edge 时可渲染 JavaScript 页面。
- 在 **Web UI → 配置** 选择引擎（`headless_engine`：`static`、`chromium` 等）。

### 通道与界面

| 功能 | 说明 |
|------|------|
| **Web UI** | 对话、配置、通道向导、Skills、定时任务 |
| **飞书** | 长连接机器人，手机遥控本机 |
| **Telegram** | Bot Token 向导绑定，长轮询收消息 |
| **Discord** | Bot Token 验证、Gateway 长连接、一键生成邀请链接 |
| **企业微信** | Webhook 回调（`config.toml` 或 API） |
| **通道审计** | `logs/channel-audit.jsonl` 记录入站/出站（调试模式更详细） |
| **安全** | Shell、目录访问等需对话中 **yes** / **no** |
| **LLM** | MiniMax、OpenAI 兼容 API、本地 llama.cpp |

---

## Web 控制台配置（推荐）

**地址：** `http://127.0.0.1:8080/webui/`（端口与 `[server] listen` 一致，默认 8080）

| 入口 | 可配置项 |
|------|----------|
| **配置** | LLM、API Key、Model · Listen · **Allowed dirs** · Skills 目录 · 爬虫引擎 |
| **通道** | 飞书、Telegram、**Discord** — 验证 Bot/应用凭证并保存 |
| **Skills** | 目录、重载、内置技能 |
| **对话** | 聊天，支持图片 |

### 首次使用

1. 安装并启动网关。
2. Web UI → **配置** → API Key + Model → **保存**（自动重启）。
3. （可选）**通道** → 飞书 / Telegram / **Discord** → 验证并保存。
4. （可选）设置 **Allowed dirs**。

### macOS 专属（配置页）

- **登录自启动**（LaunchAgent）
- **截图权限** — 系统「屏幕录制」授权

### Windows 专属（配置页）

- **登录自启动**（当前用户注册表 Run）
- **后台运行** — 勾选后隐藏控制台；保存时可自动切换到独立后台进程（关终端不影响 gateway）
- **立即切换到后台** — 配置页一键 detach

---

## 手动编辑 config.toml

| 系统 | 路径 |
|------|------|
| Windows | `%LOCALAPPDATA%\Jarvis\config.toml` |
| macOS | `~/Library/Application Support/Jarvis/config.toml`（=`/Users/<用户名>/Library/Application Support/Jarvis/config.toml`） |

```toml
[llm]
model_provider = "minimax"
api_base = "https://api.minimaxi.com"
api_key = "你的 API Key"
model = "MiniMax-M3"

[server]
listen = "127.0.0.1:8080"
allowed_dirs = "/Users/you/projects"

[channels.feishu]
app_id = "cli_xxx"
app_secret = "xxx"

[channels.discord]
bot_token = "your-discord-bot-token"
```

环境变量：`MINIMAX_API_KEY`、`JARVIS_CONFIG`、`JARVIS_HOME` 等。

---

## 飞书提示

- 每个 `app_id` 只运行 **一台** gateway。
- 授权提示请回复 **yes** / **no**。
- 建议在 **Web UI → 通道** 配置飞书。

## Discord 提示

- 在 [Discord Developer Portal](https://discord.com/developers/applications) 创建应用并启用 **Bot**。
- 开启 **Message Content Intent**（读取消息文本）。
- Web UI **通道 → Discord**：粘贴 Bot Token → 验证 → 保存；向导会给出 **邀请链接**。
- 将 Bot 邀请到你的服务器后，在频道 @Bot 或私信即可对话（与 Web/飞书共用 Agent）。

---

## 校验安装包（可选）

```bash
shasum -a 256 Jarvis-Setup-0.0.6-macos-aarch64.pkg
```

与 Releases 中的 `.sha256` 对比。

---

## 常见问题


**问：Mac 有没有像 NSIS 的一键安装？**  
答：有，**`Jarvis-Setup-*-macos-*.pkg`**。



**问：8080 被占用？**  
答：Web **配置** 或 `config.toml` 修改 Listen。



---

## 链接

| | |
|---|---|
| English | [README.md](README.md) |
---

## 许可证

MIT — 软件按「原样」提供；API 与本机命令执行由使用者自行负责。
