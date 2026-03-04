# Mind Forge

A Claude Code skill that runs structured multi-agent debates. An orchestrator spawns 4 AI personas that propose solutions in parallel, cross-score each other, and the best approach wins by consensus.

## Architecture

```
/forge "your problem"
   │
   ├─ Phase 0: Orchestrator scans codebase, picks preset
   │
   ├─ Phase 1: 4 × forge-proposer agents (parallel)
   │     ├─ Sysops proposes...
   │     ├─ Shield proposes...
   │     ├─ Razor proposes...
   │     └─ Claw proposes...
   │
   ├─ Phase 2: 4 × forge-scorer agents (parallel)
   │     ├─ Sysops scores all...
   │     ├─ Shield scores all...
   │     ├─ Razor scores all...
   │     └─ Claw scores all...
   │
   ├─ Phase 3: Orchestrator tallies → winner + synthesis
   │
   └─ Phase 4: Cleanup (shutdown agents, delete team)
```

### Components

| File | Purpose |
|------|---------|
| `SKILL.md` | Skill definition — triggers on "forge", "council", "debate" |
| `commands/forge.md` | `/forge` slash command — the orchestrator |
| `agents/forge-proposer.md` | Proposer agent — adopts a persona, proposes a solution |
| `agents/forge-scorer.md` | Scorer agent — rates all proposals with anti-gaming rules |

## Council Presets

### Infrastructure (default for ops/devops/system)
| Agent | Role | Optimizes For |
|-------|------|---------------|
| **Sysops** | SRE | Reliability, automated recovery, observability |
| **Shield** | Security Eng | Attack surface, least privilege, audit trails |
| **Razor** | Minimalist | Fewer moving parts, less code |
| **Claw** | Domain Expert | Technically correct, system-specific |

### Product (features/UX/roadmap)
| Agent | Role | Optimizes For |
|-------|------|---------------|
| **Pulse** | PM | Ship fast, data-validated |
| **Pixel** | UX Designer | User simplicity |
| **Bolt** | Backend Eng | Clean APIs, idempotent ops |
| **Lens** | QA Lead | Testability, edge cases |

### Architecture (system design/tech)
| Agent | Role | Optimizes For |
|-------|------|---------------|
| **Atlas** | Architect | Boundaries, contracts |
| **Cache** | Perf Eng | Latency, measure-first |
| **Vault** | Security Arch | Zero-trust, scoped tokens |
| **Sage** | Staff Eng | Boring tech, proven patterns |

Custom personas supported — just describe who you want.

## Scoring

| Criterion | Weight | 10/10 Means |
|-----------|--------|-------------|
| Simplicity | 25% | Junior dev could maintain it |
| Robustness | 25% | Works at 3 AM unattended |
| Security | 20% | Minimal attack surface |
| Maintainability | 15% | Easy to change in 6 months |
| Correctness | 15% | Fully solves the problem |

### Anti-Gaming
- Agents can't score themselves highest on every criterion
- Scores of 8+ or 3- need justification
- Must differentiate — no safe "everyone gets 7" cop-outs

### Synthesis
The winner doesn't just win — best ideas from losers get folded into the final recommendation.

## Install

```bash
# Clone
git clone https://github.com/Nadav-Fux/mind-forge.git ~/.agents/skills/mind-forge

# Symlink skill + command
ln -s ~/.agents/skills/mind-forge ~/.claude/skills/mind-forge
cp ~/.agents/skills/mind-forge/commands/forge.md ~/.claude/commands/forge.md
```

## Usage

```
/forge Should we use Redis, Cloudflare KV, or in-memory LRU for caching?
```

```
/forge --preset=product Add real-time notifications or batch email digests?
```

```
Run a forge with a DBA, frontend dev, DevOps, and CTO about our DB migration.
```

## License

MIT
