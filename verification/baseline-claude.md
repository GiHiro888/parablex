# Baseline run — parablex-coding (final-quality)

Reference output for the [VERIFICATION](../VERIFICATION.md) overlap step. A real run of `parablex-coding` in this repo's home environment, so other-model/other-runtime runs on the same target can be compared against it.

- **model:** claude-sonnet-4-6
- **runtime:** Claude Code
- **skill:** parablex-coding (final-quality)
- **target:** `expense-app.html` — a single-file HTML receipt/expense-tallying app (~1211 lines, localStorage). Same app family as [`origin/fable5-audit-transcript.md`](../origin/fable5-audit-transcript.md), but a **later, hardened version** (most of Fable 5's High/Medium items are already fixed).
- **date:** 2026-06-13
- redaction: paths/personal info removed; the source contains no embedded personal data.

---

# Overall rating
**A−** (release-ready. No critical/high issues. A few Low items + minor structural debt.)

# Critical issues
None.

# High
None. (Notably, the prior High from the Fable 5 source — "no orphan-record repair UI" — is now **resolved**: orphan section + revert/delete UI at L254-279, L386-393, L855-895.)

# Medium
1. **CSP relies on `'unsafe-inline'`** (L6: `script-src 'self' 'unsafe-inline'`; `style-src 'unsafe-inline'`). The page uses one inline `<script>` and inline `style=` attributes, so CSP cannot block injected inline script — XSS safety rests **entirely on `escapeHtml`** (L439-441), not on CSP. Acceptable for a single file, but the CSP is closer to documentation than enforcement. `confidence: high`
2. **`crypto.randomUUID()` (L669) is undefined over plain `http://`** → `addEntry` throws and registration fails. Works on `https`/`file://`/`localhost`. If ever served over http, new entries break silently for the user. `confidence: high`

# Low
1. **Crafted-import inconsistency**: `isValidExpense` (L451-453) accepts `settlement_id` independent of `status`. A hand-edited JSON with `status:"未"` + `settlement_id` (or `"済"` without a matching settlement) can slip in; the orphan UI mitigates the latter, the former can double-count via `revertToUnpaid` (L957-983). Import-only vector. `confidence: mid`
2. **Asymmetric import warnings**: `importMergeData` warns about resulting orphans (L1162, L1176-1181); wholesale `importData` (L1093) has no such check. `confidence: high`
3. **`editingId` not reset on import/clear-all**: stale "editing" indicator can remain; the submit guard (L640-645) prevents real harm — display-only inconsistency. `confidence: high`
4. **Duplicate form-reset code** in 3 places (L658-663, L672-677, L706-711) → extract a `resetForm()`. `confidence: high`
5. **No integrity check** between `settlement.total` and the sum of its member expenses on import. `confidence: mid`
6. **Magic number** `maxlength="2020"` (L144) undocumented (≈ 20 items × 100 chars + newlines). `confidence: high`
7. **Blocking `alert()`/`confirm()`** used throughout — fine for a personal tool, but not styleable and interrupts flow. `confidence: high`

# Missing requirements
None. Orphan repair UI, mobile layout, backup management, and date-format validation are all present.

# Good implementations
- **Consistent XSS defense**: `escapeHtml` on all dynamic insertions including attribute values; whitelist validation via `PAYMENT_METHODS` / `TAX_RATES` / `ID_PATTERN` / `DATE_PATTERN` (L308-311, L444-468).
- **CSV-injection mitigation**: `csvSafe` (L1053-1057) escapes leading `= + - @ \t \r`; UTF-8 BOM for Excel (L1070).
- **Layered data protection**: load-time `isValidDb` + corrupted-data quarantine (L535-551), auto-backup before destructive ops (`createBackup`), `MAX_BACKUPS` pruning (L562-567), `clearAll` 3-step confirm + backup prompt (L1192-1208).
- **Backup management UI** with restore/delete (L988-1051) — resolves the source's Medium "unbounded backup accumulation".
- **Backward/forward compatibility**: new fields optional in `isValidExpense` (L454-466).
- **calcTax float mitigation** (L323-328): integer math + epsilon avoids `1080/1.08 → 999.99…` errors.
- **Mobile**: `.table-wrapper { overflow-x:auto }` (L52) + media query (L76-92) + scroll hint — resolves the source's Medium "mobile overflow".
- **CSP present** + all-`addEventListener` event delegation (no inline handlers in body).

# Features to implement next
1. **Reduce CSP reliance on `'unsafe-inline'`**: move the inline script/styles out to enable a nonce/hash policy (only if the hosting model warrants hardening).
2. **Refactor**: extract `resetForm()`; unify the `importData`/`importMergeData` file-read boilerplate.
3. **Reset `editingId`** on import/clear-all.
4. *(optional)* **Integrity check** of `settlement.total` vs member-expense sum on import.

`confidence: high` (all findings verified directly against the source line numbers).
