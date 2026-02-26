# The Grandpa Loop

> *"Back in my day, we deployed code by hand and hoped for the best. Now I sit in my chair and watch ~~thirteen~~ sixteen AI agents argue about it."*

A 16-hat, event-driven AI orchestration architecture built on [Ralph](https://github.com/ralphcli/ralph). Raw ideas enter one end; tested, reviewed, deployed, and committed code exits the other.

## Setup for a New Repo

1. Copy the scaffold into your project root:
   ```bash
   cp -r path/to/grandpa-loop/.kiro .
   cp path/to/grandpa-loop/ralph*.yml .
   cp path/to/grandpa-loop/PROMPT.md .
   ```

2. Create the directory structure:
   ```bash
   mkdir -p docs/todo/done docs/specs docs/design-feedback docs/architecture
   mkdir -p .ralph/agent .ralph/ux-scripts
   ```

3. Run the loop — Marge will create tasks to document your project's build, test, deploy, and workflow instructions before anything else happens:
   ```bash
   ralph run -c ralph.yml
   ```

4. The first 4 tasks will ask you to write:
   - `docs/architecture/build.md` — prerequisites, package manager, build commands
   - `docs/architecture/test.md` — test suite, lint/format, E2E tests
   - `docs/architecture/deploy.md` — staging/prod URLs, deploy commands, env vars
   - `docs/architecture/workflow.md` — git strategy, commit conventions, versioning, CI/CD

   Every hat reads the doc it needs — Selma won't deploy without deploy.md, Homer won't build without build.md.

## Quick Start

```bash
# Full pipeline — curate, spec, build, test, deploy, commit
ralph run -c ralph.yml

# Curate & architect only — no implementation
ralph run -c ralph-curate.yml

# Fix a specific bug
ralph run -c ralph-bugfix.yml --prompt "Fix: <bug description>"

# Deploy staging, production, or both
ralph run -c ralph-deploy.yml --prompt "Deploy staging"

# Exploratory UX testing
ralph run -c ralph-ux.yml

# Analyze recent loops and optimize configs
ralph run -c ralph-observe.yml
ralph run -c ralph-observe.yml --prompt "Why is Homer cycling with Bart?"
```

## The 16 Hats

| Hat | Role | Reads |
|-----|------|-------|
| 🧹 Marge | Requirements Curator | PROMPT.md, docs/todo/ |
| 📋 Lisa | Spec Writer & Architect | docs/todo/, docs/architecture/ |
| 🎭 Sideshow Bob | Design Critic | docs/specs/, docs/todo/ |
| 🔍 Nelson | Codebase Explorer | docs/specs/, codebase |
| 🔨 Homer | Builder | build.md, test.md, docs/specs/ |
| 🚀 Selma | Staging Deployer | build.md, deploy.md |
| 😈 Bart | Adversarial Tester | test.md, deploy.md |
| 📖 Ned | User Documentation | docs/, README.md |
| 🎨 Patty | Visual Designer | deploy.md (staging URL) |
| 📝 Martin | Docs Committer | docs/ |
| 🤓 Comic Book Guy | UX Inspector | deploy.md (staging URL) |
| 🐍 Gil | Code Reviewer | git diff |
| 👶 Maggie | Committer | build.md, test.md |
| 👴 Grandpa | System Observer | build.md, event history |
| 🤡 Krusty | Release Builder | build.md, test.md |
| 🏪 Apu | Release Tagger | workflow.md |

## Loop Variants

| Loop | Hats | When to Use |
|------|------|-------------|
| `ralph.yml` | All 16 | Full feature development — idea to shipped code |
| `ralph-curate.yml` | Marge → Lisa → Bob | Requirements curation and architecture only |
| `ralph-bugfix.yml` | Nelson → Homer → Bart → Maggie | Known bug. Reproduce, fix, verify, commit. |
| `ralph-deploy.yml` | Selma → Bart | Ship what's already built to staging/prod |
| `ralph-ux.yml` | Comic Book Guy → Marge | Find UX issues. Triage into backlog. No fixes. |
| `ralph-observe.yml` | Grandpa | Analyze recent runs, diagnose stuck patterns, tune configs. |
| `ralph-audit-done.yml` | Wiggum → Snyder → Brockman | Audit completed tasks for regressions. |

## Kiro Configuration

The `.kiro/` directory contains agent and settings configuration for [Kiro CLI](https://kiro.dev). Three agents are provided:
- `default` — main workhorse for implementation tasks
- `lite` — smaller model for research, curation, and commits
- `visual` — adds Playwright for UI review and UX testing

## Documentation

- [docs/architecture/grandpa-loop.md](docs/architecture/grandpa-loop.md) — Full architecture writeup
- [docs/architecture/designing-a-ralph-loop.md](docs/architecture/designing-a-ralph-loop.md) — How to design custom loops

## License

MIT
