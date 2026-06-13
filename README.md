# Parablex

> *Reverse-engineering the audit & research doctrine of Anthropic's Claude Fable 5 into a reusable codex.*

**English** | [日本語](README.ja.md)

Parablex (`parablex-kit`) is a family of [Claude Code](https://claude.com/claude-code) skills that **extracts and reproduces the operating discipline** — the *method*, not the raw capability — behind high-quality audits and research.

The name blends **parable** (a lesson-bearing story — the same lineage as Anthropic's *Fable* model) with **-ex** (as in *codex* / *vertex*): a decoded, systematized codex of how a top-tier model approached the work.

## Concept
Part of what makes a high-capability model's output good is not knowledge but **discipline — a way of thinking**. Parablex distills that into a shared [DOCTRINE.md](DOCTRINE.md) (9 principles) that each use-case skill inherits, stabilizing the **floor of every output** (coverage, evidence, structure) regardless of which model or person runs it.

## What it does / does not do (scope)
No overpromising — a self-application of the doctrine's "stay in your lane."

**It provides (the floor it guarantees)**
- **Coverage** — a fixed checklist reduces missed categories
- **Evidence** — every finding cites `file:line` / a source; speculation is separated from fact
- **Structured, actionable output** — severity, fix approach, implementation order, isolated human-decision items
- **Reproducibility** — the same frame on every run
- **Calibrated confidence** — self-reported certainty (high / mid / low)

**It does not provide (the ceiling)**
- An uplift in the model's own reasoning or discovery ability — **it does not guarantee Fable-5-level insight or performance**
- Depth is model-dependent; a weaker model yields "thorough but shallow"
- Output quality is not a fixed expected value — it varies by model × task

## Modules
| Skill | Purpose | Triggers (Japanese) |
|---|---|---|
| [parablex-coding](parablex-coding/SKILL.md) | Code audit / review / feature design (3 modes) | 「監査」「コードレビュー」「audit」 |
| [parablex-research](parablex-research/SKILL.md) | Tech research / selection / comparison | 「調査」「技術選定」「research」 |
| [parablex-writing](parablex-writing/SKILL.md) | Doc / spec / proposal drafting & review | 「文書作成」「推敲」「writing」 |
| [parablex-plan](parablex-plan/SKILL.md) | Stress-testing a plan or design | 「計画レビュー」「設計レビュー」「plan」 |

> Trigger words are Japanese by design (the author works in Japanese). Edit the `name`/`description` in each `SKILL.md` frontmatter to localize.

## The 9 principles (DOCTRINE)
Conclusion first · Evidence-based · Severity ranking · Explicit confidence · Isolate human decisions · Exhaustive checklist · Actionable next steps · Note strengths too · Stay in your lane

## Usage (Claude Code)
- Place each module's `SKILL.md` under `.claude/skills/<module>/` → invoke by trigger word
- e.g. "audit this file" / "select a React state-management library"
- Mode example: "audit in feature-plan mode"

## Layout
```
parablex-kit/
  DOCTRINE.md                  # shared method (9 principles + rubrics)
  README.md / README.ja.md
  parablex-coding/
    SKILL.md
    references/
      checklist-common.md      # language-agnostic
      checklist-frontend.md    # HTML/CSS/JS/TS
      checklist-backend.md     # PHP/Python/Node/API/DB
      checklist-sql.md         # SQL / databases
      checklist-wordpress.md   # WordPress / PHP
  parablex-research/
    SKILL.md
  parablex-writing/
    SKILL.md
  parablex-plan/
    SKILL.md
  origin/
    fable5-audit-transcript.md # source material (redacted)
```

## Extending
Add a new module (e.g. `parablex-writing`, `parablex-plan`); the DOCTRINE stays shared.

## Origin
Generalized from three real audit passes that Claude Fable 5 performed on a single-file web app (2026-06): pre-release security, feature-addition design, and post-implementation quality.

## License
MIT — see [LICENSE](LICENSE). © 2026 GiHiro888.
