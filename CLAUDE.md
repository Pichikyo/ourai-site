# ourai-site

XR・AI・リアル空間を横断する体験設計スタジオ「往来（OURAI）」のコーポレートサイト。

## 技術スタック
- 静的HTML + CSS + バニラJS（フレームワーク・ビルドなし）
- デプロイ先: Cloudflare Pages（確定）

## 起動/実行方法
- ローカル確認: `.claude/launch.json` に「ourai-site」登録済み（python3 http.server・ポート8742）
- デプロイ: `npx wrangler pages deploy . --project-name ourai-site --commit-dirty=true`
- 本番URL: https://ourai-site.pages.dev （固定・常に最新）

## 重要ファイル
- **design.md がデザイン方針の正本。変更前に必ず読むこと**（フォント・色・字間・改行ルール・全確定値）
- style.css に全ページ共通スタイルを集約（デザイントークンは :root）
- llms.txt / company-record.html: AIクローラー向け。ナビからリンクしない独立運用

## ⚠️ 最重要の運用ルール
- **このフォルダの全ファイルはデプロイで公開される。** 台帳・計画書・レビュー等の内部資料は絶対にここに置かない（過去に事故あり）。ここに置いてよいのは公開物と design.md / CLAUDE.md のみ
- 実績の数字は独自に書き換えない。数字には出典または「自社調べ（時点）」表記が必要
- 掲載してよい案件・数字のルールは社内台帳が正本（場所はメモリ参照）。**サイトに載っていない案件名を新たに載せない**

## デザイン上の禁止事項（design.md より抜粋）
- 紫は絶対NG／グラデーションNG／ホバーリフトNG
- 見出し・リードの泣き別れ禁止（.wb 文節区切りを崩さない）
- 手動改行は br-pc クラス（PC専用）。素の <br> を本文に足さない

## コミット規約
- Conventional Commits（feat:/fix:/refactor:等）を継続

## 触ってはいけないもの
- .env（存在する場合、中身は確認不要・変更禁止）
- FEATURED実績（index.htmlのworksData 3件／works.html（旧works.html・2026/07/11改名）の特集カード8枚、2026-07-07実測）・KVスライダー・ハンバーガーの構造（変更時はともみさんの明示指示があるときのみ）
