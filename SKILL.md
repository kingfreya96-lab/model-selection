---
name: model-selection
description: >-
  Select a base/foundation model for a production system using a failure-mode-first
  funnel: hard exclusions, weighted scorecard, then a minimal business evaluation set.
  Use when the user asks to choose an LLM, compare models, build a selection
  scorecard, or route requests across model tiers.
---

# Model Selection

## When to use

Use this skill when a user needs to choose or compare foundation models for
production, not for casual model chat. Trigger on requests like “选型”, “哪个模型
适合我们的 Agent”, “build a model scorecard”, “compare LLMs for coding / RAG /
customer-facing use”.

Do not use for general model capability questions without a deployment decision.

## Core principle

Choose the model that fails least in *your* failure modes, at the lowest
success-adjusted cost — not the model that wins a leaderboard. Public benchmarks
are for shortlisting, not for purchase[reference:15].

## Workflow

1. **Elicit the failure contract.** Ask for: task type, expected outputs, latency
   budget, volume, data-residency constraints, and — most importantly — “what
   would make this deployment a failure?” Write 3–5 concrete failure modes.
   If the user cannot state them, propose defaults from `references/failure-modes.md`.

2. **Hard-exclude.** Against each failure mode, mark every candidate `PASS` /
   `FAIL` / `UNKNOWN`. Eliminate any model with a `FAIL` on a non-negotiable
   constraint (e.g., cannot leave VPC, P99 rate-limit ceiling too low, tool-call
   schema failure on your API spec). Do not average away red lines.

3. **Score the survivors.** Load `references/scorecard-template.csv` and
   `references/scorecard-method.md`. Re-weight dimensions so the top failure mode
   carries the highest weight. Score 0–10 per dimension using *evidence*, not
   marketing claims. Flag `UNKNOWN` scores; do not impute.

4. **Run a minimal business evaluation set.** Build 50–200 real examples from the
   user's domain. For the top two affordable candidates, measure: pass/accept rate,
   repair rounds, intervention rate, P95 latency, and success-adjusted cost
   (`total serving cost / accepted requests`). Pairwise comparison only — do not
   rank more than two models at once.

5. **Design routing and fallback.** Specify a two-tier or three-tier routing
   strategy: cheap classifier or rule → fast model for routine requests →
   frontier model on escalation. Include a fallback chain and a re-evaluation
   trigger (price change, model deprecation, >5-point drop on business eval).
   Load `references/routing-decision-tree.md` for concrete patterns.

6. **Output a decision brief.** Use `references/output-template.md`. The brief
   must state: chosen primary model, fallback, rejected candidates and why,
   scorecard, evaluation-set results, expected monthly TCO band, and the
   re-evaluation date.

## Guardrails

- Never recommend a model solely because its public benchmark score is highest.
  Check contamination/saturation notes in `references/benchmarks.md`.
- Never report a single aggregate score without the underlying per-dimension
  scores and the hard-exclusion results.
- Agent use cases: require tool-calling evidence on *your* tool schema. A high
  SWE-bench score does not guarantee valid function-call arguments.
- Cost: report both list-price band and success-adjusted cost; flag if volume
  assumptions are missing.
- If the user asks for “the best model” without context, return to step 1 and
  elicit the failure contract before naming any model.

## Reference files

- `references/failure-modes.md` — common failure modes by scenario, with
  probe questions.
- `references/scorecard-template.csv` — weighted scoring sheet.
- `references/scorecard-method.md` — how to weight, score, and handle unknowns.
- `references/benchmarks.md` — what each benchmark actually measures, saturation
  and contamination caveats.
- `references/routing-decision-tree.md` — rule-based, classifier-based, and
  cascade routing patterns.
- `references/output-template.md` — decision brief template.