---
name: llm-council
description: "Multi-agent debate council for architecture decisions. Use when the user asks to debate, compare approaches, run a council, or wants multiple perspectives on a technical decision. Triggers: council, debate, compare approaches, llm council, agent debate, score proposals, multi-agent decision"
user-invocable: true
---

# LLM Council — Multi-Agent Debate Pattern

Run a structured debate between 4 AI agents with distinct personas. Each proposes a solution, then they cross-score each other. The best approach wins by consensus.

## When to Use

- Architecture decisions with multiple valid approaches
- Choosing between tools, frameworks, or patterns
- Evaluating trade-offs (security vs. simplicity, speed vs. robustness)
- Any decision where you want more than one perspective

## The 4 Default Personas

| Name | Role | Focus | Bias | Skeptical Of |
|------|------|-------|------|--------------|
| **Sysops** | SRE / Infrastructure | Reliability, observability, automation | Wants monitoring + alerting for everything | Simplicity claims that skip error handling |
| **Shield** | Security Engineer | Attack surface, secrets, permissions | Wants defense-in-depth | Convenience shortcuts |
| **Razor** | Minimalist / Pragmatist | Simplicity, fewer moving parts | Wants the least code possible | Over-engineering, premature abstraction |
| **Claw** | Domain Expert | Deep knowledge of the specific system | Wants technically correct solutions | Generic approaches that ignore domain specifics |

### Customizing Personas

The user can request different personas. When they do, assign each:
- A **name** (short, memorable)
- A **role** (1-3 words)
- A **focus area** (what they optimize for)
- A **bias** (what they tend to favor)
- A **skepticism** (what they push back on)

## Process (4 Phases)

### Phase 1: Frame the Problem

Before launching agents, clearly state:
1. The decision/problem to solve
2. Key constraints (existing systems, budget, timeline)
3. What "success" looks like
4. Any approaches already considered

### Phase 2: Parallel Proposals (4 agents simultaneously)

Use `TeamCreate` to create a council team, then spawn 4 agents via the `Agent` tool, each with their persona's system prompt:

**Agent Prompt Template (Proposal Phase):**
```
You are {NAME}, a {ROLE} on an LLM Council debate.

Your focus: {FOCUS}
Your bias: You tend to favor {BIAS}
Your skepticism: You push back on {SKEPTICISM}

## Problem
{PROBLEM_DESCRIPTION}

## Constraints
{CONSTRAINTS}

## Your Task
Propose a concrete solution. Include:
1. **Approach** (2-3 sentences)
2. **Implementation** (specific steps, files, commands)
3. **Pros** (3-5 bullet points)
4. **Cons** (be honest about weaknesses)
5. **Risk assessment** (what could go wrong)

Stay in character. Be opinionated. Disagree with approaches that conflict with your values.
```

### Phase 3: Cross-Scoring (4 agents simultaneously)

After collecting all proposals, launch 4 new agents (or message existing teammates). Each agent scores ALL proposals (including their own) on 5 criteria:

**Scoring Criteria:**

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Simplicity | 25% | Fewer moving parts, less code, easier to understand |
| Robustness | 25% | Handles failures, edge cases, degraded states |
| Security | 20% | Minimizes attack surface, protects secrets, least privilege |
| Maintainability | 15% | Easy to modify, debug, hand off to someone else |
| Correctness | 15% | Actually solves the stated problem completely |

**Agent Prompt Template (Scoring Phase):**
```
You are {NAME}, a {ROLE}.

You just participated in a council debate. Here are all 4 proposals:

## Proposal 1: {NAME_1} ({ROLE_1})
{PROPOSAL_1}

## Proposal 2: {NAME_2} ({ROLE_2})
{PROPOSAL_2}

## Proposal 3: {NAME_3} ({ROLE_3})
{PROPOSAL_3}

## Proposal 4: {NAME_4} ({ROLE_4})
{PROPOSAL_4}

## Score each proposal (1-10) on these criteria:
- Simplicity (25%): Fewer moving parts, less code
- Robustness (25%): Handles failures and edge cases
- Security (20%): Minimizes attack surface
- Maintainability (15%): Easy to modify and debug
- Correctness (15%): Solves the actual problem

Format your response as:
### Scores
| Proposal | Simplicity | Robustness | Security | Maintainability | Correctness | Weighted Total |
|----------|-----------|------------|----------|-----------------|-------------|----------------|
| {NAME_1} | X/10 | X/10 | X/10 | X/10 | X/10 | X.X |
| ... | ... | ... | ... | ... | ... | ... |

### Reasoning
Brief explanation for each score. Be fair but stay in character.

### Winner
Who you think should win and why (1 sentence).
```

### Phase 4: Verdict

After all scores are in:

1. **Calculate weighted averages** across all 4 scorers for each proposal
2. **Announce winner** with final scores
3. **Note consensus** — did all agents agree? Split vote?
4. **Synthesize** — can the winning approach incorporate good ideas from runners-up?

**Tie-breaking rules:**
- If two proposals are within 0.5 points: recommend a hybrid
- If one proposal wins unanimously (all 4 rank it #1): strong recommendation
- If votes are evenly split (2-2): present both as viable, ask user to decide

## Implementation Notes

- Use `subagent_type: "general-purpose"` for all agents
- Run proposal agents in parallel (single message with 4 Agent tool calls)
- Run scoring agents in parallel (single message with 4 Agent tool calls)
- Each agent should have access to: Read, Grep, Glob, Bash (for research)
- Agents do NOT need Write/Edit access (they're advisors, not implementers)
- Use `model: "sonnet"` for agents to save cost (opus for complex problems)

## Example Invocation

User: "I need to decide how to handle database migrations in our Next.js app. Options: Prisma Migrate, Drizzle Kit, raw SQL scripts, or Supabase migrations."

This triggers:
1. Frame: Next.js app, Supabase backend, team of 2, need reproducible migrations
2. Propose: 4 agents each argue for their preferred approach
3. Score: 4 agents rate all proposals on the 5 criteria
4. Verdict: Weighted average determines winner

## Quick Start

When the user says `/council` or asks for a council debate:

1. Ask: "What decision do you need help with?" (if not already stated)
2. Create team: `TeamCreate` with name like "council-{topic}"
3. Spawn 4 agents with personas + problem context
4. Collect proposals → run scoring round → announce winner
5. Clean up: shut down agents, delete team
