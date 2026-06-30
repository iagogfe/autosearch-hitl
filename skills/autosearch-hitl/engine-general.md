# General optimization engine

Domain-agnostic. Run the phases in order. Only start the loop (Phase 5) after Phase 4
produces a valid baseline.

## Phase 1 — Metric

Confirm with the user: the single objective number and the direction (lower/higher is
better). If there is no objective number, go back to `SKILL.md` and refuse
(pre-condition 2).

## Phase 2 — Measurement

Detect an existing measurement: test suite, benchmark, profiler, eval script.
- Exists → use it. Note the exact command.
- Doesn't exist → **help create one** (e.g. load test, coverage script, eval harness)
  and get **explicit user confirmation** that it measures the right thing before
  proceeding.

## Phase 3 — Isolate + make reversible (choose the mechanism by domain)

**Code in a git repo:**
1. `git checkout -b autoopt/<tag>` from the current state (NEVER on main/master).
2. Create a worktree: `git worktree add ../<repo>-autoopt-<tag> autoopt/<tag>`.
3. **Bootstrap** in the worktree:
   - Install deps from the lockfile: `package.json`→npm/pnpm/yarn; `uv.lock`/`pyproject.toml`→`uv sync`; `requirements.txt`→`pip install -r`.
   - Copy local non-versioned config (`.env*`) **only if the measurement needs it**. **Never** `git add`/commit those files (they are secrets).
   - Pick a free port by probing if the service starts (avoid colliding with a running instance).
4. **Protect git from bootstrap artifacts (CRITICAL).** Before any commit, add to the
   worktree `.gitignore`, on their own lines: `node_modules` (no trailing slash — a
   trailing slash does NOT ignore a symlink named `node_modules`), `results.tsv`,
   `autosearch-hitl-results.tsv`, `.env` and `.env.*`. If you shared deps via a symlink,
   confirm with `git status` that the symlink and `results.tsv` show up as ignored.
   Skipping this leaks the symlink into commits and corrupts the real `node_modules`
   on merge.
5. **Verify:** run the measurement command once in the worktree. A valid number = baseline.
6. Bootstrap/verification failed → **fallback**: a dedicated branch in the directory
   itself (with a warning that there's no file isolation), or abort and ask for help.

**Non-git domain** (a prompt file, a set of configs, data):
1. Snapshot the artifact (copy to `./.autosearch-hitl-snapshot/`).
2. Restore from the snapshot on any attempt that gets worse.

## Phase 4 — Baseline

Reuse the number already obtained in Phase 3 (the initial verification) and record it
as the baseline.
- Code: `results.tsv` at the worktree root, NOT versioned.
- Non-git: `autosearch-hitl-results.tsv` in the artifact's directory.

Format (TAB-separated):

```
id	metric	status	description
0	<baseline>	keep	baseline
```

## Phase 5 — Autonomous loop (autoresearch-style)

ALWAYS work in the dedicated worktree/branch (or on the snapshot). Keep a counter
`no_improvement = 0`.

REPEAT:
1. Pick ONE concrete improvement idea for the artifact.
2. Apply the change.
3. Record the attempt. Code: commit **only the artifact files**
   (`git add <specific paths>` && `git commit`) — **NEVER `git add -A`**, which would
   drag the bootstrap symlink and `results.tsv` into history. Non-git: the snapshot already exists.
4. Run the measurement command → get the number.
   - If the measurement fails/crashes: judge — trivial fix (typo, import) → fix and
     re-run; fundamentally broken idea → record `crash`, revert, move on.
5. Compare:
   - **Improved** (in the defined direction) → KEEP (advance the branch);
     `no_improvement = 0`; status `keep`.
   - **Equal or worse** → REVERT (`git reset --hard` to the previous commit, or restore
     the snapshot); `no_improvement += 1`; status `discard`.
6. Log the row in the TSV (`id`, `metric`, `status`, `description`).
7. **Stop:** if `no_improvement >= 5`, end the loop and go to Phase 6.

Never ask "should I continue?" mid-loop — it is autonomous until it stalls.

## Phase 6 — Report

When finished, present:
- baseline → best metric reached (with the gain).
- what worked and what didn't (summary of the `keep`/`discard` rows).
- the path to `results.tsv`.
- the branch/worktree (or snapshot) with the winning artifact.
- offer: merge the branch, or remove the worktree/branch/snapshot.
