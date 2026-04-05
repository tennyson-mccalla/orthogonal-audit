# Orthogonal Audit

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that audits code from multiple orthogonal directions, rotating perspective each pass to find bugs invisible to single-angle reviews.

Traditional code review — whether human or AI — tends to apply the same cognitive lens repeatedly. Orthogonal auditing deliberately rotates the analysis axis between passes (e.g., concurrency -> security -> UX -> data integrity), maximizing the chance of catching bugs that any single perspective would miss.

## Inspired By

This skill is inspired by Martin Brodeur's paper [*The Logos Made Code: Orthogonal Probing and the Geometry of LLM Entropy*](https://doi.org/10.5281/zenodo.19421068) and the accompanying [Hacker News discussion](https://news.ycombinator.com/item?id=47630600).

The key insight: LLMs audit code from the same blind spots they wrote it from (Generator-Auditor Symmetry). Rotating a single model across orthogonal axes outperforms using multiple different models on the same axis. Brodeur's experiments showed ~4-5x improvement in bug-class discovery yield over conventional same-axis methods on a 350,000-line TypeScript codebase.

## How It Works

1. **Identify the surface** — Determine what code to audit (recent diff or full codebase)
2. **Choose an axis** — Pick the lens most aligned with the code's primary risk
3. **Run the pass** — Launch parallel review agents, each covering sub-patterns within the axis
4. **Fix and verify** — Fix all P1 (ship-blockers) and P2 (fix-before-release) issues
5. **Rotate orthogonally** — Pick the axis with maximum cognitive distance from the last
6. **Repeat** — Continue until a pass produces only P3 (cosmetic/trivial) findings

### The Six Axes

| Axis | Focus |
|------|-------|
| **Concurrency & Thread Safety** | What happens when things run simultaneously? |
| **Security & Trust Boundaries** | Where does external input enter and what can it do? |
| **Resource Lifecycle & Data Integrity** | What gets created, and does it always get cleaned up? |
| **Failure Propagation & UX** | What does the human see when something goes wrong? |
| **API Contract & Type Correctness** | Do interfaces promise what implementations deliver? |
| **Observability & Debuggability** | When something breaks in prod, can you figure out what happened? |

### Priority Tiers

- **P1**: Crashes, data loss, security holes — ship-blockers
- **P2**: Broken workflows, confusing errors, wrong behavior users will hit — fix before release
- **P3**: Cosmetic, edge-case, "would be nice" — fix when convenient

The audit stops when a pass finds only P3 issues.

## Installation

Copy this skill into your Claude Code skills directory:

```bash
# Clone the repo
git clone https://github.com/tennyson-mccalla/orthogonal-audit.git

# Copy to your Claude Code skills directory
cp -r orthogonal-audit ~/.claude/skills/orthogonal-audit
```

Or if you prefer a one-liner:

```bash
git clone https://github.com/tennyson-mccalla/orthogonal-audit.git ~/.claude/skills/orthogonal-audit
```

## Usage

Once installed, Claude Code will automatically recognize the skill. Trigger it with phrases like:

- "audit this"
- "final review"
- "make sure this is solid"

Or invoke it directly:

```
/orthogonal-audit
```

## License

MIT
