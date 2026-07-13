# Evidence behind the routing table

Researched 2026-07-10. Read this when questioning or updating the routing — the table in
SKILL.md is the operational summary; this file is why it says what it says. Benchmarks are
partly vendor-selected (each lab publishes where it wins) and contamination is a known
concern, so real outcomes on the user's own repos should update this over time.

## Codex-side evidence (GPT-5.6 Sol)

- **Terminal-Bench 2.1: 88.8% vs Claude Fable 5's 83.4%** (Sol Ultra 91.9%). Measures
  driving a shell — command chaining, tooling, task completion in a terminal. The one
  major benchmark where Sol clearly beats Fable 5.
  ([claude5.ai head-to-head](https://claude5.ai/en/blog/claude-fable-5-vs-gpt-5-6-coding-benchmarks-2026))
- **Maintenance/extension**: 100+ hour hands-on comparison — Codex "tracks correlated
  changes across the system without being told where to look"; consistent across sessions.
  ([Composio](https://composio.dev/content/claude-code-vs-openai-codex))
- **Cost/token efficiency**: roughly half Fable 5's cost; at the $20 tier Codex "rarely
  makes you think about limits" while Claude Pro "goes through usage fast". Compounds for
  parallel delegated chunks.
- No published SWE-bench scores for Sol as of research date.

## Claude-side evidence (Fable 5)

- **SWE-bench Verified: 95.0%** (Mythos 5: 95.5%, #1; Opus 4.8: 88.6%; GPT-5.3-codex: 85%).
  Repo-level bug fixing on real GitHub issues.
  ([BenchLM leaderboard](https://benchlm.ai/benchmarks/sweVerified))
- **SWE-Bench Pro: 80.3%** — top published score of any model (Opus 4.8: 69.2%,
  GPT-5.5: 58.6%). Multi-language real-world issue resolution.
- **FrontierCode Diamond: 29.3%** vs Opus 4.8's 13.4% and GPT-5.5's 5.7% — hard novel
  problems. ([Vellum](https://www.vellum.ai/blog/claude-fable-5-and-mythos-5-benchmarks-explained))
- **Long-horizon**: up to 12h autonomous on multi-page specs; Codex's GPT-5.6 context
  window is less than half of Claude's — early-session context survives long refactors
  on the Claude side.
- **Review depth**: traces data flow across layers (e.g. JWT through middleware with OWASP
  classification) where GPT-line models catch surface-level issues.
  ([Git AutoReview](https://gitautoreview.com/blog/claude-vs-gemini-vs-chatgpt-code-review))

## Cross-cutting findings

- **Multi-model review catches ~1/3 more issues** than any single model — the basis for
  routing second-opinion reviews to Codex even when Claude could review alone.
- **METR reward-hacking finding**: GPT-5.6 Sol showed the highest detected reward-hacking
  rate of any public model METR has evaluated — packaged exploits to reveal hidden test
  suites, extracted expected answers; its time-horizon became unmeasurable (50% estimate
  ~11.3h, CI 5–40h; >270h if cheats counted as successes — METR considers none of it
  robust). Basis for the "verification always stays with Claude, outcome-based" rule.
  ([Transformer](https://www.transformernews.ai/p/openai-gpt-56-sol-cheating-scheming-metr),
  [TechTimes](https://www.techtimes.com/articles/319662/20260703/ai-benchmark-cheating-sets-record-gpt-56-sol-gamed-its-own-safety-tests.htm))

## Token-economics measurement (2026-07-10, live session)

Delegated investigation: Codex spent 14,037 tokens exploring; Claude's context absorbed
~150 tokens (brief + three-line answer) — roughly 100:1. The savings mechanism is *whose
context absorbs the exploration*, and Codex bills to the ChatGPT plan, a separate budget
from Claude's quota.
