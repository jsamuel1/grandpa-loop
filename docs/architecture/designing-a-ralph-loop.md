# Designing a Ralph Loop for a Major Rewrite

> How we designed an AI orchestration pipeline to rewrite a TypeScript WebSocket relay into a Rust Zenoh router — and what we learned about matching loop topology to task shape.

---

## The Problem

MyProject's relay is a ~300-line TypeScript server running on Bun. It handles WebSocket connections, message routing, agent process management, and discovery. It works. But every new feature requires hand-rolling protocol logic that Zenoh provides as built-in primitives: pub/sub, storage, presence, peering, historical queries.

The v3 architecture replaces the relay with a Rust binary wrapping zenohd — Zenoh's router daemon. The TypeScript disappears. New Rust crates (`myproject-core`, `myproject-relay`) take its place. This isn't a refactor. It's a rewrite.

We needed a Ralph loop that could handle this — and our existing loops weren't designed for it.

## What We Had: The Builtin Presets

Ralph ships with 15 builtin presets. We evaluated each against the relay rewrite:

### `refactor` — Too Thin

Two hats: Refactorer and Verifier. The Refactorer makes atomic changes, the Verifier confirms tests still pass. This assumes you already know what to change and that you're preserving behavior in the same codebase.

A rewrite doesn't preserve behavior in the same codebase. We're building new Rust code alongside existing TypeScript. The `refactor` preset has no planning phase, no gap analysis, no task ordering. It's a good inner loop but a terrible outer loop.

### `gap-analysis` — Right Idea, Wrong Output

Three hats: Analyzer, Verifier, Reporter. Compares specs against implementation and writes an ISSUES.md. The pattern is exactly right — compare what exists against what should exist. But it produces a report, not a task queue. And it's read-only — no implementation phase.

### `research` — Pure Discovery

Two hats: Researcher and Synthesizer. Gathers information, writes findings. No code changes. Useful for the analysis phase but stops there.

### `spec-driven` — Contract-First, But Greenfield

Four hats: Spec Writer → Critic → Implementer → Verifier. Writes specs with Given-When-Then criteria, then implements. Good for new features. But it starts from a blank spec — it doesn't start from "here's 300 lines of working code that needs to become something else."

### `pdd-to-code-assist` — The Full Pipeline

Nine hats: Inquisitor → Architect → Critic → Explorer → Planner → Task Writer → Builder → Validator → Committer. The most complete preset. It goes from rough idea to committed code through iterative Q&A, design, adversarial review, codebase research, planning, task generation, TDD implementation, validation, and commit.

This is the closest to what we need. But it's designed for greenfield features — the Inquisitor asks questions to refine requirements, the Architect synthesizes designs. For a rewrite, the requirements already exist (the v3 architecture doc) and the "design" is a mapping exercise (v2 code → Zenoh primitives), not a creative exercise.

### The Full Simpsons Loop — Overkill

Our custom 13-hat Simpsons loop (ralph.yml) handles everything: requirements curation, spec writing, adversarial review, codebase exploration, TDD implementation, staging deployment, adversarial testing, visual review, UX testing, documentation, and system observation.

For a Rust rewrite, we don't need Selma (staging deployer), Patty (visual designer), Comic Book Guy (UX inspector), Ned (documentation), Martin (docs committer), or Grandpa (system observer). That's 6 hats doing nothing useful. The overhead of routing events through unused hats wastes iterations.

## The Design: Two Phases, Seven Hats

The relay rewrite has a natural phase boundary: you need to understand the mapping before you can implement it. Phase 1 produces the plan. Phase 2 executes it.

### Phase 1: Analysis & Planning

Three hats borrowed from different presets:

**Analyst** — Combines `gap-analysis`'s deep code reading with `research`'s exploration. Reads all v2 relay source, reads the v3 architecture doc, and produces a gap report that maps v2 features to Zenoh primitives. The output isn't ISSUES.md (problems to fix) — it's a mapping document (what exists → what it becomes).

**Planner** — Borrowed from `pdd-to-code-assist`'s Task Writer, but adapted. Instead of converting a design document into tasks, it converts a gap report into ordered implementation tasks. The ordering is critical: Cargo workspace scaffolding → shared types → Zenoh session → plugins → features. Each task must leave `cargo build && cargo test` green.

**Critic** — Borrowed from `spec-driven`'s Spec Critic and the Simpsons' Sideshow Bob. Reviews the task plan for dependency ordering, atomicity, and Zenoh API correctness. This is the gate between "we have a plan" and "we start writing Rust."

### Phase 2: Implementation

Four hats in a tight loop:

**Explorer** — Borrowed from the Simpsons' Nelson and `pdd-to-code-assist`'s Explorer. Before each task, researches the relevant Zenoh APIs and v2 reference code. Writes a context file that the Builder reads. This is critical because Zenoh's API is unfamiliar territory — the Builder shouldn't be discovering API patterns mid-implementation.

**Builder** — Combines `refactor`'s atomic-step discipline with `code-assist`'s TDD cycle. Implements one task per iteration: RED (write failing Rust test) → GREEN (minimal implementation) → REFACTOR (clippy, fmt) → COMMIT. The key difference from the `refactor` preset: this is writing new code, not modifying existing code. The "test before change" step becomes "write a test that defines the behavior."

**Verifier** — Borrowed from `refactor`'s Verifier but extended. Runs `cargo test`, `cargo build`, `cargo clippy`, reads the diff, checks YAGNI. On pass, emits both `verify.passed` and `task.next` — the second event triggers the Explorer to research the next task, creating a pipeline effect.

**Closer** — Borrowed from the Simpsons' Maggie. Final push, summary document, LOOP_COMPLETE.

### The Event Flow

```
Phase 1:
  analyze.start → Analyst → gaps.found → Planner → tasks.planned → Critic
                                                                      ↓
                                                              tasks.approved
                                                                      ↓
Phase 2:                                                        Explorer
  ┌──────────────────────────────────────────────────────────────────┘
  ↓
  context.ready → Builder → task.done → Verifier → verify.passed + task.next
                    ↑                                      ↓
                    └──── verify.failed ◄──────────────────┘
                                                           ↓
                                                     task.next → Explorer → ...

  (when all tasks done)
  Builder → refactor.complete → Closer → LOOP_COMPLETE
```

The inner loop (Explorer → Builder → Verifier → Explorer) is the workhorse. Each cycle implements one task. The Verifier's dual publish (`verify.passed` + `task.next`) creates a pipeline: while the current task's verification result is being processed, the Explorer is already researching the next task.

## Why Not Just Use `pdd-to-code-assist`?

It's the closest builtin, but three things don't fit:

1. **The Inquisitor/Architect Q&A phase is unnecessary.** The v3 architecture doc already exists. We don't need iterative questioning to refine requirements — we need a mapping exercise from v2 code to v3 architecture. The Analyst hat does this directly.

2. **The Task Writer doesn't understand dependency ordering for Rust crates.** It converts a plan into tasks, but it doesn't know that Cargo workspace setup must come first, that shared types must precede consumers, or that Zenoh session config must precede plugins. The Planner hat has Rust-specific ordering knowledge.

3. **There's no Explorer in the inner loop.** `pdd-to-code-assist` has an Explorer, but it runs once before all tasks. For a Zenoh rewrite, each task may need different API research — the Explorer needs to run per-task, not once.

## Why Not Just Use `refactor`?

It's the right inner loop but has no outer loop:

1. **No analysis phase.** The `refactor` preset assumes you know what to change. For a rewrite, you need to map the entire surface area first.

2. **No task ordering.** The `refactor` preset processes one step at a time, but doesn't plan the sequence. For a Rust crate with dependency ordering, the sequence matters.

3. **No Zenoh-specific verification.** The Verifier in `refactor` checks "tests still pass." For a rewrite, we also need "the Zenoh API usage is correct" and "the key expressions follow the schema."

## Design Decisions

### 7 hats, not 13

The full Simpsons loop has 13 hats. We cut 6:
- **Selma (deployer)** — no staging deployment for Rust crates
- **Bart (adversarial tester)** — `cargo test` + Verifier is sufficient; no E2E against staging
- **Patty (visual designer)** — no UI changes
- **Comic Book Guy (UX inspector)** — no user-facing changes
- **Ned (documentation)** — docs can come later
- **Grandpa (observer)** — 100 iterations max, not a long-running loop that needs tuning

Each removed hat saves event routing overhead and context budget. Fewer hats = faster iterations = more tasks completed per hour.

### Builder commits directly

In the full Simpsons loop, Homer builds and Maggie commits separately. This separation exists because the full loop has deployment and testing between build and commit — you don't want to commit before staging is verified.

For a Rust rewrite, there's no staging. The verification is `cargo test && cargo clippy`. The Builder can commit immediately after verification, saving an entire hat activation per task.

### Explorer runs per-task, not once

`pdd-to-code-assist` runs the Explorer once after design approval. For a Zenoh rewrite, each task may involve different Zenoh APIs — liveliness for presence, storage plugins for persistence, pub/sub for messaging. The Explorer runs before each task to research the specific APIs needed.

This costs one extra hat activation per task but prevents the Builder from discovering API surprises mid-implementation.

### Phase boundary is an event, not a config

Both phases live in one ralph.yml. The phase boundary is the `tasks.approved` event — Phase 1 hats (Analyst, Planner, Critic) only trigger on Phase 1 events, Phase 2 hats (Explorer, Builder, Verifier, Closer) only trigger on Phase 2 events. Ralph's event routing handles the transition naturally.

An alternative would be two separate configs (`ralph-analyze-relay.yml` and `ralph-build-relay.yml`) run sequentially. We chose one config because:
- The Planner may need to revise tasks if the Critic rejects — this feedback loop crosses the phase boundary
- One config = one `ralph run` command = less manual coordination
- The specs directory (`specs/relay-v3/`) is the shared state between phases

### Scoped to Phase 1 of v3

The v3 migration has 4 phases (Core+Relay → CLI+TUI → Web+Desktop → Cloud). This loop handles only Phase 1. The PROMPT.md explicitly lists what NOT to build. This prevents scope creep — the Builder won't start implementing CLI commands or web app changes.

Each subsequent phase gets its own ralph config, its own PROMPT.md, and its own specs directory. The phases share the Cargo workspace and the `myproject-core` crate.

## What We Learned

**Match loop topology to task shape.** A rewrite has a different shape than a feature, a bugfix, or a refactor. The analysis phase is heavier (mapping an entire codebase), the planning phase needs domain-specific ordering (Cargo dependencies), and the implementation phase needs per-task research (unfamiliar APIs). No single builtin preset fits all three.

**Cut hats aggressively.** Every hat costs context budget and iteration time. If a hat's trigger events never fire, it's dead weight in the config. Start with the minimum viable loop and add hats only when you observe a gap.

**The phase boundary matters.** Separating "understand the work" from "do the work" prevents the Builder from making architectural decisions mid-implementation. The Analyst and Planner make those decisions once, the Critic validates them, and the Builder executes without improvising.

**Builtins are building blocks, not templates.** We didn't use any single preset. We borrowed the Analyst pattern from `gap-analysis`, the Planner from `pdd-to-code-assist`, the Critic from `spec-driven`, the Explorer from the Simpsons loop, the Builder from `refactor` + `code-assist`, and the Verifier from `refactor`. The presets are a vocabulary of patterns, not a menu of complete solutions.
