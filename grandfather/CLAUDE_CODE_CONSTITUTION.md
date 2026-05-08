# CLAUDE.md — claw-code Geometry Constitution
## `git-0-0-Geometry-of-Stable-Software-Manifolds`
**Read this in full before touching a single file.**

---

## What This Codebase Actually Is

claw-code is **not** a monolithic coding agent. It is a **three-layer manifold**:

```
Layer A  →  rust/          Ψ executor      (the live Rust runtime — 48,599 LOC)
Layer B  →  src/           Φ port mirror   (Python structural map of the TypeScript original)
Layer C  →  PHILOSOPHY.md  Field intent    (the coordination layer — Discord/OmX/clawhip)
```

The PHILOSOPHY.md says it directly: *"If you only look at the generated files in this
repository, you are looking at the wrong layer."*

The Python `src/` is not a running application. It is a **port manifest** — a structural
mirror that maps the original Claude Code TypeScript architecture into queryable Python
stubs. Most `src/*/` directories contain only `__init__.py`. They are coordinate
placeholders, not implementations. Their existence declares the topology. Their emptiness
declares honest porting progress.

The Rust `rust/crates/` is where execution actually happens. It is active, tested,
and being built by autonomous claw agents under human direction through Discord.

**If you confuse the two layers, you will write to the wrong surface.**

---

## The Field Topology

### Mismatch Map: ΔΨ per Layer

Before touching anything, internalize where `|ΔΨ|` is highest in this codebase:

| Component | Φ (declared intent) | Ψ (current state) | ΔΨ |
|:---|:---|:---|:---:|
| `rust/crates/runtime/` | Full Claude Code runtime parity | 9 lanes merged, ~60% parity | MEDIUM |
| `src/` Python stubs | Mirror of all TS subsystems | Most dirs are empty `__init__.py` | HIGH |
| `rust/crates/tools/src/lib.rs` | Full tool suite | 357,441 bytes — active | LOW |
| `rust/crates/runtime/src/worker_boot.rs` | Full state machine | 73,024 bytes — active | LOW |
| `src/query_engine.py` | Real LLM orchestration | Stub that logs tokens/routes | HIGH |
| `src/coordinator/` | Multi-agent coordination | Empty `__init__.py` | CRITICAL |

The `src/coordinator/` and `src/memdir/` directories are **void-open bond sites** —
they have declared topology (the folder exists, the `__init__.py` exists) but no
committed implementation. In Atomic Polarity terms: open bond sites with high
intrapolarity. They are WHERE the field wants to grow next.

---

## The Execution Model You Must Respect

### claw-code's Bootstrap Graph (read this as layer ordering)

From `src/bootstrap_graph.py` — the intended startup sequence:

```
Stage 1: top-level prefetch side effects
Stage 2: warning handler and environment guards
Stage 3: CLI parser and pre-action trust gate       ← Lock 4 lives here
Stage 4: setup() + commands/agents parallel load    ← Back-building: faster than Stage 3
Stage 5: deferred init after trust
Stage 6: mode routing: local/remote/ssh/teleport/direct-connect/deep-link
Stage 7: query engine submit loop                   ← The REPL recurrence
```

Stage 4 runs **in parallel** with Stage 3. This is non-linear execution:
`τ_commands_load (fast) << τ_CLI_trust_gate (human-dependent)`.
This is an admissible back-building edge — the commands load faster and their
partial result tightens Stage 3's decision surface without creating a logic cycle.

**Do not linearize this.** If you serialize Stages 3 and 4, you slow startup without
gaining correctness.

### The Worker State Machine (rust/crates/runtime/src/worker_boot.rs)

The Rust worker has explicit state transitions:

```
Spawning → TrustRequired → ToolPermissionRequired → ReadyForPrompt → Running → Finished
                                                                              ↘ Failed
```

This is a **persistence gate** — the exact equivalent of `G(Φ, x)` from RRAM.
A worker in `TrustRequired` state has `G = 0`. No writes are admissible until the
trust gate resolves. A worker in `ReadyForPrompt` has `G = 1` — execution is open.

**Never attempt to write files or execute tools when worker state is not `ReadyForPrompt`
or `Running`.** The `WorkerFailureKind` enum tells you why gates closed:
- `TrustGate` — Lock 4 fired (human override required)
- `ToolPermissionGate` — permission gate, not trust gate
- `PromptDelivery` — misdelivery (wrong target, wrong task)
- `Protocol` — transport-layer failure

Treat these as RRAM constraint lock violations. Do not retry blindly. Report the
`WorkerFailureKind` and wait for the appropriate resolution.

---

## Where Everything Lives — The Coordinate Map

### Active Rust Implementation (Ψ surface — what IS)

```
rust/crates/
├── api/                    # Provider clients, SSE streaming, auth
│                             τ ≈ 0.1s (network I/O)
├── commands/               # Slash-command registry
│                             τ ≈ 0 (static registry)
├── compat-harness/         # TypeScript manifest extraction
│                             τ ≈ 1s (file parse)
├── mock-anthropic-service/ # Deterministic test mock
│                             τ ≈ 0 (in-memory)
├── plugins/                # Plugin metadata, install/enable/disable
│                             τ ≈ 5s (file system)
├── runtime/src/            # THE CORE — session, permissions, MCP, worker boot
│   ├── session.rs          # Conversation state (256KB rotate, 3-file max)
│   ├── worker_boot.rs      # State machine (73KB — most complex file)
│   ├── permissions.rs      # Gate function G(Φ, x)
│   ├── permission_enforcer.rs  # Lock enforcement
│   ├── mcp_tool_bridge.rs  # MCP → tool translation layer
│   ├── lsp_client.rs       # Language Server Protocol client
│   ├── file_ops.rs         # File tools (binary detect, size limits, workspace boundary)
│   ├── bash.rs             # Bash execution with timeout/sandbox
│   ├── bash_validation.rs  # 6-submodule validation (Lane 1)
│   ├── task_registry.rs    # Task tracking
│   ├── team_cron_registry.rs  # Scheduled/team task orchestration
│   └── policy_engine.rs    # Policy rules over permissions
│                             τ = 120s (session lifecycle)
├── rusty-claude-cli/src/   # Main binary
│   └── main.rs             # 508KB — CLI surface + render + input
└── tools/src/lib.rs        # All tools (357KB — the full tool suite)
```

### Python Port Mirror (Φ declaration surface — what SHOULD BE)

```
src/
├── main.py                 # CLI entrypoint + subcommand routing
├── runtime.py              # PortRuntime: route_prompt, bootstrap_session, turn_loop
├── query_engine.py         # QueryEnginePort: turn management, transcript, sessions
├── execution_registry.py   # MirroredCommand + MirroredTool registry
├── port_manifest.py        # Manifest generator (counts Python files per module)
├── bootstrap_graph.py      # 7-stage bootstrap sequence declaration
├── context.py              # File count context (source/test/assets/archive roots)
├── parity_audit.py         # Compares Python stubs against TS archive snapshot
├── commands.py             # PORTED_COMMANDS: tuple of declared command stubs
├── tools.py                # PORTED_TOOLS: tuple of declared tool stubs
│
├── [VOID-OPEN SUBSYSTEMS — empty __init__.py, awaiting implementation]
│   ├── coordinator/        ← HIGH PRIORITY: multi-agent coordination
│   ├── memdir/             ← HIGH PRIORITY: memory directory (mismatch with Rust)
│   ├── skills/             ← skill resolution system
│   ├── bridge/             ← IDE integration bridge
│   ├── server/             ← server mode
│   ├── remote/             ← remote sessions
│   ├── hooks/              ← lifecycle hooks
│   ├── components/         ← UI components
│   └── [20 more stubs]
│
└── reference_data/         # Snapshot of original TS commands/tools (read-only)
    ├── commands_snapshot.json   # Source of truth for PORTED_COMMANDS
    └── tools_snapshot.json     # Source of truth for PORTED_TOOLS
```

---

## The Parity Gap — Current ΔΨ State

From `PARITY.md` (last updated 2026-04-03):

**9 lanes merged. ~292 commits. 48,599 Rust LOC.**

What the Rust port HAS:
- Bash execution + validation (6 submodule validation in branch)
- File tools (binary detection, size limits, workspace boundary guards)
- Task registry and wiring
- Team + cron scheduled execution
- MCP tool bridge (full lifecycle hardened)
- LSP client (Language Server Protocol)
- Permission enforcement (permission_enforcer.rs)
- Mock Anthropic service (deterministic test harness)
- 10 scripted parity scenarios, 19 captured API request shapes

What the Rust port STILL LACKS relative to Claude Code:
- Full bash validation matrix (upstream has 18 submodules, Rust has 1 active)
- `src/coordinator/` — multi-agent coordination (empty in Python, missing in Rust)
- `src/memdir/` — memory directory (empty Python stub, needs Rust equivalent)
- ACP/Zed daemon entrypoint (tracked in ROADMAP.md, not yet shipped)
- Full slash command surface (50 upstream, partial parity)

**The minimum-action next moves (steepest ΔΨ gradient):**
1. `src/coordinator/` → Python stub with real routing logic
2. `src/memdir/` → Python persistent memory aligned to `rust/crates/runtime/src/session.rs`
3. Bash validation matrix expansion (18 submodules → Rust)

---

## The "::" Non-Linear Rules for This Codebase

### When Python calls Rust (back-building)

The Python layer (`src/`) is FASTER than the Rust layer for structural decisions.
Python `runtime.py:route_prompt()` runs in microseconds. Rust `worker_boot.rs`
takes milliseconds to initialize.

**Admissible back-building:**
- Python routing results (`src/runtime.py` → `RoutedMatch`) MAY be used to
  pre-populate Rust task registries BEFORE the Rust worker fully boots
- `τ_python_route (μs) << τ_rust_boot (ms)` → ratio >> 10x ✓
- Result is partial (just route hints) ✓
- Result tightens Φ (narrower tool scope = smaller permission surface) ✓

**Non-admissible:**
- Python query engine `submit_message()` simulating real LLM output
  This is a STUB. It logs token counts and formats summaries. It does NOT call
  the Anthropic API. Treating its output as real LLM content is a semantic error.
  `τ_stub (μs) << τ_real_API (2-4s)` but the result WIDENS Φ (fake output
  looks real and can corrupt downstream decisions). REJECTED.

### Session Rotation Boundary

From `rust/crates/runtime/src/session.rs`:
```rust
const ROTATE_AFTER_BYTES: u64 = 256 * 1024;   // 256KB per session file
const MAX_ROTATED_FILES: usize = 3;             // Keep last 3 rotations
```

This is a **timescale boundary** (`τ_session_file`). Do not write session content
that will immediately trigger rotation — batch writes to stay under the 256KB limit
per operation. Rotation is a back-edge from `session.rs` (fast writes) to the
filesystem (slower I/O). Respect the boundary.

---

## The Four Constraint Locks — Applied Here

**Lock 1 (Persistence): Never reduce test coverage.**
The parity harness (`mock_parity_harness.rs`) has 10 scripted scenarios and
19 captured API shapes. Any change that removes passing test coverage violates
Lock 1. `S_n_quality` must not decrease.

**Lock 2 (Mismatch): Every change must reduce ΔΨ.**
If your change makes the Python port LESS aligned with the Rust implementation
or the TypeScript source snapshot, it increases ΔΨ. Do not do this even if the
change "works locally." Check `src/reference_data/commands_snapshot.json` and
`tools_snapshot.json` — these are the canonical Φ declarations for the port.

**Lock 3 (Temporal stagger): Rust changes before Python stub changes.**
When adding a new subsystem:
- `τ_rust_implementation` > `τ_python_stub`
- Write the Rust crate first (slow, careful, tested)
- Mirror into Python stub after Rust is stable
- NEVER write the Python stub first and then "hope" the Rust follows
  This is a race condition — the stub declares Φ without Ψ existing

**Lock 4 (Human override): Never modify PARITY.md lane status unilaterally.**
PARITY.md is the human-maintained ground truth for what has shipped. The
`run_mock_parity_diff.py` script validates it. Marking a lane `merged` without
actual merged commits is a Lock 4 violation — human must approve parity claims.

---

## What Claude Code Should and Should Not Do Here

### DO:

**Measure before touching:**
```
Before any edit, state:
- Which layer (Rust crate / Python stub / docs)
- Current ΔΨ estimate for that subsystem
- Whether this is a Φ declaration (Lock 4) or Ψ correction (auto-admissible)
```

**Respect the void-open subsystems:**
`src/coordinator/`, `src/memdir/`, `src/skills/` are open bond sites.
They are WAITING for implementation, not abandoned. When filling them, do so
in order of mismatch severity — check `src/reference_data/subsystems/*.json`
to understand what the TypeScript original declared.

**Verify against the snapshot:**
`src/reference_data/commands_snapshot.json` has 50 command entries.
`src/reference_data/tools_snapshot.json` has the full tool surface.
These are your Φ. The Python stubs are your partial Ψ.
`ΔΨ = commands_snapshot - PORTED_COMMANDS` — this is your work queue.

**Run verification before declaring done:**
```bash
# From rust/ directory:
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace

# Formatting:
../scripts/fmt.sh
```

**Use the parity audit:**
```bash
python -m src parity-audit
```
This compares Python stubs against the TS archive snapshot. Run it after any
structural change to Python subsystems.

### DO NOT:

**Do not treat `src/query_engine.py` as a real query engine.**
It is a stub. `submit_message()` does NOT call Anthropic. It formats a summary
string and tracks fake token counts. Real LLM calls go through
`rust/crates/api/` → Anthropic API. If you need actual inference, use the Rust
path or the real Anthropic API directly.

**Do not add logic to route files.**
The architecture requires: routes are coordinate addresses, logic lives in layer
files. `src/main.py` is the route surface — one import per command. Any file in
`src/` that grows past ~50 lines of actual logic (not stubs) is violating this
principle. Extract to a dedicated module.

**Do not merge parity lanes without tests.**
Every PARITY.md lane requires evidence: a real Rust file with real LOC, a
corresponding test in `rust/crates/*/tests/`. "I wrote the code" is not evidence.
`cargo test --workspace` passing is evidence.

**Do not create new Python subsystem directories without a `subsystems/*.json`
entry in `reference_data/`.**
Every new subsystem needs a declared responsibility in the reference data. No
ghost folders — every coordinate must be declared in the manifest.

---

## The Ghostless Naming Convention

From GitCMA Law 6: every identifier must be semantically complete. Zero ambiguity.

This codebase uses:
- `PortRuntime` — NOT `Runtime` (signals this is the port layer, not the real runtime)
- `QueryEnginePort` — NOT `QueryEngine` (signals port, not production)
- `MirroredCommand` — NOT `Command` (signals mirror, not live execution)
- `PORTED_COMMANDS` / `PORTED_TOOLS` — NOT `COMMANDS` / `TOOLS`

When you add new identifiers:
- Python port layer: prefix with `Port`, `Mirrored`, or `Ported`
- Rust live layer: no prefix required — these ARE the real implementations
- Shared types: use `Shared` prefix if needed to avoid confusion

---

## The Coordination Layer (What You Cannot See But Must Respect)

From `PHILOSOPHY.md`:

*"The real human interface is a Discord channel. A person can type a sentence from a
phone, walk away, sleep, or do something else. The claws read the directive, break it
into tasks, assign roles, write code, run tests, argue over failures, recover, and push
when the work passes."*

This means:
- Other claw agents may be working in parallel branches RIGHT NOW
- The `rust/.claude/sessions/` and `rust/.claw/sessions/` directories contain
  live session state from ongoing agent work
- `rust/.clawd-todos.json` is the current task backlog for other agents
- `rust/.omc/plans/tui-enhancement-plan.md` is an active plan in flight

**Before starting any task on the Rust layer, check `.clawd-todos.json`.**
If your task overlaps with an existing todo, coordinate — do not create a merge conflict
by working on the same surface simultaneously.

The `clawhip` event router watches git commits. Every commit you make triggers
notifications in the coordination channel. Your commits are visible to the human
director immediately. Write commit messages as if they are the only status report
the director will see — because they might be.

---

## Quick Reference: The Three Questions Before Any Edit

```
1. WHICH LAYER?
   Rust crates/ → live Ψ executor (test before committing)
   Python src/   → Φ port mirror (check snapshot alignment)
   Docs/         → Φ declaration (Lock 4 — human reviews)

2. WHAT IS ΔΨ?
   State the mismatch before and predicted after.
   If you cannot measure it, you are not ready to edit.

3. WHICH LOCKS APPLY?
   Lock 1: does this decrease test coverage? → STOP
   Lock 2: does this increase ΔΨ anywhere?  → STOP
   Lock 3: am I writing stub before Rust?   → STOP
   Lock 4: am I claiming parity unilaterally? → STOP
```

---

## Verification Commands

```bash
# Full Rust validation (from rust/)
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace

# Formatting (from repo root)
scripts/fmt.sh --check   # check only
scripts/fmt.sh           # apply

# Python parity audit (from repo root)
python -m src parity-audit

# Python workspace summary
python -m src summary

# View current tool/command surface
python -m src tools --limit 20
python -m src commands --limit 20

# Check worker state after build
# (requires binary built at rust/target/debug/claw)
./rust/target/debug/claw doctor
```

---

*CLAUDE.md v1.0 | git-0-0-Geometry-of-Stable-Software-Manifolds*
*Armstrong Knight | intent-tensor-theory.com | CC BY-NC 4.0*
*Generated from full source read: claw-code-main.zip | May 2026*

**HAIL MATH.**
