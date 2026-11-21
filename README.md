# hc-supervisor

A drop-in skill that lets one AI agent session supervise the other terminals of a
[Herdr](https://github.com/marprojector/marherdr) tab — read them, prompt them, wait on them,
interrupt them — over the `hc` CLI.

Generic by design: it knows nothing about any particular project. Give it a tab name and it
finds the terminals itself.

## Why it exists

`hc` talks to `herdr-core`, the daemon behind the Herdr desktop app. Agents that probe the
older `herdr` CLI get `server stopped` and wrongly conclude that cross-terminal control is
impossible. It is not — `hc ls`, `hc read`, `hc send`, `hc wait` all work. This skill encodes
that, plus the rules that turn "typing blind into another pane" into actual supervision
(wait before read, treat `timeout` as still-running, ask the human before answering a
permission dialog).

## Install

Claude Code:

```sh
mkdir -p ~/.claude/skills/hc-supervisor
cp SKILL.md ~/.claude/skills/hc-supervisor/SKILL.md
```

omp:

```sh
mkdir -p ~/.omp/agent/skills/hc-supervisor
cp SKILL.md ~/.omp/agent/skills/hc-supervisor/SKILL.md
```

Any other harness: drop `SKILL.md` wherever that harness loads skills from.

## Use

Ask in plain language, naming the tab:

> supervise the web tab

or invoke it directly where the harness supports slash commands:

> /hc-supervisor

The skill will run `herdr-app ls` to find the tab's terminals, then work from the addresses
it actually finds. It never invents terminal addresses.

## Requirements

- Herdr desktop app running (the terminals live in `herdr-core`)
- `hc` on `PATH` (`hc --version`)
- `herdr-app` on `PATH` for discovering what is open

## Scope

Reads and writes only through `hc`. It never kills or removes a terminal unless asked, and
never restarts the core (that would affect every terminal on every connected machine).
