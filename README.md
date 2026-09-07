# How Much Did Your Judgment Change AI?

> **The Human Cognitive Leverage Ratio (HCLR): A Method for AI Users to Assess Their Influence on Model Outputs**

**中文版 / Chinese version: [README_CN.md](README_CN.md)**

---

## Abstract

AI can generate a lot of content, but generating a lot is not the same as doing good work. Given the same AI output, different users handle it very differently: some accept it as-is, some polish it sentence by sentence, and some make just a few critical judgments that change the direction of the result.

HCLR (Human Cognitive Leverage Ratio) is a self-assessment method for AI users. It records the process from the initial AI draft to the final deliverable and observes how much the user's **sparse cognitive interventions** (few but critical human judgments) influenced the model output. Through **double confirmation**, it tracks whether the result was actually adopted and whether it gained audience recognition.

---

## 1. Why HCLR

Most existing tools only record whether users are satisfied (thumbs up/down). They cannot answer:

- Did the user actually change the AI output?
- At what level did the change occur — wording, content, conclusions, the whole plan, or the problem framing?
- How many key judgments did the user need to make the result acceptable?
- Over time, is the user getting better at producing larger effective changes with fewer interventions?

HCLR provides a lightweight, sustainable way to record and observe these questions.

## 2. Core Definitions

**HCLR is a self-assessment method for the sparse intervention capability of AI users.** Within a given AIGC model, task domain, and observation period, the user records the changes their limited explicit interventions made to the generated output, and performs two rounds of confirmation.

Conditional representation:

```
HCLR(u | m, d, T)
```

- `u`: user
- `m`: model and version (a relatively stable tool environment)
- `d`: task domain
- `T`: observation period

**The model is treated as a given tool environment.** HCLR does not require fully separating human and model contributions, but changing models, task domains, or audiences should be marked as a series breakpoint.

### Process Chain

```
O0 → h → O1 → C1 → C2
```

| Symbol | Meaning |
|---|---|
| `O0` | Original AI output (initial output without intervention) |
| `h` | Human intervention (active, autonomous critical judgment) |
| `O1` | Output after intervention (final deliverable) |
| `C1` | First confirmation: whether the user adopts the result |
| `C2` | Second confirmation: whether the audience recognizes the result, based on real feedback |

## 3. How to Record

### 3.1 Basic Unit: Task Event

The basic unit is a **task event**: one AI collaboration task from original generation through human interventions to a confirmable final output. Individual interventions are tracked within the event but are not scored separately, avoiding double-counting the same change.

### 3.2 Record Fields

- `task_id`, `domain`, `model` (model and version), `audience` (audience type)
- `O0` (original output, frozen and immutable), `O1` (final output)
- `interventions[]` (list: text, type, timestamp)
- `I` (intervention amount)
- `C1` (0/0.5/1), `C2` (0–1/percentage/pending), `C2_source` (user/landing/auto_timeout/auto_confirm), `C2_note` (basis for confirmation)
- Timestamps (created / confirmed / second confirmation)

Full field definitions: [`schema/hclr-record.schema.json`](schema/hclr-record.schema.json)

### 3.3 Human Intervention

An intervention is a **critical judgment actively and autonomously initiated by the user**, for example: fact correction, goal or scope adjustment, constraint addition, conclusion revision, analytical framework change, style and audience adaptation.

Two principles:

1. **Interventions must be initiated by the human.** AI must not decide (e.g., via active learning) when or what the human should review, or restrict review boundaries.
2. **Sparsity is an observed property after the fact**, not a pre-allocated feedback budget.

### 3.4 Intervention Amount (I)

Three metrics are supported; declare which one is used:

| Metric | Definition | Note |
|---|---|---|
| Tokens / chars | Token or character count of intervention text | **Default in automated recording** (objective, zero annotation burden) |
| Judgments | Number of independent semantic judgments / key opinions | Default in manual recording |
| Turns | Number of intervention turns | Simplest |

> **Important boundary**: explicit intervention amount is not equal to total cognitive investment (reviewing, searching, fact-checking, reasoning costs are not counted). HCLR measures "explicit intervention leverage," not a complete cognitive ability score.

### 3.5 Double Confirmation

**First confirmation C1** (generation stage): whether the user adopts the result — `1` adopted / `0.5` partially adopted (with minor edits) / `0` not adopted.

**Second confirmation C2** (after the result reaches the audience): whether the audience recognizes it, based on real feedback — `1` recognized / `0.5` partially recognized (e.g. ~90% with minor edits) / `0` not recognized; any acceptance percentage may be recorded (e.g. 25%); `∅` pending.

**C2 automated closed-loop collection** (confirmation never stalls on missing feedback):

| Source | Trigger | Value |
|---|---|---|
| `landing` | Result landed locally (saved/executed and not revoked by the user) | treated as 100% recognized |
| `user` | User explicitly returns an acceptance level | as returned |
| `auto_timeout` | No response within 24 h of a confirmation request | defaulted to recognized |
| `auto_confirm` | Still no response after an active follow-up confirmation | default accepted |

- C2 is the user's interpretation of **real feedback**, not a prediction made at generation time, and does not require researchers to survey the audience directly;
- **Landing equals confirmation**: a result actually saved/executed and not revoked is the strongest implicit signal of recognition and needs no separate prompt;
- **Pending is not failure**: when feedback is insufficient, the closed-loop rules above supply a default so the pipeline never stalls; delayed feedback can be backfilled and override the default;
- User-explicit values and automatic defaults are counted separately (explicit recognition rate vs. closed-loop recognition rate); reports should disclose the share of each source.

**Result states:**

| State | C1 | C2 | Meaning |
|---|---|---|---|
| S0 | 0 | — | Not adopted |
| S1 | 1 | ∅ | Adopted, pending confirmation |
| S2 | 1 | < 0.5 | Adopted, not recognized |
| S3 | 1 | ≥ 0.5 | Adopted and recognized (incl. partial) |

## 4. Scoring & Reporting

### 4.1 Result-State Statistics

| Metric | Definition | Note |
|---|---|---|
| Direct adoption rate | Adopted without intervention / all tasks | Baseline: initial output quality |
| Post-intervention adoption rate | C1=1 / tasks with intervention | Whether intervention yields adoptable results |
| Second-round completion rate | With C2 / C1=1 tasks | Feedback backfill completeness |
| Confirmed recognition rate | C2=1 / tasks with C2 | Recognition rate within confirmed samples |
| Average intervention amount | ΣI / number of tasks | Intervention cost per task |

> Distinguish **second-round completion rate** from **confirmed recognition rate** — they have different denominators; mixing them is a common mistake.

### 4.2 HCLR Core Indicator

```text
HCLR = total model output tokens (or chars) / total user intervention tokens (or chars)
Personal aggregation: HCLR(u|m,d,T) = ΣO / ΣI
```

- **Numerator O**: total model output tokens within the task event (including O0, O1, and intermediate rounds);
- **Denominator I**: total user intervention tokens (**excluding the initial task description**; only feedback/corrections/constraints/direction adjustments to the output count);
- Double confirmation (C1/C2) is retained as a **result-state record** (S0–S3) and does not enter the formula;

> This indicator measures the leverage relationship of "output scale / intervention scale"; it does not measure output quality, cognitive investment, or social recognition. Read it together with the result-state statistics.

### 4.3 Periods & Missing Data

- Default observation period: calendar month (configurable);
- Mark breakpoints when model / domain / audience changes;
- With fewer than 10 tasks, show raw records only, no trend conclusions;
- Pending is not 0; provide optimistic (all pending as recognized) and conservative (all pending as not recognized) bounds;
- When the pending ratio exceeds 30%, second-round metrics should be marked "unreliable."

### 4.4 Leaderboard Conditions (Community Scenarios Only)

Cross-person comparison is **only allowed under strict conditions**: unified model and version; unified task set and initial outputs; unified intervention metric; result states confirmed by third parties or the community; auditable full interaction records; no selective recording (failed tasks must be included); review of anomalous results.

## 5. Important Boundaries

- HCLR is a **self-assessment reference**, not an objective truth standard;
- Not a general intelligence or overall cognitive level scale;
- Not the only way to evaluate AI usage ability;
- Primarily for **within-user longitudinal self-comparison**, not for absolute cross-person ranking;
- Not suitable as a high-stakes tool for hiring, dismissal, or performance ranking;
- A rising curve can only serve as a **reference** for changes in sparse intervention capability, not proof of cognitive improvement; a falling curve may result from harder tasks, model changes, or audience changes.

### Known Limitations

1. **Denominator does not measure cognitive investment**: intervention amount only captures explicit form;
2. **Common method bias**: all state confirmations come from the same user; C2 is the user's interpretation of feedback;
3. **Selection bias**: successful/important tasks are more likely to receive second-round backfill;
4. **Incomplete attribution**: the O0→O1 change is not necessarily entirely caused by human intervention (model stochasticity);
5. **Gaming risk**: users may inflate scores by choosing tasks, compressing expressions, or selectively recording.

## 6. Privacy & Data

- Records may contain business secrets, personal information, and client data;
- **Local-first**: records are stored locally by default and uploaded to no server;
- Data belongs to the user; it can be deleted or exported at any time;
- Any aggregation, anonymization, or third-party use (e.g., model provider optimization) requires a separate agreement covering purpose, duration, revocability, and contribution attribution;
- Model providers should distinguish: instant "good/bad" buttons ≠ first-round adoption confirmation ≠ second-round audience outcome confirmation; outstanding post-intervention results must not be fully attributed to the model.

## 7. Applications

| Level | Scenario | Note |
|---|---|---|
| Individual | Self-observation and review | Primary use case |
| Organization | Team collaboration comparison | Requires unified model/task conditions; not for performance judgment |
| Model provider | High-leverage intervention discovery | Repeated human interventions reveal model blind spots, evaluation items, and training material (second-layer application) |
| Community | Leaderboard / challenge | Strict condition control required, see 4.4 |

## Original Manuscript

The full method manuscript (with formulas, measurement protocol, and validation agenda):

- [Method manuscript (EN)](PAPER_EN.md)
- [Method manuscript (CN)](PAPER_CN.md)
- [Source ledger](PAPER-sources.json)

Collection protocol:

- [HCLR Collection Protocol (EN)](PROTOCOL.md) — how to continuously collect HCLR data in any conversation system
- [采集协议规范（中文）](PROTOCOL_CN.md)

> The manuscript is labeled as a v0.4 method draft for dissemination, not a formal peer-reviewed publication. See the document statement at the top of each paper for details.

## Empirical Pilot

HCLR is being validated with real conversation data (task events from AI-assistant dialogues; automated statistics, one-line user confirmation).

- [Pilot case (EN)](examples/PILOT_CASE_EN.md)
- [Pilot case (CN)](examples/PILOT_CASE_CN.md)

## FAQ

- **Who is HCLR for?** Anyone who regularly uses AI to produce deliverables and wants to observe how their judgments influence model outputs.
- **How do I start?** Record task events (O0 → interventions → O1 → C1 → C2) over 2–4 weeks of real work. See the manuscript for the full protocol.
- **What does a score mean?** A reference value for your explicit intervention leverage under a specific model, task, and period. It is not a general intelligence score.
- **Is HCLR a validated measurement?** No. All current scores are experimental references; large-scale reliability and validity testing has not yet been completed.
- **Can it be used for hiring or performance decisions?** Not recommended until reliability, validity, and gaming resistance are established.

## 8. License

This repository (including documents and code) is under a **custom license**:

- **Non-commercial use is free**, provided that you (1) **open a GitHub Issue in this repository** to notify the author before use, and (2) **credit the author** in any derivative work;
- **Commercial use** requires a separate agreement with the author;
- Applies to all content in this repository (README, schema, code, etc.).

See [LICENSE](LICENSE) for details.

---

## Author

[Rootosophy](https://github.com/Rootosophy)

## Version

0.4 (finalized method manuscript, 2026-08-12). Empirical pilot data continue to accumulate in real conversations; see [examples/PILOT_CASE_EN.md](examples/PILOT_CASE_EN.md).
