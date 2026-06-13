# checklist-wordpress — WordPress / PHP（テーマ・プラグイン）チェックリスト

## 入力・出力（XSS/インジェクション）
- [ ] 入力：`sanitize_text_field` / `sanitize_email` / `absint` 等で無害化
- [ ] 出力：`esc_html` / `esc_attr` / `esc_url` / `wp_kses` でエスケープ（文脈別）
- [ ] DB：`$wpdb->prepare()` 必須。`$wpdb->query` への直接埋め込み禁止
- [ ] 翻訳関数経由の出力も `esc_html__` 等を使用

## 認証・権限・CSRF
- [ ] 権限確認：`current_user_can()` を操作前に
- [ ] nonce：フォーム/AJAX/管理アクションに `wp_nonce_field` ＋ `check_admin_referer` / `wp_verify_nonce`
- [ ] AJAX：`wp_ajax_` と `wp_ajax_nopriv_` の使い分け（未ログイン公開可否を意識）
- [ ] REST API：`permission_callback` を必ず実装（省略＝全公開）

## ファイル・実行
- [ ] 直アクセス防止：各PHP先頭に `if ( ! defined( 'ABSPATH' ) ) exit;`
- [ ] アップロード：MIME/拡張子検証・`wp_handle_upload`・実行可能ファイル拒否
- [ ] `eval`/`extract`/動的include にユーザー入力を渡さない

## 設計・保守
- [ ] フック（`add_action`/`add_filter`）優先。コア・親テーマの直接改変なし（子テーマ使用）
- [ ] プレフィックス付与（関数・グローバル・オプション名の衝突回避）
- [ ] エンキュー：`wp_enqueue_script/style`（直書き`<script>`回避）・依存とバージョン指定
- [ ] 秘匿情報は `wp-config.php`/環境変数（リポジトリに含めない）
- [ ] 非推奨API・PHPバージョン互換・`WP_DEBUG`本番Off
