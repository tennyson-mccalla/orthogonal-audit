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

## Anti-patterns that destroy audit value

These are the failure modes that turn a real audit into theater. If you catch yourself doing any of them, stop and restart the pass properly.

- **Inline analysis instead of parallel sub-agents.** Reading the files yourself in the main context and then writing findings is NOT an audit — it's one reasoning pass dressed up as several. The whole point of "orthogonal" is independent perspectives that can disagree with each other. Even on small surfaces, you MUST dispatch parallel sub-agents per pass. No exceptions for "the diff is small" or "the files are already loaded."
- **Asserting bugs without verifying they exist.** Before claiming "X is broken because Y," grep for callers, check if a guard already exists elsewhere, and confirm the symptom is reachable. A "bug" that's actually already handled three frames up the call stack is a false finding that erodes trust.
- **Reasoning about behavior instead of running it.** When the verification step (Step 4) says "build and run tests," do it — even if the user normally handles building. Audits are the one workflow where reasoning is not enough.
- **Speed bragging.** If a multi-file audit took less time than it would take a careful human to *read the diff line-by-line*, it was too fast. As a rough floor: ~1 minute of wall-clock per ~50 lines of changed code, per pass. Less than that means sub-agents weren't actually doing independent work.
- **"P2 floor reached" without enumerating coverage gaps.** Stopping the audit is fine; claiming the surface is clean without listing what you didn't check is not. See Step 7.

## Process

### Step 1: Identify the Surface

Determine what code to audit. Use `git diff` for recent changes, or if the scope is broader (like a full codebase review), collect all production source files. Exclude generated code, build artifacts, and vendored dependencies.

Count the changed/relevant lines. Use the count to set realistic expectations for how long each pass will take (see "Speed bragging" above).

### Step 2: Choose the First Axis

Pick the axis most aligned with the code's primary risk. See `references/axes.md` for the full catalog.

Common first picks by project type:
- **Networked apps**: Start with Security/Trust Boundaries
- **Concurrent/async code**: Start with Concurrency & Thread Safety
- **User-facing apps**: Start with Failure Propagation & UX
- **Data pipelines**: Start with Data Integrity & Resource Lifecycle

### Step 3: Run the Pass (parallel sub-agents, mandatory)

Dispatch **2–3 parallel sub-agents** per pass. This is not optional. Each agent must run in its own context — not as inline analysis in the main conversation.

Why: independent contexts produce independent reasoning. When two of three agents flag the same issue from different angles, it's a real finding. When one agent flags it and the others don't, it's worth investigating but treat it as lower confidence. Inline analysis collapses to a single perspective and gives you false confidence.

For each agent provide:
1. The full diff or source code (don't make them re-discover the surface)
2. The specific axis and sub-focus they own (from `references/axes.md`)
3. Patterns to look for, with examples
4. Required output format: file, line, what's wrong, severity (P1/P2/P3), suggested fix, and **how the agent verified the bug exists** (grep result, traced call site, etc.)
5. Explicit instruction: "Do not report a finding unless you have verified the bug is reachable. Speculative findings are worse than missed ones."

### Step 4: Fix and Verify

Aggregate findings from all agents. Cross-check overlaps — if multiple agents flagged the same issue, raise confidence. If only one did, verify before fixing.

For every P1 and P2:
1. **Verify the bug is real before fixing.** Grep for existing handlers, check if the symptom is reachable from any call site, confirm it's not already mitigated upstream.
2. Edit the code.
3. **Build to verify compilation.** Even if the user normally handles building, run a build during audits. If the user has a "don't run the app" rule, that doesn't apply to compilation/test runs — only to launching the app interactively.
4. **Run tests to verify correctness.** If no tests cover the changed area, say so explicitly in the summary.

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

### Step 7: Summary with Coverage Gaps

After completing all passes, output:

1. **A consolidated findings table** grouped by pass/axis: file, line, severity, finding, fix applied (or deferred with reason).
2. **A "Coverage Gaps" section** that explicitly enumerates what was NOT examined. This is mandatory. Examples of what belongs here:
   - Files that were skimmed rather than read line-by-line
   - Axes that were skipped (and why)
   - Findings judged out of scope (and where they should be addressed instead)
   - Hypotheses not verified (e.g., "I claimed X is a bug but did not trace whether the call site is reachable")
   - Tests not run (and which assertions remain unverified as a result)
   - Cross-subsystem interactions deferred to a later audit
3. **Time spent per pass** (rough wall-clock or token estimate). If any pass was suspiciously fast, flag it for re-running.

A "P2 floor reached" claim without a Coverage Gaps section is not a complete audit. Reviewers should be able to look at the gaps and decide whether to run another pass.
