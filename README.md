# Jarvis

**Local AI agent for Windows & macOS** — single binary, Web console, Feishu bot, tools & skills.  
No Node.js. No Docker required for basic use.

**中文介绍 → [README_CN.md](README_CN.md)**

---

## Download

This repository publishes **pre-built installers only** (no source code).

| Platform | File | Notes |
|----------|------|--------|
| **Windows (x64)** | [`Jarvis-Setup-*-x64.exe`](https://github.com/OWNER/jarvis-agent/releases/latest) | **Recommended** — NSIS wizard installer |
| **Windows portable** | `Jarvis-*-windows-x64-portable.zip` | Unzip and run `JarvisAgent.exe` |
| **macOS (Apple Silicon)** | `Jarvis-Setup-*-macos-aarch64.pkg` | **Recommended** — double-click PKG (like NSIS) |
| **macOS (Intel)** | `Jarvis-Setup-*-macos-x86_64.pkg` | Same as above |
| **macOS portable** | `Jarvis-*-macos-aarch64.tar.gz` | Manual extract + terminal |

👉 **[All releases](https://github.com/OWNER/jarvis-agent/releases/latest)**

Replace `OWNER` with your GitHub username or org when publishing.

---

## Quick install

### Windows

1. Download **Jarvis-Setup-*-x64.exe** from [Releases](https://github.com/OWNER/jarvis-agent/releases/latest).
2. Run the installer → data goes to `%LOCALAPPDATA%\Jarvis\`.
3. Configure via **Web UI** (see below) or edit `config.toml`.
4. Start gateway (Start menu shortcut or `JarvisAgent.exe --gateway`).
5. Open **http://127.0.0.1:8080/webui/**

### macOS (PKG — recommended)

1. Download **Jarvis-Setup-*-macos-*.pkg** from [Releases](https://github.com/OWNER/jarvis-agent/releases/latest).
2. Double-click the PKG → follow the installer → **Jarvis.app** is placed in **Applications**.
3. Open **Jarvis** from Applications (starts gateway automatically).
4. Open **http://127.0.0.1:8080/webui/** → **Settings (配置)** to enter API key and options.

User data: `~/Library/Application Support/Jarvis/config.toml`

### macOS (portable tar.gz)

```bash
tar -xzf Jarvis-*-macos-aarch64.tar.gz && cd jarvis-portable
chmod +x JarvisAgent
./JarvisAgent --init
./JarvisAgent --gateway --config ./config.toml
```

Then open **http://127.0.0.1:8080/webui/**

---

## Web UI configuration (recommended)

After the gateway is running, use the browser console instead of editing TOML by hand.

**URL:** `http://127.0.0.1:8080/webui/`  
(Port matches `[server] listen` in config — default `8080`.)

| UI tab | What to configure |
|--------|-------------------|
| **Settings / 配置** | `[llm]` Provider, API Base, **API Key**, Model · `[server]` Listen, **Allowed dirs**, Skills dir, headless engine |
| **Channels / 通道** | **Feishu** (App ID + Secret, verify & save) · **Telegram** (bot token via wizard) |
| **Skills** | Skills directory, reload, built-in `skills/` |
| **Chat** | Talk to the agent; image upload supported |

### Typical first-time flow

1. Start gateway (installer shortcut, **Jarvis.app**, or `JarvisAgent --gateway`).
2. Open **http://127.0.0.1:8080/webui/**.
3. Click **Settings / 配置** (gear icon in the top bar).
4. Fill in **API Key** and **Model** (e.g. MiniMax-M3) → **Save configuration**.
   - Gateway restarts automatically; the page reloads when ready.
5. (Optional) **Channels / 通道** → Feishu → enter App ID & Secret → **Verify & save**.
6. (Optional) Set **Allowed dirs** to folders the agent may read (comma-separated paths).

### macOS-only (in Settings)

- **Login autostart** — enable gateway on login (LaunchAgent).
- **Screen capture** — grant permission for screenshot tool; use **Open Screen Recording settings**.

### Windows-only (in Settings)

- **Login autostart** — registry / Task Scheduler options where supported.

---

## Manual configuration (`config.toml`)

Config file locations:

| OS | Path |
|----|------|
| Windows (installer) | `%LOCALAPPDATA%\Jarvis\config.toml` |
| macOS (PKG) | `~/Library/Application Support/Jarvis/config.toml` |
| Portable | Next to `JarvisAgent` binary |

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

## What you can do

| Feature | Description |
|---------|-------------|
| **Web UI** | Chat, config, channels, skills, terminal |
| **Feishu / Lark** | Long-connection bot — control your PC from mobile |
| **Tools** | Files, shell, search, git, screenshot, disk/system reports |
| **Skills** | Extend with `SKILL.md` in `skills/` |
| **Safety** | Sensitive actions require **yes** / **no** in chat |
| **LLM** | MiniMax, OpenAI-compatible APIs, local llama.cpp |

---

## Feishu tips

- **One** gateway per Feishu `app_id` (do not run the same bot on two machines).
- Reply **yes** or **no** when asked to authorize shell or directory access.
- Prefer configuring Feishu under **Web UI → Channels** instead of hand-editing secrets.

---

## Verify checksum (optional)

```powershell
# Windows
Get-FileHash .\Jarvis-Setup-0.0.1-x64.exe -Algorithm SHA256
```

```bash
# macOS
shasum -a 256 Jarvis-Setup-0.0.1-macos-aarch64.pkg
```

Compare with the `.sha256` file in Releases.

---

## FAQ

**Q: Is source code in this repo?**  
A: No — releases and docs only.

**Q: Mac equivalent of Windows NSIS?**  
A: **`.pkg` installer** (`Jarvis-Setup-*-macos-*.pkg`). Optional `.dmg` for drag-to-Applications.

**Q: Port in use?**  
A: Change **Listen** in Web UI Settings or `config.toml` (e.g. `127.0.0.1:8081`).

**Q: MCP / Cursor?**  
A: `JarvisAgent --mcp --config path/to/config.toml`

---

## Links

| | |
|---|---|
| 中文文档 | [README_CN.md](README_CN.md) |
| macOS packaging | [docs/PACKAGING-MACOS.md](docs/PACKAGING-MACOS.md) (for maintainers with source) |
| Releases | https://github.com/OWNER/jarvis-agent/releases/latest |

---

## License

MIT — use at your own risk. You are responsible for API usage and commands executed on your machine.
