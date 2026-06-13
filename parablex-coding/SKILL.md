# parablex-coding — コード監査・レビュー・機能設計

> 言語非依存のコード監査スキル（旧 release-audit を昇格・汎用化）。
> 共通の型は [../DOCTRINE.md](../DOCTRINE.md) を継承：結論先出し・根拠主義(`file:行`)・重大度・confidence・人間判断分離・**監査中はコード変更なし**。

# inputs
- 対象：任意言語のソース（ファイル / ディレクトリ / 差分）
- モード（任意）：`security` | `feature-plan` | `final-quality`（既定＝`security`）
- スタック自動判定 → 該当 references チェックリストを適用

# outputs
- 監査レポート（**コード変更なし**・計画／評価のみ）
- 指示時のみVault保存：`30_Reviews/{date} {対象} audit.md`（`type: review`, `scope: project`）
- 再利用知見は `#to-knowledge` 付与（lesson-extractor連携）

# モード別の出力構成
## security（公開前セキュリティ監査）
リスク一覧（A〜D分類）→ 重大度(Critical/High/Med/Low) → 改修対象 → 修正方針 → 実装順序 → 人間確認事項 → confidence
## feature-plan（機能追加の事前設計）
結論先出し → 機能別難度 → データ構造影響 → 既存データ/IF互換性 → 複雑化リスク → 事前決定事項 → 実装順序 → 延期項目 → 小分けタスク案(T1..) → 人間確認事項
## final-quality（実装後の最終品質）
総合評価(A〜D) → 致命的問題 → High / Medium / Low → 要件漏れ → 良い実装（評価点）→ 次に実装すべき機能

# 言語非依存リスク分類（網羅チェック）
- **A. セキュリティ**：インジェクション(SQL/コマンド/パス/XSS/SSRF/テンプレ/デシリアライズ)・認証認可(IDOR/権限昇格)・秘匿情報・信頼境界
- **B. データ整合性/堅牢性**：例外処理・原子性/トランザクション・保存失敗・破損隔離・スキーマ移行・後方/前方互換
- **C. 入力検証/境界値**：型/範囲/形式/enum・長さ上限・null/空・特殊値(負数/Infinity/巨大)・エンコーディング
- **D. 信頼性/性能/保守性**：リソースリーク・N+1/非有界・冪等性/競合・重複/デッド/マジックナンバー・UI誤操作/Undo/a11y

# 手順（Pseudo）
1. 対象を読込（行番号付き）。スタック判定。**コードは一切変更しない**
2. モード観点 ＋ A〜D分類 ＋ スタック別 references で走査
3. 指摘は必ず `file:行番号` ＋ 再現/影響 ＋ なぜ問題か を併記
4. 重大度・総合評価を付与。人間判断事項を分離
5. confidence明記（high=コード確認 / mid=理論 / low=未確認）
6. 指示あればVault保存・知見抽出

# スタック別チェックリスト
- フロントエンド(HTML/CSS/JS/TS)：[references/checklist-frontend.md](references/checklist-frontend.md)
- バックエンド(PHP/Python/Node/API/DB)：[references/checklist-backend.md](references/checklist-backend.md)
- 全言語共通：[references/checklist-common.md](references/checklist-common.md)

# 期待出力サンプル（final-quality抜粋）
```markdown
# 総合評価
**B**（公開可能水準。致命的問題なし。High 1件の修正を推奨）
# 致命的問題
なし
# High
- 孤児明細の修復UI不在（src/app.js:866-868 で案内も実行手段なし）
# Medium
1. バックアップの無限蓄積（削除UI・上限なし）— storage.js:803
# 良い実装
- 全動的挿入でエスケープ適用・属性値含め漏れなし
# 次に実装すべき機能
1. 孤児明細の修復UI（最優先）
```
