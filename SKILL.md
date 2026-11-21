---
name: hc-supervisor
description: Supervise a Herdr tab from any AI agent session (Claude Code, omp, scripts). Lets you read, prompt, wait on, and interrupt the terminals of an open Herdr tab over `hc`, so one session can act as the supervisor of the others. Use when the user mentions Herdr, hc, a tab name, supervising/delegating between terminals, or asks to drive another agent pane.
---

# hc-supervisor

Drive the other terminals of a Herdr tab from this session. `hc` talks to `herdr-core` (the
daemon behind the Herdr desktop app), so it works even though `herdr status` reports its own
legacy socket as down — that socket is not the one `hc` uses.

## 1. Find the targets — never assume addresses

```bash
herdr-app ls            # spaces → tabs → terminals (JSON; ground truth of what is open)
hc ls                   # flat view + every terminal on every connected machine
```

Pick the user's tab by name (e.g. `marfolio`) and record each terminal address (`t9n`, `t9t`).
An address is stable until the terminal is closed. Do not invent addresses; if the tab is not
in `herdr-app ls`, say so.

Status values: `idle` (stopped, waiting for input), `working` (mid-turn), `done`, `blocked`
(needs a decision or a permission prompt), `error`.

## 2. Supervise loop

```bash
hc read <t> --lines 0     # what is on screen right now
hc read <t> --lines 200   # screen + scrollback
hc send <t> "text"        # paste text and press Enter
hc keys <t> enter         # keys: enter, esc, up, down, ctrl+c, tab
hc wait <t>               # block until the agent stops working
hc wait <t> --output "X"  # block until this text appears (for non-agent programs)
hc interrupt <t>          # stop key, then report what happened
```

Rules that make this supervision instead of gambling:

- **Always `wait` before `read`.** `read` while `working` shows a partial turn.
- `reason: timeout` from `wait` is a check-in, not a failure — wait again. Treat it as "still
  going", never as "done".
- If status is `blocked` or a dialog/permission prompt is visible, report it to the user and ask
  before pressing `enter`/`y`. A stuck prompt stalls that pane, and the user may want to answer.
- A prompt sent while the target is `working` is queued and delivered at the end of its turn.
  Say so rather than re-sending.
- Never `hc server stop` or restart the core: it affects every terminal and every machine.
- Never `hc kill` or `hc rm` unless the user asked for that terminal to end.

## 3. Supervising from an agent that cannot spawn subagents

`hc` is the transport; there is no agent-to-agent bus inside omp and no reading of raw
`/dev/ttys*`. If an agent tells the user it cannot reach another pane, the missing piece is
almost always this: it probed `herdr status` (legacy socket, down) instead of `hc`.
Verify with `hc ls` before believing any claim that cross-terminal control is impossible.

## 4. Reporting back

Give the user: which terminal, the command or prompt issued, the status, and the concrete output
(or the one decision that is blocking). Do not describe intentions as results — quote output.
