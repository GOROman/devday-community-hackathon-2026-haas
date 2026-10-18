# Webhook署名fixtureの確認

2026-10-18、secret scanning通知の対象 `app/src/backend/mcp-events.test.ts:8` を、値を外部へ出力せず調査した。

検出された旧定数は、既知の連番バイト32個をBase64化したテストfixtureだった。同じ値を参照するファイルは当該テストだけで、SQLiteはメモリ内、送信transportは差し替えたモック、直接のfetch呼出しもなかった。MCP署名処理が要求する接頭辞がStripeの形式に似ているため検知されたと判断した。実Stripe資格情報を使用・検証していない。

紛らわしい固定文字列を削除し、用途が明確なtest-only文字列をbtoaで符号化してMCP形式へ組み立てるfixtureへ変更した。署名検証・不正入力・再送・失効・鍵更新の振る舞いは維持する。

実行：`bun test app/src/backend/mcp-events.test.ts`。16件成功、失敗0、107 assertions。

Git履歴の削除、資格情報の変更、secret scanning警告のdismissは行っていない。元プロジェクトへの変更も行っていない。
