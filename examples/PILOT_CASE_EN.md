# HCLR Empirical Case: Conversation-Embedded Pilot

> Version: 2026-09-07 ｜ Model environment: DeepSeek v4 flash (pilot-001–005) / v4 pro (pilot-006–010)

## Background

After publishing the HCLR method manuscript, a **conversation-embedded collection** design was adopted to obtain real empirical data: AI assistant conversations are used directly as the source of task events. The user needs no extra form filling; statistics are fully automated.

## Design

| Element | Definition |
|---|---|
| Task event | One topical conversation (from request to confirmed output) |
| Model environment m | The bound large model (pilot-001–005: DeepSeek v4 flash; pilot-006–010: DeepSeek v4 pro); the AI assistant is the execution/presentation layer and does not enter the m parameter |
| O0 | Assistant's first-round output |
| h | User's intervention message |
| O1 | Output after intervention |
| O (numerator) | Total model output tokens (or chars) in the task event, including all generation rounds |
| I (denominator) | Total user intervention tokens (or chars), **excluding the initial task description** |
| HCLR | O / I (output/intervention leverage ratio) |
| C1 (first confirmation) | Result state: adopt / partial / reject |
| C2 (second confirmation) | Result state: approved / partial / rejected / pending (or custom adoption rate pctN); source: user (explicit) / landing (landing equals confirmation) / auto_timeout (24h default) / auto_confirm (accepted after follow-up) |

## Process

1. At the close of each topical conversation, the assistant requests C1 (one-line reply);
2. P is suggested by the assistant and confirmed by the user;
3. At a conclusive output, the assistant requests C2; an output landed locally (saved/executed and not revoked) counts as 100% recognition (landing); no response within 24 h defaults to recognition (auto_timeout), and an active follow-up with no response accepts the default (auto_confirm) — confirmation never stalls on missing feedback;
4. A summary report is generated every 10 tasks or weekly;
5. Raw records stay local (not published in the repository, to protect privacy).

## Sample Data (10 Tasks)

| Task | Domain | O | I | HCLR (O/I) | C1 | C2 | State | Example intervention |
|---|---|---|---|---|---|---|---|---|
| pilot-001 | Empirical design | 209 | 28 | 7.46 | partial | - | S1 | "OK, but I hope every conversation can be auto-counted without extra operations" |
| pilot-002 | Conceptual clarification | 134 | 60 | 2.23 | adopt | - | S1 | "Correction: you are not the model being used; the bound large model is …" |
| pilot-003 | HCLR revision & finalization | 8650 | 214 | 40.42 | adopt | - | S1 | "There are no auxiliary indicators anymore; P1–P5 as integers is fundamentally wrong." |
| pilot-004 | TOA release & promotion | 6424 | 385 | 16.69 | adopt | - | S1 | "Keep the same details. Also, my goal is that users can freely use this Skills…" |
| pilot-005 | Method form analysis | 2858 | 88 | 32.48 | adopt | - | S1 | "Are Skills usually single-shot? But HCLR needs statistics on every conversation…" |
| pilot-006 | Brand IT build-out | 1128 | 72 | 15.67 | adopt | approved | S3 | "I can't find the saved meeting record file, please confirm" |
| pilot-007 | Brand IT build-out | 397 | 0 | - | adopt | partial | S3 | (no intervention) |
| pilot-008 | Brand IT build-out | 507 | 0 | - | adopt | partial | S3 | (no intervention) |
| pilot-009 | Brand IT build-out | 507 | 186 | 2.73 | adopt | approved | S3 | "Confirmed, the platform license fee is an annual fee. Also: the lead requires cost savings, no extra regions… Next, draft an in-person meeting invitation for the vendor" |
| pilot-010 | Brand IT build-out | 781 | 0 | - | adopt | pct25 | S2 | (no intervention; C2=25%: draft too formal, user rewrote in a colloquial style for finalization) |

## Current Snapshot

```text
Tasks: 10
HCLR = ΣO / ΣI = 21595 / 1033 = 20.91
Numerator O (model output chars total): 21595 | avg per task: 2160
Denominator I (intervention chars total, excluding task description): 1033 | avg per task: 103
Result states:
  Post-intervention adoption rate (C1 incl. partial): 10/10 = 100%
  Explicit recognition rate (source=user, C2>=0.5): 4/5 = 80%
  Closed-loop recognition rate (incl. auto, C2>=0.5): 4/5 = 80%
```

> Note: O/I are computed from real message statistics in the Hermes session database (O = assistant text chars total; I = user intervention chars total, excluding the initial task description). pilot-001–005 used deepseek-v4-flash; pilot-006–010 used deepseek-v4-pro. C1 is confirmed for all 10 tasks (10/10 adopted, incl. pilot-001 partial). pilot-001–005 are methodology-document outputs with no C2 yet (under the closed-loop protocol, their confirmation requests will default to closed after timeout rather than staying pending); pilot-006–010 have C2 confirmed (approved×2, partial×2, pct25×1), all explicitly by the user (source=user). pilot-007/008/010 had no interventions (I=0), so HCLR is undefined (shown as "-") — the single-round output was adopted without corrections; pilot-010 was only 25% adopted: the draft was too formal, and the user rewrote it in a colloquial style for finalization.

## Significance

- **Zero-burden validation**: the user only answers one line of C1 at conversation close; P confirmation is one phrase;
- **Real data**: O, I, C1, and C2 are real conversation records, not simulated data;
- **Direct paper backfill**: as data accumulates, Sections 6–7 of the manuscript move from "validation plan" to "initial validation results";
- **Counterfactual control**: unified metric (chars); pilot-001–005 share one model (v4 flash) and pilot-006–010 share one model (v4 pro), so longitudinal comparison holds within each phase.

Raw records are kept locally under `pilot-data/` (consistent with the LICENSE: full user conversation records are not published).
