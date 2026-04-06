# agent-sandbox

Run AI coding agents safely in a sandboxed macOS user. The agent gets a **read-only** GitHub PAT — it can clone and commit locally but **cannot push, create PRs, or merge**. When the agent is done, a watcher service pushes the branch and creates a draft PR for your review.

Works with any coding agent: [Claude Code](https://claude.ai/code), [Codex](https://github.com/openai/codex), [Aider](https://aider.chat), [Goose](https://github.com/block/goose), or any CLI tool.

## Quickstart

```bash
# 1. Install
npm install -g agent-sandbox

# 2. Make sure gh CLI is authenticated (used to push and create draft PRs)
gh auth login

# 3. Clone a repo (creates sandbox user if needed, prompts for PAT, adds MCP)
agent-sandbox clone https://github.com/you/your-repo

# 4. In one terminal, start the PR watcher
agent-sandbox watch

# 5. In another terminal, run your agent
cd your-repo
agent-sandbox
```

That's it. The `clone` command handles all setup — creating the macOS sandbox user, getting a read-only PAT, cloning the repo, and configuring MCP. Running `agent-sandbox` in the repo launches Claude as the sandbox user.

## How it works

```
┌─────────────────────────────┐     ┌──────────────────────────────┐
│  sandbox-agent (macOS user) │     │  your user                   │
│                             │     │                              │
│  Any coding agent CLI       │     │  agent-sandbox watch         │
│  Git PAT: read-only         │────>│  gh CLI (authenticated)      │
│  Cannot push or create PRs  │ JSON│  Pushes branch               │
│                             │file │  Creates DRAFT PR            │
│  Commits locally, then      │     │                              │
│  requests a draft PR        │     │  You review & merge          │
└─────────────────────────────┘     └──────────────────────────────┘
```

1. `agent-sandbox clone` creates a sandboxed macOS user with a read-only GitHub PAT
2. `agent-sandbox` runs your coding agent as that user — it can read and commit but not push
3. The agent calls `create_draft_pr` (via MCP tool, added automatically) when it's done
4. The watcher (your user, your credentials) pushes the branch and creates a draft PR
5. You review the draft PR and merge when ready

## Usage

### Clone a repo

```bash
agent-sandbox clone https://github.com/owner/repo
```

First time, this will:
- Create the `sandbox-agent` macOS user (prompts for sudo)
- Open your browser to create a read-only GitHub PAT
- Clone the repo
- Add `.mcp.json` so the agent gets a `create_draft_pr` tool

### Run an agent

```bash
cd repo
agent-sandbox                                            # runs claude (default)
agent-sandbox --agent codex -- --full-auto               # runs codex
agent-sandbox --agent aider                              # runs aider
agent-sandbox -- --dangerously-skip-permissions -p "fix" # pass args to claude
```

### Start the PR watcher

```bash
# Foreground
agent-sandbox watch

# Or install as a background service (launchd)
agent-sandbox watch --install
```

## Commands

| Command | Description |
|---------|-------------|
| `agent-sandbox` | Run agent as sandbox user in current dir (default: claude) |
| `agent-sandbox clone <url>` | Clone repo into sandbox (creates user if needed) |
| `agent-sandbox watch` | Start the PR watcher (foreground) |
| `agent-sandbox watch --install` | Install watcher as a background launchd agent |
| `agent-sandbox watch --uninstall` | Remove the launchd agent |
| `agent-sandbox request` | Request a draft PR via CLI |
| `agent-sandbox serve` | Start the MCP server (stdio) |
| `agent-sandbox uninstall` | Remove everything |

## Security model

- The agent runs as a **separate macOS user** (`sandbox-agent`) with no access to your credentials
- The agent's PAT is **read-only** — it cannot push, create PRs, or merge
- The watcher pushes branches and creates **draft PRs** using your `gh` auth
- All PRs require your review before merging
- The MCP server has no credentials — it only writes request files to a watched directory

## Uninstall

```bash
agent-sandbox uninstall
```
