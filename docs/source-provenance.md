# ソース来歴

- 元の公開プロジェクト：`pupuchan1/future-company`
- 採用ブランチ：`codex/unified-voice-backend`
- 採用コミット：`d79c6db3d11865a8d13841c836d60460500a7b63`
- コミット日時：2026-10-18 16:30:29 JST
- 取り込み先：`app/`

取り込み時点のmain（`5baf023830d3ee9cf0c5ad456b1f3710e11667f0`）はフロント雛形とブランド資料まででした。React・会話記録・Node音声・MCP・Worker・結果返却を含む単一の統合ブランチを採用し、別ブランチを独自に混ぜていません。

アーカイブの200ファイルを取り込み、Git履歴・未追跡ファイル・node_modules・ビルドキャッシュ・私有状態・実環境のdotenvは含めていません。元の構成・workspaces・lockfile・README・検証記録・著作権・ライセンス表示を保持しています。元資料中のIssue/PRリンクや作業者のローカルパスは過去の記録で、提出環境で起動に必要なパスではありません。

唯一の設定変更は `app/deployment/wrangler.production.jsonc` です。実環境のアカウント・DB・relay設定値を持ち込まないため、同梱の `wrangler.example.jsonc` と同じプレースホルダーへ置き換えました。本番デプロイはしていません。

元プロジェクト全体への新しいライセンスは付与していません。第三者表示は[素材ライセンス](../app/frontend/public/licenses/)と[ElevenLabs UIのライセンス](../app/frontend/src/features/voice/elevenlabs/LICENSE.md)に保持しています。

取り込み時はアーカイブとの差分、禁止ファイル名、典型的なAPIキー・秘密鍵文字列、設定例、workspacesとlockfile、相対リンクと起動ファイルの存在を静的に確認しました。文字列確認は秘密情報の不在を完全に証明するものではありません。

当初は依存導入が承認レビューで拒否されたが、その後のユーザーの明示的な実動テスト依頼により導入・試験を実施した。対象コミット、252件の自動試験、ビルド・起動・ブラウザ確認の条件は[実測記録](submission-verification.md)を参照。製品コードは元の採用コミットから変更していない。
