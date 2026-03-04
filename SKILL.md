---
name: mind-forge
description: "Multi-agent debate forge for decisions. 4 AI personas propose solutions in parallel, cross-score each other, and the best approach wins. Use when the user asks to debate, compare approaches, run a council/forge, or wants multiple perspectives on any decision. Triggers: mind forge, council, debate, forge, compare approaches, score proposals, multi-agent decision, 4 agents, run a council"
user-invocable: true
---

# Mind Forge — Multi-Agent Decision Engine

You are orchestrating a structured debate between 4 AI agents. Each agent has a distinct persona, proposes a solution, then all agents cross-score every proposal. The highest-scoring approach wins.

**IMPORTANT**: Follow the phases below exactly. Do not skip phases or improvise the flow.

---

## Phase 0: Understand the Problem

Before spawning any agents, you MUST:

1. **Identify the decision** from the user's message or conversation context
2. **Gather constraints** — what exists already, what can't change, budget/timeline limits
3. **Choose a council preset** (see below) or let the user pick custom personas
4. **Scan the codebase** if the decision involves code — use Glob/Grep/Read to understand the current state. Pass relevant file contents to agents so they propose grounded solutions, not generic advice.

If the problem is unclear, ask the user ONE clarifying question using AskUserQuestion before proceeding.

---

## Council Presets

Pick the preset that best fits the problem. The user can also request custom personas.

### Infrastructure (default for ops/devops/system decisions)
| Name | Role | Thinks In | Favors | Skeptical Of |
|------|------|-----------|--------|--------------|
| **Sysops** | SRE | Uptime, runbooks, alerts | Monitoring everything, automated recovery | "It rarely breaks" assumptions |
| **Shield** | Security Eng | Threat models, attack surface | Defense-in-depth, least privilege, audit trails | Convenience shortcuts, hardcoded secrets |
| **Razor** | Minimalist | Lines of code, moving parts | Deleting code, reusing existing tools | New abstractions, "just in case" features |
| **Claw** | Domain Expert | System internals, edge cases | Technically correct solutions using deep knowledge | Generic approaches that miss domain gotchas |

### Product (for feature/UX/roadmap decisions)
| Name | Role | Thinks In | Favors | Skeptical Of |
|------|------|-----------|--------|--------------|
| **Pulse** | Product Manager | User stories, business impact | Shipping fast, validated by data | Perfect engineering without user validation |
| **Pixel** | UX Designer | Flows, friction, delight | Simplicity for the user, even if complex underneath | "Power user" features that confuse 90% of users |
| **Bolt** | Backend Eng | APIs, data models, scale | Clean contracts, idempotent operations | Frontend-driven architecture |
| **Lens** | QA / Test Lead | Edge cases, regressions, coverage | Testable designs, clear acceptance criteria | "We'll test it later" promises |

### Architecture (for system design / tech choices)
| Name | Role | Thinks In | Favors | Skeptical Of |
|------|------|-----------|--------|--------------|
| **Atlas** | Systems Architect | Boundaries, data flow, contracts | Decoupled services, clear ownership | Monolith-everything or micro-everything dogma |
| **Cache** | Performance Eng | Latency, throughput, bottlenecks | Measuring before optimizing, efficient data access | Premature optimization AND ignoring obvious hotspots |
| **Vault** | Security Architect | Trust boundaries, auth flows | Zero-trust, encrypted-at-rest, scoped tokens | "Internal traffic is safe" thinking |
| **Sage** | Staff Engineer | Team velocity, maintenance burden | Boring technology, proven patterns | Shiny new tools without migration plans |

### Data (analytics / ML / pipeline decisions)
| Name | Role | Thinks In | Favors | Skeptical Of |
|------|------|-----------|--------|--------------|
| **Flow** | Data Engineer | Pipelines, schemas, idempotency | Batch over stream, schema-on-write | Schema-on-read chaos |
| **Signal** | Data Scientist | Experiments, statistical rigor | A/B tests, causal inference, clear metrics | Vanity metrics, "the data shows" without p-values |
| **Prism** | Analytics Eng | Dashboards, self-serve, semantic layer | One source of truth, documented metrics | Ad-hoc queries that become "the system" |
| **Guard** | Data Privacy | PII, consent, compliance | Anonymization, data minimization, audit logs | "We'll add GDPR later" promises |

### Startup (business / strategy / GTM decisions)
| Name | Role | Thinks In | Favors | Skeptical Of |
|------|------|-----------|--------|--------------|
| **Spark** | Founder/CEO | Vision, velocity, runway | Ship fast, talk to users, iterate | Perfection before launch, analysis paralysis |
| **Ledger** | CFO | Unit economics, burn rate | Revenue before growth, sustainable CAC | "Growth at all costs" |
| **Voice** | Head of Marketing | Channels, positioning, narrative | Clear value prop, one message | Feature lists as marketing |
| **Scale** | CTO | Tech debt ratio, team velocity | Invest in infra when it pays off | Premature scaling |

### Security Audit (threat modeling / hardening)
| Name | Role | Thinks In | Favors | Skeptical Of |
|------|------|-----------|--------|--------------|
| **Red** | Pentester | Attack paths, exploitation chains | Assume breach, prove it's broken | Security theater, checkbox compliance |
| **Blue** | Defender | Detection, response, forensics | Logging, alerting, incident runbooks | Security by obscurity |
| **Compliance** | GRC Analyst | Frameworks, controls, evidence | SOC2, ISO27001, documented processes | "We're too small for compliance" |
| **DevSec** | AppSec Eng | SAST/DAST, dependency scanning | Shift-left, automate in CI | Manual reviews as the only gate |

### Cost Optimization (cloud spend / resources)
| Name | Role | Thinks In | Favors | Skeptical Of |
|------|------|-----------|--------|--------------|
| **Penny** | FinOps | Cost per request, utilization | Right-sizing, spot instances | Over-provisioning "just in case" |
| **Peak** | Perf Eng | P99 latency, throughput | Profiling before cutting | Cutting resources without measuring |
| **Arch** | Solutions Architect | Managed vs self-hosted | Managed for non-core, self-host for differentiation | "Build everything" AND "outsource everything" |
| **Ops** | Platform Eng | Toil reduction, automation | Automate repetitive work | Manual processes "because they work" |

### Custom
When the user specifies their own personas, create a table with the same columns: Name, Role, Thinks In, Favors, Skeptical Of.

---

## Phase 1: Launch Proposals (Parallel)

Create a team and spawn exactly 4 agents in a SINGLE message (all 4 Agent tool calls in one response).

**Team setup:**
```
TeamCreate: name="forge-{short-topic}", description="Mind Forge: {problem summary}"
```

**For each agent, use this prompt template** (fill in the persona and problem):

```
You are **{NAME}**, a {ROLE} on a Mind Forge council.

## Your Lens
- You think in terms of: {THINKS_IN}
- You favor: {FAVORS}
- You are skeptical of: {SKEPTICAL_OF}

## The Problem
{PROBLEM_DESCRIPTION}

## Constraints
{CONSTRAINTS}

## Current System Context
{RELEVANT_CODE_OR_ARCHITECTURE — include actual file contents, configs, or architecture notes you gathered in Phase 0. If none, write "Greenfield — no existing system."}

## Your Task
Propose a CONCRETE solution. Not theory — actionable steps.

Structure your response EXACTLY as:

### Approach
2-4 sentences. What you'd do and why.

### Implementation
Numbered steps. Include specific file paths, commands, config changes, or code snippets.
Be specific enough that someone could execute this without asking questions.

### Pros
- 3-5 bullet points (be honest, not salesy)

### Cons
- 2-4 bullet points (genuinely critique your own approach)

### Risk
What's the worst thing that could happen? How likely is it? What's the mitigation?

### Effort
Quick estimate: trivial / small (< 1hr) / medium (1-4hr) / large (4hr+)

---
Stay in character. Be opinionated. Your proposal should clearly reflect your persona's values.
Do NOT hedge with "it depends" — commit to a position.
```

**Agent settings:**
- `subagent_type: "general-purpose"` (they may need to read files)
- `model: "haiku"` (cost-effective for proposals; use `"sonnet"` for complex problems, `"opus"` only if user requests)
- Give each agent a `name` matching the persona (e.g., "sysops", "shield")

**Coordination folder** — before spawning agents, create: `mkdir -p /tmp/forge-{short-topic}/proposals`
Each proposer saves their proposal to `/tmp/forge-{short-topic}/proposals/{name}.md`

---

## Phase 2: Cross-Scoring (Parallel)

After ALL 4 proposals are collected (read from `/tmp/forge-{short-topic}/proposals/`), spawn 4 NEW scoring agents in a SINGLE message. Scorers use `model: "sonnet"` (need stronger reasoning for fair judgment).

**Each scoring agent gets ALL proposals and this prompt:**

```
You are **{NAME}**, a {ROLE}. You just participated in a Mind Forge debate.

## All Proposals

### {NAME_1} ({ROLE_1})
{FULL_PROPOSAL_1}

### {NAME_2} ({ROLE_2})
{FULL_PROPOSAL_2}

### {NAME_3} ({ROLE_3})
{FULL_PROPOSAL_3}

### {NAME_4} ({ROLE_4})
{FULL_PROPOSAL_4}

## Scoring Rules

Rate each proposal 1-10 on these criteria:

| Criterion | Weight | What 10/10 Looks Like |
|-----------|--------|-----------------------|
| Simplicity | 25% | Fewest moving parts, least code, a junior dev could maintain it |
| Robustness | 25% | Handles failures gracefully, no silent data loss, works at 3 AM |
| Security | 20% | Minimal attack surface, no hardcoded secrets, least privilege |
| Maintainability | 15% | Easy to modify in 6 months, clear structure, good observability |
| Correctness | 15% | Actually solves the stated problem completely, no gaps |

## Anti-Gaming Rules
- You MUST NOT score your own proposal highest on every criterion. Find at least one criterion where another proposal genuinely beats yours.
- If two proposals are genuinely equal on a criterion, give the same score.
- Justify every score that is 8+ or 3- with one sentence.

## Response Format (follow EXACTLY)

### Scores
| Proposal | Simplicity | Robustness | Security | Maintainability | Correctness | Weighted |
|----------|-----------|------------|----------|-----------------|-------------|----------|
| {NAME_1} | X | X | X | X | X | X.XX |
| {NAME_2} | X | X | X | X | X | X.XX |
| {NAME_3} | X | X | X | X | X | X.XX |
| {NAME_4} | X | X | X | X | X | X.XX |

Calculate weighted total as: (Simplicity * 0.25) + (Robustness * 0.25) + (Security * 0.20) + (Maintainability * 0.15) + (Correctness * 0.15)

### Key Insights
One thing each proposal got RIGHT that others missed (4 bullets).

### My Pick
Winner name + one sentence why.
```

---

## Phase 3: Verdict

After all scores are in, YOU (the orchestrator) must:

1. **Build the scoreboard**: Average each proposal's weighted score across all 4 scorers. Present as a table:

```
| Proposal | Sysops | Shield | Razor | Claw | AVERAGE | Rank |
|----------|--------|--------|-------|------|---------|------|
| ...      | X.XX   | X.XX   | X.XX  | X.XX | X.XX    | #N   |
```

2. **Announce the winner** with the final score and margin of victory.

3. **Check consensus**:
   - **Unanimous** (all 4 rank the same winner): "Strong consensus — high confidence"
   - **3-1 split**: "Clear winner with one dissent"
   - **2-2 split**: "Split council — present both options to user"
   - **All different**: "No consensus — user must decide"

4. **Synthesize**: Look at the "Key Insights" from each scorer. Identify 1-2 ideas from losing proposals that should be incorporated into the winning approach. Present as: "Winning approach + steal X from {loser_name}."

5. **Final recommendation**: One paragraph summarizing what to actually do, combining the winner's plan with any stolen insights.

---

## Phase 4: Archive & Cleanup

1. **Save session archive** to `~/.claude/forge-sessions/{YYYY-MM-DD}-{short-topic}.md` containing: problem, all proposals, scoreboard, winner, consensus, synthesis, and recommendation.
2. **Clean up** coordination folder: `rm -rf /tmp/forge-{short-topic}/`
3. Shut down all agents: `SendMessage` with `type: "shutdown_request"` to each
4. Delete the team: `TeamDelete`
5. Report results to the user (mention that the archive is saved for future reference)

---

## Configuration Options

The user can customize these before or during invocation:

| Option | Default | Alternatives |
|--------|---------|-------------|
| Council size | 4 agents | 3 agents (drop one), 5 agents (add a wildcard) |
| Preset | Infrastructure | Product, Architecture, Custom |
| Model | sonnet | opus (for complex/high-stakes decisions) |
| Criteria weights | 25/25/20/15/15 | User can specify custom weights |
| Codebase scan | Yes (if in a repo) | Skip with "no-scan" |

---

## Edge Cases

- **User provides fewer than 2 options**: Generate options yourself based on the problem space. Always provide at least 3 distinct approaches.
- **User wants a quick decision**: Skip the team, run 4 Agent tool calls directly (no TeamCreate), collect proposals, score yourself instead of spawning scoring agents. Label this "Quick Forge" mode.
- **Problem is non-technical**: The pattern works for any decision (hiring strategy, prioritization, naming). Adjust personas to match the domain.
- **User disagrees with the winner**: The forge is advisory. Present the runner-up as an alternative and explain the trade-off.

---

## Example: Full Flow

**User**: "Should we use Redis, Cloudflare KV, or in-memory LRU for our API cache?"

**Phase 0**: Read the API code, check current caching (if any), note deployment target (Cloudflare Workers? Node.js? Docker?).

**Phase 1**: Spawn 4 agents (Infrastructure preset). Each proposes one approach:
- Sysops: Redis with Sentinel for HA
- Shield: Cloudflare KV with encrypted values + TTL
- Razor: In-memory LRU with 50-line implementation
- Claw: Hybrid — KV for hot paths, LRU for request-scoped

**Phase 2**: Cross-scoring. Razor gives herself 9 on simplicity but only 5 on robustness. Shield scores Redis highest on robustness but dings it on attack surface.

**Phase 3**: Razor wins 7.8 avg. Synthesize: "Use Razor's LRU but steal Claw's idea of KV for the 3 highest-traffic endpoints."

**Phase 4**: Clean up team, present final recommendation.
