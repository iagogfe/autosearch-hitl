# LLM layer (fast-path)

Use when the goal is to minimize `val_bpb` in the `autoresearch` training setup.

## Pre-check (abort if it fails)

- The repo contains `program.md` at the root and it's readable; if it doesn't exist or
  isn't readable, **ABORT** with an honest explanation — don't improvise or reimplement
  the loop. Suggest the user check they're in the right repo (autoresearch).
- The repo contains `train.py` and `prepare.py` (it's the autoresearch repo).
- An NVIDIA GPU is available (`nvidia-smi` responds).
- Data exists in `~/.cache/autoresearch/`; if not, ask to run `uv run prepare.py`.

If any item is missing, explain and offer to help set it up — don't start the loop.

## Delegation

This path does NOT reimplement the loop. Read `program.md` at the repo root and follow
it literally:

- **Setup:** `autoresearch/<tag>` branch, verify data, initialize `results.tsv`.
- **Loop:** edit `train.py` → commit → `uv run train.py > run.log 2>&1` →
  `grep "^val_bpb:" run.log` → keep (advance the branch) or `git reset` → log to the TSV.

## Differences vs the general engine

- **Worktree:** stays **off** on this path (single GPU; shared data cache) — don't enable it.
- **Stop:** follows the "never stop" contract of `program.md` (until manual
  interruption), instead of the general engine's stop-on-stagnation.
