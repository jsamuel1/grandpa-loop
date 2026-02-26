# The Grandpa Loop

> *"Back in my day, we deployed code by hand and hoped for the best. Now I sit in my chair and watch thirteen AI agents argue about it."*

A 16-hat, event-driven AI orchestration architecture built on [Ralph](https://github.com/ralphcli/ralph). Raw ideas enter one end; tested, reviewed, deployed, and committed code exits the other.

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

## Loop Variants

| Loop | Hats | When to Use |
|------|------|-------------|
| `ralph.yml` | All 16 | Full feature development — idea to shipped code |
| `ralph-curate.yml` | Marge → Lisa → Bob | Requirements curation and architecture only |
| `ralph-bugfix.yml` | Nelson → Homer → Bart → Maggie | Known bug. Reproduce, fix, verify, commit. |
| `ralph-deploy.yml` | Selma → Bart | Ship what's already built to staging/prod |
| `ralph-ux.yml` | Comic Book Guy → Marge | Find UX issues. Triage into backlog. No fixes. |
| `ralph-observe.yml` | Grandpa | Analyze recent runs, diagnose stuck patterns, tune configs. |

## Kiro Configuration

The `.kiro/` directory contains agent and settings configuration for [Kiro CLI](https://kiro.dev). Customize the agents for your project — the `default` agent is the main workhorse, `lite` uses a smaller model for lightweight tasks, and `visual` adds Playwright for UI review.

## Documentation

- [docs/architecture/grandpa-loop.md](docs/architecture/grandpa-loop.md) — Full architecture writeup
- [docs/architecture/designing-a-ralph-loop.md](docs/architecture/designing-a-ralph-loop.md) — How to design custom loops

## License

MIT
