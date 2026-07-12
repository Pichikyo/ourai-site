---
name: ourai-site-run-and-deploy
description: ourai-site（往来の公式サイト、静的HTML+CSS、フレームワークなし）をローカルで確認する方法と、Cloudflare Pagesへデプロイする方法を知りたいとき、または「表示確認したい」「公開したい」「デプロイして」「本番に反映して」と言われたときに読み込む。特にデプロイ前は必ず読むこと（フォルダ内の全ファイルがそのまま公開されるため、内部資料の混入事故を防ぐチェックリストを含む）。
---

# ourai-site: 実行とデプロイ

対象: `/Users/pichikyo/dev/ourai-site/`（静的HTML17ページ + `style.css`。ビルドツールなし、フレームワークなし）

このスキルは「動かし方」と「公開の仕方」に特化した早見表です。デザインのルールは `design.md`、運用ルールは `CLAUDE.md` が正本です。**このスキルの内容と正本が矛盾する場合は正本が勝ちます。**

---

## 1. ローカルで確認する

このリポジトリはビルド不要な静的サイトなので、Pythonの簡易サーバーだけで確認できます。

```bash
cd /Users/pichikyo/dev/ourai-site
python3 -m http.server 8742
```

- ブラウザで `http://localhost:8742/` を開く（例: `http://localhost:8742/index.html`）
- ポート8742は `.claude/launch.json` に「ourai-site」という設定名で登録済み（実ファイルで確認済み: `runtimeExecutable: python3`, `runtimeArgs: ["-m","http.server","8742"]`, `port: 8742`）。Claude Codeの`preview_start`系ツールを使う場合はこの設定名を使えば同じコマンドが起動する。
- 停止は `Ctrl+C`

---

## 2. 本番デプロイは GitHub Actions（CI）経由が正（2026-07-12更新）

**`main`ブランチへの `git push` がそのまま本番デプロイになる。** `.github/workflows/deploy.yml` が疎通済み（2026-07-12昼確認・実装済み）で、push時に自動で `wrangler pages deploy` が実行される。手動で`wrangler`コマンドを叩く運用は**もう正ではない**（このスキルの旧版は手動wrangler前提で書かれていたが、その記述が古かった）。

```bash
cd /Users/pichikyo/dev/ourai-site
git add -A
git commit -m "..."
git push origin main   # ← これで本番デプロイが走る（CI経由）
```

| 項目 | 値 |
|---|---|
| デプロイ方式 | GitHub Actions（`.github/workflows/deploy.yml`）経由。`push`（main）で本番デプロイ、`pull_request`はプレビューURLをPRにコメント |
| デプロイ先 | Cloudflare Pages |
| プロジェクト名 | `ourai-site` |
| 本番URL | https://ourai-site.pages.dev （固定） |
| デプロイ対象ディレクトリ | `.`（リポジトリ直下、CI実行時も同様）＝**このフォルダの中身がそのまま丸ごとアップロードされる**（3章のチェックリストはCI経由でもそのまま有効） |
| CI内の`--commit-dirty=true` | CI実行環境上の未コミット差分（チェックアウト直後は無いはずだが）があってもデプロイを続行する指定 |

進捗確認: `gh run list --workflow=deploy.yml` / `gh run view <run-id>`（GitHub CLI）で見られる。

**例外（CIが落ちている時の緊急対応など）**に限り、従来どおり手動デプロイも使える。

```bash
cd /Users/pichikyo/dev/ourai-site
npx wrangler pages deploy . --project-name ourai-site --commit-dirty=true
```

カスタムドメインへの移行予定は**未確認**（CLAUDE.md・design.mdに記載なし）。当面は `ourai-site.pages.dev` が本番URL。

**案件間の注意**: この「push=デプロイ」運用は**往来（ourai-site）限定**。グループ会社トーモ（toomo-site）はGit連携なしの `deploy.sh` 手動実行が正（`~/dev/toomo-site/.claude/skills/toomo-site-run-and-deploy/SKILL.md`参照）。他案件の手順書を読むときは決め打ちしないこと。

---

## 3. 【最重要】このフォルダの全ファイルは公開される

`wrangler pages deploy .` は **カレントディレクトリ配下のファイルをほぼそのままアップロードする**。`.gitignore` に書いてあってもwranglerのアップロード対象からは除外されない（`.gitignore`はgit専用の除外設定であり、wranglerの公開範囲とは別物）。実際にこのリポジトリの `.gitignore` は以下の3行のみで、内部資料除外の効果は一切ない。

```
.DS_Store
Thumbs.db
*.log
```

**過去にこの認識違いで事故が起きている（CLAUDE.md記載）。台帳・計画書・レビューメモなどの内部資料をこのフォルダ直下やサブフォルダに置いた状態でデプロイすると、そのまま誰でも閲覧できるURLで世界に公開される。**

置いてよいもの（CLAUDE.mdより）:
- 公開ページのHTML/CSS/JS/画像
- `design.md`（デザイン正本、公開されても実害が薄い内部文書として許容されている）
- `CLAUDE.md`（同上）

置いてはいけないもの:
- 台帳・見積・契約書・レビューメモ・議事録などの内部資料全般
- `.env`（存在する場合、変更禁止・当然デプロイ物にも含めない）
- スクリーンショットや下書きなど公開想定でない一時ファイル

---

## 4. デプロイ前チェックリスト（毎回実施）

デプロイコマンドを打つ**前**に、必ず以下を実行してフォルダの中身を目視確認すること。

### 4-1. トップレベルの一覧を見る

```bash
cd /Users/pichikyo/dev/ourai-site
ls -la
```

- 見慣れないファイル・フォルダが増えていないか確認する
- 特に `.md` ファイルは `CLAUDE.md` と `design.md` の2つ以外に増えていたら要注意（内部メモが紛れ込んだ可能性）
- 台帳・見積・議事録らしきファイル名（例: `台帳`, `見積`, `メモ`, `_draft`, `_internal` を含むもの）がないか目で見る

### 4-2. git的に「見えていないファイル」がないか確認する

```bash
git status
```

- `Untracked files` に見慣れないファイルが出ていないか確認する（2026-07-07時点でも `design.md` や `budget.html` など正規の未コミットファイルが出る状態なので、「未追跡＝即NG」ではなく中身を見て判断すること）

### 4-3. 隠しフォルダの中身も確認する

```bash
find . -maxdepth 2 -name ".*" -not -name ".git" -not -name ".gitignore"
```

- 実行時点で `.claude/`, `.hallmark/`, `.wrangler/` が存在することを確認済み（2026-07-07）。これらはツール作業用のフォルダで、意図的に置いているものだが、この中に内部資料を保存しないこと。「隠しフォルダだから公開されない」は誤り。**wranglerは隠しフォルダも含めてアップロードしうる前提でチェックする。**

### 4-4. ファイル名で内部資料っぽいものを横断検索する

```bash
find . -iname "*台帳*" -o -iname "*見積*" -o -iname "*議事録*" -o -iname "*internal*" -o -iname "*draft*"
```

- 何も出てこなければOK。何か出てきたら、デプロイ対象外の場所（例: `~/Dropbox/_Cowork_Project/`）に退避してからデプロイする。

### 4-5. 未確認事項の状況が変わっていないか確認する

`ourai-site-quirks-and-debugging` の7〜8章（6章参照）に集約されている未確認事項（contact.htmlフォームの送信先運用・feature-dinosaur/picoの公開ステータス・budget.htmlの価格根拠など）は、公開してよいか判断が必要な内容を含む。デプロイ対象のページがこれらに関わる変更を含む場合、または前回確認時から状況が変わっていそうな場合は、デプロイを実行する前にともみさんに確認する。疑わしければ確認、確認が取れなければデプロイを止める。

### 4-6. 上記すべてクリアしたらデプロイ

```bash
npx wrangler pages deploy . --project-name ourai-site --commit-dirty=true
```

---

## 5. デプロイ後の確認

```bash
open https://ourai-site.pages.dev/
```

- トップページが最新の変更を反映しているか目視確認する
- 変更したページ（例: `budget.html` を触ったなら `https://ourai-site.pages.dev/budget.html`）も個別に開いて確認する

---

## 6. 要確認

contact.htmlフォーム送信先の運用状況・feature-dinosaur/picoの公開ステータス・アナリティクス導入・カスタムドメイン移行・budget.htmlの価格根拠など、契約外の未確認事項は `ourai-site-quirks-and-debugging` の7〜8章に集約されている。デプロイ前の判断が必要な場合はそちらを参照すること（4章の4-5も参照）。

---

## このスキルを使わない場面

- デザインの色・フォント・余白・改行ルールを確認したい → `ourai-site-design-contract` を読む（正本は `design.md`）
- 実績の掲載ルール・コンテンツの書き方・文言のトンマナを知りたい → `ourai-site-content-playbook` を読む
- 変更してよいもの／ともみさんの明示指示が必要なもの（KVスライダー・ハンバーガー等）の判断 → `ourai-site-change-control` を読む
- 実装上の癖・既知の詰まりどころ・デバッグ手順を知りたい → `ourai-site-quirks-and-debugging` を読む

---

## 出所とメンテナンス

このスキルの記載内容が古くなっていないか、以下のワンライナーで再検証できる。

```bash
# CLAUDE.mdの実行/デプロイ節が変わっていないか
grep -n "起動/実行方法\|デプロイ\|npx wrangler" /Users/pichikyo/dev/ourai-site/CLAUDE.md

# launch.jsonのポート・コマンドが変わっていないか
cat /Users/pichikyo/dev/ourai-site/.claude/launch.json

# .gitignoreの除外内容（wranglerの公開範囲とは別物である点に変化がないか）
cat /Users/pichikyo/dev/ourai-site/.gitignore

# 隠しフォルダ構成の変化を確認
find /Users/pichikyo/dev/ourai-site -maxdepth 2 -name ".*"

# contact.htmlの送信先が変わっていないか
grep -n "<form" /Users/pichikyo/dev/ourai-site/contact.html

# feature-dinosaur/picoが今もナビ・一覧からリンクされているか
grep -rn "feature-dinosaur\|feature-pico" /Users/pichikyo/dev/ourai-site/*.html

# アナリティクス導入の有無を再チェック
grep -rli "gtag\|analytics\|plausible\|clarity" /Users/pichikyo/dev/ourai-site/*.html

# budget.htmlの価格表記が変わっていないか
grep -n "円\|万" /Users/pichikyo/dev/ourai-site/budget.html | head -10
```
