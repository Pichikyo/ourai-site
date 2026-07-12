---
name: ourai-site-design-contract
description: ourai-site（往来サイト）のHTML/CSSを1行でも書く・直す前に必ず読む。色・フォント・余白・改行・中央寄せ・ホバー等デザイン面の変更や、design.md/CLAUDE.mdの規律に沿っているか確認したい時、紫やグラデーションやhoverリフトを使いたくなった時、type scaleを下げたくなった時、見出しやリードの改行・泣き別れが気になる時に読み込むこと。
---

# ourai-site デザイン契約 遵守ガイド

## 0. 唯一の正本はdesign.md

**このスキルは正本ではない。** このプロジェクトのデザイン方針の正本は `/Users/pichikyo/dev/ourai-site/design.md` （2026-07-07時点183行）と、その要約を含む `/Users/pichikyo/dev/ourai-site/CLAUDE.md` （同23-28行目に禁止事項の抜粋あり）。

**このスキルとdesign.mdが矛盾したら、design.mdが勝つ。** このスキルはdesign.mdを読みやすく早見表化しただけの案内であり、新しい規律を作ったり正本を上書きしたりしない。作業前に必ずdesign.md本体を開くこと。このスキルだけを読んで済ませない。

---

## 1. 絶対禁止事項（早見表）

| 禁止 | 根拠（design.md） | 破るとどうなるか |
|---|---|---|
| 紫（purple） | Theme節「紫なし」 | ブランドのアンカーヒュー264(blue-indigo)から外れ、往来の色設計が崩壊する。style.css:80 で `--purple: var(--color-accent);   /* eliminates blue-purple gradients site-wide */` と定義されている。**`--purple`という変数名自体は残っているが、値をaccent色（ブルー）に再定義することで出力される色が紫にならないようにしている**（変数名は残存、出力色は紫ではない、という実装意図） |
| グラデーション | Theme節「グラデーションなし」 | editorial路線（WhO準拠のフラットな無彩色UI）が崩れ、量産AI感が出る |
| 影（box-shadow） | 「真似る基準値」表：影＝ほぼ無し | WhOの「線は極細・影なし」という基準値から逸脱し、サイト全体の質感が浮く |
| hoverリフト（translate/scale） | Microinteractions stance「リフト禁止」 | Motion節の「transform/opacity/colorのみ・layoutプロパティ禁止」に抵触しうる。style.css:212,234,445,513,587,612,662 は色変化のみで問題なし。**ただし style.css:427 の `.concept__more:hover .arrow { width: 64px; }`（widthはlayoutプロパティ）と style.css:465-469 の `a:hover .circle { transform: translateX(4px); }`（translate使用）の2件は、design.mdのこのルールに抵触しうる既存実装として現存する（要ともみさん確認・2026-07-07grep確認済み）。「translate/scale型のhoverは0件」ではない** |

この4つは design.md 本文とCLAUDE.md 24-27行目の両方に明記されている二重確認事項。「小さい変更だから」で例外を作らない。上記の.arrow/.circleの2件は新規のルール違反を作る前例にはならない（design.md自身が禁止するパターンが既に残っているだけであり、これに倣ってよいという意味ではない）。

---

## 2. 色はoklchトークンのみ・生の色コード禁止

style.css:10-84 の `:root` に実装済みの11トークン（2026-07-07時点で実装確認済み）:

```
--color-paper / --color-paper-2 / --color-white   … 背景（ほぼ白〜純白）
--color-ink / --color-ink-2 / --color-ink-3        … テキスト（黒〜グレー）
--color-rule                                       … 罫線・ボーダー
--color-accent / --color-accent-ink                … ブランドブルー（アンカーヒュー264）とその上のテキスト
--color-focus                                      … フォーカスリング
--color-marker                                     … マーカー強調（1ページ1文まで）
```

- ブルー（`--color-accent`）は差し色。**1ビューポートあたり面積3%以下**に抑える（design.md Theme節）。UIは無彩色に徹し、色は実績写真に語らせる。
- 新しい色が必要になっても `#3355ff` のような生カラーコードを直書きしない。既存トークンで足りない場合はdesign.mdの合意を取ってからトークンを追加する。

**破るとどうなるか**: 紫混入や彩度過多になりやすく、往来の「無彩色UI＋写真主役」という編集方針が壊れる。oklchで統一しているのは色相・彩度を数値で厳密固定するため。

---

## 3. spacingは4-pointトークン必須・生px/rem原則禁止

style.css:45-53 に実装済み:

```
--space-3xs: 0.25rem(4px)  --space-2xs: 0.5rem(8px)  --space-xs: 0.75rem(12px)
--space-sm: 1rem(16px)     --space-md: 1.5rem(24px)  --space-lg: 2rem(32px)
--space-xl: 3rem(48px)     --space-2xl: 5rem(80px)    --space-3xl: 7.5rem(120px)
```

セクション上下paddingは `--space-2xl`（80px）、見出し下は `--space-xl`（48px）が既定（design.md Spacing節）。

**要注意（実態のギャップ）**: design.mdは「生の rem/px は原則書かない」としているが、2026-07-07時点でリポジトリの実HTMLには例外的な生px直書きが複数残っている（例: about.html:76-82 の `style="padding:40px 36px"` `style="font-size:20px;margin:10px 0 14px"` 等、company-record.htmlの`max-width:880px`系）。これは「許可された自由」ではなく**まだ直しきれていない古い書き方（放置すると後で困る）**として扱うこと。新規にコードを書く/直す際は生pxを増やさず、可能なら既存の生pxも気づいた範囲でトークンに寄せる。ただし大規模な一括置換はCLAUDE.mdの「触ってはいけないもの」に抵触しうるため、ともみさんの指示なしに広範囲リファクタしない。

**破るとどうなるか**: 生pxが増えるほど後から「余白の基準値」を変えたい時（実際2026-07-06にtype scaleを一回り上げた実績あり）に全ページを目視で洗い出す羽目になる。トークン化は将来の一括変更コストを潰すための保険。

---

## 4. フォントは2種のみ・廃止済みフォントを復活させない

- 見出し（Display）: **Zen Kaku Gothic New**（700/900）
- 本文（Body）: **Noto Sans JP**（400/500）
- 読み込みは style.css:7 の1行 `@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@400;500;700&family=Zen+Kaku+Gothic+New:wght@500;700;900&display=swap')` のみ（2026-07-07時点でGoogle Fonts追加読み込みは0件）。

**廃止済み・復活禁止**:
- Instrument Serif / Noto Serif JP（セリフ体全般）
- Poppins

design.md Typography節に明記の通り「明朝は使わない」。2026-07-07時点でリポジトリ全体を検索した結果、Poppins・セリフフォント名の記述は**0件**（完全除去済み）。

`font-feature-settings: "palt"` で和文を詰めた上で `letter-spacing` を効かせる手法（WhOと同じ）を崩さない。

**破るとどうなるか**: セリフやPoppinsが混ざると「太さの対比でメリハリを作る」という2フォント構成の設計思想が崩れ、過去にリブランドで捨てた古いトンマナが再侵入する（git log: 902ecf3以降のリデザインで意図的に廃棄したもの）。

---

## 5. type scaleは2026-07-06確定・これ以上下げない

style.css:34-42 に実装済み（design.mdと数値一致・確認済み）:

| トークン | 実測値(px換算) | 用途 |
|---|---|---|
| --text-xs | 13px | 小ラベル/日付/タグ |
| --text-sm | 15px | 説明文/キャプション |
| --text-base | 16px | 本文 |
| --text-md | 18px | リード/やや大きい本文 |
| --text-lg | 20px | サブ見出し |
| --text-xl | 26px | セクション見出し |
| --text-2xl | 32px | 中大見出し |
| --text-3xl | 40px | 大見出し/stat |
| --text-display | clamp(40px, 4vw+16px, 56px) | ヒーロー大見出し（据え置き） |

**2026-07-06、この数値体系は「全体を一回り上げる」形で確定した。理由は50代決裁者の可読性優先（ともみさんの営業判断）。**「小さい方がおしゃれ」という好みは承知の上で可読性を取った、という経緯がdesign.md本文に明記されている。**これ以上は下げない**という決定であり、次にどんな依頼が来ても勝手にサイズを縮めない。

**破るとどうなるか**: 決裁者層（50代想定）の可読性を落とし、営業判断として一度確定した合意を無断で覆すことになる。デザイン的な「洗練」を理由にサイズを下げる提案が来ても、まずこの経緯をともみさんに確認する。

---

## 6. 泣き別れ禁止と.wbクラス

**泣き別れ**（用語定義: 「運営」「自治体」のような一語や意味の塊が、行末と次行頭にまたがって分断される現象）は禁止。

- リード文・見出しは文節単位を `<span class="wb">`（style.css:156 `display: inline-block`）で括る。span間に空白・改行を入れない。
- 実例（index.html:57）:
  ```html
  <p class="lead"><span class="wb">企業や自治体の</span><span class="wb">新しい挑戦を、</span>...</p>
  ```
- CSSの `text-wrap: balance`（h1.page-title, h2, .lead）や `text-wrap: pretty`（本文p）も併用されているが、**design.md本文に「balanceだけでは泣き別れは防げないことが実証済み」と明記**されている。CSSは補助であり、`.wb`が本体。

**破るとどうなるか**: ブラウザ幅やフォント環境によって単語が行またぎで分断され、読みにくい・素人っぽい見た目になる。text-wrap:balance任せにすると再発する（実証済みの過去の失敗）。

---

## 7. .br-pcクラスとヒーローh1の例外

- 意図的な詩的改行（`<br>`）には必ず `class="br-pc"` を付ける。style.css:849-850 で680px以下は `display: none`（SP/モバイルでは改行を解除）。
- **例外**: ヒーローh1「仮想と現実を、/往来する。」のみ、680px以下でも全幅で改行を維持する（design.md「短く世界観の核のため」）。他のbr-pcはすべてSPで解除される前提で書く。
- 素の `<br>`（br-pcクラスなし）を本文に追加しない（CLAUDE.md 27行目）。

**破るとどうなるか**: SP画面で不自然な改行位置が固定され、レスポンシブでガタガタになる。ヒーローh1の例外を忘れて他の見出しと同じ扱いにすると、ブランドの世界観コピーの見え方が変わってしまう。

---

## 8. リードは2行以内ルール

- リード文（`.lead`）は1行に収まるのが最強。2行になる場合、2行目が5文字以下ならコピー自体を書き直す。
- 例外: `.br-pc`による意図的な詩的改行（index ヒーロー・SERVICEリードなど）は3行まで許容。

**破るとどうなるか**: 中途半端な行末（1〜4文字だけ次行に落ちる）は見た目が間延びし、editorial路線の「字間・行間の美しさ」を損なう。

---

## 9. 左端ライン統一ルール（2026-07-07確定）・中央寄せ禁止

- セクションの左端は常に `.wrap` 基準（style.css:104 `max-width: var(--max)（1180px）; margin: 0 auto;`）で全ページ・全セクション共通。この `.wrap` 自体のmargin:autoはページ全体を中央に置くための土台であり、禁止対象ではない。
- **禁止されているのは`.wrap`の"内側"で個別要素をさらに中央寄せすること**。`class="wrap" style="max-width:…"` の上書きや、`style="max-width:520px; margin:16px auto 0"` のような要素単位の中央寄せは、縦の左端ラインが乱れるため禁止。
- 読み幅を絞りたい時は、`.wrap`の内側に**margin指定なしのmax-widthコンテナ**を置く（左寄せのまま幅だけ絞る）。実例: about.html:146,161やcompany-record.html:55等の `<div style="max-width:880px">`（marginなし＝左寄せを維持）。
- **例外は2つ**: `.cta-foot`（全幅センター演出）と KV/CONCEPT等の独自レイアウト。実例としてabout.html:179、works.html（旧works.html・2026/07/11改名）:477、budget.html:121、case-detail.html:105の `<p style="max-width:520px;margin:16px auto 0;...">` は、いずれも `.cta-foot` セクション直下にあるため許容パターン（確認済み）。この例外パターンを`.cta-foot`の外で真似しない。

**破るとどうなるか**: 各セクションの左端が微妙にズレて見え、editorial系サイトの生命線である「縦の罫線的な整い」が崩れる。中央寄せは一見きれいに見えても、隣接セクションとの左端不揃いが目に付きやすい。

---

## 10. CTAとテキストリンクの型

- Primary CTA: 背景 `--color-ink`、文字色 `--color-accent-ink`、radius 2px、padding `14px 40px`、字間.06em
- Secondary CTA: 枠線 1.5px solid `--color-ink`、背景transparent、同radius/padding
- テキストリンクは「ラベル ——→」形式（矢印は長めの罫線）。WhOのVIEW MOREに倣う。

---

## 11. ページ別の写真許可

| ページ種別 | 写真使用 |
|---|---|
| index / case / feature-* | 実績写真OK |
| About / Contact / News | タイポグラフィのみ |

白基調（`--color-paper` / `--color-paper-2`）はフッターまで一貫させる。footer は `--color-paper-2` 背景（style.css確認済み）。

---

## 12. このスキルを使わない場面

- ローカル起動・デプロイ手順を知りたい → **ourai-site-run-and-deploy**
- 実績数字・案件名・原稿内容（何を書くか）の方針を知りたい → **ourai-site-content-playbook**
- 「この変更をしていいか」「触ってはいけないもの」の可否判断そのものを知りたい → **ourai-site-change-control**
- KVスライダーやハンバーガーメニューの実装が壊れた時の原因調査 → **ourai-site-quirks-and-debugging**

このスキルは上記のいずれでもなく、「design.mdの規律に沿っているか・どう読むか」を確認したい時だけ使う。

---

## 13. 要確認

デザイン契約の守備範囲外の未確認事項（contact.htmlフォーム送信先・feature-dinosaur/picoの公開ステータス・アナリティクス・カスタムドメイン・budget.html価格根拠）は `ourai-site-quirks-and-debugging` の7〜8章に集約されている。契約外の未確認事項はそちらを参照すること（重複記載による更新漏れを防ぐため、本スキルでは一覧を持たない）。

デザイン契約の守備範囲内の要確認事項はこちら:

- style.css:427（`.concept__more:hover .arrow`のwidth変化）と style.css:465-469（`a:hover .circle`のtranslateX）が、design.mdのMotion節「layoutプロパティ禁止」「リフト禁止」に抵触しうる（1章参照）。修正するか許容するかはともみさん判断が必要。

---

## 出所とメンテナンス

正本の日付・内容がずれていないか、以下のワンライナーで再検証すること。

```bash
# design.mdの改行・左端ルール確定日、type scale確定日を確認
grep -n "2026-07-0[67]" /Users/pichikyo/dev/ourai-site/design.md

# oklchトークン11色がstyle.cssに実装されているか
grep -n "^  --color-" /Users/pichikyo/dev/ourai-site/style.css

# 4-pointスペーシングトークンの現在値
grep -n "^  --space-" /Users/pichikyo/dev/ourai-site/style.css

# type scaleの現在値
grep -n "^  --text-" /Users/pichikyo/dev/ourai-site/style.css

# 廃止済みフォント（Poppins/セリフ）が復活していないか（0件が正常）
grep -rn "Poppins\|Instrument Serif\|Noto Serif" /Users/pichikyo/dev/ourai-site/*.css /Users/pichikyo/dev/ourai-site/*.html

# hoverにtransform(translate/scale)やlayoutプロパティ変化が紛れ込んでいないか（.arrow/.circleの既知2件を除き0件が正常。既知2件も見つかったら都度ともみさんに確認）
grep -n "hover" /Users/pichikyo/dev/ourai-site/style.css

# .wrap外での禁止パターン（中央寄せ）が増えていないか目視確認
grep -rn 'style="[^"]*margin:[^"]*auto' /Users/pichikyo/dev/ourai-site/*.html

# .br-pcの680pxブレークポイントが変わっていないか
grep -n "br-pc" /Users/pichikyo/dev/ourai-site/style.css
```
