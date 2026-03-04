# Mind Forge

A Claude Code skill that runs structured multi-agent debates. An orchestrator spawns 4 AI personas that propose solutions in parallel, cross-score each other, and the best approach wins by consensus.

**Not just for architecture** — works for product decisions, cost optimization, security audits, startup strategy, and anything else where multiple perspectives help.

## Real Example: The Guardian Debate

We needed to protect Docker container patches from being overwritten by updates. Four agents debated:

- **Sysops** proposed a dual-layer system with docker events + health monitor (200+ lines)
- **Shield** proposed host-side Python with SHA256 integrity + JSON patches (300 lines)
- **Razor** proposed adding 25 lines to the existing health monitor
- **Claw** proposed a Docker entrypoint wrapper

**Razor won unanimously (42.7/50)** by proving the problem didn't exist — patches were already persistent via bind mounts. Only verification was needed. Three agents built solutions for a non-existent threat. The debate caught the false premise.

> *"This is building a bank vault to protect a $20 bill."* — Razor, scoring Shield's proposal

Full transcript with scores: [`examples/guardian-debate.md`](examples/guardian-debate.md)

## Architecture

```
/forge "your problem"
   │
   ├─ Phase 0: Orchestrator scans codebase, picks preset
   │
   ├─ Phase 1: 4 × forge-proposer agents (parallel)
   │     Each proposes with: Approach, Implementation, Pros, Cons, Risk, Effort
   │
   ├─ Phase 2: 4 × forge-scorer agents (parallel)
   │     Each scores all proposals 1-10 on 5 criteria (anti-gaming enforced)
   │
   ├─ Phase 3: Orchestrator validates scores → scoreboard → winner + synthesis
   │
   └─ Phase 4: Cleanup (shutdown agents, delete team)
```

### Components

| File | Purpose |
|------|---------|
| `SKILL.md` | Skill trigger — activates on "forge", "council", "debate" |
| `commands/forge.md` | `/forge` orchestrator with validation + all presets |
| `agents/forge-proposer.md` | Proposer — persona + strict output format + self-check |
| `agents/forge-scorer.md` | Scorer — weighted criteria + anti-gaming + self-check |
| `references/presets.md` | All 7 council presets with full persona definitions |
| `examples/guardian-debate.md` | Real debate with proposals, scores, and outcome |

## 7 Council Presets

| Preset | Agents | Best For |
|--------|--------|----------|
| **Infrastructure** | Sysops, Shield, Razor, Claw | Ops, devops, system decisions |
| **Product** | Pulse, Pixel, Bolt, Lens | Features, UX, roadmap |
| **Architecture** | Atlas, Cache, Vault, Sage | System design, tech choices |
| **Data** | Flow, Signal, Prism, Guard | Analytics, ML, pipelines |
| **Startup** | Spark, Ledger, Voice, Scale | Business, strategy, GTM |
| **Security Audit** | Red, Blue, Compliance, DevSec | Threat modeling, hardening |
| **Cost Optimization** | Penny, Peak, Arch, Ops | Cloud spend, resources |

Custom personas supported — just describe who you want.

## Scoring

| Criterion | Weight | 10/10 Means |
|-----------|--------|-------------|
| Simplicity | 25% | A junior dev could maintain it |
| Robustness | 25% | Works at 3 AM with no one watching |
| Security | 20% | Minimal attack surface, no shortcuts |
| Maintainability | 15% | Easy to change in 6 months |
| Correctness | 15% | Fully solves the stated problem |

### Anti-Gaming
- Agents can't score themselves highest on every criterion
- Scores of 8+ or 3- require one-sentence justification
- Must differentiate — no safe "everyone gets 7" cop-outs
- Orchestrator validates scores before building scoreboard

### Premise Challenging
Agents are encouraged to challenge the problem statement itself. The most valuable output often isn't the best solution — it's discovering the question was wrong. (See: Guardian Debate)

### Synthesis
The winner doesn't just win — best ideas from losers get folded into the final recommendation.

## Install

```bash
# Clone and set up
git clone https://github.com/Nadav-Fux/mind-forge.git ~/.agents/skills/mind-forge
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
/forge --preset=startup Should we raise a seed round now or bootstrap to revenue?
```

```
Run a forge with a DBA, frontend dev, DevOps, and CTO about our DB migration.
```

## Why This Works

A single AI agent will give you one perspective and commit to it. It won't:
- Challenge whether the problem is real
- Admit when a simpler approach beats its own
- Score its own idea honestly against alternatives

Mind Forge forces 4 independent perspectives to compete, then uses structured scoring with anti-gaming rules to surface the genuinely best approach — even when that approach is "do almost nothing."

## License

MIT
