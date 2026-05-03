# AI Agentic Tools Dev Container Guide

This dev container is built from [calvinw/ai-agentic-tools](https://github.com/calvinw/ai-agentic-tools) and includes a comprehensive toolkit for AI-assisted development.

## Container Image

**Base:** Node.js 22-slim  
**Published to:** `ghcr.io/calvinw/ai-course-devcontainer:latest`  
**Rebuilt:** Automatically on Dockerfile changes, weekly via GitHub Actions

---

## System Tools & Utilities

The container includes essential CLI tools pre-installed:

### File & Process Tools
- `curl`, `wget` — HTTP clients
- `git` — Version control
- `vim` — Text editor
- `jq` — JSON processor
- `bat` — Syntax-highlighted cat
- `ripgrep` — Fast file search
- `fd-find` — User-friendly find alternative
- `tree` — Directory tree viewer
- `fzf` — Fuzzy finder
- `lsof`, `procps`, `iproute2` — Process/network inspection
- `make` — Build automation
- `bubblewrap` — Sandboxing utility

### Specialized Tools
- `glow` — Markdown renderer
- `gh` — GitHub CLI
- `upterm` — Terminal sharing
- `miller` — Data transformation tool
- `pspg` — PostgreSQL pager
- `poppler-utils` — PDF utilities

### Development Environment
- `python3` with `pip` and `venv`
- UTF-8 locale support
- `/workspace` as default working directory

---

## AI Coding Assistants

Pre-installed via npm:

- **Claude Code** — Anthropic's agentic coding tool (installed separately)
- **OpenCode** — Open source AI coding agent
- **GitHub Copilot** — GitHub's AI pair programmer
- **Crush** — Charm's beautifully themed terminal assistant
- **Google Gemini** — Google's AI coding assistant
- **OpenAI Codex** — OpenAI's code generation model
- **Alibaba Qwen Code** — Qwen's AI coding assistant
- **Supergateway** (v3.4.3) — MCP bridge for Codex integration

---

## Available Scripts

All scripts are located in `/usr/local/lib/ai-tools/` and added to PATH. Run them directly from anywhere.

### Environment & Setup

**`setup-env.sh`**
- Generates Ed25519 SSH key pair at `~/.ssh/id_ed25519` (if not present)
- Appends `$HOME/.local/bin` to `~/.bashrc` for local package discovery
- Enables pip/pipx-installed tools to be discoverable in shell

**`install-mcps.sh`**
- Reads `configs/mcp-servers.conf` and registers Model Context Protocol servers
- Installs MCPs to all AI tools: Claude, OpenCode, Gemini, Crush, Copilot, Codex
- Safe to re-run; existing entries are replaced with current values
- Example: `install-mcps.sh` (runs automatically on container creation)

**`uninstall-mcps.sh`**
- Removes all MCP registrations listed in `configs/mcp-servers.conf`
- Cleans up MCP server entries from all tool configurations

### Skills Management

**`setup-skills.sh`**
- Initializes `.skillshare/` directory structure with `skills/` subdirectory
- Creates sample "hello-world" skill template
- Generates `config.yaml` specifying target platforms (Claude, OpenCode, Copilot, Gemini, Crush, Codex)
- Downloads and installs the `skillshare` CLI tool if not present
- Run once per project: `setup-skills.sh`

**`sync-skills.sh`**
- Deploys all skills from `.skillshare/skills/` to configured AI platforms
- Uses `.skillshare/config.yaml` to determine target platforms
- Run after modifying skills: `sync-skills.sh`

**`unsync-skills.sh`**
- Reverses skill synchronization (removes skills from agents)
- Cleans up deployed skills from all platforms

### Optional Add-ons

**`install-datascience.sh`**
- Installs Python data science ecosystem
- Includes: numpy, pandas, matplotlib, seaborn, requests
- Installs: Jupyter, Quarto, TinyTeX
- Optional; run only if needed: `install-datascience.sh`

**`install-dolt.sh`**
- Installs Dolt — a version-controlled SQL database
- Optional; run only if needed: `install-dolt.sh`

**`install_upterm.sh`**
- Installs Upterm for terminal sharing capabilities
- Already included in container; script available for updates

### Repository Synchronization

**`sync-from-upstream.sh`**
- Synchronizes changes from the upstream ai-agentic-tools repository
- Used for keeping container definition in sync with source

**`lib-mcp-parse.sh`**
- Library script for parsing MCP configuration files
- Sourced by other scripts; not meant to be run directly

---

## Configuration Files

### `configs/mcp-servers.conf`
Defines which Model Context Protocol servers to install. Format:
```
# SSE MCP (no authentication):
dolt=https://bus-mgmt-databases.mcp.mathplosion.com/mcp-dolt-database/sse

# HTTP MCP with authentication:
# stitch=https://stitch.googleapis.com/mcp|http|X-Goog-Api-Key:$STITCH_API_KEY
```

Run `install-mcps.sh` after editing to register changes.

### `.skillshare/`
Project-level skills directory (single source of truth for custom skills).
- **Always edit skills here**, never in individual tool directories
- Run `sync-skills.sh` after modifying to deploy to all agents
- Contains: `config.yaml`, `skills/` subdirectory with individual skill definitions

### `opencode.json`
Project-level OpenCode configuration (project root, highest precedence).
- Sets default model and provider settings
- Example: Configures Deepseek V4 Flash via OpenRouter
- Overrides `~/.config/opencode/opencode.json` and `.opencode/` configs

### `.devcontainer/devcontainer.json`
Dev container configuration.
- References container image: `ghcr.io/calvinw/ai-course-devcontainer:latest`
- Runs setup on creation: `setup-env.sh && install-mcps.sh && setup-skills.sh && skillshare install github.com/anthropics/skills/skill-creator && sync-skills.sh`
- Declares secrets for GitHub Codespaces: `STITCH_API_KEY`, `OPENROUTER_API_KEY`

---

## Typical Workflow

1. **First container creation** — Runs automatically:
   - `setup-env.sh` — SSH key + PATH setup
   - `install-mcps.sh` — Register MCPs from `configs/mcp-servers.conf`
   - `setup-skills.sh` — Initialize skills infrastructure
   - `skillshare install github.com/anthropics/skills/skill-creator` — Install skill-creator tool
   - `sync-skills.sh` — Deploy skills to all agents

2. **Adding new MCPs** — Edit `configs/mcp-servers.conf`, then:
   ```
   # install-mcps.sh
   ```

3. **Creating/modifying skills** — Edit files in `.skillshare/`, then:
   ```
   # sync-skills.sh
   ```

4. **Installing additional skills**:
   ```
   # skillshare install github.com/anthropics/skills/skill-creator
   # skillshare install github.com/your-org/your-skill
   # sync-skills.sh
   ```

5. **Adding optional tools**:
   ```
   # install-datascience.sh
   # install-dolt.sh
   ```

---

## Starting Agents

All launcher scripts live in `permissions/` (baked into container image):

- `claude.sh` — Runs Claude Code with sandbox mode
- `opencode.sh` — Runs OpenCode with allow-all permissions
- `copilot.sh` — Runs GitHub Copilot with allow-all
- `crush.sh` — Runs Crush with yolo mode
- `codex.sh` — Runs OpenAI Codex
- `gemini.sh` — Runs Google Gemini

---

## Environment Variables

**For authentication in Codespaces:**
- `STITCH_API_KEY` — Stitch MCP authentication
- `OPENROUTER_API_KEY` — OpenRouter API access

Declare these in `.devcontainer/devcontainer.json` under `"secrets"` and add values in GitHub Codespaces settings.

---

## Source Repository

All tools, scripts, and Dockerfile: [calvinw/ai-agentic-tools](https://github.com/calvinw/ai-agentic-tools)

Container rebuilt automatically on changes to:
- Dockerfile
- Scripts in `scripts/` directory
- Weekly via GitHub Actions
