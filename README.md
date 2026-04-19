# Autonomous Dev Agents System

A multi-agent Claude Code system that runs a continuous development loop inside a Discord bot. Agents take tasks from a backlog, write PRDs, write TRDs, build features, review PRs, audit code quality, auto-fix broken CI, and keep the queue stocked — all on cron schedules, around the clock, without babysitting.

**You own the `main` branch and decide when to ship.**

> **Runs on a VPS.** We deploy this on a Linux VPS with a self-hosted GitHub Actions runner so agents run 24/7 and can auto-fix CI failures. See [GUIDE.md](./GUIDE.md) for VPS setup.

---

## How it works

```
PRD (Product Manager) → TRD (Developer) → Code (Developer) → Review (Reviewer) → Your merge
                                                          ↑
                          CI breaks? → CI Fixer auto-patches and opens a fix PR
                          Code quality drift? → Auditor files AUDIT tasks to the backlog
```

Twelve agents, each with a single job:

| Agent | Schedule | Model | Job |
|-------|----------|-------|-----|
| Developer | every 10 min | Sonnet | Picks up Ready tasks; writes plan + TRD; builds feature once TRD approved; commits and pushes incrementally; opens/updates draft PR; moves task through backlog states |
| TRD Watcher | every 5 min | Sonnet | Scans In Progress + In Review for pending TRDs; reviews for architectural soundness, PRD coverage, scope, and test plan; approves or requests changes |
| Reviewer | every 10 min | **Opus** | Picks oldest In Review PR; skips if no new commits since last review; checks CI; reviews code against standards; approves or requests changes |
| Merge Watcher | every 5 min | Sonnet | Detects new main-branch merges; unblocks Blocked tasks; merges main into all open PR branches; handles conflict resolution |
| Project Manager | every 30 min | Opus | Keeps Ready queue stocked at 2–3 tasks; closes stale tasks; surfaces off-plan ideas to proposals; validates roadmap health |
| Product Manager | every 4h | Opus | Writes PRDs for the next 2 upcoming goals that don't have one |
| Domain Researcher | daily 7am | Opus | Researches one market topic per run; appends findings to product-notes.md; files proposals if warranted |
| System Reviewer | daily 9pm | Opus | Audits last 24h of agent logs; scores 7 dimensions; identifies problems; files proposals; syncs changes to this public repo |
| **Codebase Auditor** | every 3h | **Opus** | Reads actual source files in one area per run; files AUDIT-NNNN tasks for real violations (specific file, line, fix) |
| **Main CI Fixer** | every 2 min* | **Opus** | Trigger-based — only runs when main CI fails; investigates failure, understands original intent, opens a fix PR |
| **PR CI Fixer** | every 2 min* | **Opus** | Trigger-based — only runs when a PR's CI fails; investigates, fixes, pushes directly to the PR branch |
| Log Trim | every 4h | Sonnet | Archives agent-log.md entries older than 24h to a monthly archive file |

*CI Fixers fire every 2 min but use a `preCommand` to skip the Claude spawn if no trigger file exists — near-zero cost when idle.

---

## Quick start

1. Set up [claude-code-discord-starter](https://github.com/anthropics/claude-code-discord-starter)
2. Read **[GUIDE.md](./GUIDE.md)** — full setup walkthrough including VPS and CI integration
3. Copy the `prompts/` templates and adapt them to your project
4. Copy `crons/jobs.json` and set your Discord channel IDs
5. Write your implementation roadmap and initial backlog
6. Run `node workspace/scripts/agent-watch.js` to monitor agents

---

## Repo structure

```
prompts/                  ← agent instruction files (customize these)
  developer.md
  reviewer.md
  trd-watcher.md
  merge-watcher.md
  project-manager.md
  product-manager.md
  domain-researcher.md
  vet-industry-researcher.md
  system-reviewer.md
  codebase-auditor.md
  main-ci-fixer.md
  pr-ci-fixer.md

crons/
  jobs.json               ← cron job definitions (plug into claude-code-discord-starter)

GUIDE.md                  ← full setup and design guide (VPS, CI, kanban, all agents)
```

---

## Requirements

- [claude-code-discord-starter](https://github.com/anthropics/claude-code-discord-starter) running
- Claude Code with Opus access (several agents use Opus — see table above)
- A project repo (any stack — prompts are fully customizable)
- A self-hosted GitHub Actions runner (optional, required for CI auto-fixers)

---

Built by [@keithbooher](https://github.com/keithbooher). See [GUIDE.md](./GUIDE.md) for the full story.
