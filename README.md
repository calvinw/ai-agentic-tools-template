# AI Agentic Tools Template

A minimal starting point for working with AI coding assistants in a dev container. The container image comes pre-loaded with all tools — this repo only needs the two config files to get everything running.

---

## What's in the Dev Container

Your container includes:

- **[Claude Code](https://code.claude.com/docs/en/overview)** — AI agentic coding tool from Anthropic
- **[OpenCode](https://github.com/opencode-ai/opencode)** — Open source code-focused AI tool
- **[Copilot](https://github.com/features/copilot)** — GitHub's AI pair programmer
- **[Crush](https://github.com/charmbracelet/crush)** — A beautifully themed assistant for command-line work
- **[Codex](https://github.com/openai/codex)** — OpenAI's agentic tool
- **[Gemini](https://github.com/google-gemini/gemini-cli)** — Google's AI coding assistant

**Configuration managed by:**
- `configs/mcp-servers.conf` — Model Context Protocol servers available to all agents
- `.skillshare/` — Custom skills available to all agents (single source of truth)

---

## Starting the Agents

All launcher scripts live in `permissions/` inside the container (baked into the image). Each one starts its tool with the right flags so you're not interrupted by permission prompts.

### Claude Code

```
# claude.sh
```

Runs `claude` with `IS_SANDBOX=1` and `--dangerously-skip-permissions`. In a dev container this is safe and makes the experience much smoother.

### OpenCode

```
# opencode.sh
```

Permissions are handled by `.opencode/opencode.json`, already configured with `read`, `write`, and `execute` set to `allow`.

### Copilot

```
# copilot.sh
```

Runs `copilot --allow-all`.

### Crush

```
# crush.sh
```

Runs `crush --yolo`.

### Codex

```
# codex.sh
```

Permissions are handled via `.codex/config.toml`, already configured for a sandbox environment.

---

## MCPs (Model Context Protocol Servers)

MCP servers extend AI tools with access to external data and services. All MCP configuration flows from a single file: `configs/mcp-servers.conf`

### Adding Additional MCPs

Edit `configs/mcp-servers.conf` and add entries using this format:

```
# SSE MCP (no authentication):
dolt=https://bus-mgmt-databases.mcp.mathplosion.com/mcp-dolt-database/sse

# HTTP MCP with authentication (credential from environment variable):
# stitch=https://stitch.googleapis.com/mcp|http|X-Goog-Api-Key:$STITCH_API_KEY
```

The `dolt` entry above is already active and provides access to a version-controlled SQL database. To add other authenticated MCPs like Stitch, uncomment the entry and provide the API key via environment variables.

After editing `configs/mcp-servers.conf`, run:

```
# install-mcps.sh
```

This reads the conf file and registers each MCP in all AI tools — Claude, OpenCode, Gemini, Crush, Copilot, and Codex. Safe to re-run; existing entries are replaced with current values.

### For Authenticated MCPs

To use authenticated MCPs like Stitch:

1. Add the secret (e.g., `STITCH_API_KEY`) at [github.com/settings/codespaces](https://github.com/settings/codespaces) under "Repository secrets"
2. Declare the secret in `.devcontainer/devcontainer.json` under `"secrets"` — this works in both GitHub Codespaces and local devcontainers:
   ```json
   "secrets": {
     "STITCH_API_KEY": "STITCH_API_KEY"
   }
   ```
3. Uncomment the MCP entry in `configs/mcp-servers.conf` and reference the secret variable (e.g., `$STITCH_API_KEY`)
4. Run `install-mcps.sh`

### Uninstalling MCPs

```
# uninstall-mcps.sh
```

Removes all MCP registrations listed in the conf file from every tool's config.

---

## Skills (All Agents)

Skills are custom slash commands available across all your AI agents. The `.skillshare/` directory is the **single source of truth** for all skills.

### Editing Skills

**Always edit skills directly in the `.skillshare/` directory.** Never manually edit skills in other tool directories — the `.skillshare/` folder is where skillshare manages your skills. Changes to skills in other locations will be overwritten during sync.

### Installing Skills

Install individual skills using:

```
# skillshare install github.com/anthropics/skills/skill-creator
# skillshare install github.com/your-org/your-skill-name
```

### Syncing Skills to All Agents

After installing or modifying skills in `.skillshare/`, sync them to all your configured AI agents:

```
# sync-skills.sh
```

This deploys skills to the platforms listed in `.skillshare/config.yaml` (Claude Code, OpenCode, Copilot, Gemini, Crush, Codex, etc.).

---

## Optional Add-ons

These scripts are available inside the container:

### Data Science Tools

```
# install-datascience.sh
```

Installs Python data science libraries, Jupyter, Quarto, and TinyTeX. Includes: numpy, pandas, matplotlib, seaborn, requests.

### Dolt Database

```
# install-dolt.sh
```

Installs [Dolt](https://github.com/dolthub/dolt), a version-controlled SQL database.

---

## Container Image

The image is built from [calvinw/ai-agentic-tools](https://github.com/calvinw/ai-agentic-tools) and published to:

```
ghcr.io/calvinw/ai-course-devcontainer:latest
```

It is rebuilt automatically on Dockerfile changes and weekly via GitHub Actions.
