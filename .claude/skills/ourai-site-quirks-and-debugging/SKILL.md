---
name: ourai-site-quirks-and-debugging
description: ourai-siteのKVスライダー・ハンバーガーメニュー・レスポンシブ崩れ・フォント表示・モーション設定など「触ると壊れやすい実装」を調べる/直す/切り分けたい時に読み込む。「トップページのスライダーが動かない」「スマホでメニューが開かない」「レイアウトが崩れた」「フォントが変わって見える」「問い合わせフォームの送信先はどこか」といった質問・不具合報告のときに参照する。
---

# ourai-site 実装の癖とデバッグ

対象リポジトリ: `/Users/pichikyo/dev/ourai-site/`

このスキルは実装の「地雷」と「崩れた時の切り分け手順」の早見表です。デザインのルール（色・字間・改行等）は正本ではないのでここには書きません。デザイン判断が必要な場合は `design.md` と兄弟スキル `ourai-site-design-contract` を必ず読んでください。

**矛盾したら正本が勝ちます。** 正本は `design.md`（デザイン）と `CLAUDE.md`（運用・禁止事項）です。このスキルの記載と実際のコードが食い違っていたら、コードとdesign.md/CLAUDE.mdを信じてください。

---

## 1. KVスライダー（トップページのメインビジュアル）

`index.html` の「①ヒーロー」セクション、`<section class="hero-kv">` 内にあります（2026-07-07時点、`index.html:53`付近）。

### 構造
- HTML側の主要ID/クラス:
  - `.kv-slider` … スライダー全体のラッパー
  - `.kv-track` / `id="kvTrack"` … スライド4枚を横並びで内包する帯（`index.html:65`）
  - `.kv-slide` … スライド1枚（`<a>`タグ、画像+キャプション`.kv-cap`）。2026-07-07時点で4枚（横須賀市／日産×BEAMS／京セラ／モスバーガー）
  - `id="kvNext"` / `id="kvPrev"` … 矢印ボタン
  - `id="kvDots"` … ドットナビ（JSが動的に生成。HTML側には空の`<div>`のみ）
- JS本体は `index.html:293-338` の `<script>` 内、即時実行関数（IIFE）1本。

### 挙動の仕様（実測）
- 5秒ごとに自動で次のスライドへ切り替わる（`setInterval(..., 5000)`、`index.html:319`）。
- `prefers-reduced-motion: reduce`（OS側の「視差効果を減らす」設定）が有効な場合、自動回転タイマーは起動しない（`reset()`関数内で`if(reduce) return;`、`index.html:300, 316-317`）。ただし矢印クリック・ドットクリック・スワイプでの手動送りは reduced-motion でも動く。
- マウスが乗っている間（`mouseenter`）は自動回転を止め、離れる（`mouseleave`）と再開する（`index.html:324-325`）。
- タッチスワイプ対応: 40px以上のスワイプで前後スライド送り（`index.html:328-335`）。
- スライド送りは `transform: translateX()` で実装（`index.html:313`）。

### ID依存という「地雷」の意味
JSは `document.getElementById('kvTrack')` などIDベースでDOMを取得しています（`index.html:296, 321-322, 299`）。つまり：
- `id="kvTrack"` / `id="kvNext"` / `id="kvPrev"` / `id="kvDots"` のどれか1つでもHTML側で消す・リネームすると、その機能だけ静かに壊れます（コンソールエラーは出るが、画面上は「反応しないボタン」になるだけで気づきにくい）。
- `track` が見つからない場合は `if(!track) return;` で即座に処理を止める設計（`index.html:297`）なので、`#kvTrack`が無いページ（例: index.html以外）ではこのスクリプトは何もしません。壊れているわけではなく「対象がないので動かない」が正常です。
- **CLAUDE.mdで「KVスライダーの構造は触ってはいけないもの」と明記されています（変更はともみさんの明示指示があるときのみ）。** IDのリネームやスライド枚数の変更は必ず本人確認を取ってください。

---

## 2. ハンバーガーメニュー（スマホ用ナビ）

全ページ共通のヘッダー内にあります。

### 構造
- `id="menuBtn"` … ハンバーガーボタン本体（`<button class="hamburger">`）
- `id="mobileNav"` … スマホ用ドロップダウンナビ（`<nav class="mobile-nav">`）
- JS本体は各HTMLファイルの末尾、`</body>`直前の`<script>`内（例: `contact.html:176-195`）。全ページに同じロジックが個別にコピーされている（共通JSファイル化はされていない＝1ページずつ同じコードが埋め込まれている点に注意。修正時は全ページへの反映漏れがないか確認すること）。JS共通化の是非は構造変更の承認判断にあたるため`ourai-site-change-control`の守備範囲。大規模改修（共通JSファイル化など）を考える場合はそちらを先に確認すること。

### 挙動
- クリックで `mobileNav` に `.open` クラスをトグル（表示/非表示はCSS側 `style.css:829` `.mobile-nav.open { display: block; }` で制御）。
- `aria-expanded` / `aria-hidden` 属性も連動して切り替える（アクセシビリティ対応）。
- `mobileNav`内の各リンクをクリックすると自動でメニューを閉じる（`contact.html:187-193`）。

### 表示されるブレークポイント
CSS側で `.hamburger { display: flex; }` が有効になるのは `max-width: 768px` と `max-width: 900px` の両方のメディアクエリ内（`style.css:833, 840`。同じルールが2箇所に重複定義されている＝実質900px以下で表示）。

**CLAUDE.mdで「ハンバーガーの構造は触ってはいけないもの」と明記されています。** ID変更や開閉ロジックの変更はともみさんの明示指示が必要です。

---

## 3. ブレークポイント一覧（2026-07-07時点、style.css実測）

`style.css`内の`@media`クエリは以下の5種類のみ。**1180pxはメディアクエリではなく `--max: 1180px`（`.wrap`コンテナの最大幅トークン、`style.css:83, 104`）なので、レスポンシブの切替点ではありません**（事前ブリーフにあった「ブレークポイント一覧680/768/860/900/1180px」は誤りなので訂正します）。

| 幅 | 該当箇所（行） | 役割 |
|---|---|---|
| `max-width: 860px` | `style.css:337`, `style.css:429` | KVスライダーを1カラム化＋アスペクト比変更／CONCEPTセクション（左画像+右テキスト）を1カラム化 |
| `max-width: 900px` | `style.css:768`, `style.css:839` | ヘッダー高さをSP用に切替（`--header-h-sp`適用）、PC用ナビ`nav ul`を非表示、カードグリッドを1カラム化、ハンバーガー表示 |
| `max-width: 768px` | `style.css:775`, `style.css:832` | `.wrap`の左右パディング縮小、ロゴサイズ縮小、見出しサイズをclamp縮小、フッターを縦積み、ハンバーガー表示、サブグリッドを1カラム化 |
| `max-width: 680px` | `style.css:589`, `style.css:843`, `style.css:849` | NEWSリストの折り返し方変更、VIRTUAL FASHIONセクションのグリッド1カラム化、**`.br-pc`（PC専用改行）を`display:none`で解除** |
| `prefers-reduced-motion: reduce` | `style.css:785` | 全要素の`transition-duration`を0.01msに強制し、`scroll-behavior`を`auto`に（後述） |

数値が近い900pxと768pxで役割が分かれているのは意図的な多段階レスポンシブです（「ナビを消す」と「余白を詰める」を別タイミングでやっている）。崩れを疑うときはまずこの表でどのブレークポイントの担当範囲かを特定してください。

---

## 4. フォント読込

`style.css:7`（ファイル冒頭）:
```css
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@400;500;700&family=Zen+Kaku+Gothic+New:wght@500;700;900&display=swap');
```
Google Fonts CDNへの1リクエストで2書体（Noto Sans JP / Zen Kaku Gothic New）をまとめて読み込んでいます。`--font-display`（見出し用）と`--font-body`（本文用）のトークン定義は `style.css:25-26`。

**フォントが崩れて見える時に疑うべき点:**
- ネットワークタブでこのGoogle Fonts系リクエストが失敗/ブロックされていないか（社内ネットワークやプライバシー拡張機能でCDNがブロックされるケースがある）。
- `@import`はCSSファイルの先頭に書く必要があり、他のルールより前に記述されていないと一部ブラウザで無視されます（現状は`style.css`の最上部にあるため問題なし、というのが2026-07-07時点の状態）。
- セリフ体やPoppinsが混ざって見える場合は、キャッシュされた旧CSSを見ている可能性が高い（過去にセリフ体・Poppinsは廃止された経緯がgit履歴にあり）。ハードリロード（Cmd+Shift+R等）でまず確認。

---

## 5. `prefers-reduced-motion`対応箇所

サイト内で「アニメーションを減らす」設定に対応しているのは、確認できた範囲で以下の2箇所のみです。

1. **CSS全体**（`style.css:785-787`）:
   ```css
   @media (prefers-reduced-motion: reduce) {
     * { transition-duration: 0.01ms !important; scroll-behavior: auto !important; }
   }
   ```
   すべての要素の`transition`を実質瞬時にし、スムーススクロール（`html { scroll-behavior: smooth; }`, `style.css:89`）も無効化する。

2. **KVスライダーの自動回転**（`index.html:300, 316-317`）: `matchMedia('(prefers-reduced-motion: reduce)')`で判定し、自動タイマー（5秒ごとの切替）だけを止める。手動操作（矢印・ドット・スワイプ）は動き続ける。

design.mdでは「モーションはtransform/opacity/colorのみ、`--dur-short`使用、`transition-all`禁止、`prefers-reduced-motion`対応」がルールとして明記されています（正本側の規律）。このスキルは「どこに実装されているか」の案内に留め、新規追加のルールはdesign.mdを見てください。

---

## 6. 表示が崩れた時の切り分け手順

上から順に、安い確認から潰していってください。

1. **ハードリロードする。** キャッシュされた古いCSS/JSを見ているだけの可能性が最も高い（Cmd+Shift+R / Ctrl+Shift+R）。
2. **ブラウザの開発者ツールでコンソールエラーを見る。** `Cannot read properties of null`のようなエラーが出ていたら、IDが変更・削除された可能性が高い（第1章のKVスライダー、第2章のハンバーガーのID依存を疑う）。
3. **どのブレークポイントで崩れているか特定する。** 開発者ツールのレスポンシブモードで幅を動かし、崩れ始める幅を確認したら、第3章の表でどのメディアクエリの担当かを特定する。
4. **`style.css`と対象HTMLの両方を見る。** インラインstyle（`style="..."`属性）で個別ページが上書きしている場合があるため、`style.css`側だけを見ても原因が分からないことがある（例: `contact.html`のフォームは`<style>`タグでページ内に直接スタイルを書いている）。
5. **フォント関連なら第4章を確認。** CDNブロック・キャッシュを疑う。
6. **KVスライダー/ハンバーガー固有の不具合なら、そのIDがHTML側に存在するかを`grep`で確認する。**
   ```bash
   grep -n 'id="kvTrack"\|id="kvNext"\|id="kvPrev"\|id="kvDots"' /Users/pichikyo/dev/ourai-site/index.html
   grep -n 'id="menuBtn"\|id="mobileNav"' /Users/pichikyo/dev/ourai-site/<対象ページ>.html
   ```
7. **ローカルで実際に動かして確認する。** 兄弟スキル`ourai-site-run-and-deploy`の手順でローカルサーバーを起動し、実機（実ブラウザ）で確認する。静的解析だけで判断しない。
8. **それでも分からなければ、直近のgitコミットを疑う。** `git log --oneline -10`で直近の変更を確認し、`git diff <直前コミット>`で差分を見る。

---

## 7. お問い合わせフォームの送信先について（要確認だった点の実態）

2026-07-07時点で`contact.html`を実際に確認したところ、フォームには送信先が実装されています。

```html
<form class="form" id="contact-form" action="https://ssgform.com/s/MrgYkzC4z61d" method="POST">
```
（`contact.html:97`）

「ssgform.com」という外部フォーム送信サービスにPOSTする実装で、`action`属性・`method`属性ともに設定済みです。ただし以下は **未検証・要確認** として残します。

- このssgform.com上でどのメールアドレス/webhookに転送される設定になっているかは、リポジトリのコードだけからは分かりません（サービス側の管理画面でしか確認できない）。
- フォーム送信が実際に成功する（201/200が返る、実際に通知が届く）かどうかの動作確認はこのスキル執筆時点で行っていません。
- reCAPTCHA等のスパム対策の有無もHTML上からは確認できていません。

**「送信先が未実装」という認識は事実と異なります。** 送信先（action URL）は存在しますが、その先の運用・受信確認は未検証・要確認です。オーナー（ともみさん）にssgform.com側の受信設定を確認してもらうか、実際にテスト送信して着信確認することを推奨します。

---

## 8. その他の未検証・要確認事項（2026-07-07時点）

- **feature-dinosaur.htmlの公開ステータス**: `works.html（旧works.html・2026/07/11改名）:155-157`で`<span class="feat-badge">公開準備中</span>`というバッジ付きでリンクされています。ページ自体は存在しリンクも生きていますが、「公開準備中」という表示が意図的なプレースホルダーなのか、コンテンツ未完成のまま公開されてしまっているのかはオーナー確認が必要です。
- **feature-pico.htmlの公開ステータス**: `works.html:133-135, 224`で`<span class="feat-badge">特集</span>`バッジ付き・通常カードとしても2箇所からリンクされています。表示上は公開済み案件として扱われていますが、内容が最終版かは要確認。
- **アナリティクス導入有無**: 全17ページを`gtag`/`analytics`/`G-`/`GTM-`で検索した結果、該当ヒットなし（2026-07-07時点）。アクセス解析タグは未導入と見られますが、導入予定の有無はオーナーに確認していません。
- **カスタムドメイン移行予定**: 全ページのOGP `og:url`は`https://ourai-site.pages.dev/...`固定。独自ドメインへの移行予定があるかどうかはコードからは分からず、要確認。
- **budget.htmlの価格根拠**: `contact.html:131`のFAQに「100万〜3,000万円のレンジが中心で、最も多いのは300万〜500万円」という記載がありますが、この数字の算出根拠（社内実績集計か肌感か）はコードから確認できません。CLAUDE.mdの「実績の数字は社内台帳が正本」という運用ルールに照らすと、価格帯の記載も何らかの根拠資料と紐づいているべきですが、リポジトリ内にその出典は見当たりません。要確認。

---

## このスキルを使わない場面

- 色・フォント・字間・改行・余白・CTAの見た目のルールを知りたい → `ourai-site-design-contract`
- ローカル起動・デプロイコマンド・Cloudflare Pagesの運用を知りたい → `ourai-site-run-and-deploy`
- 実績の追加・案件名の掲載可否・コピーの書き方を知りたい → `ourai-site-content-playbook`
- ページ構成の変更や大きな改修の進め方・レビュー体制を知りたい → `ourai-site-change-control`

---

## 出所とメンテナンス

このスキルの内容は2026-07-07時点のリポジトリ実態です。以下のワンライナーで再検証してください。

```bash
# KVスライダーIDの現況確認
grep -n 'kvTrack\|kvNext\|kvPrev\|kvDots' /Users/pichikyo/dev/ourai-site/index.html

# ハンバーガーIDの現況確認
grep -n 'id="menuBtn"\|id="mobileNav"' /Users/pichikyo/dev/ourai-site/*.html

# ブレークポイント一覧の再確認
grep -n "@media" /Users/pichikyo/dev/ourai-site/style.css

# prefers-reduced-motion対応箇所の再確認
grep -n "prefers-reduced-motion" /Users/pichikyo/dev/ourai-site/style.css /Users/pichikyo/dev/ourai-site/*.html

# フォント読込行の確認
sed -n '1,10p' /Users/pichikyo/dev/ourai-site/style.css

# contact.htmlフォームの送信先確認
grep -n "<form" /Users/pichikyo/dev/ourai-site/contact.html

# feature-dinosaur/picoの公開バッジ状況確認
grep -n "feature-dinosaur\|feature-pico" /Users/pichikyo/dev/ourai-site/works.html

# アナリティクス導入有無の再確認
grep -ln "gtag\|analytics\|G-\|GTM-" /Users/pichikyo/dev/ourai-site/*.html
```
