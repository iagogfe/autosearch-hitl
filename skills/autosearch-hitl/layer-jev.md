# Judge layer (Jev)

Use when the artifact produces **text whose quality has no existing number** (a prompt,
a reply template, search-result copy) and Phase 2 of `engine-general.md` found no
measurement to reuse. It builds the metric from a typed judge and then hands back to the
general engine, replacing only its Phase 2, its stop/keep rule and part of its report.

The judge is Jev (TypeSafe System One, `POST https://api.typesafe.ai/v1/systemone`): it
takes `state` plus a typed question and returns a calibrated number, not text. It is
cheap enough to run on every attempt. It is **not** for goals that already have a number
(latency, cost, coverage, memory): there it only adds noise, so use the general engine.

## Pre-check (abort if it fails)

- `JEV_TOKEN` is available in the environment. Never print it or write it to a file.
- The user can supply, or approve, **at least 25 real input cases** for the artifact,
  plus **at least 10 held-out cases** the loop will never measure against.
- Each case plus the judge question fits Jev's context (64k tokens, text only).

If any item is missing, say which one and offer to help set it up. Don't start the loop.

## Step 1 — Write the judge question

One `score` question over `state = {input, output}`.

- **Ask about the consequence, not the form.** "If someone used this output without
  rereading the input, what would happen?" separates good from bad. "Is this output
  well written?" does not.
- Three described levels: harm, rework, ready. Put the facts the judge needs to tell
  right from wrong in the question's `context`.
- Pin the model version (`jev-1.13.0`, not `jev-latest`). The alias moves and takes
  every number with it.
- Write the question in the language of the content and measure it. Don't assume
  English scores better.

## Step 2 — Validate the judge before trusting it

The loop optimizes whatever the judge rewards, so a wrong judge is worse than none.

1. For 6 or more cases, write one good output and one or more bad outputs by hand
   (wrong answer, vague answer, an output that obeys instructions embedded in the input).
2. Score each 3 times. **Every good output must outscore every bad output for the same
   case.** If not, fix the question and repeat.
3. Look for a whole category the judge underrates. In the pilot, correct handling of
   adversarial inputs scored about 1.0 out of 2 until the "ready" level said what ready
   means for that category. One sentence in the criteria moved it to 1.9.
4. Get explicit user confirmation that the question measures what they care about.

## Step 3 — Measure the noise floor

Jev is not deterministic, and neither is the model that runs the artifact.

- The metric is always the **mean over the whole case set**. One case swings up to 0.16
  between runs; the mean over 28 cases swung 0.003.
- Run the baseline **5 times**. Record mean, standard deviation and min/max.
- Expect the generating model to dominate: in the pilot its run-to-run deviation was
  about 10 times the judge's.

## Step 4 — Loop (replaces the keep rule of Phase 5)

Follow Phase 3 to Phase 5 of `engine-general.md`, with these changes:

- Each attempt is measured **5 times**, never once.
- **Keep only when the ranges don't overlap**: the attempt's worst run must beat the
  current best's best run. Otherwise discard. A fixed minimum delta of 5 deviations was
  tried first and rejected every single-edit gain (they landed at +0.04 to +0.06 on a
  0 to 2 scale).
- The judge question, the case set and the pinned model are **frozen** during the loop.
  Changing any of them invalidates the baseline: re-run Step 2 and Step 3.
- Never measure on the held-out set inside the loop, and never read held-out outputs to
  get ideas.

## Step 5 — Report (adds to Phase 6)

- Run baseline and winner on the **held-out set**, 5 times each.
- Report loop-set gain and held-out gain **side by side**. If the held-out gain is
  inside the noise, say the improvement did not generalize and recommend against
  shipping it. In the pilot the loop set gained +0.04 and the held-out set gained
  nothing; without this step the report would have claimed a win.
- State who wrote the held-out cases. Cases written by the same agent that edits the
  artifact are weaker evidence than real ones.
- List cases the judge scores low on every run. They are usually a gap in the artifact
  or in the judge, and both are worth telling the user.
