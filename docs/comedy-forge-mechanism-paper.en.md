# Comedy Forge: Implementation Paper for a Protocol-Driven Multi-Round Comedy Generation System (Aligned to Latest Skill)

**Version**: v1.1 (synced with current `skill.mdc`)  
**Status**: Implementation-level specification  
**Repository**: `Geeprox/ComedyForge`

---

## Abstract

This paper provides an implementation-level description of Comedy Forge, a protocol-driven system for stand-up and manzai generation. The system operationalizes comedy writing as a constrained iterative optimization loop: persona-conditioned draft generation, audience simulation and blind review, evolutionary selection, comment-driven rewriting, and set-level assembly optimization. Relative to one-shot generation, the latest system introduces and formalizes: (1) active-subset diversity sampling with core/long-tail dynamic weighting, (2) `angle-preset`-driven 10-angle quota planning with plan/report validation, (3) L0-L5 expectation-violation tiering, (4) seed-driven mood-group shifts with expanded mood bias mapping, (5) wildcard A/B plus conditional C variants under pool-cap governance, (6) vA/vB plus conditional vC polishing, and (7) structured rewrite-signal feedback loops. We provide formal constraints, data contracts, equations, thresholds, exception handling, complexity estimates, and validation protocols for reproducible implementation.

**Keywords**: comedy generation, protocolized agents, audience simulation, evolutionary optimization, comment-driven rewriting, set-list optimization

---

## 1. Introduction

### 1.1 Problem

Comedy generation is a stage-structure problem, not merely a fluency problem. A practical system must satisfy:

- intra-joke structure (setup -> punchline -> rhythm -> callback),
- inter-joke set coherence (open -> build -> climax -> close),
- interpretable feedback-to-rewrite pathways,
- strict style/persona boundaries under diversity constraints.

### 1.2 Objective

Given theme and runtime parameters, produce a stage-ready, auditable, and iteratively improvable set with full round-level diagnostics.

---

## 2. System Architecture

### 2.1 Layers

- Entry and contracts: `.comedy-forge/skill.mdc`
- Core protocols: `system/*.md`
- Workflow execution: `workflows/*.md`
- Output templates: `templates/*.md`
- Runtime compatibility: `compatibility.targets = [codex, claude-code]`

### 2.2 Pipeline

```mermaid
flowchart TD
  A[Parameter validation] --> B[Active-subset persona sampling and drafts]
  B --> C[Diversity, quota, and dedup checks]
  C --> D[Audience template sampling and blind review]
  D --> E[Score aggregation and Polarizer detection]
  E --> F[Comment-to-rewrite signalization]
  F --> G[Elite retention + wildcard expansion + A/B/C variants]
  G --> H[Distillation and gain logging]
  H --> I[Composite set-list search]
  I --> J[vA/vB(/vC) polishing]
  J --> K[Standardized output]
```

### 2.3 Isolation

- Creator side cannot see raw individual votes/comments.
- Judge side cannot see hidden creator persona internals.
- Only structured rewrite signals are exposed cross-boundary.

### 2.4 Dual-Side Execution Templates (Mandatory)

Creation must run in an isolated creator context:

```xml
<persona_creator>
You are a comedy creator with the required professional baseline.
Tonight's mask: [8-dimensional persona variables]
Task: write jokes about {{theme}}.
Constraint: reason strictly through {{lens}} and stay in character.
</persona_creator>
```

Review must run in an isolated blind-judge context:

```xml
<persona_judge>
You are a blind-review audience member with randomized demographics.
Rate each joke from 1-5 stars and provide one short comment.
[Strictly do not reference creator persona settings or version history]
</persona_judge>
```

Constraint: reset `persona_judge` before each audience vote; never pass `persona_creator` fields into judge prompts.

---

## 3. Parameter Space and Global Objective

### 3.1 Input

\[
\Theta = \{type, theme, min\_rounds, max\_rounds, survival\_rate, batch\_size, audience\_size, absurd\_booster, angle\_preset\}
\]

Constraints:

- \(type \in \{standup, manzai, auto\}\)
- \(3 \le min\_rounds \le 20\)
- \(5 \le max\_rounds \le 30\)
- \(0.1 \le survival\_rate \le 0.5\)
- \(10 \le batch\_size \le 500\)
- \(10 \le audience\_size \le 200\)
- \(3 \le absurd\_booster \le 50\)
- \(angle\_preset \in \{balanced, conservative, exploratory\}\)
- \(min\_rounds \le max\_rounds\)

### 3.2 Output

\[
output/comedy\text{-}forge\text{-}\{theme\}\text{-}\{YYYYMMDD\text{-}HHMMSS\}.md
\]

### 3.3 Objective

\[
\max \mathcal{L}=\alpha Q_{single}+\beta Q_{set}+\gamma Q_{diversity}+\delta Q_{rewrite}-\lambda R_{overlap}
\]

---

## 4. Runtime Contracts (from `skill.mdc`)

### 4.1 scoring_contract

- `manzai_duo_chemistry_first = true`
- `seed_mood_shift_enabled = true`
- negative mood-group ratio: default 40%, negative-seed 60%, positive-seed 25%-30%
- expanded mood bias enum: tired/irritable/optimistic/anxious/bored/relieved/cynical
- dual-threshold polarizer detection

### 4.2 evolution_contract

- `survival_rate_applied = true`
- wildcard expansion mode with max 5 origins/round
- base variant count = 2 (A/B), conditional max = 3 (C)
- candidate pool hard cap = 500

### 4.3 assembly_contract

- second-pass polishing enabled
- base variants = 2 (vA/vB), conditional max = 3 (vC)
- polish mode pool: `compress/callback_boost/clarity_boost/persona_lock/risk_soften`
- composite set-list objective weights:

\[
w = \{0.55, 0.20, 0.15, 0.10, -0.15\}
\]

### 4.4 diversity_contract

- `sampling_mode = active_subset_then_sample`
- `weighting_mode = core_long_tail_dynamic`
- `angle_preset_default = balanced`
- `angle_preset_values = [balanced, conservative, exploratory]`
- 10-angle weighted profile is selected by preset (core/long-tail balancing)
- `core_weight_total_range = [0.65, 0.85]`
- `long_tail_weight_total_range = [0.15, 0.35]`
- health metrics: coverage, entropy, Top1 share, pair repeat, drift, JSD, quota hit rate

---

## 5. Data Contracts

### 5.1 Joke

```yaml
id: "R2-17"
round_created: 2
persona:
  baseline_professional: true
  identity: "Middle Manager"
  generation: "Millennials"
  tone: "Socially Anxious"
  lens: "Legal Compliance"
  language_style: "Academic"
  secret: "Side-hustle losses"
  current_state: "Just called by HR"
  theme_relation: "Observer"
content:
  setup: "..."
  punchline: "..."
  tags: [Observation Mismatch, L3]
style:
  angle: "Observation Mismatch"
  expectation_level: "L3"
  attack_level: 7
scores:
  overall: 4.35
metadata:
  is_polarizer: false
  is_wildcard: false
  rewrite_applied_actions: ["Add premise bridge"]
```

### 5.2 Audience

```yaml
id: "A-17"
dimensions:
  mood: "anxious"
vote: 3
comment: "Structure works but ending is predictable"
```

### 5.3 Rewrite Signal

```yaml
joke_id: R2-15
round: 2
rewrite_signals:
  - tag: ENDING_PREDICTABLE
    priority: P2
    action: Shift key reversal token later
    confidence: 0.81
keep_elements:
  - Keep the central conflict line
```

---

## 6. Persona and Diversity Sampling

### 6.1 Two-Layer Persona

- constant professional baseline
- variable dimensions: identity, generation, tone, lens, language style, secret, current state, theme relation

### 6.2 Boundary Rules

- identity must be long-horizon role labels
- short-term events must stay in `current_state`
- `secret` and `current_state` cannot be semantic duplicates

### 6.3 Active-Subset Sampling

Two-step sampling:

1. activate subset `active_values(dim)`
2. sample jokes within activated subset with dynamic weights

Activation ranges (capped by dimension cardinality):

- batch 10-29: 4-6
- batch 30-79: 6-10
- batch 80-200: 8-14
- batch 201-500: 10-16

### 6.4 Per-Value Sample Guard

\[
expected\_count\_per\_value = \frac{batch\_size}{active\_count(dim)}
\]

- `<30`: [2,6]
- `30-79`: [3,10]
- `80-200`: [4,18]
- `201-500`: key dims >= 8

### 6.5 Coverage Floors

- identity >=4 (>=5 when batch>=50)
- lens >=5 (>=6 when batch>=50)
- current_state >=4 (>=5 when batch>=50)
- language style >=3 (>=4 when batch>=50)
- single-generation share <=50%
- adjacent style violation rate = 0

---

## 7. Angle Quotas, Level Quotas, and Health Metrics

### 7.1 `angle-preset`-Driven 10-Angle Profile

The system supports three presets:

- `balanced` (default, Core/Long-tail = 0.75/0.25)
- `conservative` (safer, Core/Long-tail = 0.85/0.15)
- `exploratory` (riskier, Core/Long-tail = 0.65/0.35)

Selection rules:

- if `angle-preset` is provided and valid, use it directly
- if omitted, default to `balanced`
- if invalid, fall back to `balanced` and set `preset_fallback=true`

`balanced` weights:

- self-deprecating 0.14, observational misdirection 0.16, absurd extrapolation 0.13, attack external 0.11, wordplay 0.10, situational simulation 0.11
- role contrast 0.08, rule loophole 0.07, meta self-mock 0.06, emotional backfire 0.04

`conservative` weights:

- self-deprecating 0.16, observational misdirection 0.18, absurd extrapolation 0.15, attack external 0.13, wordplay 0.11, situational simulation 0.12
- role contrast 0.05, rule loophole 0.04, meta self-mock 0.03, emotional backfire 0.03

`exploratory` weights:

- self-deprecating 0.12, observational misdirection 0.14, absurd extrapolation 0.11, attack external 0.10, wordplay 0.09, situational simulation 0.09
- role contrast 0.12, rule loophole 0.09, meta self-mock 0.08, emotional backfire 0.06

### 7.2 Quota Conversion

\[
raw_i=batch\_size\cdot weight_i,\quad base_i=\lfloor raw_i \rfloor
\]

Use largest-remainder allocation for residual slots.

### 7.3 Preset-Specific Quota Floors

`balanced`:

- 10-29: long-tail>=15%, active>=2, angle_top1_share_max=0.35
- 30-79: long-tail>=20%, active>=3, angle_top1_share_max=0.35
- 80-200: long-tail>=22%, active>=4, angle_top1_share_max=0.35
- 201-500: long-tail>=25%, active>=4, angle_top1_share_max=0.35

`conservative`:

- 10-29: long-tail>=10%, active>=1, angle_top1_share_max=0.40
- 30-79: long-tail>=12%, active>=2, angle_top1_share_max=0.40
- 80-200: long-tail>=14%, active>=3, angle_top1_share_max=0.40
- 201-500: long-tail>=15%, active>=4, angle_top1_share_max=0.40

`exploratory`:

- 10-29: long-tail>=20%, active>=2, angle_top1_share_max=0.33
- 30-79: long-tail>=28%, active>=3, angle_top1_share_max=0.33
- 80-200: long-tail>=32%, active>=4, angle_top1_share_max=0.33
- 201-500: long-tail>=35%, active>=4, angle_top1_share_max=0.33

### 7.4 Expectation Levels (L0-L5)

- \(L0 \le 10\%\)
- \(L1 \le 15\%\)
- \(L2 \ge 30\%\)
- \(L3 \ge 25\%\)
- \(L4 \ge 10\%\)
- \(L5 \ge 5\%\)

Conflict priority: `L2/L3 > L4/L5 > L1/L0`.

### 7.5 Diversity Health Report

Per-round mandatory checks:

- coverage_ratio (key dims >=0.80)
- coverage_ratio (other dims >=0.70)
- normalized_entropy (key dims >=0.72)
- normalized_entropy (other dims >=0.65)
- top1_share (key dims <=0.35)
- top1_share (other dims <=0.40)
- pair_repeat_rate(secret,current_state) <=0.03
- angle_quota_hit_rate >=0.80
- long_tail_share_actual >= long_tail_share_min(angle_preset, batch_size)
- angle_preset in {balanced, conservative, exploratory}
- elite_filter_drift in [0.30,0.70]
- distribution_jsd(key dims) in [0.08,0.35]

---

## 8. Audience Simulation, Template Pool, and Mood Bias

### 8.1 Weighted Sampling Tiers

very-high/high/medium/noise hierarchy is applied across 12 dimensions.

### 8.2 Correlated Resampling

- GenZ ↔ Student +0.8
- Irritable ↔ Layoff +0.7
- Insider-veteran ↔ Critical +0.6

### 8.3 Seed-Driven Mood Group

Negative group: `tired/irritable/anxious/cynical`

- default 40%
- negative seed 60%
- positive seed 25%-30%

Explicit example: when `seed="rainy-night-depression"` (e.g., "雨夜沮丧"), the negative mood-group share rises from 40% to 60%, making round-level scoring stricter on average.

Mood bias:

- tired -0.15
- irritable -0.25
- optimistic +0.10
- anxious -0.10
- bored -0.12
- relieved +0.05
- cynical -0.18

### 8.4 Template Pool for Audience>100

No fixed single matrix. Use rotating subgroup templates (T1/T2/T3), disallow immediate repetition across consecutive rounds.

---

## 9. Scoring and Polarizer Detection

### 9.1 Dimension Weights

Standup: 30/25/25/20.  
Manzai: 35/25/25/15 (duo-first).

Implementation constraint: when `type=manzai`, judges must prioritize duo chemistry, catch-response timing, and two-person rhythm integrity over single-performer punchline density.

### 9.2 Aggregation

1. weighted raw score
2. mood-biased score
3. clamp to [1,5]
4. integer vote
5. trim top2/bottom2
6. compute mean/std

### 9.3 Polarizer

Small sample (n<=100): tail conflict + sigma>1.15.  
Large sample (n>100): plus peer variance / cross-group std / sigma>1.2.

---

## 10. Evolution: Elite, Wildcard, A/B/C Variants

### 10.1 Elite

\[
elite\_count = \lceil batch\_size\cdot survival\_rate \rceil
\]

### 10.2 Wildcard Expansion

Max 5 wildcard origins per round. Base A/B variants; conditional C variants triggered by:

- `REPEAT_WITHIN_SET`
- `ENDING_PREDICTABLE`
- `PERSONA_COLLAPSE`
- or `std_dev > 1.2` with high controversy potential

Pool size:

\[
pool_{next}=batch\_size + 2\cdot wildcard\_origins + variant\_c\_count
\]

If overflow >500: reduce C first, then new blood; never cut elite or A/B.

### 10.3 Variant Roles

- A: prioritize P1 fixes
- B: prioritize P2/P3 explorations
- C: switch narrator/rhythm while preserving core conflict

---

## 11. Comment-Driven Rewriter

### 11.1 Slice Weights

- `vote<=2`: 1.0
- `vote==3`: 0.5
- `vote>=4`: keep-elements only

### 11.2 Tag Extensions

In addition to structural/target/language/rhythm tags, latest skill adds:

- `REPEAT_WITHIN_SET`
- `ENDING_PREDICTABLE`
- `PERSONA_COLLAPSE`

### 11.3 Scoring

\[
tag\_score = \sum(comment\_weight\cdot confidence)
\]

Select Top1-3 tags, execute by `P1>P2>P3`, with max 3 actions per joke.

### 11.4 Scope and Safety

Forced scope: wildcard, borderline-eliminated, low-score (`overall<3.4`).  
Creators receive structured signals only, never raw comments.

---

## 12. Distillation and Gain Logging

- eliminated jokes: strict compressed format `id|concept|score|reason`
- hard limits: `concept <= 10 Chinese chars (or equivalent short phrase)`, `reason <= 15 Chinese chars`
- forbidden extras: no raw lines, hidden persona internals, or historical version details
- elite jokes: full text + version chain + rewrite actions + deltas
- per-round `rewrite_feedback_log` mandatory
- actions with two consecutive negative net deltas are downweighted

---

## 13. Final Assembly and vC Polishing

### 13.1 Individual Pre-score

\[
individual = 0.45\cdot overall + 0.20\cdot callback + 0.20\cdot clarity + 0.15\cdot freshness
\]

### 13.2 Composite Set Score

\[
set\_score = 0.55\cdot avg(individual) + 0.20\cdot flow + 0.15\cdot callback\_chain + 0.10\cdot diversity - 0.15\cdot overlap
\]

### 13.3 Search

- candidates<=14: exhaustive
- candidates>14: prune then beam (suggested beam=8)

### 13.4 Second-Pass Polishing

Base: vA (compress), vB (callback boost).  
Conditional vC triggered for clarity/persona/risk conditions.

Polish mode pool: `compress/callback_boost/clarity_boost/persona_lock/risk_soften`.

---

## 14. Termination and Exception Handling

Early stop requires all:

- `round >= min_rounds`
- `elite_mean >= 4.2`
- `elite_std < 0.6`
- `polarizer_rate < 10%`

Exception branches:

1. trimmed sample < 6 -> resample audience
2. subgroup imbalance > ±10% -> rebalance and recompute
3. action conflicts -> resolve by priority + persona consistency
4. pool overflow -> cut C then new blood

---

## 15. Complexity and Cost

Let rounds \(R\), batch \(B\), audience \(A\):

- scoring core: \(O(R\cdot B\cdot A)\)
- dedup worst-case: \(O(B^2)\) (only when B>100)
- set search: approx. \(O(k\cdot beam)\)

Cost controls:

- threshold-triggered dedup
- candidate cap at 14 for search
- incremental structured logs

---

## 16. Validation Protocol

### 16.1 Mode Regression

- minimal: 10/10/3
- deep: 100/200/5

### 16.2 Constraint Regression

- coverage/entropy/top1/pair-repeat/quota-hit
- L0-L5 quota satisfaction
- manzai duo-first scoring effect

### 16.3 Rewrite Gain

\[
mean(net\_delta) > 0
\]

and verify downweighting of repeatedly negative actions.

### 16.4 Assembly Gain

- set_score trend
- callback_chain improvement
- overlap penalty reduction

---

## 17. Limitations and Future Work

- weight profiles and template pools are still heuristic priors; online calibration is needed
- tag mapping remains sensitive to semantic ambiguity
- protocol is mature, but unified CLI runtime can further productize execution

---

## 18. Conclusion

Comedy Forge turns intuition-heavy comedy drafting into protocolized optimization. The system ensures input-space quality through diversity contracts, closes the feedback loop through structured rewrite signals, and optimizes show-level quality through composite set objectives. This paper is synchronized with the latest skill and can be used directly for implementation, review, and reproducibility across Codex and Claude Code.

---

## Appendix A: Parameters

| Parameter | Range | Description |
|---|---|---|
| type | standup/manzai/auto | style mode |
| min-rounds | 3-20 | minimum rounds |
| max-rounds | 5-30 | maximum rounds |
| survival-rate | 0.1-0.5 | elite survival ratio |
| batch-size | 10-500 | base candidate size |
| audience-size | 10-200 | reviewer size |
| absurd-booster | 3-50 | absurdity boost |
| angle-preset | balanced/conservative/exploratory | angle quota preset |

## Appendix B: Protocol Mapping

| Mechanism | File |
|---|---|
| Entry contracts | `.comedy-forge/skill.mdc` |
| Persona & audience generation | `.comedy-forge/system/persona-generator.md` |
| Scoring & controversy detection | `.comedy-forge/system/scoring-protocol.md` |
| Comment-driven rewrites | `.comedy-forge/system/comment-rewrite-rules.md` |
| Evolution & wildcard variants | `.comedy-forge/system/evolution-rules.md` |
| Diversity & quota planning | `.comedy-forge/system/batch-diversity.md` |
| Batch creation workflow | `.comedy-forge/workflows/batch-creator.md` |
| Audience workflow | `.comedy-forge/workflows/audience-simulator.md` |
| Distillation workflow | `.comedy-forge/workflows/distillation-engine.md` |
| Final assembly workflow | `.comedy-forge/workflows/final-assembly.md` |
| Output schema | `.comedy-forge/templates/output-format.md` |
