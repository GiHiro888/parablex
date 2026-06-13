# VERIFICATION — Proving a parablex run "functioned"

How to demonstrate, with evidence, that a parablex skill **functioned** on a given model/runtime — not merely that it ran. A raw action log alone proves only invocation (Level 0); it is **necessary but not sufficient**.

## Levels of "functioned"
| Level | Claim | Evidence |
|---|---|---|
| 0 | **Invoked** | the skill loaded/triggered (action log / transcript) |
| 1 | **Format-adherent** | the output follows the prescribed structure (rubric below) |
| 2 | **Content-valid** | findings are real: cited `file:line` exist, the issue is genuine (true-positive sampling) |
| 3 | **Portable / equivalent** | same skill + same target on another model/runtime yields equivalent structure and substantially overlapping findings vs a reference run |

Levels 0–2 prove "it functioned (here)". Level 3 is required to claim "it functioned **equivalently on another model**".

## Proof package (minimum)
1. **Environment meta** — model id, runtime, skill commit. (attribution)
2. **Fixed input** — same target file/dir, same mode. (control)
3. **Output artifact** — the audit report itself (not just the log).
4. **Rubric scorecard** — the 9-principle checklist (below), each ✓/✗.
5. **True-positive sampling** — pick N findings; confirm the cited line exists and the issue is real; record false positives.
6. *(Level 3)* **Overlap vs reference** — compare findings to a reference run; record overlap % and misses.
7. *(Level 3)* **Re-run stability** — run twice; structure and major findings stable.

## Rubric — DOCTRINE 9-principle adherence (Level 1)
Score the output, not the model's reputation:
- [ ] **Conclusion first** — a verdict/rating leads the report
- [ ] **Evidence-based** — every finding cites `file:line` (code) or a source (research)
- [ ] **Severity ranked** — explicit Critical/High/Med/Low (or impact×likelihood)
- [ ] **Confidence stated** — high/mid/low present
- [ ] **Human decisions isolated** — a separate section for decisions only a human can make
- [ ] **Exhaustive checklist walked** — coverage across the relevant categories
- [ ] **Actionable next steps** — fix approach + ordering, not just problems
- [ ] **Strengths noted** — good implementations acknowledged
- [ ] **Stayed in lane** — audit only, no code changes / no fabrication

Pass threshold (suggested): **≥ 8/9** for Level 1.

## True-positive sampling (Level 2)
- Sample size: min(5, total findings).
- For each: (a) does the cited line/range exist? (b) is the described issue real in the code? (c) is the severity defensible?
- Record: true positives / false positives / unverifiable. Suggested pass: **≥ 80% true-positive**, **0 fabricated line references**.

## Overlap vs reference (Level 3)
- Reference = a prior run on the same target (e.g. `origin/fable5-audit-transcript.md`, or `verification/baseline-claude.md`).
- Map findings by issue (not wording). Report: shared / reference-only (misses) / new.
- Suggested pass: **≥ 60% of reference High/Medium findings reproduced**.

## Run template (copy per verification)
```markdown
# parablex verification — <YYYY-MM-DD>
- model: <id>          runtime: <name/version>
- skill: parablex-<module> (<mode>)   kit commit: <hash>
- target: <file> (<lines> lines)
- reference (if Level 3): <path>

## Rubric (Level 1):  _ /9
[paste the 9 checkboxes with ✓/✗]

## True-positive sampling (Level 2):  TP _ / FP _ / unverifiable _
| # | finding | line exists? | real? | severity ok? |

## Overlap vs reference (Level 3):  shared _ / misses _ / new _

## Verdict
Level reached: 0 / 1 / 2 / 3.  Notes:
```

> Baselines: `origin/fable5-audit-transcript.md` (Fable 5, source) and `verification/baseline-claude.md` (Claude, this repo) are provided as reference runs for the overlap step.
