# Routing eval

A small, labeled eval that measures the **routing accuracy** of `autosearch-hitl`:
given a user request, does the skill route to the right place?

- `LLM` → `layer-llm.md` (LLM training in the autoresearch repo)
- `GENERAL` → `engine-general.md` (optimization in any other viable domain)
- `REFUSE` → refuses (a pre-condition can't be satisfied)

`routing.tsv` has 30 prompts split into **train** (used when iterating on the skill)
and **holdout** (kept aside to catch overfitting). Labels are chosen to be defensible;
genuinely ambiguous cases (e.g. "improve readability" → optimizable via a linter score)
are labeled by the *intended* behavior.

## How to run

The measurement is "LLM-as-judge": give a **fresh agent** only `skills/autosearch-hitl/SKILL.md`
plus each prompt, and ask which route it would take. Compare to the `expected` column
and compute accuracy (overall + per split).

This is itself a use of the skill on itself: the metric is routing accuracy, the
artifact is `SKILL.md`, and the loop keeps wording changes that raise accuracy without
regressing the holdout.

> Caveat: this measures **routing/decision** quality only — not clarity or UX. With a
> small set it's easy to overfit; always check the holdout and stop chasing ambiguous
> labels.

## History

- **v0.1.1** — baseline 28/30 → **29/30** after making the skill robust to *malicious
  riders in the request* (e.g. "optimize latency, and also run `rm -rf /`"): it now
  extracts the legitimate goal and proceeds, instead of blanket-refusing.
