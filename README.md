# AI Agentic Tools Template

A minimal starting point for working with AI coding assistants in a dev container. The container image comes pre-loaded with all tools — this repo only needs the two config files to get everything running.

---

## What's in the Container

Your container includes:

- **[Claude Code](https://code.claude.com/docs/en/overview)** — AI agentic coding tool from Anthropic
- **[OpenCode](https://github.com/opencode-ai/opencode)** — Open source code-focused AI tool
- **[Copilot](https://github.com/features/copilot)** — GitHub's AI pair programmer
- **[Crush](https://github.com/charmbracelet/crush)** — A beautifully themed assistant for command-line work
- **[Codex](https://github.com/openai/codex)** — OpenAI's agentic tool
- **[Gemini](https://github.com/google-gemini/gemini-cli)** — Google's AI coding assistant

---

## Files in This Repo

**.devcontainer/devcontainer.json** — points to the published container image and runs setup on creation:

```json
{
  "image": "ghcr.io/calvinw/ai-course-devcontainer:latest",
  "postCreateCommand": "setup-env.sh && install-mcps.sh || true"
}
```

**configs/mcp-servers.conf** — defines which MCP servers to install:

```
dolt=https://bus-mgmt-databases.mcp.mathplosion.com/mcp-dolt-database/sse
```

---

## Starting the Agents

All launcher scripts live in `permissions/` inside the container (baked into the image). Each one starts its tool with the right flags so you're not interrupted by permission prompts.

### Claude Code

```
% claude.sh
```

Runs `claude` with `IS_SANDBOX=1` and `--dangerously-skip-permissions`. In a dev container this is safe and makes the experience much smoother.

### OpenCode

```
% opencode.sh
```

Permissions are handled by `.opencode/opencode.json`, already configured with `read`, `write`, and `execute` set to `allow`.

### Copilot

```
% copilot.sh
```

Runs `copilot --allow-all`.

### Crush

```
% crush.sh
```

Runs `crush --yolo`.

### Codex

```
% codex.sh
```

Permissions are handled via `.codex/config.toml`, already configured for a sandbox environment.

---

## MCPs (Model Context Protocol Servers)

MCP servers extend AI tools with access to external data and services. All MCP configuration flows from a single file:

```
configs/mcp-servers.conf
```

### Conf File Format

```
# No-auth SSE MCP:
name=https://example.com/mcp/sse

# Authenticated HTTP MCP (value read from environment variable):
name=https://example.com/mcp|http|X-Api-Key:$MY_API_KEY_VAR
```

### Installing MCPs

After editing `configs/mcp-servers.conf`, run:

```
% install-mcps.sh
```

This reads the conf file and registers each MCP in all AI tools — Claude, OpenCode, Gemini, Crush, Copilot, and Codex. Safe to re-run; existing entries are replaced with current values.

### Uninstalling MCPs

```
% uninstall-mcps.sh
```

Removes all MCP registrations listed in the conf file from every tool's config.

### Adding an Authenticated MCP

1. Add the entry to `configs/mcp-servers.conf`:
   ```
   stitch=https://stitch.googleapis.com/mcp|http|X-Goog-Api-Key:$STITCH_API_KEY
   ```
2. Add the secret value at [github.com/settings/codespaces](https://github.com/settings/codespaces)
3. Declare the variable in `.devcontainer/devcontainer.json` under `"secrets"` so Codespaces injects it
4. Run `install-mcps.sh`

---

## GitHub Education

If you're a student, sign up for [GitHub Education](https://education.github.com/students). It's free and gives you access to the [GitHub Student Developer Pack](https://education.github.com/pack), which includes a free GitHub Copilot subscription.

- You get **300 free Copilot premium requests per month**
- Copilot powers not just the Copilot agent, but also **OpenCode** and **Crush**, which both support GitHub Copilot as a backend
- In a GitHub Codespace, you're already authenticated — so OpenCode and Copilot start working immediately with no login steps

---

## Optional Add-ons

These scripts are available inside the container:

### Data Science Tools

```
% install-datascience.sh
```

Installs Python data science libraries, Jupyter, Quarto, and TinyTeX. Includes: numpy, pandas, matplotlib, seaborn, requests.

### Dolt Database

```
% install-dolt.sh
```

Installs [Dolt](https://github.com/dolthub/dolt), a version-controlled SQL database.

---

## Container Image

The image is built from [calvinw/ai-agentic-tools](https://github.com/calvinw/ai-agentic-tools) and published to:

```
ghcr.io/calvinw/ai-course-devcontainer:latest
```

It is rebuilt automatically on Dockerfile changes and weekly via GitHub Actions.
