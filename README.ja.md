# Parablex

[English](README.md) | **日本語**

> *Reverse-engineering the audit & research doctrine of Anthropic's Claude Fable 5 into a reusable codex.*

Fable 5 の監査・調査で機能していた**実行規律（思考の型）を抽出・再現**する Claude Code スキルファミリー（`parablex-kit`）。性能そのものではなく、**規律＝出力の floor（一貫性・網羅・根拠）** を移植する。

**名前の由来**：`parable`（寓話＝教訓を宿す物語。AnthropicのモデルFableと同じ系譜）＋ `-ex`（codex／vertex 調＝解読・体系化されたコーデックス）。上位モデルの手法を逆算・蒸留して誰でも使える形にする、という本ファミリーの趣旨を表す。

## コンセプト
高精度モデルが示す品質の一部は、知識でなく **規律（思考の型）** に由来する。その型を共通の [DOCTRINE.md](DOCTRINE.md)（9原則）として抽出し、用途別スキルが継承する。これにより、モデルや担当が替わっても **出力の下限（floor）＝網羅・根拠・構造** を安定化できる。

## もたらすもの／もたらさないもの（スコープ）
得られた知見が「何を」もたらすかを明確にする。誇大表現はしない（DOCTRINE「越権しない」の自己適用）。

**もたらす（保証範囲・floor）**
- 観点の網羅：固定チェックリストで見落としカテゴリを削減
- 根拠の明示：全指摘に `file:行`／出典、推測と事実の分離
- 構造化された実行可能アウトプット：重大度・修正方針・実装順序・人間判断の分離
- 再現性・一貫性：同じ枠組みでの反復実行
- confidence較正：確実度の自己申告

**もたらさない（限界・ceiling）**
- モデル自身の推論力・発見力の底上げ（＝**Fable 5級の洞察・性能は保証しない**）
- 深さはモデル依存：低性能モデルでは「網羅的だが浅い」結果になりうる
- アウトプット品質は固定の期待値ではなく、適用モデル × タスクで変動する

## モジュール
| スキル | 用途 | 主トリガー |
|---|---|---|
| [parablex-coding](parablex-coding/SKILL.md) | コード監査・レビュー・機能設計（3モード） | 「監査」「コードレビュー」「audit」 |
| [parablex-research](parablex-research/SKILL.md) | 技術調査・選定・比較検討 | 「調査」「技術選定」「research」 |

## 9原則（DOCTRINE）
結論先出し / 根拠主義 / 重大度ランク / confidence明記 / 人間判断の分離 / 網羅チェックリスト / 実行可能な次手 / 強みも明記 / 越権しない

## 使い方（Claude Code）
- `.claude/skills/{parablex-coding,parablex-research}/` に SKILL.md を配置 → トリガー語で起動
- 例：「このファイルを監査」「React状態管理ライブラリを技術選定」
- モード指定例：「feature-plan モードで監査」

## 構成
```
parablex-kit/
  DOCTRINE.md                  # 共通の型（9原則・ルーブリック）
  README.md / README.ja.md
  parablex-coding/
    SKILL.md
    references/
      checklist-common.md      # 全言語共通
      checklist-frontend.md    # HTML/CSS/JS/TS
      checklist-backend.md     # PHP/Python/Node/API/DB
  parablex-research/
    SKILL.md
```

## 拡張
用途追加は新モジュール（例 `parablex-writing` / `parablex-plan`）を足すだけ。DOCTRINE は共有。

## 由来
ある単体HTMLアプリに対し Claude Fable 5 が実施した監査3回（公開前セキュリティ／機能追加の事前設計／実装後の最終品質）の手順を一般化したもの。

## ライセンス
MIT（[LICENSE](LICENSE) 参照）. © 2026 GiHiro888.
