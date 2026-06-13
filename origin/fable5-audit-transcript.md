# Fable 5 Audit Transcript — Source Material

**English** | [日本語（逐語・正本 / verbatim original）](fable5-audit-transcript.ja.md)

> Full transcript of the actual Claude Fable 5 audit passes this kit was distilled from.
> This English version is a **translation**; the authoritative **verbatim** record is the Japanese original ([fable5-audit-transcript.ja.md](fable5-audit-transcript.ja.md)).
> Only environment-specific info (paths, usernames), personal info, and the target app name are redacted (`<repo>` / `<home>` / `expense-app`). Findings, line numbers, and reasoning are otherwise faithful to the original.
>
> Three passes: (1) pre-release security, (2) feature-addition design, (3) post-implementation quality. Fable 5 audited only; a different model implemented.

---

## Fable 5 — turn 1

I'll locate and audit the expense-app HTML file.

## Fable 5 — turn 2

**[tool: Bash]**
```json
{
  "command": "find <repo> -maxdepth 3 -iname \"*.html\" -not -path \"*/node_modules/*\" 2>/dev/null",
  "description": "Find HTML files in project"
}
```

## Fable 5 — turn 3

**[tool: Bash]**
```json
{
  "command": "wc -l -c \"<repo>/works/expense-app.html\"; ls -la <repo>/works/",
  "description": "Check file size and works directory contents"
}
```

## Fable 5 — turn 4

**[tool: Read]**
```json
{
  "file_path": "<repo>/works/expense-app.html"
}
```

## Fable 5 — turn 5

Audit complete. Target: `works/expense-app.html` (364 lines, single-file HTML, localStorage persistence). No code changes; plan only.

# expense-app.html — Pre-Release Audit Report

## 1. Current risk inventory

### Security
| # | Risk | Location |
|---|---|---|
| S1 | **Stored XSS**: `item.shop` inserted via `innerHTML` without escaping. Input like `<img src=x onerror=...>` executes script | `render()` L281, `showDetail()` L309 |
| S2 | **XSS via import**: `importData()` ingests JSON without validation → a crafted backup file injects arbitrary HTML/attributes into `shop` / `id` (the `onclick="showDetail('${s.id}')"` at L296 is an attribute-injection point) | L342-352, L296 |
| S3 | **CSV injection**: only `"` is escaped. No mitigation for cells starting with `=` `+` `-` `@` → formula execution when opened in Excel (e.g. `=cmd\|...`, exfiltration via `=HYPERLINK(...)`) | `exportCSV()` L322 |
| S4 | **No CSP**: no meta CSP; heavy inline-onclick usage makes it structurally impossible to add | entire file |

### Data corruption
| # | Risk | Location |
|---|---|---|
| D1 | **Startup crash**: `JSON.parse` in `loadFromStorage()` has no try/catch → corrupted localStorage halts the whole `onload`; app won't start | L149-152 |
| D2 | **Import overwrites everything**: no confirm dialog, no schema validation, replaces all of `db` → selecting the wrong file instantly wipes all data. Malformed structure (e.g. missing `expenses`) permanently crashes `render()` | L342-352 |
| D3 | **Silent save failure**: `saveToStorage()` doesn't handle QuotaExceededError → memory and localStorage diverge; input lost on reload | L154-156 |
| D4 | **ID precision issue**: `Date.now() + Math.random()` (float) restored via HTML attribute through `parseFloat` → precision loss on stringification may make selective settle/delete target the wrong row | L176, L188, L220 |
| D5 | **No schema version**: no migration path for future structure changes | entire file |

### Input validation
| # | Risk | Location |
|---|---|---|
| V1 | Amount: allows negatives, `Infinity` (`1e999`), decimals, huge values. If JPY is assumed, integer enforcement is needed | `addEntry()` L170-171 |
| V2 | Date: can register while empty → blanks leak into sort / CSV output | L168, no validation |
| V3 | Vendor: no length limit | L169 |

### Accidental deletion / UI / maintainability
| # | Risk | Location |
|---|---|---|
| U1 | `clearAll()`: single confirm only, no pre-backup prompt, no undo | L354-361 |
| U2 | `deleteSelected()`: physical delete, no undo | L219-236 |
| U3 | Settlement history truncated to latest 10 (`slice(0,10)`), not shown on screen → users think it "disappeared" | L287 |
| U4 | `exportCSV()` destructively sorts `db.expenses` (side effect: stored order changes) | L321 |
| U5 | Heavy `alert()` use; `label` lacks `for` (accessibility); `URL.revokeObjectURL` not called | entire file |

## 2. Priority
- **High** (required before release): S1, S2, S3, D1, D2, D3
- **Medium**: D4, V1, V2, U1, U3, S4
- **Low**: V3, U2, U4, U5, D5

## 3. Files to change
- `works/expense-app.html` only (keep single-file structure; no new files/libraries)

## 4. Concrete fix approach
- **S1/S2 (XSS)**: add a shared `escapeHtml()` and apply to every `innerHTML` insertion; or rebuild via `textContent`. Change `onclick="showDetail('${s.id}')"` to `addEventListener` + `dataset` lookup (eliminates attribute injection)
- **S3 (CSV)**: prefix `'` when a cell starts with `=` `+` `-` `@` `\t` `\r` (standard Excel formula-neutralization)
- **D1 (corruption resilience)**: wrap `loadFromStorage()` in try/catch; on parse failure, move corrupted data to an `expense_db_corrupted_{timestamp}` key before resetting (never silently discard)
- **D2 (import defense)**: (1) add a schema validator (`expenses`/`settlements` are arrays; check each element's type/required keys; string-length caps) (2) confirm "this overwrites current data" before import (3) auto-stash current data to a separate localStorage key before overwriting (4) error notice on parse failure
- **D3 (save failure)**: try/catch in `saveToStorage()`; on failure, clearly state "Could not save. Exporting a backup is recommended"
- **D4 (ID)**: switch new IDs to `crypto.randomUUID()` (string). Migrate existing data to strings on load (unify comparisons to string equality)
- **V1/V2**: amount = positive integers only (`Number.isSafeInteger` + upper bound); date = required; vendor = ~100-char cap
- **U1**: before clear-all, offer to auto-download a JSON backup (or a two-step confirm requiring typing "delete")
- **U3**: add a "showing latest 10 (of N total)" note, or pagination
- **U4**: copy before sorting via `[...db.expenses].sort()`
- **S4 (CSP)**: after removing all inline onclick and switching to `addEventListener`, add `<meta http-equiv="Content-Security-Policy">` (large scope — see #6)

## 5. Implementation order
1. **D1 → D3**: harden save/load (the verification base for other fixes; lay an unbreakable foundation first)
2. **First half of S1/S2**: introduce `escapeHtml()` and apply at all render sites
3. **D2**: import validation / confirm / stash
4. **S3**: CSV formula-injection mitigation + U4 (non-destructive sort) together
5. **V1/V2/V3**: strengthen input validation
6. **D4**: change ID scheme + migrate existing data
7. **U1/U2/U3**: accidental-deletion prevention / history note
8. **S4 + U5**: remove all inline handlers → CSP meta tag → finishing touches (label / revokeObjectURL etc.)
9. Browser regression check after each stage (register → selective settle → delete → CSV → backup → restore → clear-all)

## 6. Decisions requiring a human
- **Amount spec**: allow negatives (refunds/offsets)? Integer-only OK? → changes V1's implementation
- **Import behavior**: keep full-overwrite (+ confirm), or offer merge (append)? → merge is a spec change
- **History 10-item limit**: keep with a note, or show all / paginate?
- **Delete undo**: a trash (soft delete) is a spec change, needs approval. A minimal two-step confirm stays within the current spec
- **CSP scope**: removing all inline handlers touches many lines (every button, dynamic rows). Deployment form (whether static hosting can set headers) also matters → feasibility and timing
- **ID migration**: whether to auto-migrate existing localStorage data (intend to prompt a backup export beforehand)

confidence: high (S1–S3, D1–D3 verified directly in code. D4's precision issue is a theoretical possibility with low real-world frequency)

I'll start from implementation step 1 on approval.

## Fable 5 — turn 6

**[tool: Read]**
```json
{
  "file_path": "<repo>/works/expense-app.html"
}
```

## Fable 5 — turn 7

Audit complete. No code changes; verified the actual data structure of the current implementation (`works/expense-app.html`).

**Conclusion first**: order = **3 memo → 4 payment method → 1 tax rate (record only) → 5 JSON merge → 2 parent-child**. Defer CSV merge, tax-amount calc, and the 2027 1% rule. Because the current validator (`isValidDb`) does not reject unknown fields, **adding all new items as optional fields needs no migration of existing data** (old JSON → new app, new JSON → old app: bidirectional compatibility holds).

---

## 1. Implementation difficulty per feature

| # | Feature | Difficulty | Main reason |
|---|---|---|---|
| 3 | Memo field | **Low** | one optional field + a render conditional only |
| 4 | Payment method | **Low** | fixed select (enum) → validation/XSS concern ~zero |
| 1 | Per-rate tax calc | **Medium** | recording/aggregation is easy; "calculating" tax requires fixing the rounding spec first |
| 5 | Data merge | **High (JSON) / not recommended (CSV)** | dedup, integrity validation, preview UI. CSV has the fatal issue in #4 below |
| 2 | Parent-child | **High** | largest data-structure impact, largest UI complexity, requires CSV-output design |

## 2. Impact on data structure

Current expense: `{id, date, shop, amount, status, settlement_id?}`

```yaml
# Proposed added fields (all optional = coexist without changing existing data)
memo: string        # Feature 3. ~200 chars max
payment: string     # Feature 4. enum: cash|credit card|PayPay|Rakuten Pay|d-payment|au PAY
taxRate: number     # Feature 1. only 8 | 10 allowed. unset = uncategorized
items:              # Feature 2. optional (opt-in)
  - name: string    # 100 chars max
    amount: number  # positive integer
    taxRate: number # future extension
```

- settlements structure: no change needed (settlement stays at the parent `amount` unit; even with Feature 2, keeping children display-only isolates the impact)
- Must add to `isValidExpense`: the "allow `undefined` + validate type/range/enum only when present" pattern
- Feature 5: no structural additions. But old-style numeric ids (from `Date.now()+Math.random()`) may remain, so dedup must compare via `String(id)`

## 3. Compatibility risk with existing JSON/CSV

**JSON: low risk**
- old backup → new app: new fields `undefined` → pass optional validation. No problem
- new backup → old app: current `isValidDb` doesn't inspect unknown fields → passes. No problem
- One caveat: making the new validation "strict when present" means a hand-edited invalid value (e.g. `taxRate: "10%"`) causes **the whole file to be rejected** (current all-or-nothing behavior). One bad record blocks the entire load → worth considering a "drop only the invalid fields and continue" approach

**CSV: medium–high risk**
- adding columns (memo, payment, tax rate) → column-count mismatch with old CSV. Minor real harm for Excel-viewing use
- **Fatal**: current CSV has no `id` column → dedup in Feature 5's CSV merge is fundamentally impossible. Re-merging the same record = double counting, guaranteed
- Hand-rolled CSV parsing (commas/newlines inside quotes, `""` escaping, BOM) is a bug breeding ground under the no-external-library constraint

## 4. UI-complexity risk
- input form: 3 → up to 6 fields. Two rows via `form-row` flex. Mitigation: **keep optional as default and the required set at the current 3 fields**
- this app uses large fonts (20–24px) → more fields means a taller layout / more scrolling. Recommend applying the "hide column if memo is empty" approach (Feature 3 requirement) to tax rate and payment too
- Feature 2's dynamic line-item-row UI is the biggest complexity driver. Expanding child rows in the list can reuse a modal (existing `showDetail` pattern) to reduce it
- Feature 5's "popup window": `window.open` can't open new HTML (single-file constraint) + is subject to popup blockers → **strongly recommend reusing the existing modal**

## 5. Tax-calc spec items to decide first
| Decision | Options | Recommendation (confidence) |
|---|---|---|
| Input method | tax-included (back-calc) / tax-excluded + tax | keep tax-included, show back-calc (high: UX unchanged) |
| Rounding | floor / round / ceil | **human decision required**. Won't assert tax-law correctness |
| Calc unit | per-record / from per-rate totals | per-record (mid: personal use. invoice style is total-unit but out of this app's scope) |
| How the tax rate is held | per-receipt / per-line-item | **per-receipt first**. Avoid mixed-rate receipts by splitting entries → extend to per-line-item when Feature 2 lands (key to avoiding rework) |
| What to store | store tax amount / store rate only | store rate only, compute tax at display time (high: changing the rounding spec won't contaminate past data) |
| 2027 food 1% | — | **isolate as a provisional spec**. Tax rate is a fixed user-selected enum (8/10); do not implement date-based auto-detection → just add to the enum once the law is finalized |

## 6. Order to implement
1. **Feature 3 (memo)**: smallest change; establish the optional-field validation pattern
2. **Feature 4 (payment)**: establish the enum-validation pattern. Same structure as 3, low risk
3. **Feature 1 (tax rate — record + per-rate aggregation only)**: defer tax-amount calc until the rounding spec is fixed
4. **Feature 5 (JSON merge only)**: start after the validations for fields added in 1–3 settle (merge depends on all-field validation)
5. **Feature 2 (parent-child)**: last. Design together with "extend tax rate to per-line-item" from 1

## 7. Items to defer
- **CSV merge** (part of Feature 5): no id column → dedup impossible. Fixing it requires adding an id column to CSV = breaks old-CSV compat. JSON merge achieves the same goal, so **recommend skipping**
- **Auto tax-amount display**: waiting on the human rounding decision
- **2027 food 1%**: isolate as a law-dependent item to confirm (provisional spec). Design so a future enum addition suffices; wait
- **Standalone window for the "merged DL file"**: replace with an in-modal preview + reuse existing `downloadBlob`

## 8. Bite-sized implementation tasks to hand to Sonnet
Each task = ~one commit, independently verifiable (consistent with the earlier "2–3 items per batch"):

| ID | Task | Depends on |
|---|---|---|
| T1 | Add `memo`: form + `addEntry` validation + `isValidExpense` + render (hide if empty) + modal display. escapeHtml required | none |
| T2 | Add `memo` CSV column. Apply `csvSafe` + `""` escaping | T1 |
| T3 | Add `payment`: select element + whitelist validation + render + CSV column | none |
| T4 | Add `taxRate`: select (8/10/uncategorized) + validation (enum) | none |
| T5 | Per-rate aggregation display: below the unsettled total, "10% target: ¥n / 8% target: ¥n / uncategorized: ¥n" | T4 |
| T6 | Add CSV tax-rate column | T4 |
| T7 | JSON merge — logic layer: file read + `isValidDb` validation + id-dup detection (`String(id)` compare) + settlement-orphan reference check function. No UI | after T1–T4 |
| T8 | JSON merge — UI layer: preview modal (n added / n duplicates skipped / warnings) + auto-backup before running (reuse existing `expense_db_backup_` pattern) | T7 |
| T9 | Parent-child — structure: `items` optional + validation (name length, amount positive integer, count cap) | after T7 |
| T10 | Parent-child — input: dynamic line-row add UI + warn on mismatch with total (mismatch warns only, still allows registration) | T9 |
| T11 | Parent-child — display: mark rows that have line items in the list + show child items in the modal | T9 |
| T12 | Parent-child — CSV output method (method must be decided first: parent rows only / mixed parent-child / separate file) | T11 + human decision |
| T13 | Tax back-calc display (after the rounding spec is fixed) | T5 + human decision |

## 9. Decisions to confirm with a human
1. **Tax rounding**: floor/round/ceil (directly tied to tax-law correctness; this audit won't assert it)
2. **Tax-rate granularity**: approve the per-receipt-first proposal (section 5)?
3. **Allow parent-child total mismatch**: handling when receipt total ≠ line-item total due to discounts/points (recommend: warn only, still allow registration)
4. **Skip CSV merge**: approve? If implementing, accepting an id column in CSV (old-CSV incompat) is a prerequisite
5. **Dedup policy on merge**: skip (recommended) / overwrite / keep both
6. **Finalize the payment-method list**: contents/order (offer a free-text field? → recommend not, due to XSS/validation cost)
7. **Change behavior on validation failure**: relax current "one bad record rejects all" → "exclude only the bad ones and continue"? (especially impactful for merge)
8. **Schema version (D5)**: keep the established "no problem for now" premise. Only when implementing Feature 5 is there minor room to reconsider, for identifying source-file generations (optional, minor)

---

**Three-configuration summary**:
- **Minimal**: T1–T4 (memo + payment + tax-rate record). Data-structure impact = 3 optional fields only
- **Safe**: minimal + T5–T8 (per-rate aggregation + JSON merge). No tax calc → no tax risk; merge guarded by preview + auto-backup + dup-skip
- **Future-extended**: + T9–T13 (parent-child + tax calc + 2027 1% + reconsider CSV merge)

On the instruction to start, I'll begin from T1.

## Fable 5 — turn 8

I'll run the audit. First, I'll review the whole current file.

## Fable 5 — turn 9

**[tool: Read]**
```json
{
  "file_path": "<repo>/works/expense-app.html"
}
```

## Fable 5 — turn 10

# Overall rating

**B** (release-ready level. No critical issues. Fixing the 1 High is recommended)

# Critical issues

None

# High

**No UI to review/repair orphan records (settled records with no settlement history)**
- The merge warning (lines 866-868) advises "consider deleting or editing to adjust," but provides no means to do so
- Evidence:
  - the unsettled table shows only `status === "未"` (unsettled) (line 622) → orphans are hidden
  - `deleteSelected` assumes the unsettled-table checkboxes (line 573) → can't delete orphans
  - `startEdit` is triggered only by the unsettled-row edit button (line 255) → can't edit orphans
  - `showDetail` is triggered from settlement-history links (lines 259-262) → an orphan's settlement id isn't in history, so it's unreachable
- Result: orphans appear only in counts, CSV, and JSON; permanently un-actionable in the UI. The user-approved spec "after checking, prompt deletion or editing adjustment" is effectively unmet

# Medium

1. **Unbounded accumulation of in-localStorage backups**
   - `expense_db_backup_*` (lines 803, 856), `expense_db_corrupted_*` (lines 389-390) have no deletion means or cap
   - repeated imports pressure the 5MB quota → an underlying cause of `saveToStorage` failure. Can't be cleaned except via dev tools

2. **No date-string format validation**
   - `isValidExpense` only checks `typeof e.date === 'string'` (line 293)
   - CSV output leaves the date unquoted (line 756) → importing JSON with a comma-containing date shifts CSV columns. XSS is prevented by escapeHtml, so this is limited to an integrity issue

3. **Mobile display**
   - 8-column table + `font-size: 22px` + no media query + no `overflow-x` wrapper → horizontal overflow on phones
   - the items badge's `title` hover (line 654) doesn't work on touch devices (viewable in the post-settlement modal)

# Low

1. **Double subtraction on inconsistent data**: a record with `status:"未"` + a `settlement_id` (allowed by isValidExpense, lines 297-299) shows in the settlement breakdown, and "return to unsettled" can re-subtract from `total` (lines 713-715). Only via crafted JSON import
2. **Merge preview over-reports orphan count**: `mergeDb` scans the whole post-merge DB (lines 366-369) → pre-existing orphans are shown as if "caused by this merge"
3. **Replace-type import accepts orphans without warning**: `importData` (line 776) has no orphan check; asymmetric with the merge side (which warns)
4. **Edit-mode lingers**: after import/clear-all, `editingId` and the "editing" indicator remain. The submit-time guard (lines 460-465) means no real harm, display-only inconsistency
5. **Implicit amount conversion**: `formatAmountInput` (lines 277-283) silently converts "1.5" → "15"
6. **Duplicate code**: form-clear in 3 places (lines 478-483, 492-497, 526-531), taxCell template in 2 places (lines 646-648, 691-693), file-read boilerplate in 2 places (importData/importMergeData)
7. **`crypto.randomUUID`**: undefined in non-HTTPS (http:// serving) → can't register. No problem on HTTPS/file:///localhost
8. **Magic number**: the basis for textarea `maxlength="2020"` (line 103) (20 items × 100 chars + 19 newlines) is undocumented
9. **No integrity check between settlement.total and line-item sum**: can't detect total drift in imported data

# Missing requirements

- a means to delete/edit-adjust orphan records (same root as High 1; the warning's promise is unimplemented)
- a CSV column for the breakdown memo (items) → **agreed out of scope** (recorded as a decision, not a missing requirement)
- isolating the 2027 1% tax rule → **already decided to defer**

# Good implementations

- **Consistent XSS defense**: `escapeHtml` applied to all dynamic insertions, including attribute values, with no gaps. `modal-title` / `edit-mode-target` use `innerText`/`textContent`
- **Whitelist validation**: payment (PAYMENT_METHODS), taxRate (TAX_RATES), settlement ID (ID_PATTERN) exclude free strings
- **CSV-injection mitigation**: `csvSafe` (lines 737-740) escapes leading formula chars; BOM for Excel compatibility
- **Layered data protection**: validate-on-load + quarantine corrupted data (lines 388-393), auto-backup before destructive ops, clearAll's 3-step confirm + backup prompt
- **Backward/forward compatibility**: all new fields optional → old backups readable, new backups readable by the old version
- **Thorough `String(id)` comparison**: handles mixed old-numeric / new-UUID IDs
- **calcTax float mitigation** (lines 214-219): integer arithmetic + epsilon removes `1080/1.08 → 999.99…`-type errors, verified
- **CSP consistency**: removing all inline handlers + event delegation coexists with the `script-src` constraint

# Features to implement next

1. **Orphan-record repair UI** (addresses High, top priority): show orphans in the unsettled table with a warning, or a dedicated section offering "return to unsettled / delete"
2. **localStorage backup management**: list/restore/delete UI for `expense_db_backup_*` + a retention cap (e.g. latest 3)
3. **Mobile support**: an `overflow-x: auto` wrapper on the table + media queries
4. **Date format validation**: add `/^\d{4}-\d{2}-\d{2}$/` to `isValidExpense`/`isValidSettlement`
5. **Refactor**: unify `resetForm()`, extract a taxCell generator, unify file-read (pay down debt before adding features)
