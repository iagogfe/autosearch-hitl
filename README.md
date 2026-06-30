<div align="center">

**English** · [Português](README.pt-BR.md)

# autosearch-hitl

**Autonomous optimization with a human in the loop:**
you define what "better" means, and the skill iterates on its own — *change → measure → keep/discard* — safely, and knows when to stop.

![license](https://img.shields.io/github/license/iagogfe/autosearch-hitl)
![release](https://img.shields.io/github/v/release/iagogfe/autosearch-hitl)
![CI](https://img.shields.io/github/actions/workflow/status/iagogfe/autosearch-hitl/ci.yml?branch=master&label=CI)
![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen)
[![skills.sh](https://img.shields.io/badge/skills.sh-install-111111)](https://www.skills.sh/iagogfe/autosearch-hitl)

</div>

`autosearch-hitl` is an **Agent Skill** that generalizes the idea behind
[`autoresearch`](https://github.com/karpathy/autoresearch) by **Andrej Karpathy**
to everyday engineering. Instead of tweaking code by hand and measuring manually,
the skill drives the experimentation loop for you — on **anything** with an
objective metric (latency, cost, CI time, test coverage, search quality, etc.),
not just model training.

## Why it exists

Optimization is repetitive: tweak something → run → check if it improved → repeat,
dozens of times. That's exactly the kind of work an agent does well — **as long as
the "did it get better?" decision is a number, not a hunch.** `autosearch-hitl`
automates that loop while keeping **you in charge of the decisions that matter**.

## Human-in-the-loop, on purpose

Nobody wants a 100%-automatic agent touching production code blindly. The skill is
built around the **human in the center**:

- It **asks what you want** and guides you — no commands to memorize.
- It has a **pre-condition gatekeeper**: before starting, it checks whether the
  problem is optimizable and, if not, **refuses and explains why** (instead of
  optimizing the wrong thing).
- When there's no way to measure, it **helps you build one** and asks you to confirm
  the measurement actually measures the right thing.
- It works on an **isolated copy** and hands you the result to review and decide the merge.

## How it works

1. **Pre-conditions (the gatekeeper).** The skill only runs if there is:
   1. *Controllable artifact* — something concrete to change.
   2. *Objective metric* — a number that says better/worse, with a direction.
   3. *Repeatable measurement* — a command that produces that number (or can be created).
   4. *Reversible change* — you can undo a bad attempt.
   Something missing and unsolvable? It refuses and explains.

2. **Safe isolation.** For code, it works in a dedicated **git worktree**
   (`autoopt/<tag>`) — never touching your branch. For other domains, it uses snapshots.

3. **Measured loop.** Apply one idea → measure → **keep if better, revert if worse**
   → log it. Each improvement becomes a commit; each regression is discarded.

4. **Stops at the right time.** It ends on its own when it **stalls** (several
   attempts with no gain) and hands you a report + the winning result.

## Installation

Easiest — via the [skills](https://www.skills.sh) CLI:

```bash
npx skills add iagogfe/autosearch-hitl
```

Or manually (skills are discovered under `~/.claude/skills/`):

```bash
git clone https://github.com/iagogfe/autosearch-hitl
cp -r autosearch-hitl/skills/autosearch-hitl ~/.claude/skills/autosearch-hitl
# (or symlink it to keep it versioned)
```

Restart your agent. In Claude Code it shows up as `/autosearch-hitl`. It also works
with other skill-compatible agents (Codex, etc.) — it's just markdown, no runtime
dependency.

## Usage

Open your agent in a project and say what you want to improve, in plain language:

```
use autosearch-hitl to reduce the latency of the POST /order endpoint
use autosearch-hitl to increase test coverage of the payments module
use autosearch-hitl to improve search quality against this eval set
```

The skill handles the rest: triage → measurement → isolation → loop → report.

### Don't know how to phrase it? Just ask plainly

You don't have to be technical. Casual ("even dumb") prompts work too:

```
how can I improve this?
this feels slow — can you make it faster?
make my tests cover more
can this be cheaper to run?
make these search results better
```

If it isn't clear how to measure "better", the skill **asks you and helps set it up**
before changing anything — so you can start even without knowing the metric.

## Results (real cases)

Applied to production code of a **payments API**, with measured gains and **zero
regressions**:

- **Latency of a critical endpoint** (`create_order`): ~46% faster, cutting
  redundant database queries (from ~22 to ~7 per request).
- **Test suite time**: **−57%** (it found a test that hung ~10s on a real DNS
  lookup and isolated it, without touching production code).
- **Search quality** (documentation service): **+162% MRR** — the right answer now
  ranks first on most queries (via BM25 + domain synonyms; 4 of 8 ideas were
  discarded by the metric).

> The skill is only as good as the metric you define. With little data it's easy to
> "overfit" the number — define a measurement that reflects real usage.

## Credits

The core idea — the autonomous *change → measure → keep/discard* loop — is by
**[Andrej Karpathy](https://github.com/karpathy)**, in the
**[`autoresearch`](https://github.com/karpathy/autoresearch)** project. This skill
merely generalizes that concept to any measurable domain, with a human-in-the-loop
focus. All credit for the original idea is his. 🙏

## License

[MIT](./LICENSE).
