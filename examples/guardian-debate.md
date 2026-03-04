# Real Example: OpenClaw Patch Guardian Debate

**Date**: March 3, 2026
**Preset**: Infrastructure (Sysops, Shield, Razor, Claw)
**Result**: Razor won unanimously (8.52/10 weighted average)

## The Problem

OpenClaw runs in Docker (`openclaw-gateway`). Updates overwrite `/app/dist/`. Custom patches to `wa_daemon.js` (WhatsApp protocol fix, contact labels, rate limiting) could be lost. We need a system to protect these patches.

**Constraints**: Existing health_monitor.py runs every 5 min via cron. Telegram alerts already work. Docker compose bind-mounts `/root/.openclaw/` as persistent storage.

---

## Phase 1: Proposals

### Sysops — Dual-Layer Docker Events + health_monitor

> "I want two layers of protection. Belt AND suspenders."

**Approach**: A systemd service listening to `docker events` triggers patch re-application via `docker cp` + `docker exec` on every container start. health_monitor.py gets a `check_patches()` function as safety net.

**Implementation**:
1. Create `/root/recovery_watchdog/patches/` with golden copies of all patched files
2. Build `patch-guardian.sh` (~150 lines bash) that listens to docker events
3. On container start: copy patched files into container, verify checksums
4. Register as systemd service for auto-start on boot
5. Add `check_patches()` to health_monitor.py (~50 lines) as 5-min fallback

**Pros**: Catches every restart instantly (docker events). Two independent systems. Proven systemd integration.

**Cons**: Two systems to maintain. Bash scripts are fragile. 200+ lines of new code. systemd service is another thing that can break.

**Effort**: medium (2-3hr)

---

### Shield — Host-Side Python with JSON Patch Definitions

> "I want cryptographic integrity verification and atomic operations."

**Approach**: A Python guardian script with JSON-defined patches, SHA256 integrity checks, and atomic file operations. Triggered by docker events + 10-min cron fallback.

**Implementation**:
1. Create `/root/recovery_watchdog/startup_guardian.py` (~300 lines)
2. Define patches as JSON: `{"file": "wa_daemon.js", "search": "...", "replace": "...", "sha256_before": "...", "sha256_after": "..."}`
3. Use Python `re.sub()` instead of sed (no shell injection risk)
4. Atomic operations: write to temp file, rename into place
5. Reuse existing `spark_secrets.py` for secrets, health_monitor for alerting

**Pros**: Cryptographic verification. Atomic operations. No shell injection. Reuses existing alerting.

**Cons**: 300 lines of new code. SHA256 hashes need updating when patches change. JSON patch definitions are brittle. Over-engineered for 4 checks.

**Effort**: large (4hr+)

---

### Razor — Add 25 Lines to health_monitor.py

> "You're all solving a problem that doesn't exist. The patches are already persistent."

**Approach**: The patches live in `/root/.openclaw/` which is a bind mount — `openclaw update` only touches `/app/dist/`. The real gap is that nobody *verifies* the patches are correct. Add a `check_patch_integrity()` function to health_monitor.py. That's it.

**Implementation**:
1. Add `check_patch_integrity()` (~25 lines) to `/root/recovery_watchdog/health_monitor.py`
2. Check 4 things: (a) protocol version in wa_daemon.js, (b) contact labels, (c) supervisor path, (d) skills count
3. If any check fails → Telegram alert (existing infrastructure)
4. Delete the 193-line `startup_guardian.sh` that was already written
5. Optional: one-liner `@reboot` cron for startup detection

**Pros**: Zero new files. Zero new infrastructure. Reuses everything that already works. A junior dev can read 25 lines.

**Cons**: Runs every 5 min, not on startup (5-min gap). Doesn't auto-fix, only alerts. Relies on one file (health_monitor.py).

**Effort**: small (< 30min)

---

### Claw — Docker Entrypoint Wrapper

> "Everyone's worried about `openclaw update` breaking wa_daemon.js, but it CAN'T — wa_daemon.js isn't in /app/."

**Approach**: Override the Docker entrypoint with a custom script that runs before OpenClaw starts. Copies golden supervisor config, verifies wa_daemon.js, runs doctor --fix, then chains to the original entrypoint.

**Key insight**: `wa_daemon.js` lives at `/root/.openclaw/wa_daemon.js` (bind-mounted workspace), NOT in `/app/dist/`. `openclaw update` replaces `/app/dist/` only. The only real risk is a full container rebuild from a fresh image.

**Implementation**:
1. Create `/guardian/startup-guardian.sh` on host
2. Volume-mount it in docker-compose.yml
3. Override `entrypoint:` in docker-compose.yml
4. Script: copy golden supervisord.conf, verify wa_daemon.js, run smoke tests, exec original entrypoint

**Pros**: Runs on EVERY start (not 5-min delay). Correctly identifies real vs imaginary risks. Uses Docker's own mechanism.

**Cons**: Overriding entrypoint is fragile. If the script fails, container won't start. docker-compose change needed. Adds startup latency.

**Effort**: medium (1-2hr)

---

## Phase 2: Scores

**Criteria**: Simplicity (25%), Robustness (25%), Security (20%), Maintainability (15%), Correctness (15%) — weighted average out of 10.

### Scored by Claw (OpenClaw Expert)
| Proposal | Simplicity | Robustness | Security | Maintainability | Correctness | Weighted |
|----------|-----------|------------|----------|-----------------|-------------|----------|
| Sysops | 5 | 8 | 6 | 5 | 7 | 6.25 |
| Shield | 4 | 8 | 8 | 4 | 7 | 6.15 |
| **Razor** | **10** | **6** | **9** | **10** | **9** | **8.65** |
| Claw | 6 | 8 | 7 | 6 | 8 | 6.95 |

> "This is the painful truth. Razor correctly identifies that wa_daemon.js and workspace files are already safe via bind mounts — the threat model everyone else (including me) inflated."

### Scored by Razor (Minimalist)
| Proposal | Simplicity | Robustness | Security | Maintainability | Correctness | Weighted |
|----------|-----------|------------|----------|-----------------|-------------|----------|
| Sysops | 5 | 8 | 7 | 5 | 7 | 6.35 |
| Shield | 3 | 9 | 9 | 3 | 7 | 6.20 |
| **Razor** | **10** | **6** | **9** | **10** | **8** | **8.45** |
| Claw | 6 | 7 | 7 | 6 | 8 | 6.75 |

> "The most technically thorough proposal... for a 300-line Python script protecting four patches that live on bind mounts and rarely change. This is building a bank vault to protect a $20 bill."

### Scored by Sysops (SRE)
| Proposal | Simplicity | Robustness | Security | Maintainability | Correctness | Weighted |
|----------|-----------|------------|----------|-----------------|-------------|----------|
| Sysops | 5 | 8 | 7 | 5 | 6 | 6.15 |
| Shield | 4 | 8 | 9 | 4 | 6 | 6.10 |
| **Razor** | **10** | **6** | **8** | **10** | **9** | **8.45** |
| Claw | 6 | 8 | 7 | 6 | 8 | 6.95 |

> "Claw and Razor raise an important point... the primary threat model I (and Shield) operated under — that updates overwrite wa_daemon.js — is likely **wrong**."

### Shield (Security Eng)
Shield did not submit scores after multiple nudges. This happens — in a real forge, the orchestrator proceeds with 3/4 scorers.

---

## Phase 3: Verdict

| Proposal | Claw | Razor | Sysops | **AVERAGE** | **Rank** |
|----------|------|-------|--------|-------------|----------|
| Sysops | 6.25 | 6.35 | 6.15 | 6.25 | #3 (tie) |
| Shield | 6.15 | 6.20 | 6.10 | 6.15 | #4 |
| **Razor** | **8.65** | **8.45** | **8.45** | **8.52** | **#1** |
| Claw | 6.95 | 6.75 | 6.95 | 6.88 | #2 |

**Winner: Razor** — 8.52/10 weighted average, **unanimous** (all 3 scoring agents ranked Razor #1)

**Consensus**: Strong — every agent, including the ones with complex proposals, admitted Razor was right.

**Synthesis**: Razor's approach + steal Claw's insight that the *real* risk is supervisor config on image rebuild (not wa_daemon.js on update). Store a golden supervisord.conf on the host and verify it.

**What was actually implemented**: `check_patch_integrity()` added to health_monitor.py (~35 lines). Checks: protocol version, contact labels, supervisor path (host-side since container mount is read-only), skills count. Auto-copies patched wa_daemon.js if the runtime copy is stale. Runs every 5 min. Zero new files.

---

## Why This Example Matters

The debate revealed that 3 out of 4 agents were solving the **wrong problem**. They assumed `openclaw update` would overwrite wa_daemon.js — but it can't, because wa_daemon.js lives in bind-mounted workspace (`/root/.openclaw/`), not in the npm package (`/app/dist/`).

Only Razor and Claw caught this. Razor's response was: "if the problem doesn't exist, don't build a solution for it." Claw built a solution anyway. Razor won because doing nothing (except verification) was genuinely the best answer.

This is the core value of Mind Forge: **surface blind spots by making agents argue from different angles.** A single agent would have built Shield's 300-line solution and moved on. The debate caught the false premise.
