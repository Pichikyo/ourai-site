# Design — 往来 (OURAI)

XR・AI・リアル空間を横断する体験設計スタジオのデザインシステム。
毎ページのリデザインはこのファイルを先に読む。ページごとに別テーマを作らない。

参照リファレンス: WhO (whohw.jp) の実測値に忠実。ただしコンテンツは往来独自。
真似る対象は「ヘッダー高さ / フォント種別 / フォントのサイズ感 / レイアウト / パーツ間の余白」。

## Genre
editorial（サンセリフ・無彩色UI・写真主役・字間広め）

## 真似る基準値（WhO 実測 → 往来採用値）

| 項目 | WhO 実測 | 往来採用 |
|---|---|---|
| ヘッダー高さ | 140px / SP 105px | 80px / SP 80px（2026-07-13にPC/SP統一。旧140/88px→瞳ちゃんデザイン調整で80pxへ） |
| ヘッダー下線 | 1px #e5e5e5 | 1px var(--color-rule) |
| 本文 | 14px・字間.05em・行間2.0 | 同じ |
| 説明文 | 13px・字間.1em | 同じ |
| 小ラベル/日付/タグ | 12px・字間.1em | 同じ |
| サブ見出し | 18px・字間.1em | 18px・字間.08em |
| セクション見出し | 24px・字間.05em | 24〜30px・字間.04em |
| ナビ文字 | 17〜18px・字間.15em | 15px・字間.14em |
| 大見出し（最大） | 36〜42px | clamp 40〜56px |
| セクション間余白 | 80〜120px | 80px（section上下） |
| 字間（全体） | 最低.05em | 同じ（body既定） |
| 角丸 | ほぼ直角 | 2px |
| 影 | ほぼ無し | 無し |

ポイント: ①文字を大きくしない ②全テキストに字間 ③行間・余白をたっぷり ④線は極細・影なし。

## Theme

カスタム。アンカーヒュー: 264 (blue-indigo)。紫なし。グラデーションなし。
UIは無彩色に徹し、ブルーは差し色として最小面積（≤3% per viewport）。色は実績写真に預ける。

```
--color-paper:       oklch(96%  0.004  255)   /* メイン背景（ほぼ白） */
--color-paper-2:     oklch(98%  0.003  255)   /* セクション交互 */
--color-white:       oklch(100% 0      0  )   /* 純白 */
--color-ink:         oklch(18%  0.010  260)   /* 主テキスト（ほぼ黒・微青） */
--color-ink-2:       oklch(45%  0.008  264)   /* 本文グレー */
--color-ink-3:       oklch(62%  0.006  260)   /* ラベル・非活性 */
--color-rule:        oklch(90%  0.004  258)   /* ボーダー / 罫線（#e5e5e5相当） */
--color-accent:      oklch(52%  0.19   264)   /* ブランドブルー（差し色のみ） */
--color-accent-ink:  oklch(99%  0.003  264)   /* アクセント上のテキスト */
--color-focus:       oklch(65%  0.22   264)   /* フォーカスリング */
--color-marker:      oklch(93%  0.13   102)   /* 蛍光ペン風マーカー。.marker で下半分のみ。1ページ1文まで */
```

## Typography

- 本文 Body: "Noto Sans JP" 400/500 — モリサワ「ゴシックBBB」の無料代替（端正な中ゴ）
- 見出し Display: "Zen Kaku Gothic New" 700/900 — モリサワ「見出ゴMB31」の無料代替（力強い見出しゴシック）
- 2フォント構成。セリフ（Instrument Serif / Noto Serif JP）は廃止。Poppins も廃止。

本文ゴシック＋見出しゴシックの「太さの対比」でメリハリを作る。明朝は使わない。
`font-feature-settings: "palt"` で和文を詰め、その上で `letter-spacing` を効かせる（WhOと同手法）。

Type scale: px ベース（WhO 基準 → 2026-07-06 全体一回りアップ確定。50代決裁者の可読性優先・ともみさん判断）。
- 13(xs) / 15(sm) / 16(base) / 18(md) / 20(lg) / 26(xl) / 32(2xl) / 40(3xl)px、ヒーロー大見出しのみ clamp(40,56) 据え置き。
- ナビ・モバイルメニューは 16px。
- 「小さい方がおしゃれ」は承知の上で可読性を取った営業判断。これ以上は下げない。

## 改行・組版ルール（2026-07-06 確定）

1. **泣き別れ禁止（リード・見出し）**: 「運営」「自治体」等が行をまたぐのは不可。
   リード文は文節単位を `<span class="wb">`（display:inline-block）で括る。span間に空白・改行を入れない。
2. **CSS 支援**: `h1.page-title, h2 { text-wrap: balance }`、本文・リード `p, .lead { text-wrap: pretty }`。リードは`max-width:none`で枠いっぱいに流す（2026/07/11ともみさん指示。短い行に折れて見えるbalance+680px上限は廃止）。
   ただし balance だけでは泣き別れは防げない（実証済み）。.wb が本体、CSSは補助。
3. **詩的改行は PC 専用**: 意図した `<br>` には `class="br-pc"` を付け、680px以下で `display:none`。
   例外: ヒーロー h1「仮想と現実を、/往来する。」のみ全幅で改行維持（短く世界観の核のため）。
4. **リードは2行以内**: 2行目が5文字以下になる文はコピーを書き直す。1行に収まる文が最強。
   例外: 詩的改行（br-pc）で意図的に行を割ったリード（index ヒーロー・SERVICE リード）は3行まで可。

## 左端ライン統一ルール（2026-07-07 確定）

- セクションの左端は常に .wrap 基準で全ページ・全セクション共通。
- 読み幅を絞りたいときは .wrap の内側に「margin指定なしの max-width コンテナ」を置く（左寄せ）。
- `class="wrap" style="max-width:…"` や `margin: … auto` によるコンテンツの中央寄せは禁止（縦ラインが乱れるため）。
- 例外: .cta-foot（全幅センター演出）と KV/CONCEPT 等の独自レイアウトはこの限りではない。

## Spacing

4-point named scale。必ず var(--space-*) を使う。生の rem/px は原則書かない。

```
--space-3xs: 0.25rem (4px)   --space-2xs: 0.5rem (8px)   --space-xs: 0.75rem (12px)
--space-sm:  1rem    (16px)  --space-md:  1.5rem (24px)  --space-lg: 2rem   (32px)
--space-xl:  3rem    (48px)  --space-2xl: 5rem   (80px)  --space-3xl: 7.5rem (120px)
```

セクション上下 padding は var(--space-2xl)（80px）。見出し下は var(--space-xl)（48px）。

## Motion

- Easing: --ease-out: cubic-bezier(0.16, 1, 0.3, 1)
- Duration: --dur-short: 220ms
- transform / opacity / color のみ。layout プロパティは禁止。
- prefers-reduced-motion: opacity-only ≤ 150ms
- リビール: なし（静的 HTML）

## Microinteractions stance

- ホバー: 色変化のみ（リンク→accent、カード→ボーダー濃色）。translate/scale は最小。リフト禁止。
- フォーカスリング: outline 2px solid var(--color-focus)、即時。
- transition: 個別プロパティ指定。transition-all 禁止。

## CTA voice

- Primary: background var(--color-ink), color var(--color-accent-ink), radius 2px, padding 14px 40px, 字間.06em
- Secondary: border 1.5px solid var(--color-ink), background transparent, 同 radius/padding
- テキストリンクは「ラベル ——→」形式（矢印は長め罫線）を多用。WhO の VIEW MORE に倣う。

## Eyebrow / sec-label ルール

英字ラベル（NEWS, SERVICE 等）はセクション見出しの上に小さく置く。字間.12em、12px、グレー。
WhO 同様「各セクションに小さな英字ラベル＋和文見出し」を許容するが、
うるさくしないため字を小さく・グレーに抑える。

## Per-page allowances

- index / case: 実績写真先行 OK
- Feature pages: 実績写真 OK（ユーザー供給）
- About / Contact / News: タイポグラフィのみ。例外: Aboutの代表ポートレート1点（profile_azuma.jpg、MESSAGEセクション、2026/07/10ともみさん指示）
- ページ遷移演出: 全ページ`main`に page-in（14px下からフェードイン0.55s）。方向スライドは不採用（2026/07/11）。prefers-reduced-motionでは無効
- 無彩色ルールの例外（先方ブランド資産・各1箇所限定・トークン化しない）: contactのLINEグリーンボタン／indexのVRChat公式カバーアート（vrchat_coverart.jpg、OFFICIAL PARTNERカード、2026/07/11ともみさん指示）

## What pages MUST share

- ロゴ / ワードマーク（黒）
- ヘッダー高さ 80px（PC/SP統一・2026-07-13改訂）/ 下線 1px
- --font-display（Zen Kaku Gothic New）+ --font-body（Noto Sans JP）
- 本文14px・字間.05em・行間2.0
- 白基調（フッターまで白を貫く）
- CTA ボタン形状・padding リズム

## What pages MAY differ on

- セクション数と順序
- ヒーローのサイズとレイアウト

## Exports

### tokens.css

```css
/* Hallmark · genre: editorial · theme: custom · anchor: oklch(52% 0.19 264)
 * display: Zen Kaku Gothic New · body: Noto Sans JP
 * reference: whohw.jp (measured) · paper-band: light · accent: minimal cool
 */
:root {
  --color-paper:       oklch(96%  0.004  255);
  --color-paper-2:     oklch(98%  0.003  255);
  --color-white:       oklch(100% 0      0  );
  --color-ink:         oklch(18%  0.010  260);
  --color-ink-2:       oklch(45%  0.008  264);
  --color-ink-3:       oklch(62%  0.006  260);
  --color-rule:        oklch(90%  0.004  258);
  --color-accent:      oklch(52%  0.19   264);
  --color-accent-ink:  oklch(99%  0.003  264);
  --color-focus:       oklch(65%  0.22   264);

  --font-display: "Zen Kaku Gothic New", "Noto Sans JP", sans-serif;
  --font-body:    "Noto Sans JP", "Hiragino Kaku Gothic ProN", "Yu Gothic", sans-serif;

  --track-body:  0.05em;
  --track-label: 0.12em;
  --track-nav:   0.14em;

  --space-3xs: 0.25rem; --space-2xs: 0.5rem;  --space-xs: 0.75rem;
  --space-sm:  1rem;    --space-md:  1.5rem;  --space-lg: 2rem;
  --space-xl:  3rem;    --space-2xl: 5rem;    --space-3xl: 7.5rem;

  --text-xs:   0.75rem;   --text-sm:  0.8125rem; --text-base: 0.875rem;
  --text-md:   1rem;      --text-lg:  1.125rem;  --text-xl:   1.5rem;
  --text-2xl:  1.875rem;  --text-3xl: 2.25rem;
  --text-display: clamp(2.5rem, 4vw + 1rem, 3.5rem);

  --ease-out:  cubic-bezier(0.16, 1, 0.3, 1);
  --dur-short: 220ms;

  --radius-card: 2px; --radius-btn: 2px; --radius-pill: 99px; --radius-input: 2px;
}
```
