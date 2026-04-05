---
name: orthogonal-audit
description: Multi-axis code audit that rotates perspective each pass to find bugs invisible to single-angle reviews. Use when implementation is believed complete, before committing or declaring work done. Triggers on "audit this", "final review", "make sure this is solid", or when work is about to be marked complete after significant implementation.
---

# Orthogonal Audit

Audit code from multiple orthogonal directions, fixing bugs at each pass, then rotating the axis until only P3 (cosmetic/trivial) issues remain.

## Priority Tiers

- **P1**: Crashes, data loss, security holes — ship-blockers
- **P2**: Broken workflows, confusing errors, wrong behavior users will hit — fix before release
- **P3**: Cosmetic, edge-case, "would be nice" — fix when convenient

**Stop condition**: When a pass finds only P3 issues, the audit is complete.

## Process

### Step 1: Identify the Surface

Determine what code to audit. Use `git diff` for recent changes, or if the scope is broader (like a full codebase review), collect all production source files. Exclude generated code, build artifacts, and vendored dependencies.

### Step 2: Choose the First Axis

Pick the axis most aligned with the code's primary risk. See `references/axes.md` for the full catalog.

Common first picks by project type:
- **Networked apps**: Start with Security/Trust Boundaries
- **Concurrent/async code**: Start with Concurrency & Thread Safety
- **User-facing apps**: Start with Failure Propagation & UX
- **Data pipelines**: Start with Data Integrity & Resource Lifecycle

### Step 3: Run the Pass

Launch 2-3 **parallel** review agents, each covering a different sub-focus within the chosen axis. Give each agent:
1. The full diff or source code
2. Specific patterns to look for (from the axis definition in `references/axes.md`)
3. Instructions to report: file, line, what's wrong, severity (P1/P2/P3), and fix

### Step 4: Fix and Verify

Aggregate findings from all agents. Fix every P1 and P2 issue directly. For each fix:
- Edit the code
- Build to verify compilation
- Run tests to verify correctness

Skip false positives or P3 issues with a brief note — don't argue, just move on.

### Step 5: Rotate the Axis

Pick the axis **most orthogonal** to what you just examined. The goal is maximum cognitive distance — if you just looked at threading, don't look at performance next (they share a lens). Look at security, or UX, or data integrity instead.

**Rotation principle**: Each pass should find bugs that were *invisible* to the previous pass's lens. If a new pass only finds things the prior pass could have caught, the axis wasn't orthogonal enough — rotate further.

### Step 6: Repeat Until P2 Floor

Continue rotating and auditing until a full pass produces only P3 findings. Then stop — diminishing returns beyond this point.

Typical audit depth:
- **Small change** (1-3 files): 2 passes usually sufficient
- **Feature** (5-15 files): 3 passes typical
- **Full codebase**: 3-4 passes before hitting P2 floor

### Step 7: Summary

After completing all passes, output a single consolidated table of everything found and fixed, grouped by pass/axis. This gives the human a clear picture of what the audit covered and what changed.
