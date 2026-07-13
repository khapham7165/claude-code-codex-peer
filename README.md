# codex-peer

A [Claude Code](https://claude.com/claude-code) skill that lets Claude work with the
[OpenAI Codex CLI](https://developers.openai.com/codex) as a **peer collaborator** —
two frontier models on one task list instead of one.

## How it works

Claude stays in the driver's seat and holds all the context. When work can be split or
offloaded, Claude:

1. **Briefs** Codex — a self-contained prompt (goal, files, constraints, acceptance
   criteria), since Codex starts each session cold.
2. **Spawns** it in the background (`codex exec`) at **full intelligence** — always the
   best model and highest reasoning effort, never downgraded "for easy tasks".
3. **Keeps working** on the other task in parallel — nobody sits idle.
4. **Verifies** Codex's result against real outcomes (does it build, does the app
   behave), never against Codex's own success report.
5. **Reports** back with clear attribution: what Codex did, what Claude did.

Task routing is evidence-based (see [`references/routing-evidence.md`](references/routing-evidence.md)):
terminal-heavy work, scoped maintenance, parallel chunks, and second-opinion code reviews
go to Codex; repo-level features, long refactors, novel problems, and final verification
stay with Claude. If Codex is unavailable (auth, quota, network), Claude does the work
itself and says so — nothing silently degrades.

## Install

Requirements: Claude Code, plus the Codex CLI installed and logged in (`codex login`).

```bash
git clone git@github.com:khapham7165/codex-peer.git ~/.claude/skills/codex-peer
```

Optional — also register Codex as an MCP server so it's available as a tool from session
start:

```bash
claude mcp add codex -s user -- codex mcp-server -c model_reasoning_effort=xhigh
```

## Use

Nothing to memorize — the skill triggers on its own when you mention Codex or when work
is parallelizable. Examples:

- "split these two tasks with codex"
- "get a codex second opinion on this diff"
- "investigate X and Y" (two independent chunks → automatic parallel split)
