# Mind Forge

A Claude Code skill that runs structured multi-agent debates. 4 AI personas propose solutions in parallel, cross-score each other, and the best approach wins by consensus.

Not just for architecture — works for any decision where multiple perspectives help.

## How It Works

```
You ──→ [Frame problem] ──→ 4 agents propose in parallel ──→ 4 agents score in parallel ──→ Winner + synthesis
```

1. **Frame** — You describe the problem. Mind Forge scans the codebase for context.
2. **Propose** — 4 agents with distinct personas each propose a concrete solution.
3. **Score** — Every agent scores every proposal (1-10) on 5 weighted criteria. Anti-gaming rules prevent self-favoritism.
4. **Verdict** — Weighted averages determine the winner. Best ideas from losers get folded into the final recommendation.

## Council Presets

### Infrastructure (ops, devops, system decisions)
| Agent | Role | Optimizes For |
|-------|------|---------------|
| **Sysops** | SRE | Reliability, automated recovery, observability |
| **Shield** | Security Eng | Attack surface, least privilege, audit trails |
| **Razor** | Minimalist | Fewer moving parts, less code, delete before add |
| **Claw** | Domain Expert | Technically correct, system-specific solutions |

### Product (features, UX, roadmap)
| Agent | Role | Optimizes For |
|-------|------|---------------|
| **Pulse** | Product Manager | Ship fast, validate with data |
| **Pixel** | UX Designer | User simplicity, friction reduction |
| **Bolt** | Backend Eng | Clean APIs, idempotent operations |
| **Lens** | QA Lead | Testability, edge case coverage |

### Architecture (system design, tech choices)
| Agent | Role | Optimizes For |
|-------|------|---------------|
| **Atlas** | Systems Architect | Boundaries, contracts, decoupling |
| **Cache** | Performance Eng | Latency, throughput, measure-first |
| **Vault** | Security Architect | Zero-trust, encryption, scoped tokens |
| **Sage** | Staff Engineer | Boring tech, proven patterns, team velocity |

Custom personas supported — just describe who you want on the council.

## Scoring Criteria

| Criterion | Weight | 10/10 Means |
|-----------|--------|-------------|
| Simplicity | 25% | A junior dev could maintain it |
| Robustness | 25% | Works at 3 AM with no one watching |
| Security | 20% | Minimal attack surface, no shortcuts |
| Maintainability | 15% | Easy to change in 6 months |
| Correctness | 15% | Fully solves the stated problem |

## Install

```bash
# Option 1: Clone and symlink
git clone https://github.com/Nadav-Fux/mind-forge.git ~/.agents/skills/mind-forge
ln -s ~/.agents/skills/mind-forge ~/.claude/skills/mind-forge

# Option 2: Manual
mkdir -p ~/.agents/skills/mind-forge
curl -o ~/.agents/skills/mind-forge/SKILL.md https://raw.githubusercontent.com/Nadav-Fux/mind-forge/main/SKILL.md
ln -s ~/.agents/skills/mind-forge ~/.claude/skills/mind-forge
```

## Usage

```
/mind-forge Should we use Redis, Cloudflare KV, or in-memory LRU for caching?
```

```
Run a forge with the Product preset — should we add real-time notifications or batch email digests?
```

```
I need a council of a DBA, a frontend dev, a DevOps engineer, and a CTO to decide on our database migration strategy.
```

### Quick Mode

For simple decisions, Mind Forge skips team creation and runs a streamlined 4-agent proposal + self-scoring flow.

## Anti-Gaming

Agents cannot score their own proposal highest on every criterion. They must identify at least one criterion where a competitor genuinely wins. Scores of 8+ or 3- require a one-sentence justification.

## Synthesis

The winner doesn't just win — Mind Forge identifies the best ideas from losing proposals and folds them into the final recommendation. You get the strongest possible answer, not just the popular one.

## License

MIT
