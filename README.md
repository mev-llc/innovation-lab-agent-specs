# Seven agent specs from the MEV Innovation Lab Delivery System

Agent definitions for [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) from a 72-hour build: a live proof of concept on New York City's [Local Law 84](https://www.nyc.gov/site/buildings/codes/ll84-benchmarking-law.page) energy data, covering ~30,000 buildings. Three engineers ran the build with these seven agents; a fourth joined for final-day polish. The agents reviewed about 127 pull requests before any human looked, returning close to 60 hours of review time. The PoC runs on $45/month of infrastructure.

These are the exact specs as they ran during the build — project paths, repo names, and stack references intact.

## What this is not

The specs are the visible part of the delivery system. On their own they are seven well-structured prompts. What made them work on this build: the orchestration around them, the quality gates, and the senior engineers who owned the plan. Running the files without that layer gets you the happy path — and not much past it.

Concretely, two dependencies are not in this repo. Every agent starts by reading a root `CLAUDE.md` — the project's conventions and quality requirements — which is project-specific and not included. And the sequencing lives outside the specs: the `/dev` command, the human gates, and the 3-iteration cap on the review–fix loop are part of the orchestration layer, not these files.

## How the pipeline runs

One command, `/dev` with a ticket number, runs the chain from a GitHub Issue to code ready for review:

```
GitHub Issue
     │
     ▼
 Architect ──► implementation plan ──► HUMAN GATE: plan approved?
     │
     ▼
  Builder ──► code, clean commits
     │
     ▼
 Reviewer ⇄ Fixer          (loop capped at 3 iterations)
     │
     ▼
 Security Reviewer · Design Reviewer · TODO Finder
     │
     ▼
 Final diff + P0–P4 summary ──► HUMAN GATE: ship?
```

Two rules hold the pipeline together:

- **Humans sign off twice.** Nothing is written to disk until an engineer approves the Architect's plan. Nothing merges until an engineer reviews the final diff.
- **Loops are capped.** The review–fix loop stops at 3 iterations; the design review loop stops at 8. No infinite loops, no runaway token spend.

Every agent runs with a fresh context — no shared conversation history. The input is the ticket and the spec; the output is an artifact (a plan, a diff, a report) passed to the next agent.

## The seven agents

| Agent | File | Job |
|---|---|---|
| Architect | `agents/architect.md` | Read-only. Explores the codebase, finds reusable utilities, writes the implementation plan. |
| Builder | `agents/builder.md` | Writes code following the plan and project conventions. |
| Reviewer | `agents/reviewer.md` | Reviews code against a P0–P4 severity rubric: correctness, security, architecture, types, performance. |
| Fixer | `agents/fixer.md` | Applies must-fix and should-fix patches from the Reviewer's report. Adds no features. |
| Design Reviewer | `agents/design-reviewer.md` | Opens the local site via Chrome DevTools MCP, screenshots at 1440px and 375px, checks the UI against shadcn/ui and Tailwind standards. |
| Security Reviewer | `agents/security-reviewer.md` | Scans for input validation gaps, injection, exposed secrets, CORS issues, and prompt injection risks. |
| TODO Finder | `agents/todo-finder.md` | Ship-readiness scan: leftover TODOs, mock data, placeholders, stray `console.log`s. |

## How to use

1. Copy the files from `agents/` into your project's `.claude/agents/` directory.
2. Adapt the stack references to yours — the specs are written against this build's stack (FastAPI + React with TypeScript, TanStack Query/Router, shadcn/ui, Tailwind CSS, SQLModel with pgvector) and will underperform against different conventions as written.
3. Write your own root `CLAUDE.md` with your conventions and quality requirements — the agents read it at runtime, and without it the review tiers have no definitions to enforce.
4. Wire your own orchestration. The specs define behavior per agent; the sequencing, gates, and caps live outside these files.

## Background

The full write-up of the build — scoping from an open dataset, the cost decisions behind the $45/month runtime, what got cut to ship in three days: **[Inside MEV's AI Innovation Lab: shipping a PoC in 72 hours](https://mev.com/blog/how-mev-shipped-a-poc-in-72-hours-with-ai-agents)**

The delivery system is how MEV's AI Innovation Lab ships client work. If you have a PoC or an early-stage product in mind: [Innovation Lab as a Service](https://mev.com/services/innovation-lab-as-servise?utm_source=github&utm_medium=readme&utm_campaign=agent-specs).

## License

MIT. See [LICENSE](LICENSE).
