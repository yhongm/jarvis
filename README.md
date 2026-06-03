# Jarvis

**Local AI agent for Windows & macOS** — one native binary, one installer per platform.  
Web console, Feishu / Telegram bots, tools, Skills, MCP, scheduled tasks, headless browser.

**中文介绍 → [README_CN.md](README_CN.md)**

---

## Why Jarvis

| | |
|---|---|
| **Language** | **Rust** — compiled to a single native executable (`JarvisAgent`). No Node.js, no Python runtime bundled for the agent itself. |
| **Distribution** | **Installer only** — Windows NSIS `.exe`, macOS `.pkg`. Same idea on both platforms: install once, config in a fixed user data folder. |
| **Extensibility** | Standard **Skills** (`SKILL.md`), **MCP** (stdio server + external MCP clients), built-in **tools**, **cron-style scheduler**, **headless browser** for web pages. |

### Single installer — pros & cons

| Pros | Cons |
|------|------|
| Fixed paths for config, memory, skills, logs — easy backup & Web UI discovery | Not a “green” portable folder you can copy to a USB stick |
| Start menu / Applications shortcut, uninstaller (Windows), system-integrated layout (macOS) | Each platform/arch needs its own build (x64 Windows, aarch64/x86_64 macOS) |
| Fewer “wrong directory / missing config” support issues | Updates = download a new installer (or maintainer script on dev machines) |
| Matches how most desktop software is shipped | Advanced users cannot drop a zip on a server without using the installer layout |

We intentionally **do not ship portable ZIP / tar.gz** on Releases — only installers.

---

## Download

Pre-built **installers only** (no source code in this repo).

| Platform | Installer |
|----------|-----------|
| **Windows (x64)** | [`Jarvis-Setup-*-x64.exe`](https://github.com/yhongm/jarvis/releases/latest) — NSIS wizard |
| **macOS (Apple Silicon)** | `Jarvis-Setup-*-macos-aarch64.pkg` |
| **macOS (Intel)** | `Jarvis-Setup-*-macos-x86_64.pkg` |

👉 **[All releases](https://github.com/yhongm/jarvis/releases/latest)**  
Replace `yhongm` with your GitHub username or org when publishing.

---

## Quick install

### Windows

1. Download **Jarvis-Setup-*-x64.exe** and run the installer.
2. Data directory: `%LOCALAPPDATA%\Jarvis\` (`config.toml`, `memory/`, `skills/`, `logs/`).
3. Start gateway (Start menu shortcut or `JarvisAgent.exe --gateway`).
4. Open **http://127.0.0.1:8080/webui/** → **Settings** for API key and options.

### macOS

1. Download **Jarvis-Setup-*-macos-*.pkg** and double-click to install.
2. Open **Jarvis** from **Applications** (starts gateway).
3. Open **http://127.0.0.1:8080/webui/** → **配置** for API key and options.

User data: `~/Library/Application Support/Jarvis/config.toml`

---

## Capabilities

### Built-in tools

The agent can call these tools (subject to **allowed dirs** and **yes/no** authorization for sensitive actions):

| Tool | Purpose |
|------|---------|
| **filesystem** | Read, write, list, glob under allowed paths |
| **shell** | Run allowlisted commands; risky ops need your approval |
| **grep** | Search file contents |
| **web_search** | Web search (when configured) |
| **git** | Git operations in allowed repos |
| **screenshot** | Screen capture (OS permissions required) |
| **disk_report** | Disk usage & cache hints |
| **system_report** | CPU / memory / process overview |
| **headless** | Fetch or crawl pages (static HTTP or Chromium) |
| **skill** | Load and route **Skills** |
| **scheduler** | Create and manage **scheduled tasks** |
| **project_scan** | Scan project layout |
| **mcp_list_servers** / **mcp_invoke** | List and call configured **MCP** servers |

### Skills (standard `SKILL.md`)

- Drop skills under the configured **skills directory** (default: user data `skills/` or built-in `skills/`).
- Each skill is a folder with **`SKILL.md`** (YAML frontmatter + instructions), compatible with common agent skill conventions.
- Optional **`scripts/`** — Python or JavaScript helpers auto-registered as tools.
- Manage paths and reload from **Web UI → Skills**.

### MCP (Model Context Protocol)

- **MCP server mode** — expose Jarvis tools to Cursor / other MCP hosts:
  ```bash
  JarvisAgent --mcp --config path/to/config.toml
  ```
- **MCP client** — configure external MCP servers in `config.toml`; agent uses `mcp_list_servers` / `mcp_invoke`.

### Scheduled tasks

- Built-in **scheduler** tool and gateway background runner.
- Define cron-like jobs in Web UI or via agent; notifications can go to Feishu / Telegram when channels are configured.

### Headless browser

- **Static fetch** — fast HTTP/HTML parse without a browser.
- **Chromium** — full JS rendering when Chrome/Edge is installed.
- Choose engine in **Web UI → Settings** (`headless_engine`: `static`, `chromium`, etc.).

### Channels & UI

| Feature | Description |
|---------|-------------|
| **Web UI** | Chat, settings, channels, skills, terminal |
| **Feishu / Lark** | Long-connection bot — control your PC from mobile |
| **Telegram** | Bot token via setup wizard |
| **Safety** | Shell, directory access, and risky ops require **yes** / **no** in chat |
| **LLM** | MiniMax, OpenAI-compatible APIs, local llama.cpp |

---

## Web UI configuration (recommended)

**URL:** `http://127.0.0.1:8080/webui/` (port = `[server] listen`, default `8080`)

| Tab | Configure |
|-----|-----------|
| **Settings / 配置** | LLM provider, API key, model · listen address · **allowed dirs** · skills dir · headless engine |
| **Channels / 通道** | Feishu, Telegram — verify & save |
| **Skills** | Directory, reload, built-in skills |
| **Chat** | Talk to the agent; image upload supported |

### First-time flow

1. Install and start gateway (shortcut or **Jarvis.app**).
2. Open Web UI → **Settings** → API key + model → **Save** (gateway restarts).
3. (Optional) **Channels** → Feishu credentials → **Verify & save**.
4. (Optional) Set **Allowed dirs** for folders the agent may read.

### macOS-only (Settings)

- **Login autostart** (LaunchAgent)
- **Screen capture** — grant permission for screenshot tool

### Windows-only (Settings)

- **Login autostart** where supported

---

## Manual configuration (`config.toml`)

| OS | Path |
|----|------|
| Windows | `%LOCALAPPDATA%\Jarvis\config.toml` |
| macOS | `~/Library/Application Support/Jarvis/config.toml` |

```toml
[llm]
model_provider = "minimax"
api_base = "https://api.minimaxi.com"
api_key = "your-api-key"
model = "MiniMax-M3"

[server]
listen = "127.0.0.1:8080"
allowed_dirs = "/Users/you/projects"

[channels.feishu]
app_id = "cli_xxx"
app_secret = "xxx"
```

Environment variables (optional): `MINIMAX_API_KEY`, `OPENAI_API_KEY`, `JARVIS_CONFIG`, `JARVIS_HOME`.

**Security:** Do not share `config.toml` — it contains API keys.

---

## Feishu tips

- **One** gateway per Feishu `app_id`.
- Reply **yes** or **no** when asked to authorize shell or directory access.
- Prefer **Web UI → Channels** over hand-editing secrets.

---

## Verify checksum (optional)

```powershell
Get-FileHash .\Jarvis-Setup-0.0.1-x64.exe -Algorithm SHA256
```

```bash
shasum -a 256 Jarvis-Setup-0.0.1-macos-aarch64.pkg
```

Compare with the `.sha256` file in Releases.

---

## FAQ

**Q: Is source code in this repo?**  
A: No — releases and user docs only. Developers with source use [DEVELOPMENT.md](DEVELOPMENT.md).

**Q: Why no portable ZIP?**  
A: Installers give a consistent layout and fewer support issues. Power users can still run `JarvisAgent` from a custom path with `--config`.

**Q: Mac equivalent of Windows NSIS?**  
A: **`Jarvis-Setup-*-macos-*.pkg`** (optional `.dmg` for drag-to-Applications when published).

**Q: Port in use?**  
A: Change **Listen** in Web UI Settings or `config.toml`.

**Q: MCP with Cursor?**  
A: `JarvisAgent --mcp --config path/to/config.toml`

---

## Links

| | |
|---|---|
| 中文文档 | [README_CN.md](README_CN.md) |
| Maintainer docs | [DEVELOPMENT.md](DEVELOPMENT.md) |
| macOS packaging | [docs/PACKAGING-MACOS.md](docs/PACKAGING-MACOS.md) |
| Windows packaging | [docs/PACKAGING-NSIS.md](docs/PACKAGING-NSIS.md) |
| Releases | https://github.com/yhongm/jarvis/releases/latest |

---

## License

MIT — use at your own risk. You are responsible for API usage and commands executed on your machine.
