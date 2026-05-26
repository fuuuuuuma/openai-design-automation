# 11. HTML 駆動の上級ワークフロー

## この章のゴール

- 全媒体の「中間表現」を **HTML** に寄せ、1 ソースから LP・スライド・PDF・OGP まで派生させる
- AI に HTML を書かせるメリット（再編集容易性・diff・移植性）を最大化する
- 公開先の枠（Notion / 自社サイト / GitHub Pages 等）に依存しない設計

---

## 11-1. なぜ HTML が「最強の中間言語」か

PPTX も Figma も Canva も独自フォーマットです。一度入れたら他に持ち出しにくい。
**HTML は人間も AI も読める / 書ける / バージョン管理できる**。CSS で見た目を変え、印刷 CSS で PDF にもなる。Puppeteer や Playwright で PNG にもなる。**1 つのソース → 多媒体展開** の起点として最強です。

---

## 11-2. パイプライン

```
[Markdown / JSON で本文]
     │
     ▼
[Codex が HTML テンプレに流し込み]
     │
     ├─► Web で公開 (GitHub Pages 等)
     ├─► PDF 書き出し（Puppeteer print）
     ├─► PNG 書き出し（Playwright screenshot）
     ├─► PPTX に変換（reveal.js → pdf → pptx）
     └─► OGP 画像生成（HTML → PNG）
```

---

## 11-3. テンプレ HTML の例

`templates/slide.html`：

```html
<!doctype html>
<html lang="ja">
<head>
  <meta charset="utf-8">
  <title>{{title}}</title>
  <link rel="stylesheet" href="/styles/slide.css">
</head>
<body class="slide">
  <header><h1>{{title}}</h1></header>
  <main>{{{body_html}}}</main>
  <footer><span>{{page}} / {{total}}</span></footer>
</body>
</html>
```

`styles/slide.css` には「1 スライド = 1920x1080 印刷」サイズで CSS Variables を仕込みます。これだけで PowerPoint 風の見た目を HTML で再現できます。

---

## 11-4. Codex への指示例

```
Codex: content/slides/q2_review.md を読み、各 H2 を 1 スライドとして
templates/slide.html に流し込み、out/slides/*.html を生成して。
最後に Puppeteer で out/slides.pdf に結合。

要件:
- 縦横 1920x1080
- 各 H2 タイトル + 配下の本文 + 1 画像（GPTImage2.0※ 経由）
- 画像プロンプトは Codex が H2 から自動派生
- 出力は ZIP にまとめて public/q2_review.zip
```

---

## 11-5. HTML → PDF（Puppeteer）

```ts
import puppeteer from "puppeteer";

const b = await puppeteer.launch();
const p = await b.newPage();
await p.goto("file://" + path.resolve("out/slides/01.html"));
await p.pdf({ path: "out/q2.pdf", width: 1920, height: 1080, printBackground: true });
await b.close();
```

複数 HTML を 1 PDF にまとめるなら `pdf-lib` で merge。

---

## 11-6. HTML → OGP 画像（PNG）

```ts
import { chromium } from "playwright";

const b = await chromium.launch();
const p = await b.newPage({ viewport: { width: 1200, height: 630 } });
await p.goto("file://" + path.resolve("out/ogp/article-42.html"));
await p.screenshot({ path: "out/ogp/article-42.png" });
await b.close();
```

ブログ記事 100 本分の OGP を 5 分で量産できます。

---

## 11-7. HTML スライド → PPTX

reveal.js でスライド化 → PDF → `pdf2pptx`（Python）で PPTX に変換。完全再現は無理ですが、テキスト編集前提の納品なら十分です。

```
md  ──► reveal.js HTML  ──► PDF  ──► PPTX (編集可)
```

---

## 11-8. CSS 設計のコツ

- **CSS Variables** にブランドを集約（`--brand-primary` 等）
- **印刷 CSS（`@media print`）** で PDF 用の余白・改ページを制御
- **`break-after: page;`** で確実な改ページ
- **Tailwind の `@apply`** をテンプレ側で使い、コンテンツ側は class を意識しなくて良い設計

---

## 11-9. AI に HTML を書かせる時の鉄則

- **コンポーネント単位で書かせる**: 一気に 500 行は事故る
- **テンプレと中身を分ける**: テンプレは固定、中身だけ AI が触る
- **Lighthouse スコアをチェックさせる**: アクセシビリティ・パフォーマンスは数値で評価
- **画像は遅延読み込み**: `loading="lazy"` を AI に必ず付けさせる
- **alt 必須**: SEO とアクセシビリティの両面で

---

## 11-10. 「HTML 1 ソース」で派生する成果物例

| 派生物 | 変換手段 |
| -- | -- |
| 公開 LP | そのまま Vercel / GitHub Pages |
| PDF 資料 | Puppeteer print |
| OGP / バナー | Playwright screenshot |
| PPTX | reveal.js → PDF → pdf2pptx |
| メルマガ HTML | inline CSS 化（Juice 等） |
| Notion 記事 | HTML2Notion API |
| 印刷物 PDF | CSS Paged Media |
| キャプチャ素材 | スマホ実機で開いて画面録画 |

---

## 11-11. 落とし穴

- **CSS の単位混在**: px と % が混ざると PDF 化で崩れる。1 ソース内で統一
- **フォントの埋め込み忘れ**: `@font-face` を必ず Local + Web 両対応
- **絵文字フォントの差**: macOS と Linux で別物。CSS で明示
- **画像パス**: 絶対パス推奨。ファイルシステム → URL の差で詰む

---

## 次の章へ

[12. Codex の機能をふんだんに使う上級編 →](12_codex_power_features.md)
