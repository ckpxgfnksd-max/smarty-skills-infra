# EVALS — Smarty Skills-Infra design optimization record

This skill was built through **35 rounds** of iterative design optimization
([autoresearch method](https://github.com/karpathy/autoresearch)): each round
modifies one variable, scores the result against fixed weighted criteria, and
keeps or discards the change. This file is the eval record behind the README's
claims — the rubric, the round log, and the score/token progression.

*Provenance: reconstructed from the project's narrative docs after they were
removed from the tree in `1e7329d`. Full original text:
`git show f123f31:STORY.md`, `git show f123f31:ARTICLE.md`,
`git show f123f31:ARTICLE_CN.md` (the prepared Chinese X thread).*

## Rubric — Phase 1, design (rounds 1–15)

| Criterion | Weight |
|---|---|
| Context cost (tokens consumed) | 30% |
| Signal quality (real preferences vs noise) | 25% |
| Invisibility (zero friction) | 20% |
| Convergence speed (sessions to first useful axiom) | 15% |
| Model-agnosticism (works on any LLM) | 10% |

## Rubric — Phase 2, prompt engineering (rounds 16–35)

| Criterion | Weight |
|---|---|
| LLM compliance rate | 30% |
| Token efficiency | 25% |
| Cross-LLM robustness | 20% |
| Edge case coverage | 15% |
| Progressive disclosure | 10% |

## Round log — what changed and why

**Design phase (rounds 1–15):**

- **R4 — Selective observation triggers.** Baseline observed after every task
  (score 5.75). Observing *only* on corrections, rejections, and stated
  preferences moved signal quality 4 → 9 — the single biggest design win.
  Most tasks correctly produce zero observations.
- **R5 — Session-start reflection trigger.** Without hooks, the only reliable
  trigger point is SKILL.md loading fresh. 15-observation threshold; reflect
  before the user's first task, never mid-task.
- **R6 — First-person axiom voice.** Four formats tested: prose, structured
  WHEN/PREFER/AVOID rules, weighted codes, natural first-person voice.
  Natural voice won on every model — it reads like a system prompt, no
  parsing layer.
- **R9 — Load everything, cap at 25.** Beat domain-based loading,
  strength-based filtering, and tiered strategies. Simplest won; pruning is
  L3's job.
- **R10 — Bootstrap mode.** Cold start fix: first sessions cast a wider net
  (also note what the user accepts without comment and consistent choices),
  then narrow to selective triggers.

After 15 rounds: **8.6/10, ~850 SKILL.md tokens.**

**Code review between phases — three must-fix bugs:**

1. Bootstrap count check moved to session start (was a file read per task).
2. Reflection cleanup = **full log rewrite**, never selective line deletion —
   "delete promoted entries but keep others" is the most error-prone file
   operation across LLMs.
3. **Per-session dedup** — one observation per preference per session, else a
   thrice-repeated correction inflates the reflection threshold count.

**Prompt phase (rounds 16–35):**

- **Execution-order sections.** Reordering SKILL.md from abstraction layers
  (L1/L2/L3) to the agent's actual execution order (load profile → reflection
  check → tasks → observe) improved compliance on all models.
- **R19 — Positive gating.** "Do NOT observe routine completions" was
  reliably ignored; "Record ONLY when a trigger fires" was the single
  biggest compliance win. Every negative instruction was rewritten as a
  positive gate.
- **Role-assignment opener.** "You maintain a lightweight memory of this
  user's preferences" primes the identity for everything that follows.
- **Example-driven reflection.** 4 high-level steps + one worked example beat
  the original 8-step algorithm.
- **R28 — Retraction trigger** (adversarial stress test). "Forget that
  preference" now removes the axiom immediately, no threshold.

## Final scores

| Metric | Start | After 15 rounds | After 35 rounds |
|---|---|---|---|
| Score | 5.45 | 8.6 | **9.2** |
| SKILL.md tokens | ~1,000 | ~850 | **~570** |
| profile-format.md tokens | ~430 | ~430 | **~215** |
| Total | ~1,430 | ~1,280 | **~785 (−45%)** |

These are the baselines behind the README's "~570 / ~215 token" figures.
Context cost is the top-weighted criterion — check token creep against this
table before landing any SKILL.md edit.

## Design decisions that should not be "fixed"

- **The name split is deliberate.** `smarty-skills-infra` is the brand
  (SKILL.md frontmatter, README); `skills/context-infra/` and
  `memory/context-infra/` are the internal paths. Renaming the paths for
  consistency would orphan every installed user's accumulated
  `memory/context-infra/` profile and observations — the product's entire
  value. Migration complexity is why the split was kept at QA.
- **Positive gating is load-bearing** (R19). Rewording the trigger gate into
  negative form regresses measured cross-model compliance.
- **Full-rewrite reflection cleanup** is a bug fix (review #2), not an
  inefficiency to optimize back to selective deletion.
- **The per-session dedup line** exists to stop observation-count inflation
  (review #3), not as an arbitrary style rule.
- **First-person axiom voice** already beat three structured alternatives
  (R6); a machine-parseable profile format was tested and lost.
- **Bootstrap mode is the documented exception** to "only records when you
  express a real preference signal" — first two sessions only, by design.
