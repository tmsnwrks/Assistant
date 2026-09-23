# ThomsonWorks DeskPilot

> **The ultra-fast, local-first AI desktop agent — a headless Rust daemon with an Obsidian Velocity Web Studio.**
>
> *Official repository by [ThomsonWorks](https://github.com/thomsonworks-in).*

DeskPilot runs as a **tiny background daemon** (`~10MB binary, ~12MB RAM idle`) and serves a polished web UI at `http://127.0.0.1:31415`. It auto-discovers every local [Ollama](https://ollama.com) model on startup, routes messages to your chosen model, surfaces full thinking-chain traces, and persists all history in local SQLite — **with zero data ever leaving your machine.**

![DeskPilot Obsidian Velocity Web Studio](assets/deskpilot_studio.png)

---

## ⚡ Why DeskPilot? (Key Features)

| Feature | Details |
| :--- | :--- |
| 🚀 **~10MB Binary · ~12MB RAM Idle** | Pure Rust headless daemon. 56% smaller than a native GUI app. Starts in milliseconds. |
| 🔒 **100% Private · Zero Telemetry** | All data stays on your machine. Works fully offline. No accounts. No subscriptions. |
| 💓 **Autonomous Heartbeat Daemon** | Background wake-loop (`HeartbeatRunner`) periodically executes scheduled triggers & workspace routines autonomously. |
| 🛡️ **Execution Watchdog & Circuit-Breaker** | `RunWatchdog` prevents runaway loops, infinite turns, and hung tool calls across agent runs. |
| 🔍 **Interactive Diff Verification** | Audit agent workspace changes via inline Git diff modals directly on Living Spec milestones. |
| 🧠 **Thinking-Chain Traces** | Collapsible 💡 reasoning traces shown inline — see exactly how the model thinks. |
| 🤖 **Auto Local Model Detection** | Detects all running Ollama models at launch. Switch models from a live dropdown. |
| 💾 **SQLite WAL Persistence** | 12-message rolling context window. All conversations stored locally across reboots. |
| ⚡ **`dp` — 2-Character Launcher** | Type `dp` to start or focus DeskPilot. Smart: boots if stopped, focuses if running. |
| 🌐 **Obsidian Velocity Web Studio** | Muted amethyst dark-mode UI at `localhost:31415`. Works in any browser, no install. |
| 🔀 **Multi-Model Routing** | Route to local Ollama models or cloud frontier models (OpenRouter, Anthropic, OpenAI). |
| 🧭 **Read-Only Plan Mode** | Inspect the workspace and generate an ordered Living Spec roadmap before any file, shell, Git, network, MCP, or subagent mutation. |
| 📦 **Declarative Workspace Manifests** | Per-repository .deskpilot/workspace.json (Google x-inspired) configuration for model overrides, verification commands, egress controls, and prompt rules. |
| 🛡️ **Interactive Tool Approval Gates** | Granular interactive user approval for subagent delegation and mutation tools with session token synchronization and permanent allow-rules. |

---

## 🚀 1-Click Install & Run (Zero Dependencies)

No Node.js, Rust, Python, or Git required. Single binary installation that automatically sets up DeskPilot, creates shortcuts, and launches the Obsidian Velocity Web Studio in your browser.

### Windows (PowerShell):
Open PowerShell and run:
```powershell
irm https://raw.githubusercontent.com/tmsnwrks/Assistant/main/install.ps1 | iex
```
*(Or from Command Prompt `cmd.exe`: `powershell -c "irm https://raw.githubusercontent.com/tmsnwrks/Assistant/main/install.ps1 | iex"`)*  
*Downloads pre-compiled binary to `%LOCALAPPDATA%\DeskPilot\bin`, configures PATH, creates the `dp` launcher, and instantly opens `http://127.0.0.1:31415`.*

### macOS & Linux (Terminal):
Open Terminal and run:
```bash
curl -fsSL https://raw.githubusercontent.com/tmsnwrks/Assistant/main/install.sh | sh
```
*Supports Apple Silicon (M1–M4), Intel Macs, and Linux. Automatically adds `dp` to your shell PATH and opens the Web Studio.*

---

## ⚡ Ultra-Short One-Liner Run Command: `dp`

Once installed, launch DeskPilot anytime with just **2 characters**:

```bash
dp
```

- **If DeskPilot is stopped:** `dp` boots the headless daemon in milliseconds and opens `http://127.0.0.1:31415`.
- **If DeskPilot is already running:** `dp` instantly brings the Web Studio into focus in your browser.
- **Headless Background Service (no browser):**
  ```bash
  deskpilot --headless
  ```

---

## 🔌 Supported Engines & Models

| Mode | Supported Runtimes & Providers | Example Models |
| :--- | :--- | :--- |
| **Local (Private / Offline)** | [Ollama](https://ollama.com), any OpenAI-compatible local runner | Qwen3, DeepSeek-R1, Llama 3.3, Mistral, Gemma 4 |
| **Cloud (Frontier Reasoning)** | [OpenRouter](https://openrouter.ai), DeepSeek API, Anthropic, OpenAI | Claude Sonnet, DeepSeek-V3/R1, GPT-4o, Gemini |
| **Semantic Memory** | SQLite WAL with rolling context + project adaptive context | Zero-latency local retrieval |

---

## 🛠️ Usage & Navigation

- **💬 Chat:** Conversational canvas with user/assistant bubbles, per-model timing badge, and collapsible 💡 thinking-chain traces.
- **🧭 Plan Mode:** Select `Plan Mode` in the composer to generate a read-only implementation roadmap. Review the imported objectives in Living Spec, then choose **Approve Plan** to return to Guided execution.
- **🤖 Model Selector:** Live dropdown in the header — grouped into `⚡ Local Offline (Zero Cost)` and `🌐 Cloud Frontier` with `💡 Reasoning` / `⚡ Fast` badges.
- **⚙ Settings:** Configure API keys, default model, and view local runtime status via `/api/settings`.
- **🛠 REST API:** Full JSON API at `:31415` — `/api/models`, `/api/chat`, `/api/settings`, `/api/message`.

### Declarative Workspace Manifest (.deskpilot/workspace.json)

Configure per-project constraints, verification harnesses, and network fences with a single version-controlled file:

`json
{
  "name": "my-service",
  "default_model": "qwen2.5-coder:7b",
  "verification": {
    "test_command": "cargo test",
    "lint_command": "cargo clippy",
    "build_command": "cargo build --release"
  },
  "egress": {
    "allowed_hosts": ["api.openai.com", "openrouter.ai"],
    "block_network": false
  },
  "rules": [
    "Always run verification tests before claiming task completion.",
    "Never bypass the workspace permission matrix."
  ]
}
`

When present in the workspace root, DeskPilot displays an active manifest pill in the Web Studio header, injects verification guidelines directly into the agent's system prompt, and enforces egress host allowlists across network tools (`web_search`, `curl`).

### Session Lifecycle, Worktrees & Task Suspend / Resume

- **Git Worktree Isolation**: Run risky or concurrent executions on isolated worktrees (`POST /api/projects/:id/worktrees`), with 1-click **Merge** to trunk or **Discard** in the Trees inspector panel.
- **Usage-Aware Semantic Compaction**: Automatically compacts long chat histories at semantic turn boundaries, preserving essential user requests, active commitments, and referenced artifacts while recording compaction events into session telemetry.
- **Built-in Review Personas**: Specialized subagents (`Security Auditor`, `Code Reviewer`, `Test Gap Reviewer`, `Code Simplification`, `Pull Request Reviewer`) analyze diffs and surface findings directly into the Living Spec validation gates.
- **Thread Lifecycle**: Rename threads, attach `#tags` for organization, and archive old threads (`include_archived` filter toggle in sidebar).
- **Task Suspend & Resume**: Serialize active multi-turn agent executions into SQLite `task_snapshots` (`POST /api/tasks/suspend` and `POST /api/tasks/resume`), allowing work to be cleanly paused and resumed across restarts.
- **Git Checkpoints & Rollback**: Automatic pre-run git checkpoints (`try_git_checkpoint`) allow 1-click workspace rollback via `POST /api/conversations/:id/undo-run`.

### Plugin Extensions

DeskPilot supports plugin processes through the Model Context Protocol (MCP). Add a plugin folder at `.deskpilot/plugins/<id>/` with a `plugin.json` manifest:

```json
{
  "name": "Calendar Tools",
  "version": "1.0.0",
  "description": "Calendar integration",
  "command": "python",
  "args": ["server.py"]
}
```

The process must implement MCP over stdin/stdout. Its tools are discovered at chat time and exposed to the model as `mcp__plugin_<id>__<tool>`. User-level plugins can be installed under `%USERPROFILE%\\.deskpilot\\plugins\\<id>\\`. Plugin execution is controlled by the project's `mcp_tools` permission, and existing `mcp_servers.json` configurations remain supported.

---

## 💻 Building From Source (Developers)

If you have Rust installed:

```bash
git clone https://github.com/thomsonworks-in/deskpilot.git
cd deskpilot
cargo run --release
```

To test:
```bash
cargo test
```

---

## 📂 Local Data & Privacy

DeskPilot stores its local SQLite database at:
- **Windows:** `%LOCALAPPDATA%\DeskPilot\deskpilot.db`
- **macOS/Linux:** `~/.deskpilot/deskpilot.db`

All conversation logs, project contexts, vector embeddings, and API keys are stored locally.

---

## 📄 License

GNU General Public License v3.0 (GPLv3). See [LICENSE](LICENSE) for details.
