# Worked example: A/B test of the `grilling` skill (vanilla vs. enhanced)

This is the evidence behind the proposed enhancement. It is deliberately **honest about a result that cut against the original hypothesis**: a capable model running the *current* skill already reasons very well. The enhancement's value is therefore not "fixing a blind skill" — it is **making rigor structural and verifiable instead of emergent and luck-dependent**, plus closing a few risk categories the vanilla run still missed.

## Method

- One generic plan (no personal data), stress-tested twice:
  - **Vanilla**: the verbatim upstream `grilling` SKILL.md body.
  - **Enhanced**: the same body + the five proposed rules.
- Both were told **not** to use web tools (so the test isolates *behavior from the prompt*, not tool access).
- A third, independent agent then **fact-checked every concrete claim in the vanilla output** against authoritative docs (gspread, Google Sheets API, Looker Studio, colabtools, Google Identity).

### The plan under test
> A scheduled job, running in Google Colab, reads a local CSV (exported daily by another tool) and writes its rows into a Google Sheet via gspread; a Looker Studio dashboard reads that Sheet and must always show the latest data. Goal: "fully automated", minimal setup, ideally no secret files. Assumptions: (a) use Colab's built-in Google auth so no credentials are managed; (b) start from the smallest OAuth scope; (c) the dashboard reads the same tab every time, so just overwrite it.

## Result 1 — the vanilla skill is strong

The fact-checker extracted **15 concrete technical claims** from the vanilla output and verified each:

| Verdict | Count |
|---|---|
| confirmed | **15** |
| refuted | 0 |
| unverifiable | 0 |

Vanilla correctly: identified that **Colab is not a scheduler** and challenged that premise; flagged Colab's ephemeral filesystem; identified `authenticate_user()` as interactive/unattended-incompatible; recommended `spreadsheets` scope + open-by-ID to avoid the broad Drive scope; warned about the **~60 writes/min/user** quota; recommended `USER_ENTERED`; caught the **shrink-row stale-data** footgun and **non-atomic clear+update**; and noted **Looker Studio caching (~15 min)**. No factual errors.

**So the original "it misses premise/idempotency/quotas" hypothesis was falsified by our own test.** Reported as-is.

## Result 2 — but the rigor is emergent, not guaranteed (the real gap)

The same audit confirmed four **structural** properties of the vanilla run, true regardless of how good the answers were:

1. **Zero verification.** Every claim was asserted from model memory with no lookup. All 15 were right *this time* — but the process "provides no evidence and would not have caught an error." This is luck, not method. In a separate real session, the same pattern confidently emitted a **wrong** mechanism — "Looker Studio binds to the worksheet by `gid`" — which authoritative docs refute (it binds by **tab name**). Nothing in the skill flags or catches that.
2. **No artifact inspection.** It never asked to see the actual CSV (encoding, delimiter, header row, sample types) before prescribing a typing/overwrite strategy.
3. **Inconsistent risk coverage.** Strong on idempotency/atomicity/quotas/secret-hygiene; **silent on** data privacy (a local CSV is now a cloud copy with whatever sharing the Sheet/dashboard has) and on **formula/CSV injection** (`= + - @` leading cells), among others.
4. **Premise challenge is real but shallow.** It challenged "Colab as scheduler" well, but did not interrogate deeper premises (which Colab tier? why Sheets+Looker at all vs. infra the user already runs?).

## Result 3 — what the enhanced skill added

On the identical plan, the enhanced version reliably produced what vanilla left to chance:

- **Tagged its unverifiable claims** `[UNVERIFIED]` (8 of them) instead of asserting them, and ended with an explicit list to confirm against docs (Rule 1 + Rule 5).
- **Listed the exact artifact checks** to run before coding: encoding (**explicitly called out Shift-JIS/CP932 mojibake risk**), header presence, delimiter, line endings, null representation, and whether any cell starts with `= + - @` (Rule 2).
- **Caught risks vanilla missed**: data **privacy / sharing** of the CSV-in-cloud; **formula injection** and the matching `RAW`-plus-sanitize tradeoff; **schema drift**; **timezone** (UTC vs JST `daily` boundary); **history/backfill** loss from overwrite; the `drive.file` picker limitation (Rule 4).

## Conclusion

The enhancement does **not** claim the skill is broken. It claims — and this test demonstrates — that the skill's quality is **model- and luck-dependent**, with **no verification, no artifact inspection, and uneven risk coverage** baked into the prompt. The five rules turn those into guarantees. The single highest-value rule is **Rule 1 (verify, or mark `[UNVERIFIED]`)**, because it is the only thing that would have caught the real-world `gid` error.

---
*Full transcripts (vanilla, enhanced, and the per-claim audit with sources) are reproducible by pasting the two skill variants and the plan above into Claude; the audit used WebSearch/WebFetch against official docs.*
