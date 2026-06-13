# checklist-backend — バックエンド(PHP/Python/Node/API/DB)チェックリスト

## セキュリティ（共通）
- [ ] SQL：プレースホルダ/ORM。文字列連結禁止
- [ ] コマンド実行：引数配列・shell経由回避・ユーザー入力を渡さない
- [ ] パストラバーサル：パス正規化＋許可ディレクトリ制限
- [ ] 認証：パスワードはハッシュ(bcrypt/argon2)、セッション/トークン適正管理
- [ ] 認可：エンドポイントごと権限チェック、IDOR防止
- [ ] CSRFトークン、CORSのorigin限定、レート制限
- [ ] 秘匿情報は環境変数/シークレットマネージャ（リポジトリに含めない）

## PHP固有
- [ ] PDO＋プリペアドステートメント、`htmlspecialchars`で出力エスケープ
- [ ] `include`/`require`/`file_get_contents`にユーザー入力を渡さない
- [ ] 本番：`display_errors=Off`・適切な`error_reporting`

## Python固有
- [ ] `pickle`/`yaml.load`を信頼境界外で使わない（`safe_load`）
- [ ] `subprocess`は引数配列＋`shell=False`
- [ ] f-string/`%`でのSQL組立て禁止

## Node固有
- [ ] `eval`/`child_process`の入力注意、`npm audit`で依存脆弱性
- [ ] 未捕捉Promise・`await`漏れ・unhandledRejection

## データ/API
- [ ] 入力バリデーション（型/必須/範囲）をエンドポイント境界で
- [ ] トランザクション境界・一意制約・外部キー
- [ ] N+1回避（eager load）・ページング・インデックス
- [ ] エラー応答で内部情報を露出しない・適切なHTTPステータス
- [ ] マイグレーションの可逆性・後方互換
