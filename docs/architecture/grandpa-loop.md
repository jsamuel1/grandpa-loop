# The Grandpa Loop

> *"Back in my day, we deployed code by hand and hoped for the best. Now I sit in my chair and watch thirteen AI agents argue about it."*
> — 👴 Grandpa Simpson, System Observer

The Grandpa Loop is a 13-hat, event-driven AI orchestration architecture built on [Ralph](https://github.com/ralphcli/ralph). Raw ideas enter one end; tested, reviewed, deployed, and committed code exits the other. Every hat is an AI agent with a single job, a defined contract, and absolutely no trust in the hat before it.

Named for the hat that watches over everything: Grandpa, the System Observer. He tunes the loop based on evidence. He's seen it all. He remembers everything — which is more than you can say for the original Grandpa, but that's what a scratchpad and an observer report are for. He complains constantly. And he's the reason the loop gets better over time.

---

## Why "Loop"?

Calling it a "loop" is generous. A loop implies a clean circle — A to B to C and back to A. The Grandpa Loop is more like a [Lissajous curve](https://en.wikipedia.org/wiki/Lissajous_curve): a path that keeps crossing over itself, re-entering at unexpected points, tracing a shape that looks chaotic until you see the underlying pattern.

When Maggie commits, Grandpa observes, and the flow circles back to Marge for the next task — that's the main orbit. But when Bart breaks Homer's build, it cuts back across the middle. When Bob rejects Lisa's spec, it doubles back on itself. When Comic Book Guy finds a UX disaster, it arcs all the way from the end to the beginning. When Patty rejects the visuals, it crosses the build-fix loop at a different angle. The docs track runs parallel, untouched by any of it.

The result isn't a circle. It's an odd, twisted figure that traces the same space from different directions depending on what went wrong and where. The only exit is `queue.empty` — and even then, Grandpa gets the last word.

Most CI/CD pipelines are linear: build → test → deploy → done. The Grandpa Loop is self-correcting and self-intersecting. Bad specs get rejected before code is written. Bad code gets caught before it ships. Bad UX gets filed as new work. The curve keeps tracing, and the product gets better each pass.

---

## The Cast

| Hat | Who | The Deal |
|-----|-----|----------|
| 🧹 **Marge** | Requirements Curator | Nothing moves without a clean task file. She turns chaos into structured tasks with testable acceptance criteria. Also triages UX issues that Comic Book Guy discovers. |
| 📋 **Lisa** | Spec Writer | The overachiever. Writes specs so precise that Homer can't misinterpret them. (He tries.) Includes interface changes, test plans, and diagrams. No ambiguous language allowed. |
| 🎭 **Sideshow Bob** | Design Critic | The adversary. Reviews Lisa's specs for completeness, feasibility, and YAGNI violations. Enjoys rejecting them. Does NOT rewrite — just points out the holes and sends Lisa back. |
| 🔍 **Nelson** | Explorer | The scout. Researches the codebase before Homer touches anything. Finds existing patterns, integration points, and broken windows. Research only — no implementation. |
| 🔨 **Homer** | Builder | Surprisingly effective when given clear specs. Follows Explore → Plan → TDD. Must pass ALL backpressure gates before declaring done. The workhorse of the loop. |
| 🚀 **Selma** | Staging Deployer | Methodical, no-nonsense. Builds the app, deploys infrastructure, syncs assets, invalidates the CDN. If anything fails, she tells you exactly why and stops. |
| 😈 **Bart** | Adversarial Tester | His job is to break things. He's good at it. Runs E2E tests, tries invalid inputs, boundary values, rapid interactions, offline mode. Also checks whether Homer snuck in extras. |
| 📖 **Ned** | Documentation | Hi-diddly-ho! Updates docs to reflect what changed. Runs in parallel with the test pipeline — docs don't block deploys, deploys don't block docs. |
| 🎨 **Patty** | Visual Designer | Nothing ships with bad UI. Checks every page at mobile, tablet, and desktop breakpoints. Verifies design system compliance and accessibility. |
| 📝 **Martin** | Docs Committer | The prefect. Commits docs the instant Ned finishes. Only touches documentation files — never source code. |
| 🤓 **Comic Book Guy** | UX Inspector | *Worst. UX. Ever.* Runs real user journey scripts against staging. Records everything. Writes scathing issue reports. If a user can't complete a journey, he blocks the commit. |
| 👶 **Maggie** | Committer | Silent but effective. Final validation, commit, push, archive the task. One commit per task. |
| 👴 **Grandpa** | System Observer | Watches for stuck loops, wasted iterations, repeated failures. Tunes the loop configuration using the scientific method — observe, hypothesize, wait, change ONE thing, verify. |

---

## The Interaction Graph

<!-- LISSAJOUS_PLACEHOLDER -->

The Lissajous shape isn't decorative — it's what the flow actually looks like. The main orbit traces from Marge through to Grandpa, but the feedback arcs from Bart, Patty, and Comic Book Guy all cut back across to Homer, crossing the curve at different angles. The CBG-to-Marge triage arc sweeps from one end to the other. Grandpa's loop-back to Marge closes the figure.

Notice how Homer is the most connected node. Everything flows through the builder — specs arrive, failures return, visual rejections bounce back. Homer is the pressure point. When the loop is slow, it's usually Homer. When the loop is stuck, it's definitely Homer. Grandpa watches this closely.

---

## Why Not Just PDD?

Prompt-Driven Development gives you a planner and a builder. That's a good start. But it's like having an architect and a bricklayer and calling it a construction company. Where's the inspector? The tester? The person who checks whether the door actually opens?

The Grandpa Loop adds three things that standard PDD patterns don't have: an observer who tunes the system itself, a manual tester reborn as AI, and a set of interlocking feedback loops that make the pipeline self-correcting rather than self-congratulating.

### Grandpa: The Observer Who Fixes the Machine

<!-- MEME_PLACEHOLDER -->

*Grandpa's approach to system tuning.*

Most orchestration systems have a fixed configuration. You set it up, you run it, and when it breaks you go in and fix it yourself. Grandpa changes that.

Grandpa sits in his armchair and watches. He watches Homer and Bart cycle back and forth three times on the same bug. He watches Lisa's specs get rejected twice in a row. He watches iterations climb without commits. He watches, and he waits, and he takes notes in his observer report.

And then — only then — he reaches for the mallet.

Grandpa's tuning follows the scientific method, which is a fancy way of saying he doesn't panic. He observes a pattern across at least two iterations. He writes down what he thinks is wrong. He waits to see if it happens again. Then he makes one minimal change to the loop configuration — maybe increasing Homer's max activations, maybe tightening a guardrail, maybe adjusting the memory budget — and he watches to see if it helps.

This is the opposite of how most people manage AI pipelines. The usual approach is to notice something's wrong, immediately change five things, and then wonder why it's still broken (or broken in a new way). Grandpa changes one thing at a time and documents every change with rationale. He's not fast. He's not flashy. But the loop gets measurably better over time because someone is actually watching it and making evidence-based adjustments.

The key insight is that the orchestration system itself is a thing that needs maintenance. Not just the code it produces — the pipeline that produces the code. Grandpa is the hat that maintains the machine, not the product. He's meta. He's the loop watching the loop.

And yes, sometimes the fix is just hitting it with a mallet. But it's a *carefully considered* mallet strike, after *extensive* observation from the armchair.

### Comic Book Guy: The Return of the Manual Tester

The software industry spent twenty years trying to eliminate manual testing. We automated everything. Unit tests, integration tests, E2E tests, contract tests, property tests, mutation tests, visual regression tests. We built elaborate CI pipelines that run thousands of checks on every commit.

And yet. Users still find bugs that no automated test catches. They click things in the wrong order. They paste emoji into number fields. They resize the browser to 400px wide and wonder why the layout exploded. They try to use the product the way a human actually uses a product, which turns out to be nothing like the way a developer writes a test.

Manual testing was never the problem. Manual testing was expensive, slow, inconsistent, and didn't scale. But the *thing it tested* — real user journeys, end-to-end, with all the messy human behavior — was always valuable. We threw out the baby with the bathwater.

Comic Book Guy brings it back.

He maintains a library of user journey scripts — not automated test scripts, but narrative descriptions of what a real user would try to do. "Log in, connect an AWS account, create a coding agent, ask it to build something." He loads up the staging environment in a real browser, walks through each script step by step, and records what happens. Did the button work? Did the page load? Did the error message make sense? Could a new user figure out what to do next?

When something doesn't work, he doesn't just file a bug. He writes a scathing review — reproduction steps, expected vs actual behavior, screenshots, and a priority assessment. *Worst. UX. Ever.* These get written as task files that Marge picks up in the next iteration. Discovery feeds the backlog.

The scripts persist across iterations and grow over time. Every time Comic Book Guy explores a new user journey, he adds a script for it. The test coverage of real user behavior expands with every loop. And unlike a human manual tester, Comic Book Guy doesn't get bored, doesn't skip steps, and doesn't forget to test the thing that broke last time.

He's also the only hat that can block a commit based on vibes. If the automated tests all pass but the UX feels wrong — confusing flow, missing affordance, dead-end state — Comic Book Guy can emit `ux.blocked` and send it back to Homer. No other hat has that power. Bart can only fail on evidence. Patty can only fail on visual criteria. Comic Book Guy can fail on "a real user would be confused here," and that's by design.

This is the hat that catches the bugs your test suite never will, because your test suite tests what you thought of, and Comic Book Guy tests what a user would actually do.

---

## The Five Feedback Loops

The Grandpa Loop isn't a straight pipeline — it's a system of interlocking feedback loops. This is what makes it self-correcting, and what separates it from a dumb CI/CD chain.

**Spec Revision** — Bob rejects Lisa's spec. Lisa rewrites. Bob re-reviews. This can cycle multiple times, and that's fine — a rejected spec costs minutes. A bad spec that reaches Homer costs hours.

**Build Fix** — Bart breaks Homer's build. Homer fixes. Selma redeploys. Bart retests. The most common loop. Homer and Bart can cycle many times before Grandpa intervenes.

**Visual Fix** — Patty rejects the UI. Homer fixes. The entire test pipeline reruns. Patty is ruthless about pixels, and she should be — users see the UI before they see the code.

**UX Fix** — Comic Book Guy blocks the commit because a real user journey is broken. Homer fixes. Full pipeline reruns. This is the most expensive loop, which is why it's last — catch everything cheaper first.

**UX Triage** — Comic Book Guy finds issues but they're not blockers. He writes them up as new tasks. Marge picks them up in the next iteration. Discovery feeds the backlog. The loop feeds itself.

---

## The Parallel Track

After Homer finishes building, two things happen simultaneously.

The **test track** flows through Selma, Bart, Patty, Comic Book Guy, and finally Maggie — deploying, testing, reviewing, and committing the code.

The **docs track** flows through Ned and Martin — updating documentation and committing it independently.

Docs don't wait for tests. Tests don't wait for docs. They converge at the end with two independent commits from one build event. This is intentional. Ned shouldn't be blocked because Bart found a bug. Martin shouldn't wait for Patty to approve pixels. Parallelism where the dependencies allow it.

---

## Backpressure: The Law of the Loop

> *If you can't prove it, it didn't happen.*

This is the single most important design principle. No hat can declare success without evidence. Not "I think it works." Not "it should be fine." Proof.

Homer must pass every quality gate — tests, compilation, linting, formatting, secrets scanning — before he's allowed to say he's done. Bart must provide structured evidence covering E2E tests, adversarial scenarios, accessibility, and YAGNI compliance. Patty must confirm responsiveness, accessibility, and design system alignment.

The evidence format is validated by the CLI. Vague prose gets rejected. This isn't bureaucracy — it's the mechanism that prevents the loop from lying to itself.

---

## Guardrails

Nine rules every hat follows. Break one and Grandpa notices.

1. **Fresh context each iteration** — read the task queue, don't rely on stale memory
2. **Backpressure is law** — evidence required before emitting done events
3. **One task at a time** — pick the oldest unclaimed task, finish it
4. **Stay in scope** — never modify files outside the current task without justification
5. **All platforms matter** — consider browser, desktop, and mobile impact
6. **Quality gates are mandatory** — pre-commit hooks run before declaring done
7. **YAGNI ruthlessly** — no speculative features, no future-proofing, no "while I'm here"
8. **KISS always** — simplest solution that works; complexity must be justified
9. **Confidence protocol** — score decisions 0–100; above 80 proceed, below 50 choose the safe default, always document uncertainty

---

## The Derivative Loops

The Grandpa Loop is the full 13-hat pipeline. Sometimes you don't need all thirteen arguing at once. For focused work, we extract subsets that inherit the same guardrails, backpressure rules, and Simpsons personality — just with fewer hats and a tighter budget.

**The Bugfix Loop** sends Nelson to reproduce the bug with a failing test, Homer to write the minimal fix, Bart to try to break it, and Maggie to commit. Four hats. Scientific method: observe, hypothesize, test, fix. If Bart's verification fails, it cycles back to Nelson with context about why the fix didn't work.

**The Deploy Loop** sends Selma to build and deploy, then Bart to verify the deployment is healthy. Takes a target — staging, production, or both. When deploying both, staging must succeed before production starts. Bart checks health endpoints, validates config, and runs visual smoke tests against each environment.

**The UX Discovery Loop** sends Comic Book Guy on a full exploratory sweep of staging, then Marge triages everything he finds into the task backlog. Pure discovery — no fixes, no code changes. Run this when you want to know what's broken before deciding what to fix.

| Loop | Hats | When to Use |
|------|------|-------------|
| **Grandpa Loop** | All 13 | Full feature development — idea to shipped code |
| **Bugfix Loop** | Nelson → Homer → Bart → Maggie | Known bug. Reproduce, fix, verify, commit. |
| **Deploy Loop** | Selma → Bart | Ship what's already built to staging, prod, or both. |
| **UX Discovery** | Comic Book Guy → Marge | Find issues. Triage into backlog. No fixes. |

To run any derivative, point Ralph at its config file. The full Grandpa Loop reads the prompt file for work and runs until the queue is empty. The derivatives can take an inline prompt describing the specific bug to fix or environment to deploy — or they'll read the prompt file like the main loop does.

> *"I used to be with it. Then they changed what 'it' was. Now what I'm with isn't 'it', and what's 'it' seems weird and scary to me. But I still tune the loop."*
> — 👴 Grandpa
