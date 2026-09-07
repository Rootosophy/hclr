# History Rewrite Notice: Privacy Cleanup

> Date: 2026-09-07 ｜ 中文版 / Chinese version: [HISTORY_NOTICE_CN.md](HISTORY_NOTICE_CN.md)

## Why was the history rewritten?

On 2026-09-07 this repository's history was rewritten. Earlier public examples (`examples/PILOT_CASE_*.md`) quoted intervention messages drawn from real business usage that contained **sensitive information** — company/brand names, third-party vendor names, real personal names, specific amounts, and internal project codes. To protect the privacy of the user and related parties, this content has been anonymized across **the entire history**.

## What was changed

- Company / brand names → generalized (e.g. "公司 / the company")
- Third-party vendor names → generalized (e.g. "服务商 / the vendor")
- Real personal names → replaced by roles (e.g. "the lead")
- Specific amounts → relative descriptions (e.g. "the platform license fee is an annual fee")
- Internal project codes and file paths → removed / generalized

**Unaffected**: HCLR values (O / I / HCLR), C1/C2 result states, and the sample structure — the method and data remain fully reproducible.

## Impact

- All commit hashes changed on 2026-09-07 (force push);
- Previous clones, forks, commit links, and CI references may no longer resolve — use current `main`;
- To verify an old version, contact the maintainer via an Issue.

## Going forward

Since this cleanup, all data entering public examples (`examples/`) is **mandatorily anonymized** (rules in PROTOCOL §7 and at the end of the examples documents); a sensitive-term scan is required before publication.
