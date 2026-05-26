# 03. Codex × GPTImage2.0 で画像を量産する

## この章のゴール

- Codex から GPTImage2.0※ を直接叩いて、サムネ・バナー・OGP を 1 コマンドで量産する
- 同一プロンプトのバリエーション生成と、参照画像入力（Image-to-Image）を使い分ける
- 失敗カットを自動で再生成するパイプラインを組む

> **※未確認**: 本章で「GPTImage2.0」と呼ぶのは OpenAI の最新画像生成 API の慣用表記です。実際の API 呼び出し時は `responses.images.create` 等の現行エンドポイントで `model: "gpt-image-2"` 相当を指定してください。モデル ID の正確な綴りは [15_unverified_terms.md](15_unverified_terms.md) を参照。

---

## 最小サンプル: 1 コマンドでサムネ生成

`scripts/gen-thumbnail.ts` を Codex に作らせます。プロンプトは次のとおり。

```
Codex: scripts/gen-thumbnail.ts を作って。
- 引数: --topic "AI 収益化" --variant 5
- OpenAI 画像 API（gpt-image-2 相当）を呼ぶ
- 出力: assets/output/thumb-{timestamp}-{n}.png
- アスペクト比 1280x720（YouTube サムネ）
- 失敗時は 3 回までリトライ
- 1 件 0.04 USD 想定。total コストを最後に標準出力
```

Codex が TypeScript を生成し、`tsx scripts/gen-thumbnail.ts --topic "AI 収益化" --variant 5` で 5 案が一括で出ます。

---

## プロンプト設計の鉄則

GPTImage2.0※ の出力品質は **プロンプトの 3 段構造** でほぼ決まります。

```
[1] 主題 (Subject)         : 何が中心に写るか
[2] スタイル (Style)        : 写真・イラスト・3D・グラフィック
[3] 構図と制約 (Composition): 余白・配色・カメラ位置・テキスト有無
```

例: 「[1] AI とコイン束のアイコン化、[2] フラットベクター・赤と黒の 2 色、[3] 中央配置・上部に余白 30%・テキストなし」。

Codex に毎回この 3 段で書かせるよう、`.codex/prompts/image.md` にテンプレを置きます。

---

## バッチ量産パイプライン

YouTube 動画 10 本分のサムネを一括生成するレシピ。

```bash
# titles.csv: 1 列目に動画タイトル
cat titles.csv | xargs -I {} codex run gen-thumbnail.ts --topic "{}" --variant 3
```

xargs と Codex の `run` サブコマンドを組み合わせると、CSV や JSONL 1 ファイル渡しで全タイトル分が並列実行されます。

> **コスト目安**: 1 画像 = 約 $0.04（HD）。10 タイトル × 3 案 = $1.2 / 回。週次運用で月 $40 弱。

---

## Image-to-Image（参照画像で再生成）

「この参考サイトのトンマナで作って」という指示は GPTImage2.0※ の image input で受け渡せます。

```ts
const res = await openai.images.edit({
  model: "gpt-image-2",
  prompt: "同じトーンで、被写体を 'ノートPCを開く女性' に置き換えて",
  image: fs.createReadStream("assets/input/reference.png"),
  n: 4,
  size: "1024x1024",
});
```

参照画像の入手手段は [10_appshots_advanced.md](10_appshots_advanced.md) の App Shots※ 経由が最速です。

---

## 後処理: アップスケールとリサイズ

GPTImage2.0※ の出力はそのままでは媒体ごとのサイズ規格に合いません。Codex に次のラッパースクリプトを書かせます。

```ts
// scripts/post-process.ts
import sharp from "sharp";

const variants = [
  { name: "youtube_thumb", w: 1280, h: 720 },
  { name: "instagram_reel", w: 1080, h: 1920 },
  { name: "x_card", w: 1200, h: 628 },
  { name: "ogp", w: 1200, h: 630 },
];

for (const v of variants) {
  await sharp("assets/output/base.png")
    .resize(v.w, v.h, { fit: "cover" })
    .toFile(`assets/output/${v.name}.png`);
}
```

`sharp` は依存が軽いので Codex の CI から呼びやすい。生成 → 後処理 → 各媒体最適サイズ書き出し、までを 1 ジョブで完結させます。

---

## 自動品質チェック

「文字が崩れた画像」を弾くために、生成直後に GPT-5 で OCR + 妥当性判定を回します。

```ts
const ocr = await openai.responses.create({
  model: "gpt-5",
  input: [
    { role: "user", content: [
      { type: "input_text", text: "この画像に文字崩れがあれば 'NG' とだけ返す。なければ 'OK'。"},
      { type: "input_image", image_url: dataUrl },
    ]},
  ],
});

if (ocr.output_text.trim() === "NG") await regenerate();
```

これで「日本語が虫食いになった画像」を機械的に再生成できます。

---

## 失敗例と回避

| 症状 | 原因 | 回避 |
| -- | -- | -- |
| 文字が grub_eng のように崩れる | 日本語が苦手な版を引いた | 「画像にテキストを入れない」プロンプトで生成 → Canva 側で文字を後乗せ |
| 同じ顔ばかり出る | プロンプトの subject 固定が強い | Variant ごとに seed を変える / `n: 4` で複数候補 |
| ロゴが勝手に出る | 学習元のクセ | プロンプトに「no logo, no watermark, no text」を必ず付ける |

---

## 次の章へ

[04. Codex × Canva / Figma プラグイン連携 →](04_codex_design_plugins.md)
