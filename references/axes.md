# Audit Axes Catalog

Each axis is a lens. Apply the specific patterns listed — don't freelance within an axis or you'll drift into adjacent territory and reduce orthogonality.

## Axis 1: Concurrency & Thread Safety

**Focus**: What happens when things run simultaneously?

Patterns to check:
- Mutable state accessed from multiple threads/actors without synchronization
- Blocking calls (`waitUntilExit`, `sleep`, synchronous I/O) on cooperative thread pools
- Spin-loops or busy-waits (polling `availableData`, tight retry loops)
- `@unchecked Sendable` hiding real safety violations vs. unnecessarily applied
- Race conditions in start/stop/init/deinit sequences
- Closures capturing mutable state across isolation boundaries
- Missing cancellation handling in long-running async operations

## Axis 2: Security & Trust Boundaries

**Focus**: Where does external input enter and what can it do?

Patterns to check:
- Command/argument injection via user-supplied filenames or paths
- Path traversal in file operations (`../` in names, digests, URLs)
- Untrusted network responses: unbounded sizes, malformed payloads, negative values
- Missing validation on decoded data (JSON, manifests, configs)
- Temp file predictability or symlink attacks
- Hardcoded secrets or credentials
- Input that controls resource allocation (page counts, image dimensions, array sizes)

## Axis 3: Resource Lifecycle & Data Integrity

**Focus**: What gets created, and does it always get cleaned up? What state survives a crash?

Patterns to check:
- Files created in temp directories with no cleanup path
- Child processes that become orphaned on cancellation or crash
- Partial filesystem state after interrupted multi-step operations
- Memory from unbounded collections (all-at-once vs. streaming/lazy)
- File handles, observers, or listeners registered without removal
- TOCTOU gaps between check-then-act sequences
- State that diverges between in-memory and on-disk representations

## Axis 4: Failure Propagation & User Experience

**Focus**: What does the human actually see when something goes wrong?

Patterns to check:
- Silent error swallowing (`try?`, empty `catch {}`, ignored return values)
- Dead code: error recovery methods that nothing calls
- Error states with no recovery path (stuck UI, no retry button)
- Missing feedback on user-initiated actions (export, save, delete)
- UI showing stale data during async operations
- Invisible data gaps (failed items silently omitted from aggregated output)
- Edge cases in normal use: empty inputs, duplicates, zero-count collections
- State machine gaps: unreachable states, missing transitions, double-entry

## Axis 5: API Contract & Type Correctness

**Focus**: Do the interfaces promise what the implementations deliver?

Patterns to check:
- Protocol conformances that compile but violate semantic contracts
- Optional return values that callers don't handle (force-unwraps on "should never be nil")
- Computed properties with hidden side effects or expensive computation
- Initializers that partially construct objects (some fields set, others deferred)
- Public API that exposes internal implementation details
- Enum switches missing cases or using default when exhaustive matching is safer
- Collection operations that assume non-empty inputs

## Axis 6: Observability & Debuggability

**Focus**: When something goes wrong in production, can you figure out what happened?

Patterns to check:
- Errors that lose context (re-thrown as generic, localizedDescription stripping details)
- Operations with no logging at decision points
- Async operations with no timeout or progress indication
- State transitions with no way to trace cause
- External process invocations with discarded stdout/stderr

## Choosing Orthogonal Pairs

Maximum distance pairs (use these for consecutive passes):
- Concurrency ↔ Failure UX (internal machinery vs. external experience)
- Security ↔ Resource Lifecycle (input boundaries vs. output cleanup)
- API Contracts ↔ Observability (static correctness vs. runtime visibility)

Minimum distance pairs (avoid these for consecutive passes):
- Concurrency ↔ Resource Lifecycle (both focus on runtime state)
- Security ↔ API Contracts (both focus on boundaries/interfaces)
- Failure UX ↔ Observability (both focus on what happens when things break)
