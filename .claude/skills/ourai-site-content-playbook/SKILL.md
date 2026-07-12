---
name: ourai-site-content-playbook
description: ourai-site（往来サイト）で実績カード・特集ページ・お知らせ・活動記録年表を追加/更新するときに読み込む。「実績を追加して」「新しい案件を載せて」「news.htmlに追記して」「company-record.htmlを更新して」「OGP画像を設定して」「画像ファイル名どうする」といった依頼が来たら必ずこのスキルを開き、worksData配列の直編集・feature-*.htmlの雛形手順・works.html（旧works.html・2026/07/11改名）への反映・出典記載ルールをこの順で確認してから作業する。
---

# ourai-site コンテンツ更新プレイブック

このスキルは「ourai-site（往来のコーポレートサイト）に新しい実績・お知らせを載せる」作業の手順書です。デザインのルール（色・フォント・余白など）はこのスキルの管轄外です。デザインに触るときは兄弟スキル **ourai-site-design-contract** を読んでください。矛盾があれば常に正本（`design.md` / `CLAUDE.md`）が勝ちます。このスキルは正本への案内と作業手順の早見表に徹します。

対象リポジトリ: `/Users/pichikyo/dev/ourai-site/`（2026-07-07時点で静的HTML17ページ + `style.css`、ビルドなし）

---

## 0. 着手前に必ず確認する絶対ルール

- **実績数字は社内台帳が正本。サイトに載っていない案件名を新たに載せない。**（`CLAUDE.md` 記載）
  - このスキルは「サイトに既にある実績の追加・更新手順」だけを扱います。「載せてよい案件かどうか」の判断はできません。新規案件を載せたいときは、まず社内台帳（場所はオーナーのメモリ参照）で掲載可否を確認し、ここでは扱わないでください。
- **実績の数字を独自に書き換えない。数字には出典または「自社調べ（時点）」表記が必須。**（`CLAUDE.md` 記載）
- **このフォルダ内の全ファイルはデプロイで公開される。** 台帳・下書き・レビューメモなど内部資料は絶対にこのリポジトリに置かない（過去に事故あり、`CLAUDE.md` 記載）。
- FEATURED（枚数はCLAUDE.mdに実測値を反映済み・2026-07-07。実数は`ourai-site-change-control`の1章を参照）・KVスライダー・ハンバーガー構造は「触ってはいけないもの」指定。構造自体を変える場合はオーナーの明示指示が必須です（`CLAUDE.md` 記載）。ただし本スキルが扱う「featuredカードの中身の差し替え・新規追加」はこの禁止事項の対象外です（KVスライダーとworksDataは別物。詳細は1章参照）。

---

## 1. 実績を追加する全手順

サイト上の「実績」は3箇所に別々のデータとして存在します。3箇所すべてを揃えて更新しないと表示が食い違います。

| 箇所 | ファイル・場所 | 役割 |
|---|---|---|
| ① トップのFEATUREDカード | `index.html` の `worksData` 配列（341〜366行目、2026-07-07実測） | トップページのおすすめ3件を出し分ける |
| ② 特集ページ | `feature-[案件名].html`（新規ファイル） | 1案件を深掘りする専用ページ |
| ③ 実績一覧ページ | `works.html` の FEATURED カード（76〜166行目付近）と ALL PROJECTS カード（180行目以降） | 実績を網羅する一覧・フィルタ |

### 手順0（ブロッキング）：社内台帳で掲載可否を確認する

新規案件名を載せる場合、手順Aに進む前に必ず社内台帳（場所が特定できない場合はともみさんに直接確認する）を開き、掲載可否をYES/NOで確認すること。確認が取れていない・取れないうちは手順A以降に着手しない。既存案件の内容更新（数字の差し替え・出典追記など）のみで新規案件名を追加しない場合はこの手順は不要。

### 手順A：index.htmlのworksData配列を直編集する（トップページのFEATURED3件を変える場合のみ）

`index.html:341-366` に以下の形でオブジェクトが3件並んでいます（2026-07-07実測、KVスライダー本体・#kvTrack等のIDとは別物なので触っても「触ってはいけないもの」には抵触しません）。

```js
const worksData = [
  {
    client:      '横須賀市',
    title:       'メタバースヨコスカ',
    description: '地域の魅力をVRChat上で体験化した観光プロモーション。アバター用スカジャンは15万ダウンロードを記録。',
    tags:        ['VRChat', '自治体', '観光PR'],
    image:       'assets/images/slide/pf_yokosuka_2023_10_01.jpg',
    link:        'feature-yokosuka.html',
  },
  // ...
];
```

- `image` を空文字 `''` にすると「CASE IMAGE」というプレースホルダー表示になります（実装は`index.html`内の該当script参照）。画像がまだ無い案件を先出しする場合はこれを使えます。
- ここは3件固定の配列で、直書き換えです。CMSやJSONファイルはありません。
- **このコメント直上に「実績データ（追加・並び替えはここを編集）」という注記が本文にあります。それ以外の場所（KVスライダーの`#kvTrack`など）は触らない。**

### 手順B：feature-[案件名].htmlを雛形から作る

既存の特集ページ（例: `feature-yokosuka.html`、191行、2026-07-07実測）が最も忠実な雛形です。新規案件は既存ファイルをコピーして中身を差し替えるのが最も事故が少ないやり方です。

雛形の構成（上から順）:

1. `<head>` — title・description・OGP（og:title/description/url/image）・favicon・`style.css`読み込み・ページ固有の`<style>`（`.feat-hero`など）
2. `<header>` — 全ページ共通ナビ（他ページからそのままコピーする。手で書き直さない）
3. `.feat-hero` セクション — パンくず（`.crumb`）、`.platform`（プラットフォーム/カテゴリ表記）、`h1.page-title`、タグ一覧（`.tags`内に`.tag.cat`+複数`.tag`）、`.period`（プロジェクト期間バッジ）
4. 本文セクション（`.wrap.blk` / `max-width:880px`）
   - `.hero-img` — メイン画像
   - リード文（1段落）
   - `.stats` — 数字を強調するブロック（`<b>数値</b><span>説明</span>`のペア）
   - **数字の直後に出典注記の`<p>`を置く**（例: `※来場者数は横須賀市公式発表（2025年10月）、DL数は自社調べ（2026年時点）`）— これが実績数字ルールの実装形です
   - `<h3>課題｜...` `<h3>往来のソリューション｜...` `<h3>プロジェクトの歩み` `<h3>成果｜...` の4見出し＋`<ul><li>`
   - `.meta-table` — Client / プロジェクト期間 / プラットフォーム / URL / 参考リリース（外部リンクは`target="_blank" rel="noopener"`必須）
5. 「他の特集プロジェクト」セクション（`.cards`、3件ピックアップ）
6. `.cta-foot` 共通CTA
7. `<footer>` 共通フッター
8. ハンバーガーメニューの`<script>`（他ページと同一、コピーでよい）

作業手順:

```bash
cd /Users/pichikyo/dev/ourai-site
cp feature-yokosuka.html feature-新案件名.html
```

1. `<title>` `<meta name="description">` `og:title` `og:description` `og:url` `og:image` をすべて新案件用に書き換える（OGP絶対URL規約は2章参照）
2. `.feat-hero` 内のパンくず・platform・h1・tags・period を差し替える
3. 本文の課題／ソリューション／歩み／成果を書く。数字を入れたら必ず出典の`<p>`を添える
4. `.meta-table` を埋める
5. 「他の特集プロジェクト」の3件は既存の別案件へのリンクのままでよい（新設した案件が育ったら他ページからもここへ相互リンクを検討）
6. ナビの `<a class="active" ...>` は特集ページには付いていない（`works.html`にのみ`active`が付く。特集ページはナビ全項目が非アクティブ、2026-07-07実測）

### 手順C：works.htmlに追加する

`works.html` には実績が2段構成で並んでいます（2026-07-07実測）。

**① FEATUREDカード**（`.feat-list`内、76行目付近〜）— 特集ページを作った案件はここにもカードを追加します。

```html
<a class="feat-card" href="feature-新案件名.html">
  <span class="feat-badge">特集</span>
  <div class="thumb"><img src="assets/images/pf_xxx.jpg" alt=""></div>
  <div class="body">
    <div class="platform">VRChat｜カテゴリ</div>
    <h3>キャッチコピー<br class="br-pc">案件名</h3>
    <div class="count">期間 <span class="period">件数など</span></div>
    <div class="more-link">特集を読む →</div>
  </div>
</a>
```

- 画像がまだ無い場合は `<div class="thumb">CASE IMAGE</div>` または `IMAGE：案件名` という文字プレースホルダーで代用できます（`feature-kek.html`・`feature-pico.html`が実例、2026-07-07実測）。
- 未公開・準備中の案件は `<span class="feat-badge">公開準備中</span>` バッジと `<div class="thumb">COMING SOON</div>` を使います（`feature-dinosaur.html`が実例）。

**② ALL PROJECTSカード**（`.cards`内、180行目以降）— 単発の活動・イベントもすべてここに1行ずつ追加します。

```html
<a class="card" href="feature-新案件名.html" data-cat="world">
  <div class="thumb">CASE IMAGE</div>
  <div class="body"><div class="platform">クライアント名</div>
  <h3>「案件名」の説明</h3>
  <div class="tags"><span class="tag cat">ワールド制作</span></div>
  <div class="count">2026.07</div></div>
</a>
```

- 特集ページへのリンクを持たない単発活動は `<a>` の代わりに `<div class="card" data-cat="...">` を使います（リンクなしカード。2026-07-07実測で多数実例あり）。
- `data-cat` に入れられる値（2026-07-07時点で実際に使われているもの）: `world`（ワールド制作） `3d`（3D制作） `avatar`（アバター・衣装制作） `event`（イベント） `pr`（PR・プロモーション） `media`（執筆・出版） `industry`（業界活動） `gov`（自治体・観光） `edu`（教育・研究）。スペース区切りで複数指定可（例: `data-cat="world edu"`）。
- フィルターボタン（`#worksFilter`内）とカードの絞り込みJS（`works.html`末尾の`<script>`、530行目付近）は`data-cat`の値と連動しています。新しいカテゴリを増やす場合はフィルターボタン側にも追加が必要（このスキルの対象外の変更なので、design.mdの構造ルールに沿ってオーナー判断を仰ぐ）。
- `#worksCount` はJSで自動集計される想定（実装は`works.html`末尾script）。手で数字を書く必要はありません。

### チェックリスト（実績追加）

- [ ] **【ブロッキング】社内台帳で掲載可否を確認した**（新規案件名を載せる場合は必須。確認できるまで手順Aへ進まない。このスキルでは可否そのものは判断しない）
- [ ] `index.html` の`worksData`（トップ3件を変える場合のみ）
- [ ] `feature-[案件名].html` を作った（OGP絶対URL・出典注記込み）
- [ ] `works.html` の FEATURED カードに追加した（特集ページがある場合）
- [ ] `works.html` の ALL PROJECTS カードに追加した（`data-cat`を正しく設定）
- [ ] 実績数字すべてに出典または「自社調べ（時点）」を明記した
- [ ] 画像ファイルを命名規則に沿って`assets/images/`配下に置いた（2章参照）
- [ ] `llms.txt` / `company-record.html` を更新した（3章参照、必要な場合）

---

## 2. 画像命名規則とOGP設定

### 画像ファイル名

2026-07-07時点で`assets/images/`配下を実測すると、命名は完全に統一されてはいません（`p_img4.jpg`、`pf_05_AvatarWork01.jpg`、`pf_kyocera_fineceramics_202408_01.jpg`など旧命名が混在）。ただし直近の案件（横須賀・日産BEAMS・京セラ・フューチャーショップ）は概ね次の形に収束しています。

```
pf_[クライアント名]_[年月または連番]_[連番].jpg
```

実例: `pf_yokosuka_2023_10_01.jpg`、`pf_nissan_heritageworld_202403_01.jpg`、`pf_kyocera_fineceramics_202408_01.jpg`

新規追加時はこの直近パターン（`pf_[クライアント]_[番号や年月]`）に合わせてください。KVスライダー専用の画像は`assets/images/slide/`配下に置く運用です（2026-07-07実測でスライド用3枚が`slide/`配下に確認できる）。

- assets合計サイズは2026-07-07時点で約2.4MB。画像を追加するたびに肥大化するので、書き出し時に圧縮を意識してください（具体的な圧縮基準はdesign.mdに規定なし、要確認）。

### OGP設定（絶対URL規約）

全ページで `og:url` と `og:image` は **相対パスではなく絶対URL** で書かれています（2026-07-07実測、全ページ共通）。

```html
<meta property="og:url" content="https://ourai-site.pages.dev/feature-新案件名.html">
<meta property="og:image" content="https://ourai-site.pages.dev/assets/images/pf_xxx.jpg">
```

- ドメインは `https://ourai-site.pages.dev` 固定（カスタムドメイン移行の予定は要確認、決まれば全ページ一括で書き換えが必要になる）。
- 新規ページを追加する際、`og:image` に専用画像がまだ無ければ、他ページと同様に `assets/images/slide/pf_yokosuka_2023_10_01.jpg`（サイト全体の既定OGP画像、2026-07-07実測で`index.html`/`news.html`/`works.html`/`company-record.html`など多数のページがこの1枚を共用）を暫定で使う手もあります。ただし本来は案件ごとの専用画像が望ましいです。
- `<title>` と `og:title` は同一文言にする（全ページで一致、2026-07-07実測）。

---

## 3. お知らせを追加する（ニュース台帳経由・2026/07/12から台帳駆動化）

**news.htmlを直接編集してはいけない。** ニュースの正本は `~/inhouse-bot/ourai-news-ledger.json`（N番号つき台帳）で、news.htmlの一覧・index.htmlの最新3件・works.md末尾のお知らせ節は、すべて `~/inhouse-bot/ourai_sync.py` が `<!-- NEWS-SYNC:START/END -->` マーカー間に生成する。手で書いた変更は次のsyncで消える。

**追加手順**:
1. 台帳の `news` 配列末尾に1件追加。`no` は既存最大の次（N036〜）、既存noの再利用禁止
   ```json
   {"no": "N036", "date": "2026.07", "category": "お知らせ", "text": "本文", "link": "feature-xxx.html", "show": true}
   ```
2. `cd ~/inhouse-bot && python3 ourai_sync.py`（news.html/index.html/works.mdの3面が自動更新）
3. `python3 generate_html.py`（社内閲覧ページ https://inhouse-tasks.pages.dev/ourai-news へ台帳を同梱デプロイ）
4. ourai-siteを `git commit → push`（CIで本番デプロイ）

**フィールドの規律**:
- `date`: `YYYY.MM` 基本、年のみ `YYYY` 可、年月不明は `date:""`＋`date_label:"—"`。形式外はsyncが警告
- `category`: `お知らせ/業界活動/イベント/3D制作/ワールド制作/執筆・登壇/その他` の7種に固定（語彙外はsyncが警告）
- `link`: 省略可。相対（feature-*.html等）またはhttp(s)絶対URLのみ（他はsyncが拒否してリンクなし化）
- `show: false` で公開3面から非表示（台帳からは消さない）。出典表記は必須ではない（company-record.htmlとの違い）
- 並びはsyncが日付降順に自動整列するので、台帳内の位置は末尾追加でよい

---

## 4. llms.txt と company-record.html の更新規律

この2つはAIクローラー向けの独立運用ページで、ナビからリンクされていません（2026-07-07実測）。**出典付き数字が義務**という点で`news.html`より厳格です。

### company-record.html — 出典必須の一次情報源

年表項目（`.news-item`）に加えて実績数字の集計テーブルがあります（74行目付近、2026-07-07実測）。

```html
<tr><th>累計ワールド制作数</th><td>20ワールド（自社調べ・2026年時点）</td></tr>
<tr><th>メタバースヨコスカ 累計来場</th><td>20万人突破（2025年10月・横須賀市公式発表）</td></tr>
```

年表側も出典を本文中に埋め込む書式です。

```html
<div class="news-item"><time>2025.10</time><span class="chip">イベント</span>
  <a href="feature-yokosuka.html">メタバースヨコスカ 2周年 — 累計来場20万人・スカジャン10万DL突破。
  記念イベントの企画・運用を担当（出典: 市公式・PR TIMES 000000006.000147904）</a>
</div>
```

**このページに数字を足す・変えるときの規律（2026-07-07実測の実装パターンから抽出）:**

- すべての数字末尾に `（出典: 発表元 リリース番号）` または `（自社調べ・[年]年時点）` を必ず添える
- 出典が自治体・企業の公式発表なら発表元名＋可能ならリリース番号/URLを本文またはPR TIMES番号で特定する
- 出典が往来自身の集計なら「自社調べ」＋集計時点の年を明記する
- news.htmlとの重複はOK（news.htmlは軽量版、company-record.htmlは出典完備の正式版という役割分担）

### llms.txt — 会社情報のプレーンテキスト要約

`llms.txt`（2026-07-07時点で1ファイル、Markdown風プレーンテキスト）には「主要実績（出典付き）」という節があり、company-record.htmlの数字を要約する形で載っています。

```
## 主要実績（出典付き）
- 横須賀市「メタバースヨコスカ」: 2ワールド累計来場20万人突破・アバタースカジャン10万DL
  （2025年10月時点・横須賀市公式発表）。最新の自社集計ではスカジャン15万DL（2026年時点）
```

- company-record.htmlの数字を変更・追加したら、`llms.txt`の該当行も同じ出典表記で同期させること（2ファイルの数字がズレると「AIクローラーに矛盾した実績を見せる」ことになるため）。
- `llms.txt`の「主要ページ」節にURL一覧があるので、新規ページ（新しいfeature-*.htmlなど）を作った場合はここにも追記を検討する（2026-07-07時点でfeature-*個別ページへのリンクはllms.txtには無く、works.html等の一覧ページのみがリンクされている。個別追加するかは要確認＝オーナー判断）。

---

## このスキルを使わない場面

- **色・フォント・余白・改行ルールなど見た目のデザインを変えたい** → `ourai-site-design-contract` を読む（design.mdの実測値がここに整理されている）
- **ローカル起動・デプロイ・Cloudflare Pages操作をしたい** → `ourai-site-run-and-deploy` を読む
- **原因不明の表示崩れ・JSエラー・既知のクセにハマった** → `ourai-site-quirks-and-debugging` を読む
- **変更してよいか判断に迷う・承認フローを確認したい** → `ourai-site-change-control` を読む
- **worksData配列やworks.htmlの構造自体（HTML/CSSの土台）を変えたい**（中身の差し替えではなく仕組みの変更） → まずオーナーの明示指示を得て、`ourai-site-design-contract`と合わせて確認する

---

## 出所とメンテナンス

このスキルの記述はすべて2026-07-07時点の実リポジトリ実測に基づきます。運用が変わったら以下のワンライナーで再検証してください。

```bash
# worksData配列の行番号がずれていないか確認
grep -n "worksData\s*=" -A2 /Users/pichikyo/dev/ourai-site/index.html

# works.htmlのdata-cat一覧（実際に使われている値）を再抽出
grep -oE 'data-cat="[^"]*"' /Users/pichikyo/dev/ourai-site/works.html | sort -u

# 全ページのOGP設定が絶対URLになっているか一括チェック
grep -n 'og:url\|og:image' /Users/pichikyo/dev/ourai-site/*.html

# 画像ファイルの命名パターンと総容量を確認
ls /Users/pichikyo/dev/ourai-site/assets/images/ && du -sh /Users/pichikyo/dev/ourai-site/assets/images

# company-record.html / llms.txt の出典表記に漏れがないか目視確認
grep -n "出典\|自社調べ" /Users/pichikyo/dev/ourai-site/company-record.html /Users/pichikyo/dev/ourai-site/llms.txt

# news.htmlの最新項目（先頭）を確認
sed -n '55,65p' /Users/pichikyo/dev/ourai-site/news.html

# CLAUDE.md本体の「触ってはいけないもの」「実績数字」ルールが変わっていないか確認
sed -n '1,40p' /Users/pichikyo/dev/ourai-site/CLAUDE.md
```

## 要確認

feature-dinosaur/picoの公開ステータス・カスタムドメイン移行予定・アナリティクス導入有無など契約外の未確認事項は `ourai-site-quirks-and-debugging` の7〜8章に集約されている。そちらを参照すること。

このスキル固有の未確認事項（content-playbookの守備範囲内のもの）:

- 画像圧縮の具体的な基準値（サイズ上限・フォーマット指定など、design.md・CLAUDE.mdに明文化なし）
- llms.txtに個別のfeature-*.htmlページを追記すべきかどうかの方針
