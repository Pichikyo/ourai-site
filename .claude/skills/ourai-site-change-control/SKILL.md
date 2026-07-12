---
name: ourai-site-change-control
description: ourai-site（往来コーポレートサイト）で「何かを変更・修正・追加する」タスクに着手する前に必ず読み込む。特に.envやKVスライダー・FEATURED一覧・ハンバーガーメニューなど構造物に触れる時、コミットメッセージを書く時、design.md/CLAUDE.mdの内容を確認せずに実装しようとしている時、過去に消したはずの要素（フッター住所・SERVICE画像等）を復活させそうな時に参照するガードレール集。
---

# ourai-site 変更管理ガードレール

このスキルは「何をどう作るか」ではなく「変更していいか・どう記録するか」を扱う。
デザインの数値・トークンは design.md、実行/デプロイ手順は起動・デプロイ系スキル、コンテンツ運用（実績追加・文言）はコンテンツ系スキルを見る。ここは**変更管理**専用。

**矛盾したら正本が勝つ。** design.md（デザイン）と CLAUDE.md（運用ルール）が一次情報源。このSKILL.mdはそこへの案内・早見表であり、数値やルールを再定義する第二の正本ではない。本ファイルと正本が食い違っていたら、正本を信じて、このファイルの記述が古いと判断すること。

## 0. 最初にやること（毎回・省略禁止）

作業を始める前に、この2ファイルを必ず読む。

```bash
cat /Users/pichikyo/dev/ourai-site/CLAUDE.md
cat /Users/pichikyo/dev/ourai-site/design.md
```

読まずに style.css の色・余白・フォントサイズを変更したり、フッター構造やナビ構造を書き換えたりしない。design.md は「ページごとに別テーマを作らない」ことを前提にした唯一のデザイン正本であり、CLAUDE.md は運用ルールの正本。

## 1. 触ってはいけないもの（ともみさんの明示指示がない限り変更禁止）

CLAUDE.md（2026-07-07時点の内容）に明記されている絶対ルール。

| 対象 | 実体（実リポジトリで確認済み） | 制約 |
|---|---|---|
| `.env` | 2026-07-07時点でリポジトリ内に**実在しない**（`.gitignore` にも記載なし）。今後作られても中身は確認不要・変更禁止 | 存在有無に関わらず触らない |
| KVスライダー | `index.html` のヒーロー内 `#kvTrack`（4枚のスライド、5秒自動回転、`#kvNext`/`#kvPrev`/`#kvDots`、タッチスワイプ、`prefers-reduced-motion`対応。JS本体は同ファイル内 `<script>` 内、`/* ── KVスライダー ── */` コメント直下） | 構造変更はともみさんの明示指示必須 |
| FEATURED（実績の特集扱い） | **要注意：呼び名が2箇所にまたがり枚数が違う。** ①`index.html` の「FEATURED WORKS」セクション（`#works-grid`、JS内 `worksData` 配列で描画、2026-07-07時点で3件）。②`works.html（旧works.html・2026/07/11改名）` の「FEATURED PROJECTS」セクション（`.feat-card` を8枚ハードコード：yokosuka/nissan-beams/mos/kyocera/futureshop/pico/kek/dinosaur）。CLAUDE.mdの旧記述「FEATURED 6枚」は実測と不一致だったため、2026-07-07にオーナー確認のうえCLAUDE.mdを実測値（①3件／②8枚）に更新済み（解決済み）。 | どちらのFEATUREDも構造・枚数変更はともみさんの明示指示必須 |
| ハンバーガー構造 | `#menuBtn`（`index.html` 内 `.hamburger`）と `#mobileNav`（`.mobile-nav`）。768px以下で表示（`style.css` 内 `@media (max-width: 768px)` と `900px` の2箇所に `.hamburger { display: flex; }` 記載あり）。開閉JSは `index.html` 末尾の `<script>` 内 | 構造変更はともみさんの明示指示必須 |
| 実績の数字 | 独自に書き換えない。数字には出典または「自社調べ（時点）」表記が必須（例: index.htmlの`※DL数・来場者数は自社調べ（2026年時点）`） | 掲載可否は社内台帳が正本。**サイトに載っていない案件名を新たに載せない** |

補足（実確認事項）:
- KVスライダーは2026-07-07時点で**4枚**（yokosuka / nissan-beams / kyocera / mos）。ブリーフや古いメモで枚数が違う場合は実ファイルを数え直すこと。
- worksData配列は「新案件を足す場所」であって「触ってはいけない場所」ではない。追加は可、既存の**構造**（配列の描画ロジック・カード生成JS）を変えるのが禁止対象。

## 2. 変更前チェックリスト

- [ ] `CLAUDE.md` を読んだ（触ってはいけないもの・運用ルールの再確認）
- [ ] `design.md` を読んだ（色・フォント・余白・改行ルールの再確認）
- [ ] 触ろうとしている箇所が上記「触ってはいけないもの」表に載っていないか確認した
- [ ] 載っている場合、ともみさんから明示の変更指示が出ているか確認した（出ていなければ提案に留め、実装しない）
- [ ] 実績数字を扱う場合、出典 or 「自社調べ（時点）」表記があるか確認した
- [ ] 新しい案件名を載せようとしていないか（社内台帳との突合が必要）確認した
- [ ] デプロイ対象フォルダ直下に内部資料（台帳・計画書・レビューメモ等）を置いていないか確認した（過去に事故あり。フォルダ内は全ファイルが公開される）

## 3. コミット規約

CLAUDE.md記載のルール：**Conventional Commits（`feat:` / `fix:` / `refactor:` 等）+ 日本語の説明文**を継続する。

実際の運用（git履歴で確認、2026-06-11〜06-23の12コミット分）:

```
feat(top): VIRTUAL FAShION セクション追加（SERVICE〜WORKS間）
fix(footer): 全ページから住所・代表者名を削除
refactor(service): 画像削除・サービス名大型化・課題/できること視認性強化
refactor(site): 全ページをトップページ方針に統一
```

観測できたパターン:
- 形式: `<type>(<scope>): <日本語の要約>`
- type は `feat` / `fix` / `refactor` の3種のみ確認できた（`docs`/`chore`等の使用例は未確認）
- scope はページ名や領域を英語小文字で（`top` / `service` / `about` / `footer` / `site` など）
- 本文（要約行の下）に変更理由・変更内容の補足を日本語で書いた例あり（例: 34d3207）
- Co-Authored-By 行が入っている例が複数あり（Claude Sonnet 4.6 名義）。AIが手を入れた変更である旨を記録する運用と読める

**注意（要確認）**: 2026-07-07時点、`git log` に載っている最新コミットは `2026-06-23` の `feat(top+service): 複数セクション改善` まで。それ以降（design.md・CLAUDE.md・budget.html・company-record.html・llms.txt・feature-dinosaur.html・feature-kek.html の新規追加、および既存11ファイルの変更）はすべて**未コミット**（`git status` で `M` または `??`）。つまり**git履歴は現在の作業ディレクトリの状態を反映していない**。「git履歴を見れば今のサイトの状態が分かる」という前提を置かないこと。コミットするかどうか・タイミングはともみさんの判断に委ねる（このスキル自身もgitの変更コマンドは一切実行しない）。

## 4. git履歴から見る確定済みの意思決定（「戻すな」リスト）

過去のコミットで**意図して削除・撤廃した**要素。同じ理由が再発するので、リデザインや復元作業で無意識に戻さないこと。

| 削除・撤廃されたもの | 該当コミット | 戻すな、な理由 |
|---|---|---|
| フッターの住所・代表者名（「株式会社 往来　代表取締役社長：東 智美」「〒107-0052 東京都港区赤坂9-6-30 ルイマーブル乃木坂408」） | `59975e1` fix(footer): 全ページから住所・代表者名を削除（12ファイル対象） | 明示的にフッターから除去する意思決定。今後フッターに社名情報を足す場合は住所・代表者名を含めない設計にすること |
| SERVICE（service.html）の横長イメージ画像 | `34d3207` refactor(service): 画像削除・サービス名大型化・課題/できること視認性強要化 | design.md の Per-page allowances でも「About / Contact / News: タイポグラフィのみ」と明記。service.htmlも画像を削除しテキスト訴求に寄せた経緯があるため、安易に画像を足し戻さない |
| セリフ体フォント（Instrument Serif / Noto Serif JP） | design.md 内に明記（「セリフ（Instrument Serif / Noto Serif JP）は廃止」）。個別のfont変更コミットはgit履歴上に単独では見えないが、design.mdが確定事項として記録 | 現行フォントは Zen Kaku Gothic New（見出し）+ Noto Sans JP（本文）の2種構成で確定 |
| Poppins フォント | design.md に明記（「Poppins も廃止」） | 同上 |
| グラデーション / ホバーリフト / 影 | design.md・CLAUDE.md両方に「絶対NG」と明記（複数箇所で重複記載されるほど強く意識されているルール） | WhO(whohw.jp)実測値に忠実な無彩色エディトリアル路線からの逸脱になるため |

## 5. このスキルを使わない場面

- デザインの色・フォント・余白の具体的な数値を知りたい → `ourai-site-design-contract` を読む
- ローカル起動・Cloudflare Pagesへのデプロイ手順を知りたい → `ourai-site-run-and-deploy` を読む
- 実績（feature-*ページ）の追加・文言の書き方・掲載ルールの実務を知りたい → `ourai-site-content-playbook` を読む
- 実装の癖・既知の落とし穴・デバッグ手順を知りたい → `ourai-site-quirks-and-debugging` を読む

このスキルは「触っていいか／どう記録するか」の判断にのみ使う。

## 6. 要確認（オーナー未確認・裏取りできなかった事項）

以下はオーナー（ともみさん）に確認できておらず、実リポジトリの調査だけでは断定できなかった。断定的に扱わないこと。

- **contact.htmlフォームの送信先**: `<form ... action="https://ssgform.com/s/MrgYkzC4z61d" method="POST">` を確認した（ssgform.comという外部フォーム送信サービスへPOSTしている構成）。ただしこのフォームサービスの契約状況・受信先メールアドレス・運用体制は未確認
- **feature-dinosaur / feature-picoの公開ステータス**: `works.html` 上のバッジ表示は確認できた。feature-dinosaur.htmlは`公開準備中`バッジ（2026年・準備中の案件として明示）、feature-pico.htmlは他の実績と同じ`特集`バッジ（公開済み扱いに見える）。ただし「サイトに実際に公開してよいか」の最終判断は社内台帳・オーナー確認が必要
- **アナリティクス導入有無**: 全HTMLファイルを `gtag` / `G-` / `googletagmanager` / `analytics` 等のキーワードで検索したが該当なし。2026-07-07時点で**未導入と見られる**（ただし将来的な導入予定の有無は未確認）
- **カスタムドメイン移行予定**: リポジトリ内に記述なし。CLAUDE.mdは本番URLを `https://ourai-site.pages.dev` と明記するのみで、独自ドメインへの移行計画は確認できなかった
- **budget.htmlの価格根拠**: ファイル自体は存在し閲覧できるが、記載されている金額の算出根拠・承認プロセスはリポジトリ内に記述がなく未確認

## 出所とメンテナンス

このSKILL.mdの内容は2026-07-07時点のリポジトリ状態を実際に読んで確認した。揮発しやすい数値（KVスライダー枚数・FEATURED枚数・最新コミット日付等）は以下のワンライナーで再検証すること。

```bash
# 正本2ファイルの内容を確認
cat /Users/pichikyo/dev/ourai-site/CLAUDE.md
cat /Users/pichikyo/dev/ourai-site/design.md

# .envの実在確認
test -f /Users/pichikyo/dev/ourai-site/.env && echo "EXISTS" || echo "NOT_EXISTS"

# KVスライダーの現在の枚数
grep -c 'kv-slide' /Users/pichikyo/dev/ourai-site/index.html

# index.html側FEATURED WORKS（worksData）の件数
grep -c "client:" /Users/pichikyo/dev/ourai-site/index.html

# works.html側FEATURED PROJECTSの件数とバッジ内訳
grep -n 'feat-badge' /Users/pichikyo/dev/ourai-site/works.html

# git履歴とコミット規約の実例（最新から遡って確認）
cd /Users/pichikyo/dev/ourai-site && git log --oneline --all

# 作業ディレクトリがgit履歴に対して未コミット変更を抱えていないか
cd /Users/pichikyo/dev/ourai-site && git status --short

# アナリティクス導入有無の再確認
grep -rl "gtag\|googletagmanager\|G-[A-Z0-9]" /Users/pichikyo/dev/ourai-site/*.html

# contact.htmlの送信先確認
grep -n "action=" /Users/pichikyo/dev/ourai-site/contact.html
```

## 直push許可の例外（2026-07-12 智美決定・A案）

**智美がチャットで内容を一つずつ指示・確認した変更の反映**（例: 実績台帳→works.mdの同期）は、PR（承認箱）を通さず main へ直push＋デプロイしてよい。会話そのものが承認にあたるため。
それ以外の変更（デザイン・構成・新ページ・文言の独自判断を含むもの）は従来どおりPR経由で智美の承認を得る。
