# 提出モノレポの実測検証

2026-10-18 16:35〜16:40 JST、Mac Studio。

- 対象コミット：`4b4599da44034e236e27a38f6c55bc8772fea79d`
- ソース：`app/`、採用元 `d79c6db3d11865a8d13841c836d60460500a7b63`
- Node.js 24.14.0（正規npmのnodeパッケージを一時利用）、npm 11.9.0、Bun 1.3.11。
- `npm ci --ignore-scripts --no-audit --no-fund` 成功、163パッケージ。最初は既定Node25のengine警告が出たため、以降のアプリ試験はNode24.14に切り替えた。
- 実API、外部MCP、実電話、第三者通知、公開デプロイは使用していない。新規の資格情報は生成していない。

## 実行結果

すべて `app/` から実行。

| コマンド | 結果 |
| --- | --- |
| `npm run test:frontend` | 89/89成功、失敗・skipなし |
| `npm run integration:test` | 86/86成功、失敗・skipなし |
| `npm run test:worker` | Node46件＋Bun31件＝77件成功、失敗なし |
| `npm run typecheck` | React・Workerとも成功 |
| `npm run build` | React・Workerとも成功 |
| `npm run lint` | exit 0。既存のOrbとLegacyPrototypeにwarning9件、errorなし |
| `npm run db:migrate:local` | ローカルD1の0001・0002適用成功 |
| `npm run dev` | React5173、Node4320、Worker8787のloopback起動成功 |

自動テストは計252件成功。ビルドは500kB超のチャンクとoutDirのwarningあり。依存のライフサイクルスクリプトは実行していないが、上記の型検査・ビルド・ローカル起動は成功した。

## Chromeによる実操作

既存ChromeをPlaywrightの独立一時プロファイルで起動し、日本語・架空データだけを使った。既存サービスの終了や既存ブラウザへのログイン操作はしていない。

### 現行の音声受付

同梱 `App.tsx` → `ReceptionPage` の通常入口 `http://127.0.0.1:5173/` を実際に開いた。

- ホーム・会話画面の表示と再読込HTTP200を確認。
- 「話してみる」でローカル案件作成へ進んだ後、relay未設定の `voice_relay_unconfigured (HTTP 503)` で停止。
- 実AI開始の `/api/live/*` はブラウザ側でもブロックして監視し、呼出し0件を確認。
- 設定なしで実音声の通しは試験していない。この停止を実AI応答成功とは扱わない。

[現行ホーム](screenshots/current-home.jpg) / [設定不足の会話画面](screenshots/current-unconfigured.jpg)

### 保管されている模擬UI

現行 `App.tsx` は `LegacyPrototype.tsx` を通常ルートに公開していない。このため、一時のVite fixtureを空きポート15173で作成し、**同梱のLegacyPrototype自体を変更せず**読み込んだ。HTMLリクエストをfixtureへ向けることで、模擬UIの `/operator` と `/session/:id/result` の再読込も確認した。fixtureは試験後に削除した。公開デプロイを撮った画面ではない。

1. 「AI受付と話す」から会話開始。
2. 文字入力で開発環境を質問し、人へ「検証用の開発言語を教えてください」と依頼。
3. 同じブラウザの別タブで担当者画面を開き、模擬呼出し→引受。
4. 「検証用の架空回答：TypeScriptとReactです。」を入力して結果報告。
5. 元タブへ回答本文と完了状態が反映されたことを確認。
6. 「会話を終了」→「終了してメモを見る」の非同期遷移を待つ。
7. 結果URLで再読込し、回答本文と「会話を再開」操作が残ることを確認。

[依頼](screenshots/legacy-request.jpg) / [担当者の回答入力](screenshots/legacy-operator.jpg) / [会話への反映](screenshots/legacy-answer.jpg) / [終了・再読込後](screenshots/legacy-result.jpg)

最初の検証スクリプトは終了操作後の遷移前に再読込していたため、結果URLへの遷移待ちを追加して再実行し、成功を確認した。別の検証スクリプトの同一URL移動時のHTTP応答参照も修正した。いずれも検証用スクリプトの修正で、製品ソースの修正は必要なかった。

## この確認で分かること

ローカル実装の自動テスト、ビルド、認証設定不足時の停止、および保管模擬UIの依頼・回答・保存の流れを実測した。公開プレビューの配信状態、実API・実マイク・実機電話との結合は別の確認である。担当者の過去の実API記録は `app/integration/VERIFICATION.md` と `app/docs/local-voice-demo-verification.md` に保持している。

## 資料同期

両スライドのプロフィール12箇所に、元画像のRGB画素を変えず円形アルファマスクを適用した。詳細版10枚と別本編6枚を全ページ描画確認。両方の最終ページのQRをレンダー画像から読み取り、`https://haas.nullevi.app/` と完全一致した。HaaS一般構想の図は番号付き4段階へ整理し、実操作した保管模擬UIの画面を明示して追加した。

デモ入口の公開・TLS・5秒自動移動・停止操作は別担当が確認済みと報告した。今回のMacからの通常DNS参照はキャッシュ遅延で名前解決できなかったため、今回のChromeによるローカル実装試験と同一の確認には数えていない。

検証で起動したローカル開発サービスと一時QAサーバーは終了し、一時fixtureは削除した。既存サービスは停止していない。

表紙以外の詳細版9枚・本編5枚には、元の1983×793ロゴを縦横比を維持して同じ左下フッターへ配置した。二重追加がないこと、全16ページの本文・図・QRとの重なりがないことを描画確認し、公開PDFとREADME用の5画像を同期した。製品コード・実測試験結果は変更していない。

## 公開フロントの撮り直し（2026-10-18 16:58 JST）

入口 `https://haas.nullevi.app/` から自動的に `https://future-company-fe-preview.aliceroad123.chatgpt.site/` へ移動したことをChromeで確認。その公開フロントで、受付、会話の確認依頼、`/operator` の引受・回答入力、元の会話への反映、終了・結果URLでの再読込を新規プロファイルで実操作し、撮り直した。公開画面のバナーは「共有用UIデモ」「会話・依頼はこのブラウザ内に保存」「AI音声・黒電話は未接続」と明記されていた。

HTTPはGET/HEAD/OPTIONS以外をブラウザで遮断し、架空の質問と「検証用の架空回答」だけを使用した。実API課金、第三者への送信、実電話発信は行っていない。現在のREADMEとプレゼンに使う `docs/screenshots/public-*.jpg` はこの撮影で取得した画像。`legacy-*` / `current-*` は前回のローカル試験の証跡として保持している。

撮影元：受付 `/`、会話 `/session/:id`、担当者 `/operator`、終了後 `/session/:id/result`。動的IDはブラウザ内のダミーセッション。公開UIの確認結果を、同梱ソースの実AI結合確認へ一般化しない。
