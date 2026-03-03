# LLM Council

A Claude Code skill that runs structured multi-agent debates for architecture decisions.

## What It Does

4 AI agents with distinct personas debate your problem, propose solutions, cross-score each other, and the best approach wins by consensus.

## The Council

| Agent | Role | Optimizes For |
|-------|------|---------------|
| **Sysops** | SRE / Infrastructure | Reliability, observability, automation |
| **Shield** | Security Engineer | Attack surface, secrets, permissions |
| **Razor** | Minimalist | Simplicity, fewer moving parts |
| **Claw** | Domain Expert | Technically correct, system-specific solutions |

## How It Works

1. **Frame** — You describe the problem and constraints
2. **Propose** — 4 agents run in parallel, each proposing a solution
3. **Score** — Each agent scores all proposals (1-10) on 5 weighted criteria
4. **Verdict** — Weighted averages determine the winner

### Scoring Criteria

| Criterion | Weight |
|-----------|--------|
| Simplicity | 25% |
| Robustness | 25% |
| Security | 20% |
| Maintainability | 15% |
| Correctness | 15% |

## Install

```bash
# Claude Code skill (symlink into your skills directory)
mkdir -p ~/.agents/skills/llm-council
cp SKILL.md ~/.agents/skills/llm-council/
ln -s ~/.agents/skills/llm-council ~/.claude/skills/llm-council
```

## Usage

In Claude Code:

```
/council How should we handle database migrations? Options: Prisma, Drizzle, raw SQL, Supabase migrations.
```

Or just describe a decision and ask for a council debate:

```
I need to decide between Redis, in-memory cache, and Cloudflare KV for our caching layer. Can you run a council on this?
```

## Customizing Personas

You can request different personas. Each needs:
- **Name** — short, memorable
- **Role** — 1-3 words
- **Focus** — what they optimize for
- **Bias** — what they tend to favor
- **Skepticism** — what they push back on

Example: "Run a council with a UX designer, a backend engineer, a product manager, and a QA lead"

## License

MIT
