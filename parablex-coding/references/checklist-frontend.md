# checklist-frontend — フロントエンド(HTML/CSS/JS/TS)チェックリスト

## セキュリティ
- [ ] `innerHTML`挿入値・属性値を全escape（格納型/インポート経由XSS）
- [ ] 外部JSON取込前にスキーマ検証＋確認＋現データ自動退避
- [ ] CSVセル先頭 `= + - @` をエスケープ（数式インジェクション）
- [ ] inlineハンドラ全廃 → イベント委譲 → CSPメタタグ
- [ ] postMessage/iframeのorigin検証、`target="_blank"`は`rel="noopener"`

## 永続化（localStorage/IndexedDB）
- [ ] `JSON.parse`のtry/catchと破損データ別キー退避
- [ ] QuotaExceededError処理（保存失敗を通知）
- [ ] バックアップキーの上限・削除UI（無限蓄積防止）
- [ ] IDは文字列(`crypto.randomUUID`)・`String(id)`比較
  - ※`crypto.randomUUID`は非HTTPSで未定義（`http://`配信は不可。file:///localhost/HTTPSは可）

## 入力・UI
- [ ] 金額=正整数 / 日付=必須・形式(`/^\d{4}-\d{2}-\d{2}$/`) / 選択肢=enum
- [ ] 全消去前に自動バックアップDL or 二段確認、Undo検討
- [ ] 破壊的ソートはコピー（`[...arr].sort()`）してから
- [ ] 多列テーブルは`overflow-x`ラッパー＋メディアクエリ（モバイル横はみ出し）
- [ ] label `for`属性・`URL.revokeObjectURL`・aria（アクセシビリティ）

## TypeScript固有
- [ ] `any`濫用・非null断定`!`の乱用なし
- [ ] 外部境界（API/フォーム/localStorage）は実行時検証（zod等）。型は実行時を保証しない
