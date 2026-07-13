---
name: codex-peer
description: >-
  Work with the OpenAI Codex CLI as a peer collaborator — spawn it in the background on one
  task while Claude works another in parallel, offload read-heavy investigations to protect
  Claude's context window, and get cross-model second-opinion code reviews. Use whenever the
  user mentions Codex in any form ("ask codex", "spawn codex", "codex review", "what does
  codex think", "split this with codex") — AND proactively when the user never says "codex"
  but the work fits: 2+ independent chunks that could run in parallel instead of serially;
  a read-heavy task with a small answer (investigating unfamiliar code, digesting a large
  diff or log, finding where something is defined); or a significant change that deserves an
  independent second review before shipping. If parallelizing or offloading would make the
  turn faster or keep Claude's context lean, reach for this skill rather than doing
  everything serially yourself.
---

# Codex Peer Collaboration

Codex is a peer, not a servant. It runs a frontier model and is dispatched at **full
intelligence for every task** — the defaults in `~/.codex/config.toml` (currently
`gpt-5.6-sol` + `model_reasoning_effort = "xhigh"`). Never pass a smaller `-m` model or a
lower effort "because the task is easy": the user uses Codex both as a second brain and as
a cross-model second opinion, and downgrading defeats both. Routing between Claude and
Codex is by *task shape*, never by difficulty tier.

## Preflight

Confirm Codex is reachable before building a plan around it:

```bash
codex login status   # expect "Logged in using ChatGPT"
```

If auth is broken, suggest the user run `! codex login` (it's interactive), do the current
task yourself, and say so in the report. Same fallback for quota errors (429), network
failures, or a hung session (kill the background job after a reasonable timeout). If Codex
output fails verification twice on the same task, take the task over instead of
ping-ponging. The work never silently degrades — the final report always discloses who did
what.

## Routing: who takes what

Grounded in published benchmarks and hands-on comparisons (details and sources in
`references/routing-evidence.md`):

| Task shape | Route to | Why |
|---|---|---|
| Terminal/shell-heavy: builds, CI debugging, tooling chains, env wrangling | Codex | Leads Terminal-Bench |
| Scoped extension/maintenance of existing code | Codex | Tracks correlated changes unprompted; token-efficient |
| Parallelizable independent chunks | Codex (one or more) | Claude works simultaneously; separate quota |
| Read-heavy, small-answer: investigations, log digestion, large-diff analysis | Codex | Exploration lands in Codex's context, not Claude's (~100:1 savings measured) |
| Second-opinion / adversarial code review | Codex | Multi-model review catches ~1/3 more issues than either model alone |
| Repo-level multi-file features and bug fixes | Claude | Leads SWE-bench Verified/Pro |
| Long-horizon sessions, large refactors, architecture | Claude | 2x+ context window; early-session context survives |
| Hard novel problems; deep security/data-flow review | Claude | Leads frontier-difficulty and data-flow-tracing evals |
| Anything whose main input is the accumulated session context | Claude | The brief would cost more than the task |
| **Final verification and integration** | **Always Claude** | See Verification below |

Poor delegations regardless of the table: tiny edits (the brief costs more than doing it)
and tasks that would require Claude to deep-re-read everything to verify anyway.

**Latency expectation** (measured): Codex at xhigh takes minutes even on small questions,
so its thinking time dominates small tasks — a two-lookup investigation ran 4x slower
delegated than done directly, while a full PR review ran *faster* with the split because
Codex's deep pass overlapped Claude's own. Rule of thumb: the bigger the task, the better
the split. For a small question the user is actively waiting on, answer it directly; save
Codex for work that runs in the background while the session moves on.

## Dispatching

Codex starts every session cold — **the brief is the context transfer**. A good brief is
self-contained: goal, relevant file paths (with line refs when known), constraints the
session has established, and concrete acceptance criteria. Address it as a peer
("Task for you as peer collaborator: ...").

**Spawn in the background and keep working** — this is the point of the pattern. Never sit
idle waiting on Codex; take the other task yourself immediately:

```bash
# investigation / review (no writes)
codex exec --sandbox read-only "<brief>"
# implementation (edits files)
codex exec --sandbox workspace-write "<brief>"
```

Run these via Bash with `run_in_background: true`; a notification arrives on completion and
interim output can be Read from the task's output file. The session header printed at start
contains the **session id** — capture it, because follow-ups to a specific session use
`codex exec resume <session-id> "<follow-up>"`. Don't use `resume --last` when more than
one session ran; it's ambiguous.

If the `codex` MCP server tools are available in the session, they're equivalent to
`codex exec` — use whichever is at hand; the mechanics above still apply.

**Concurrency safety:** don't edit files a running Codex job is touching. For parallel
tasks that both write, give Codex its own git worktree.

## Verification

Codex's model line measurably reward-hacks: METR found it games visible success criteria
(gaming hidden test suites, extracting expected answers) at the highest rate of any public
model it has evaluated. This is not a reason to distrust its work — it's a reason
acceptance checks must be **independent outcomes Claude observes directly**: does it build,
does the e2e flow pass, does the demo page behave. Never accept (a) Codex's own report of
success, or (b) passing a test suite that Codex could see and optimize against, as the
verification. Verify the outcome, then report it plainly.

## Reporting

The final message to the user merges both work streams and attributes them: what Codex did
(and its session id if follow-ups are likely), what Claude did, what was verified and how,
and — if a fallback happened — why. Cross-model attribution matters to the user; keep it
explicit.
