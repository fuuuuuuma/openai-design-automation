# OpenAI だけでデザイン業務を全自動化する 2026 年完全ガイド

> **対象**: クリエイター・YouTuber・LP 制作者・スライド量産担当・1 人マーケター
> **方針**: OpenAI API キー 1 本を起点に、Codex / Manus / ChatGPT / 外部プラグイン（Canva / Figma / HeyGen / Remotion / Higgsfield / Seedance）まで横断し、画像・動画・スライド・LP・Web サイト・アプリ生成までを全自動化する
> **更新**: 2026-05 時点
> **ライセンス**: CC0 1.0 Universal
> **総文字数**: 約 80,000 字（マルチバイト含む）
> **構成**: 全 15 章を 1 ページに統合

---

## 目次

- [01. 全体像と OpenAI 1 本縛りの設計思想](#01-全体像と-openai-1-本縛りの設計思想)
- [02. Codex 環境と API キー管理（OpenAI Developer プラグイン）](#02-codex-環境と-api-キー管理openai-developer-プラグイン)
- [03. Codex × GPTImage2.0 で画像を量産する](#03-codex--gptimage20-で画像を量産する)
- [04. Codex × Canva / Figma プラグイン連携](#04-codex--canva--figma-プラグイン連携)
- [05. Codex × HeyGen / Remotion で動画を作る](#05-codex--heygen--remotion-で動画を作る)
- [06. Codex 単体で LP / Web サイト / アプリ開発](#06-codex-単体で-lp--web-サイト--アプリ開発)
- [07. Manus × GPTImage2.0 でスライド資料を生成](#07-manus--gptimage20-でスライド資料を生成)
- [08. Manus × Higgsfield × Seedance で映像生成](#08-manus--higgsfield--seedance-で映像生成)
- [09. ChatGPT × Canva「マジックレイヤー」修正フロー](#09-chatgpt--canvaマジックレイヤー修正フロー)
- [10. 裏技: App Shots で参考サイトを呼び出す](#10-裏技-app-shots-で参考サイトを呼び出す)
- [11. HTML 駆動の上級ワークフロー](#11-html-駆動の上級ワークフロー)
- [12. Codex の機能をふんだんに使う上級編](#12-codex-の機能をふんだんに使う上級編)
- [13. 実践ワークフロー集](#13-実践ワークフロー集)
- [14. トラブルシュート & 既知の落とし穴](#14-トラブルシュート--既知の落とし穴)
- [15. 未確認用語リストと参考リンク](#15-未確認用語リストと参考リンク)

---

## 01. 全体像と OpenAI 1 本縛りの設計思想

### なぜ「OpenAI だけで」完結できるのか

2026 年現在のデザイン業務自動化は、もはや 1 ツールでは完結しません。画像生成、動画生成、デザイン編集、Webサイト構築、それぞれに最適化された専用サービスが乱立しています。
そこで本ガイドは「**OpenAI の API キー 1 本を起点として、外部ツール群を呼び出して全自動化する**」というアーキテクチャを採用します。OpenAI の API キーで生成系（GPTImage2.0※ / GPT-5 / Realtime API）を駆動し、Codex や Manus のエージェント能力で外部 SaaS のプラグイン API を叩く構成です。

> **※未確認**: 「GPTImage2.0」は本資料では「OpenAI が提供する最新画像生成 API の総称」として扱います。2026-05 時点での正式モデル ID は OpenAI Platform の Models 一覧で都度確認してください（例: `gpt-image-1` 系の後継）。詳細は [15 章](15 章) を参照。

---

### 設計原則 5 つ

#### 1. 「主語」を OpenAI にする

外部ツールを使うときも、必ず Codex / Manus / ChatGPT のいずれかの OpenAI エージェントが「主語」として実行ループを回します。プラグインや MCP サーバーは「呼ばれる側」。これにより、人間が触るインターフェースが OpenAI 系に統一され、操作学習コストが圧倒的に減ります。

#### 2. API キーは 1 本に集約

OpenAI Developer プラグイン（VS Code / Codex CLI 向け）で API キーを 1 ヶ所に集約管理します。`OPENAI_API_KEY` を環境変数にせず、専用のシークレットストアに入れることで、誤コミットを防ぎます。詳しくは 02 章。

#### 3. 中間表現は Markdown / HTML / JSON のいずれか

ツール間でデータを受け渡す中間表現は、人間が読めて LLM が再編集できる Markdown / HTML / JSON のいずれかに統一します。Figma の独自バイナリや Canva の独自フォーマットを最終成果物としてだけ使い、編集途中はテキストで保つ設計です。

#### 4. 「やり直し可能性」を最優先

生成 AI は失敗します。必ず「生成 → プレビュー → 修正」のループを 2〜3 周回す前提で組みます。1 発で完成を目指さない。Codex のチェックポイント機能、Manus のスナップショット、ChatGPT のチャット履歴を「セーブポイント」として使い倒します。

#### 5. 公開系の操作は自動化しない

YouTube への投稿、X への自動投稿、課金アクションの実行は、本ガイドではすべて「生成・保存まで」で止めます。BAN リスクとアカウント停止の被害が大きすぎるため、最終ボタンは人間が押す原則です。

---

### カバーする生成物の種類

本ガイドで全自動化する成果物は次の通りです。

| カテゴリ | 具体物 | 主担当エージェント |
| -- | -- | -- |
| 静止画 | サムネ・バナー・OGP・図解 | Codex × GPTImage2.0 |
| スライド | 営業資料・教材・登壇資料 | Manus × GPTImage2.0 |
| 短尺動画 | Reels / Shorts / TikTok | Codex × Remotion / Manus × Seedance |
| アバター動画 | 解説動画・自己紹介 | Codex × HeyGen |
| LP / Webサイト | キャンペーン LP / コーポレート | Codex 単体 |
| Web アプリ | 1 機能 SaaS / 社内ツール | Codex 単体 |
| デザイン修正 | 既存 Canva / Figma の改訂 | ChatGPT × Canva マジックレイヤー※ |

---

### 「OpenAI 縛り」が崩れる例外

外部生成系を使う場面が 2 つだけあります。

1. **動画生成の Seedance**（中国 ByteDance 系・物理表現に強い）  
2. **Higgsfield**（カメラワーク制御が得意なモーション系）

これらは OpenAI Sora 系で代替可能な日もありますが、2026-05 時点では「画作りの幅」を優先して併用します。ただし呼び出し元の指揮は Manus（OpenAI モデルが頭脳）にすることで、運用主体は OpenAI に保ちます。

---

### 章間の依存関係

```
01 overview ─┬─► 02 apikey ─► 03 gptimage ─► 04 design plugins
             │                            └─► 05 video plugins
             ├─► 06 codex solo dev
             ├─► 07 manus slides ─► 08 manus higgsfield seedance
             ├─► 09 chatgpt canva magic
             └─► 10 appshots ─► 11 html workflow ─► 12 codex power
                                                  └─► 13 workflows real
```

最短ルートは 01 → 02 → 03 → 06 → 13。ここまでで「Codex 1 本で LP と画像と簡易アプリが作れる」状態になります。

---


---

## 02. Codex 環境と API キー管理（OpenAI Developer プラグイン）

### この章のゴール

- Codex CLI を導入し、OpenAI API キーを「**コードから見えない場所**」に置く
- 「OpenAI Developer プラグイン」※経由で API キーを統一管理する
- コスト上限と利用ログを最初の 10 分で仕込む

> **※未確認**: 「OpenAI Developer プラグイン」は本資料では「VS Code / Codex CLI 向けに OpenAI が配布する公式拡張」を指す呼称として用います。2026-05 時点で複数の派生（GitHub Marketplace 版・VS Code 拡張版・JetBrains 版）が並走しているため、最新の正式名称は OpenAI 公式ドキュメントで確認してください。

---

### ステップ 1. Codex CLI のインストール

```bash
## 公式 CLI（OpenAI が配布する Codex CLI）
npm i -g @openai/codex-cli
codex --version
```

代替として、JetBrains IDE 用プラグイン、VS Code 拡張版、Cursor 同梱版があります。本ガイドでは macOS の Terminal + Codex CLI を前提に進めます。

---

### ステップ 2. API キーの取得とスコープ

OpenAI Platform にログイン → **Settings → API keys → Create new secret key** で発行。発行時に次を必ず設定します。

- **Name**: `codex-design-auto-2026`（用途が分かる名前）
- **Permissions**: `Restricted`（read/write を最小化）
- **Project**: 専用プロジェクトを切る（コスト分離）
- **Expiration**: 90 日（更新を強制してローテーション）

> 1 つのキーに全責務を持たせない。画像生成専用、動画生成専用、開発専用で 3 本に分けるとローテーションが楽です。

---

### ステップ 3. OpenAI Developer プラグインに登録

OpenAI Developer プラグイン側で次の設定を行います（VS Code 拡張版を例に）。

1. コマンドパレット → `OpenAI: Sign in with API Key`
2. プロジェクト選択 → `codex-design-auto-2026`
3. **Storage backend**: macOS Keychain（Windows は Credential Manager）
4. **Default model**: `gpt-5-codex`（コード）/ `gpt-image-2`※ （画像）

これで `OPENAI_API_KEY` を環境変数に置かなくても、Codex / プラグイン経由の生成は Keychain から自動取得されます。

> **※未確認**: `gpt-image-2` というモデル ID は本資料の慣用表記。正確なモデル ID は Platform 上で確認すること。

---

### ステップ 4. シークレットを「絶対にコミットしない」設定

```bash
## プロジェクトルートで
echo ".env*"            >> .gitignore
echo "*.key"            >> .gitignore
echo "credentials.json" >> .gitignore
echo ".openai/"         >> .gitignore

## pre-commit フックで secret 検出
npm i -D @secretlint/secretlint @secretlint/secretlint-rule-preset-recommend
npx secretlint --init
```

`secretlint` を pre-commit で走らせると、`sk-` 始まりの文字列が含まれた瞬間に commit がブロックされます。

---

### ステップ 5. コスト上限と利用ログ

API 暴走対策は必須です。OpenAI Platform → **Settings → Limits** で次を設定します。

- **Hard limit**: 月 $200（個人運用の目安）
- **Soft limit**: 月 $100（メールで警告）
- **Email alerts**: ON
- **Usage tracking**: プロジェクト別で集計

加えて、Codex CLI 側でも 1 リクエスト単位の上限を入れます。

```bash
## ~/.codex/config.toml
[limits]
max_tokens_per_request = 100000
max_cost_per_session_usd = 5.0
warn_at_usd = 1.0
```

---

### ステップ 6. プロジェクトの基本構成

```
my-design-auto/
├── CLAUDE.md          # Codex/Claude 共通の運用ルール（後述）
├── AGENTS.md          # サブエージェント定義（Codex 機能）
├── .codex/
│   ├── config.toml    # モデル / 制限 / プラグイン設定
│   └── prompts/       # 再利用プロンプト
├── assets/
│   ├── input/         # 参照画像・参考サイトのスクショ
│   └── output/        # GPTImage2.0 で生成した PNG
├── designs/
│   ├── canva/         # Canva 連携用 JSON
│   └── figma/         # Figma 連携用ノード ID マップ
└── README.md
```

`CLAUDE.md`（または `AGENTS.md`）を最初に整備すると、Codex が毎回その指示を踏まえて動くようになります。例えば「画像は 1024x1024 か 1536x1024 のみ」「LP は Tailwind v4 で」など。

---

### ステップ 7. 動作確認

```bash
codex "Hello から動作確認。OPENAI_API_KEY 経由ではなくプラグインから読めているか、最初の応答に keychain と入れて返答してください"
```

応答に `keychain` が含まれていれば、シークレット経路が正しく通っています。

---


---

## 03. Codex × GPTImage2.0 で画像を量産する

### この章のゴール

- Codex から GPTImage2.0※ を直接叩いて、サムネ・バナー・OGP を 1 コマンドで量産する
- 同一プロンプトのバリエーション生成と、参照画像入力（Image-to-Image）を使い分ける
- 失敗カットを自動で再生成するパイプラインを組む

> **※未確認**: 本章で「GPTImage2.0」と呼ぶのは OpenAI の最新画像生成 API の慣用表記です。実際の API 呼び出し時は `responses.images.create` 等の現行エンドポイントで `model: "gpt-image-2"` 相当を指定してください。モデル ID の正確な綴りは [15 章](15 章) を参照。

---

### 最小サンプル: 1 コマンドでサムネ生成

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

### プロンプト設計の鉄則

GPTImage2.0※ の出力品質は **プロンプトの 3 段構造** でほぼ決まります。

```
[1] 主題 (Subject)         : 何が中心に写るか
[2] スタイル (Style)        : 写真・イラスト・3D・グラフィック
[3] 構図と制約 (Composition): 余白・配色・カメラ位置・テキスト有無
```

例: 「[1] AI とコイン束のアイコン化、[2] フラットベクター・赤と黒の 2 色、[3] 中央配置・上部に余白 30%・テキストなし」。

Codex に毎回この 3 段で書かせるよう、`.codex/prompts/image.md` にテンプレを置きます。

---

### バッチ量産パイプライン

YouTube 動画 10 本分のサムネを一括生成するレシピ。

```bash
## titles.csv: 1 列目に動画タイトル
cat titles.csv | xargs -I {} codex run gen-thumbnail.ts --topic "{}" --variant 3
```

xargs と Codex の `run` サブコマンドを組み合わせると、CSV や JSONL 1 ファイル渡しで全タイトル分が並列実行されます。

> **コスト目安**: 1 画像 = 約 $0.04（HD）。10 タイトル × 3 案 = $1.2 / 回。週次運用で月 $40 弱。

---

### Image-to-Image（参照画像で再生成）

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

参照画像の入手手段は [10 章](10 章) の App Shots※ 経由が最速です。

---

### 後処理: アップスケールとリサイズ

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

### 自動品質チェック

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

### 失敗例と回避

| 症状 | 原因 | 回避 |
| -- | -- | -- |
| 文字が grub_eng のように崩れる | 日本語が苦手な版を引いた | 「画像にテキストを入れない」プロンプトで生成 → Canva 側で文字を後乗せ |
| 同じ顔ばかり出る | プロンプトの subject 固定が強い | Variant ごとに seed を変える / `n: 4` で複数候補 |
| ロゴが勝手に出る | 学習元のクセ | プロンプトに「no logo, no watermark, no text」を必ず付ける |

---


---

## 04. Codex × Canva / Figma プラグイン連携

### この章のゴール

- Codex から Canva と Figma を MCP / プラグイン経由で操作する
- 既存テンプレに対して「差分修正だけ AI に投げる」運用に切り替える
- 1 つの素材を Canva と Figma 両方に同期させる

---

### 4-1. Canva プラグイン

#### セットアップ

Canva Pro 契約 → Canva Developer Portal → Personal Access Token を発行 → Codex MCP サーバー設定に追加。

```json
// ~/.codex/mcp.json
{
  "servers": {
    "canva": {
      "command": "npx",
      "args": ["-y", "@canva/mcp-server"],
      "env": { "CANVA_ACCESS_TOKEN": "${env:CANVA_TOKEN}" }
    }
  }
}
```

`CANVA_TOKEN` は Keychain に保存し、`env` 経由で注入します。**直書きしない**。

#### できること

- デザイン検索 → ID 取得
- 既存デザインの複製と編集
- 画像・テキスト要素の差し替え
- PDF / PNG / MP4 でエクスポート

#### Codex への依頼例

```
Codex: 「2026春LP_メイン」というデザインを 5 言語に複製して。
日本語 → 英語 / 韓国語 / 繁体字 / 簡体字 / スペイン語。
テキスト翻訳は GPT-5 を経由。レイアウト崩れを避けるため文字数を 1.1 倍以内に圧縮。
完了したら各 PDF を assets/output/lp_lang/ に保存。
```

Codex が翻訳 → Canva API での差し替え → エクスポート、までを一括実行します。

---

### 4-2. Figma プラグイン（公式 MCP）

#### セットアップ

Figma → Settings → Personal Access Tokens を発行 → MCP に追加。

```json
{
  "figma": {
    "command": "npx",
    "args": ["-y", "@figma/mcp"],
    "env": { "FIGMA_TOKEN": "${env:FIGMA_TOKEN}" }
  }
}
```

#### できること

- ファイル / フレーム / ノード ID の取得
- Variables の読み書き（デザイントークン）
- コンポーネント差し替え
- 画像書き出し（PNG / SVG / PDF）
- コメント投稿

#### 「コードからデザインを動かす」例

```
Codex: design-system ライブラリの「Button/Primary」コンポーネントを、
Variables の brand-color = #FF3B30 → #0070F3 に差し替えて。
影響する 14 ファイルすべて plays 用に書き出して、output/figma_export/ に保存。
```

これで色変更とエクスポートが 1 コマンドで終わります。手作業なら半日コース。

---

### 4-3. Canva ↔ Figma 同期

両者の中間表現として **デザイントークン JSON** を採用します。

```json
// design-tokens.json
{
  "color": {
    "brand-primary": "#0070F3",
    "brand-secondary": "#111111"
  },
  "spacing": { "xs": 4, "sm": 8, "md": 16, "lg": 24 },
  "typography": {
    "h1": { "size": 48, "weight": 700, "line-height": 1.2 }
  }
}
```

Codex に書かせるスクリプト 2 本：

- `sync-tokens-to-figma.ts`: JSON → Figma Variables
- `sync-tokens-to-canva.ts`: JSON → Canva ブランドキット

トークン 1 ファイルを更新するだけで、両方の SaaS に同じ値が伝播します。

---

### 4-4. 差分修正 (Patch Mode)

毎回全生成しないために「差分指示」だけ送る運用にします。

```
Codex: 既存 LP「春キャンペーン」のヒーロー画像のみ差し替え。
- 元: 桜のイラスト
- 新: 桜 + ノートPC を中央配置、トンマナそのまま
- それ以外の要素は触らない
```

GPTImage2.0※ で新画像生成 → Canva プラグインで該当ノードのみ replace_image → PDF 書き出し。**全体は触らない** が鍵。

---

### 4-5. 「修正だけ Canva」「初稿は Figma」の役割分担

経験則ですが次が機能します。

| フェーズ | ツール | 理由 |
| -- | -- | -- |
| ワイヤー設計 | Figma | コンポーネント化と Variables が強い |
| 初稿〜中稿 | Figma | 履歴管理・複数人レビュー |
| 最終調整・量産 | Canva | テンプレ流用・媒体別書き出しが速い |
| 緊急修正 | ChatGPT × Canva マジックレイヤー※ | ブラウザだけで完結 |

---

### 4-6. 既知の罠

- **Canva 無料プランは API が制限される** — Pro / Teams 契約推奨
- **Figma のフレーム ID は URL から抜くのが速い** — `?node-id=` の値
- **画像差し替え後にキャッシュが残る** — Canva はブラウザ側でリロード必須
- **トークン名の衝突** — Figma Variables と Canva ブランドキットで命名規約を同じにする

---


---

## 05. Codex × HeyGen / Remotion で動画を作る

### この章のゴール

- アバター動画は **HeyGen**、プログラマブル動画は **Remotion** で作り、両者を Codex から統合制御する
- 「台本 → 音声 → 動画 → 字幕 → 書き出し」を 1 ジョブで完了させる
- 動画の差分修正（文言だけ差し替え）を自動化する

---

### 5-1. HeyGen プラグイン

#### セットアップ

HeyGen Enterprise / Creator 契約 → API キー発行 → Codex MCP 登録。

```json
{
  "heygen": {
    "command": "npx",
    "args": ["-y", "@heygen/mcp-server"],
    "env": { "HEYGEN_API_KEY": "${env:HEYGEN_KEY}" }
  }
}
```

#### できること

- 既存アバター（人物 / 自撮りクローン）の一覧
- 台本入力 → アバター動画生成
- 多言語ボイス（日本語 / 英語 / 韓国語等）
- 書き出し（MP4 / WebM）

#### Codex への指示例

```
Codex: scripts/heygen-batch.ts を作って。
- 入力: scripts/scripts.json（[{"avatar": "fuuma_v3", "voice": "ja-male-1", "text": "..."}]）
- HeyGen API を呼んで MP4 を生成
- 完了通知は Slack Webhook に投げる（本文に * を使わない）
- 同時並列は 3 ジョブまで
```

---

### 5-2. Remotion プラグイン

Remotion は「React で動画を書く」フレームワーク。Codex と相性が最高で、JSX を書ければそのまま動画になります。

#### セットアップ

```bash
npx create-video@latest my-remotion-pj --template hello-world
cd my-remotion-pj
npm install
```

#### Codex に書かせる例

```
Codex: components/HookOpening.tsx を作って。
- 0〜1.5 秒: 黒背景にテキスト「3秒で説明します」フェードイン
- 1.5〜2.5 秒: 主役画像 hero.png をフェード + 拡大 1.0 → 1.05
- BGM: assets/bgm-light.mp3 を音量 0.3 で
- フォント: Noto Sans JP 700
- 解像度: 1080x1920 (Reels)
```

Remotion のコンポジションが自動生成され、`npx remotion render` で MP4 出力されます。

---

### 5-3. HeyGen + Remotion のハイブリッド

「アバター（HeyGen）+ 字幕・装飾（Remotion）+ 効果音（自前）」を 1 本にまとめる構成です。

```
[HeyGen で本体動画.mp4]
        │
        ▼
[Remotion で字幕・効果・BGM レイヤー]
        │
        ▼
[ffmpeg で合成 + 書き出し]
```

Codex に「`scripts/pipeline-video.ts` を書け」と指示すれば、HeyGen API 呼び → Remotion レンダリング → ffmpeg 合成、までを 1 つの TypeScript で繋げられます。

---

### 5-4. 差分修正フロー

YouTube ショート量産で頻発する「文言だけ差し替えたい」を高速化します。

```
Codex: 動画 ID heygen-abc123 の台本だけ差し替え。
- 旧: 「AI で月 100 万」
- 新: 「AI で月 300 万」
- 他の演出・尺・BGM はそのまま再利用
```

HeyGen の `script` フィールドだけ更新 → 再レンダリング。約 2〜3 分で差し替え完了。

---

### 5-5. 字幕の自動生成

OpenAI Whisper API（または GPT-4o Audio）で書き起こし → Codex で SRT 整形 → Remotion で表示。

```ts
// 1. 文字起こし
const tr = await openai.audio.transcriptions.create({
  model: "whisper-1",
  file: fs.createReadStream("source.mp4"),
  response_format: "srt",
});
// 2. 日本語の意味区切り改行（LLM 判定）
const srt = await refineSrtWithLLM(tr.text);
// 3. Remotion で焼き込み
```

意味区切りは fugashi/MeCab ではなく **GPT-5 で文脈判断** させると自然です（個人運用ルールでも徹底）。

---

### 5-6. 動画コスト管理

| 項目 | 単価目安 | 月運用想定 |
| -- | -- | -- |
| HeyGen 60秒動画 | 約 $0.5 〜 $2 | 30 本で $15〜$60 |
| Remotion 自前レンダ | 電気代のみ | 〜 |
| Whisper 文字起こし | $0.006/分 | 10 時間で $3.6 |
| GPT-5 字幕整形 | $0.02 / 1 万字 | $1 程度 |

トータル月 $80 〜 $150 で「短尺動画 30 本 + 字幕付き」が回ります。

---

### 5-7. 既知の落とし穴

- **HeyGen のアバター生成は本人同意必須** — 商用利用ではライセンス契約を確認
- **Remotion の `delayRender()` を忘れると Promise が壊れる** — 非同期 fetch 時は必須
- **MP4 の音ズレ** — 30fps と 29.97fps を混ぜると 1 分で 2 フレームずれる。pipeline 全体を 30 固定に統一
- **書き出し時のフォント未埋め込み** — Remotion 側で `staticFile()` 経由のフォント読み込みを徹底

---


---

## 06. Codex 単体で LP / Web サイト / アプリ開発

### この章のゴール

- 外部プラグインを使わず Codex CLI 1 本だけで LP / コーポレートサイト / 1 機能 SaaS を完成させる
- 「対話 1 回 → アプリ起動」のテンポを掴む
- 公開（Cloudflare Pages / Netlify など、未確認のサービス名は読者側で選定）まで自動化する

---

### 6-1. なぜ Codex 単体で完結できるか

Codex は「コードを書く」だけでなく **Bash 実行・ファイル操作・パッケージ導入・テスト実行・サーバー起動** までを 1 セッションで行えます。
つまり「LP を作って」と命令すると、以下を自分で完結します：

1. `pnpm create next-app@latest` でプロジェクト生成
2. Tailwind v4 のセットアップ
3. ヒーロー / フィーチャー / 価格 / FAQ / CTA セクション作成
4. `pnpm dev` で起動 → スクショ確認
5. レビュー → 微修正 → 完成

人間がやることは「方針の指示」と「ブランドカラー渡し」だけです。

---

### 6-2. LP 1 本を 30 分で

#### プロンプト例

```
Codex: AI 副業学習サービスの LP を作って。
- スタック: Next.js (App Router) + Tailwind v4 + shadcn/ui
- 構成: hero / 3 features / pricing 3 plan / testimonials 3 / FAQ 4 / CTA
- 色: #0F172A（背景）/ #38BDF8（アクセント）/ 白文字
- フォント: Inter + Noto Sans JP
- 画像: 後で差し替え予定。今は placeholder OK
- レスポンシブ必須。SP は 1 カラム
- /api/contact に POST するフォーム
- 起動: pnpm dev で 3000 番に。最後にスクショして見せて
```

Codex が 10 分前後で全部組み、自分で `pnpm dev` を起動、Playwright でスクショを撮って返してきます。修正は 1 行で済む。

---

### 6-3. ヒーロー画像の差し替え

LP の hero 画像は 03 章 のスクリプトで GPTImage2.0※ から生成。

```
Codex: hero 画像を gen-thumbnail.ts で生成して。
- topic: "AI を学ぶ社会人イメージ"
- size: 1920x1080
- 出力後、components/Hero.tsx の bgImage を差し替えて再起動
```

差し替え後は HMR で自動反映。

---

### 6-4. SaaS（1 機能アプリ）

ユーザー登録 → 入力フォーム → 結果保存、までを Codex 1 本で。

```
Codex: 「日記ジャーナル」アプリを作って。
- スタック: Next.js + Prisma + SQLite + Auth.js (Email magic link)
- 機能: 日記の投稿 / 一覧 / 編集 / 削除
- バックエンドは Route Handler、フロントは Server Components
- スタイル: shadcn/ui のデフォルト
- 起動して localhost:3000 を見せて
```

これで MVP は完成します。Codex が DB マイグレーション・Auth 設定・テストデータ投入まで一気通貫で済ませます。

---

### 6-5. 公開（デプロイ）

> 公開系はユーザー手動の確認を必ず挟む方針（環境による）

Codex に「Cloudflare Pages にデプロイして」と指示すると、

1. `wrangler login`（ブラウザが開く → 人間が承認）
2. `pnpm build` + `wrangler pages deploy`
3. デプロイ URL を取得

までを行います。`wrangler` の認証ステップだけ人間が踏めば、それ以降は完全自動。**ただし課金・公開のような取り消せない操作は事前に必ず確認** が運用ルール（個人運用方針）。

---

### 6-6. テストと CI

```
Codex: e2e テストを Playwright で書いて。
- /, /signin, /journal/new, /journal/[id] を回って 200 を確認
- フォーム送信が API に届くこと
- スクショは tests/screenshots/ に保存
- GitHub Actions で push 時に自動実行
```

`.github/workflows/ci.yml` まで自動生成されます。

---

### 6-7. 「失敗を許す」設計

Codex は時々ミスります。回避策：

- **小さい commit を強制**: AGENTS.md に「1 機能 = 1 commit」と明記
- **dev サーバーの起動確認を必須に**: 「最後に必ず http GET で 200 を確認」
- **テスト無しの完了報告を禁止**: 「テストを書いてから完了と言う」

これだけで Codex の「動かないコードの納品」が激減します。

---

### 6-8. 例: 個人運用で実際に作るもの（参考）

| 用途 | スタック | 所要時間 |
| -- | -- | -- |
| 教材販売 LP | Next.js + Stripe Checkout | 60 分 |
| 視聴者の質問フォーム | Next.js + Resend + KV | 30 分 |
| YouTube 字幕生成ツール | Next.js + Whisper + Remotion | 4 時間 |
| Notion 連携の予約表 | Next.js + Notion API | 90 分 |
| ChatGPT のラッパー（社内向け） | Next.js + OpenAI SDK | 120 分 |

---


---

## 07. Manus × GPTImage2.0 でスライド資料を生成

### この章のゴール

- Manus（自律 AI エージェント）に GPTImage2.0※ を呼ばせて、PPTX / PDF / Slides 用の素材を量産する
- 「目次 → 各スライド画像 → ノート → 最終 PPTX 出力」を 1 セッションで完了させる
- 大量の営業資料・社内教材を「テーマだけ与える」で生産する

> **※未確認**: Manus は 2025〜2026 にかけて急速に機能拡張中。本資料の手順は 2026-05 時点の挙動を前提に書きます。UI 名や設定パスは変わる可能性あり。

---

### 7-1. Manus の何がスライドに向いているか

Manus は「**ブラウザ・OS 操作・コード実行・ファイル管理**」を 1 エージェントで完結します。スライド作成においては：

- Markdown → PPTX の変換
- HTML → PDF の変換
- ブラウザで参照サイトを巡回（外部素材の収集）
- 画像生成 API（GPTImage2.0※ 等）を Manus から呼ぶ
- ZIP でまとめて納品

までを「1 つのタスク内」でやり切ります。**人間は最初の指示 1 回だけ**。

---

### 7-2. 基本フロー

```
[Manus に「営業資料 30 ページを作って」と指示]
   │
   ├─ Manus が目次を生成 → 人間にプレビュー
   │   ├─ OK → 続行
   │   └─ NG → 目次のみ修正
   │
   ├─ 各ページの本文をテキストで生成（GPT-5 で）
   ├─ 各ページのキー画像を GPTImage2.0※ で生成
   ├─ python-pptx で組み立て
   └─ output/sales_2026.pptx として書き出し
```

---

### 7-3. Manus への指示テンプレ

```
ゴール: 「AI 収益化スクール」の営業資料 PPTX を作る
ページ数: 24
構成:
  - 1: タイトル
  - 2: スクール概要
  - 3-5: 課題提起
  - 6-12: ソリューション（7 つの特徴）
  - 13-16: 受講生の声 + 実績
  - 17-20: カリキュラム
  - 21: 料金プラン 3 つ
  - 22: 入会の流れ
  - 23: FAQ
  - 24: CTA
画像方針: 各ページ右側に GPTImage2.0※ で生成したイメージ画像
配色: 紺と金（コーポレート寄り）
フォント: Noto Sans JP
出力: output/sales_v1.pptx + output/sales_v1.pdf 両方
```

---

### 7-4. GPTImage2.0※ 呼び出しの自動化

Manus 内で実行する Python スクリプトの雛形：

```python
from openai import OpenAI
client = OpenAI()

def gen_image(prompt: str, out: str):
    r = client.images.generate(
        model="gpt-image-2",  # ※未確認 / 実 ID は要確認
        prompt=prompt,
        size="1024x1024",
        n=1,
    )
    with open(out, "wb") as f:
        f.write(b64decode(r.data[0].b64_json))
```

Manus はこの関数を生成して各ページから呼び出します。

---

### 7-5. python-pptx での組み立て

```python
from pptx import Presentation
from pptx.util import Inches

p = Presentation("templates/corporate.pptx")
for page in pages:
    s = p.slides.add_slide(p.slide_layouts[5])
    s.shapes.title.text = page["title"]
    body = s.placeholders[1]
    body.text = page["body"]
    s.shapes.add_picture(page["image"], Inches(5), Inches(1.5), Inches(4.5))

p.save("output/sales_v1.pptx")
```

テンプレ `templates/corporate.pptx` を 1 つ用意しておけば、Manus が中身を毎回入れ替えてくれます。

---

### 7-6. 多言語展開

「同じ資料を 5 言語に展開して」と Manus に追加指示すれば、

- 各ページの text を GPT-5 で翻訳
- 画像のキャプションだけ言語別に再生成
- ファイル名を `sales_en.pptx` `sales_zh.pptx` のように分けて保存

までを自動で行います。

---

### 7-7. Google Slides 経由の場合

PPTX を Manus に Google Drive へアップロード → Slides に変換 → 編集権限を共有、までやらせます。

```
Manus: 出力した sales_v1.pptx を Google Drive にアップロードし、
Google Slides に変換、リンクは閲覧者:閲覧のみで発行。
URL を返して。
```

OAuth 連携が必要なので、初回だけ人間がブラウザでログイン承認します。

---

### 7-8. 大量量産のコツ

- **テンプレを 3 種類用意**: コーポレート / カジュアル / 教材。最初の指示で選ばせる
- **ブランドキットを JSON 化**: 色・フォント・余白を 1 JSON にしてテンプレ別に切替
- **画像生成を非同期で並列化**: 24 ページ分の画像を逐次にすると 10 分かかるが、並列なら 2 分
- **失敗ページのみ再生成**: 「Manus、ページ 12 だけ画像作り直して」で済むよう、ページ ID を必ず付与

---

### 7-9. コスト感

24 ページ資料 1 本：

| 項目 | コスト |
| -- | -- |
| GPT-5 文章生成 24 ページ分 | $0.5 〜 $1 |
| GPTImage2.0※ 24 枚 | $1 前後 |
| Manus 実行時間 8 分 | プラン内（要 Manus 課金） |
| 合計 | **$2 〜 $3 / 1 本** |

---


---

## 08. Manus × Higgsfield × Seedance で映像生成

### この章のゴール

- Manus を指揮系統として、Higgsfield※ と Seedance※ の 2 系統の動画生成を組み合わせる
- カメラワーク（Higgsfield 強み）× 物理表現・人物動作（Seedance 強み）の役割分担
- 「画像 → 短尺動画 → 連結 → BGM・字幕 → 公開用 MP4」までの全自動パイプライン

> **※未確認**:
> - 「Higgsfield」はカメラ制御に強いとされる動画生成サービスの慣用表記（ユーザー原表記「Higgesfieid」を正式名称候補に修正併記）。
> - 「Seedance」は ByteDance 系の動画生成モデル（ユーザー原表記「SeeDance」も同義として扱う）。
> どちらも正確な API 仕様は 2026-05 時点の公式情報を要確認。

---

### 8-1. なぜ「OpenAI 縛り」なのに外部モデルを使うのか

01 章 の通り、OpenAI Sora 系で大半は賄えますが、

- **カメラワーク**（ドリー / クレーン / オービット）の自由度は Higgsfield※ の方が現時点で高い
- **人物の自然な動き**（ダンス / 演技）と背景物理（水・布）は Seedance※ がリードする日がある

そのため「OpenAI モデルを脳として、外部モデルを画作りに使う」役割分担を採用します。**指揮は Manus**、つまり OpenAI 系のエージェントが投げる構図を守ります。

---

### 8-2. パイプライン全景

```
[GPTImage2.0※ で静止画ベース生成]
        │
        ├─► [Higgsfield※ で 5 秒尺の画→動画（カメラワーク重視）]
        │
        └─► [Seedance※ で 5 秒尺の画→動画（演技重視）]
                │
                ▼
        [Manus が 6〜8 ショットを並べて 30 秒尺の編集]
                │
                ▼
        [ffmpeg で BGM + 字幕焼き込み → 出力 MP4]
```

---

### 8-3. ショット設計

Manus に渡す JSON 例：

```json
{
  "project": "ai-school-promo-2026",
  "duration_sec": 30,
  "shots": [
    { "id": 1, "type": "hero", "image_prompt": "都会のカフェで PC を開く女性、朝の光", "video_model": "higgsfield", "camera": "slow zoom in" },
    { "id": 2, "type": "context", "image_prompt": "ノート画面に AI のアイコン、上から俯瞰", "video_model": "higgsfield", "camera": "top-down 30deg" },
    { "id": 3, "type": "human", "image_prompt": "笑顔で電話に出る男性会社員、オフィス", "video_model": "seedance", "motion": "natural smile" },
    { "id": 4, "type": "product", "image_prompt": "教材ロゴをホログラム表示", "video_model": "seedance", "motion": "logo rotate" }
  ],
  "bgm": "assets/bgm/uplift_120bpm.mp3",
  "captions": "captions/promo.srt"
}
```

Manus はこの JSON を読んで、画像生成 → 動画生成 → 結合 → BGM → 字幕、まで一気通貫で実行します。

---

### 8-4. Higgsfield※ 呼び出し例

```ts
const r = await fetch("https://api.higgsfield.ai/v1/videos", {
  method: "POST",
  headers: { Authorization: `Bearer ${HIGGSFIELD_KEY}` },
  body: JSON.stringify({
    image: dataUrl,
    motion: "slow_zoom_in",
    duration: 5,
    aspect: "9:16",
  }),
});
```

（**※未確認**: 上記エンドポイントは想定 API。実 URL とパラメータ名は公式 Docs を確認）

---

### 8-5. Seedance※ 呼び出し例

```ts
const r = await fetch("https://api.seedance.bytedance.com/v1/i2v", {
  method: "POST",
  headers: { "X-Api-Key": SEEDANCE_KEY },
  body: JSON.stringify({
    image_url: imageUrl,
    motion_prompt: "natural smile, head turn slightly",
    duration: 5,
    fps: 24,
  }),
});
```

（**※未確認**: 上記エンドポイントは想定 API）

---

### 8-6. Manus が組む編集スクリプト

```python
from moviepy.editor import VideoFileClip, concatenate_videoclips, AudioFileClip

clips = [VideoFileClip(f"shots/{i+1}.mp4") for i in range(len(shots))]
final = concatenate_videoclips(clips, method="compose")

bgm = AudioFileClip("assets/bgm/uplift_120bpm.mp3").subclip(0, final.duration)
final = final.set_audio(bgm.audio_fadeout(2))
final.write_videofile("output/promo.mp4", fps=30)
```

字幕は ffmpeg の `subtitles` フィルタで焼き込み：

```
ffmpeg -i output/promo.mp4 -vf "subtitles=captions/promo.srt" output/promo_sub.mp4
```

---

### 8-7. 失敗カットの差し戻し

Manus に「ショット 3 の人物の表情が硬い」と伝えるだけで、

1. 該当 shot の prompt を微修正（"warm smile, looking off-camera"）
2. Seedance※ で再生成
3. 該当 shot だけ差し替え
4. 結合を再実行

を自動で回します。**全体再生成しない**のがコスト・時短の鍵。

---

### 8-8. 公開直前の人間チェック

- 商標 / 肖像権 / 音楽ライセンス
- 字幕の意味区切り（[srt スキル](../README.md) と同じ要領）
- 16:9 / 9:16 の切替時のクロップ
- 終了画面の CTA URL タイポ

ここだけは必ず人間が見ます。公開系の不可逆操作（YouTube / TikTok 投稿）は手動運用が原則。

---

### 8-9. コスト目安

| 項目 | 単価 | 30 秒尺 |
| -- | -- | -- |
| Higgsfield※ 5 秒尺 | 約 $0.4 〜 $1 | 3 ショットで $1.2 〜 $3 |
| Seedance※ 5 秒尺 | 約 $0.3 〜 $0.8 | 3 ショットで $0.9 〜 $2.4 |
| GPTImage2.0※ 静止画 6 枚 | $0.24 | $0.24 |
| Manus 実行時間 | プラン内 | プラン内 |
| 合計 | — | **$3 〜 $6 / 1 本** |

---


---

## 09. ChatGPT × Canva「マジックレイヤー」修正フロー

### この章のゴール

- ブラウザだけで完結する「即時修正フロー」を確立する
- Canva の「マジックレイヤー」※ を ChatGPT 経由で指示して、CLI を立ち上げずにデザイン修正を完結
- スマホからの修正にも耐える運用にする

> **※未確認**: 「Canva マジックレイヤー」はユーザー指定の表記。2026-05 時点で Canva の Magic Studio 群（Magic Eraser / Magic Edit / Magic Grab / Magic Switch / Magic Write / Magic Expand 等）に同名機能が存在するか公式に確認されていない。本章では「Canva の生成 AI 編集機能群を ChatGPT から呼び出して使う」概念として扱う。正式名称は [15 章](15 章) を参照。

---

### 9-1. シナリオ

外出先で「**LP のヒーロー画像、明日の朝までに 3 案出したい**」となった時、ChatGPT のスマホアプリだけで完結させたい。
ChatGPT には Canva 連携 GPT（Canva 公式 GPT を含む）があるので、以下が可能です：

1. ChatGPT に「LP のヒーロー画像を 3 案、Canva で作って」と入力
2. Canva 側でデザインが自動生成 → URL が返ってくる
3. URL を開いて気に入ったものを採用 / 微修正
4. PNG ダウンロード

---

### 9-2. セットアップ

1. ChatGPT の **GPTs** から「Canva」を検索 → 公式 GPT を「会話に追加」
2. 初回連携でブラウザの Canva ログインを許可
3. 連携完了。これ以降は ChatGPT 内から `@Canva` でデザイン操作可能

---

### 9-3. 修正フローのテンプレ

#### 9-3-1. 文字だけ差し替え

```
@Canva 「春キャンペーン LP」のヒーローテキストを
「新春 50% OFF」から「春の応援キャンペーン」に変更して。
配置・フォント・色はそのまま。完了したら PNG でダウンロード URL を出して。
```

#### 9-3-2. 画像オブジェクトの除去

```
@Canva 「営業資料 1 ページ」の左下の女性人物を
マジックレイヤー※（Magic Eraser 相当）で削除して、
背景を自然に補完して。
```

#### 9-3-3. 構図の再配置

```
@Canva このスライド、テキストを左 50%・画像を右 50% に並べ替えて。
余白は均等。タイトルだけ上に固定。
```

---

### 9-4. 「うまくいかない時」の AI 通訳

Canva の指示は曖昧だと失敗します。ChatGPT に **通訳役** を頼みます。

```
私: 「画像をふんわりさせたい」を Canva の正しい指示に翻訳して
ChatGPT: → "Apply a soft gaussian blur with radius 4, then reduce
   saturation by 15%. Keep the subject in focus."
→ そのまま @Canva に貼ると正確に伝わる
```

人間の感覚的な日本語 → 機械が理解する英語パラメータ、を翻訳させる運用がポイント。

---

### 9-5. ChatGPT 経由ならではの強み

- **複数 GPT を併用**: 「Canva」+「DALL·E」+「Excel」を 1 つのチャットで往復
- **履歴が残る**: 何月何日にどう修正したかが ChatGPT 履歴に蓄積（CLI だと残らないことが多い）
- **スマホ完結**: 外出先で修正 → そのまま Slack に共有
- **音声入力**: 「これをふんわり、もっと暖色」と話すだけで修正

---

### 9-6. 限界と回避

| 限界 | 回避策 |
| -- | -- |
| 細かいピクセル指定が苦手 | 「左から 240px」より「中央寄せ + 余白 10%」のように相対指定 |
| バッチ修正は遅い | 30 件以上は Codex × Canva プラグイン（04 章）に戻す |
| バージョン管理が弱い | Canva の Version History を必ず ON。週次でエクスポート保管 |
| 言語の混在 | 1 つの会話内では指示言語を統一（日本語なら全部日本語） |

---

### 9-7. 「ChatGPT で修正、Codex で量産」の分担

最強の組合せ：

```
[アイデア出し]      ChatGPT 音声入力
[1〜3 案の試作]     ChatGPT × Canva 連携
[気に入った案を選定] 人間
[同テンプレを 50 件量産] Codex × Canva プラグイン（バッチ）
[最終チェック]      ChatGPT で目検 + ファクトチェック
```

ChatGPT は「対話」と「即興」、Codex は「量」と「一貫性」。役割を分けます。

---

### 9-8. Insight（小ネタ）

- **「マジックレイヤー」※ という呼称**は社内ドキュメントでは便利。新人にも伝わる
- **Canva 公式 GPT は無料プランでも一部使える**。本格運用は Pro
- **モバイル版 ChatGPT** は音声入力 → そのまま @Canva で送れるため、運転中の指示にも対応（運転中の操作は法令厳守）

---


---

## 10. 裏技: App Shots で参考サイトを呼び出す

### この章のゴール

- 参考サイト・参考デザインを **キャプチャ → 構造化 → AI 入力** までを 1 操作で完了させる
- 「あの LP みたいに作って」を実装可能な指示に翻訳する
- 著作権・商標に踏み込まない範囲で「参照」する作法を守る

> **※未確認**: 「App Shots」はユーザー指示の原表記（推測される正式名称候補は「AppShots」「App Shots」など）。本章ではスクリーンショット系の汎用ツール群を指す呼称として扱います。同等の機能は CleanShot X / Shottr / iOS のスクショ → ChatGPT 共有 でも代替可能。

---

### 10-1. なぜ「参考スクショ」が最強の入力か

GPT-5 や GPTImage2.0※ は **画像入力に対するプロンプト精度が劇的に上がりました**。「文字で 100 行説明する」より「1 枚スクショ + 5 行コメント」の方が再現性が高い。
だからこそ、参考サイトを高速にスクショして AI に渡す動線が、デザイン自動化の生産性を左右します。

---

### 10-2. 基本フロー

```
[参考サイト発見]
   │
   ▼
[App Shots※ 等でフルページ + 各セクションをキャプチャ]
   │
   ▼
[Codex / ChatGPT / Manus に画像 + 抽出ポイントを渡す]
   │
   ▼
[「同じ構造・別ブランドで」と指示して再生成]
```

---

### 10-3. キャプチャの取り方 3 種

#### A. 全画面ショット（OS 標準）

- macOS: `Cmd + Shift + 3`
- Windows: `Win + Shift + S`

雑だが速い。雑談ベースの方針相談に使う。

#### B. 要素単位ショット（CleanShot X / Shottr 等）

- 要素にホバー → 自動でその DOM を矩形選択
- スクロール領域を縦長で 1 枚にする「scrolling capture」

LP の構成要素を 1 つずつ AI に渡す時の主力。

#### C. App Shots※ 系（自動巡回 + 整理）

- URL を 1 つ入れると、PC / SP / タブレット 3 サイズで自動キャプチャ
- フォルダに自動整理（YYYY-MM-DD / ホスト名）
- ZIP で Codex / Manus に丸ごと渡せる

---

### 10-4. AI に渡す時のテンプレ

```
このスクショ（参考_LP_2026.png）と同じ構造で、
私のブランド向けに LP を作って。
- ブランド名: AI 収益化ラボ
- カラー: #0B1426 / #F5C518
- フォント: Noto Sans JP + Inter
- 入れ替え: ヒーロー画像 / 文言 / 価格 / CTA
- 維持: グリッド / 余白 / セクション順序
- 著作権配慮: 画像・文言は完全に置き換える。レイアウト「だけ」参考にする
```

「レイアウトだけ参考」と明示するのが重要。**直接の流用は著作権侵害**。インスピレーション源として使う。

---

### 10-5. Codex に「スクショから自動 LP」させる

```
Codex: assets/refs/lp_reference_2026.png を読んで、構造を解析。
- セクション分割
- 各セクションの主要見出し / サブ文言 / CTA / 画像位置を抽出
- 上記を JSON で出力（lp_structure.json）
- その JSON を元に、Next.js + Tailwind で別ブランド向け LP を生成
- 中身（文章・画像）は私のブランドに合わせて新規生成
```

GPT-5 のマルチモーダル能力 + Codex のコード生成で「**スクショ → 動く LP**」が 1 セッションで完了します。

---

### 10-6. 動画の参照

「あのバズった TikTok みたいに」も同じ要領で。

1. App Shots※ で動画の **キーフレーム 6 枚** を切り出し
2. GPT-5 にカット割り（編集の切り替え位置）を抽出させる
3. その情報を Remotion / Manus に渡して新規動画を作る

「**動画 → カット表 JSON → 別映像**」の翻訳が裏技ポイント。

---

### 10-7. 失敗パターンと回避

| 失敗 | 原因 | 回避 |
| -- | -- | -- |
| AI が参考画像のロゴまでコピーする | プロンプトに「no logo」を入れていない | 必ず「remove all logos, brand names, and watermarks」と明記 |
| 色味だけ似て構造が違う | スクショに余計な部分が映り込んでいる | セクション単位でクロップ → 1 枚 1 セクション |
| 文章の表現が「いかにも AI」 | 参考の文言をそのまま LLM に処理させた | ChatGPT に「私のブランドの口調で全部書き直して」と続けて指示 |

---

### 10-8. 知っておくべき著作権ライン

- レイアウト・構成・配色の「アイデア」は保護対象になりにくい
- 文章・画像・ロゴ・キャラクター・スクリーンショット内のコピーは保護対象
- 商標は「同業他社」では強くアウト
- 「参考にした」と公言するのは構わないが、**そっくりなものを公開** は別問題

迷ったら「文章は完全に書き直し」「画像は新規生成」「色は同系統だがコードは別」のラインを守る。

---

### 10-9. 裏技まとめ

- スマホで撮ったスクショを iCloud 同期 → そのまま Mac の Codex に渡す
- 動画は QuickTime で 0.5 秒区切りにフレーム書き出し → 6 〜 12 枚で要点把握
- Notion / Linear / Slack のスクショも参考になる（社内 UI の改善で重宝）
- 古い自分の制作物もスクショ → AI に渡せば「過去最高作」を超える土台になる

---


---

## 11. HTML 駆動の上級ワークフロー

### この章のゴール

- 全媒体の「中間表現」を **HTML** に寄せ、1 ソースから LP・スライド・PDF・OGP まで派生させる
- AI に HTML を書かせるメリット（再編集容易性・diff・移植性）を最大化する
- 公開先の枠（Notion / 自社サイト / GitHub Pages 等）に依存しない設計

---

### 11-1. なぜ HTML が「最強の中間言語」か

PPTX も Figma も Canva も独自フォーマットです。一度入れたら他に持ち出しにくい。
**HTML は人間も AI も読める / 書ける / バージョン管理できる**。CSS で見た目を変え、印刷 CSS で PDF にもなる。Puppeteer や Playwright で PNG にもなる。**1 つのソース → 多媒体展開** の起点として最強です。

---

### 11-2. パイプライン

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

### 11-3. テンプレ HTML の例

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

### 11-4. Codex への指示例

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

### 11-5. HTML → PDF（Puppeteer）

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

### 11-6. HTML → OGP 画像（PNG）

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

### 11-7. HTML スライド → PPTX

reveal.js でスライド化 → PDF → `pdf2pptx`（Python）で PPTX に変換。完全再現は無理ですが、テキスト編集前提の納品なら十分です。

```
md  ──► reveal.js HTML  ──► PDF  ──► PPTX (編集可)
```

---

### 11-8. CSS 設計のコツ

- **CSS Variables** にブランドを集約（`--brand-primary` 等）
- **印刷 CSS（`@media print`）** で PDF 用の余白・改ページを制御
- **`break-after: page;`** で確実な改ページ
- **Tailwind の `@apply`** をテンプレ側で使い、コンテンツ側は class を意識しなくて良い設計

---

### 11-9. AI に HTML を書かせる時の鉄則

- **コンポーネント単位で書かせる**: 一気に 500 行は事故る
- **テンプレと中身を分ける**: テンプレは固定、中身だけ AI が触る
- **Lighthouse スコアをチェックさせる**: アクセシビリティ・パフォーマンスは数値で評価
- **画像は遅延読み込み**: `loading="lazy"` を AI に必ず付けさせる
- **alt 必須**: SEO とアクセシビリティの両面で

---

### 11-10. 「HTML 1 ソース」で派生する成果物例

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

### 11-11. 落とし穴

- **CSS の単位混在**: px と % が混ざると PDF 化で崩れる。1 ソース内で統一
- **フォントの埋め込み忘れ**: `@font-face` を必ず Local + Web 両対応
- **絵文字フォントの差**: macOS と Linux で別物。CSS で明示
- **画像パス**: 絶対パス推奨。ファイルシステム → URL の差で詰む

---


---

## 12. Codex の機能をふんだんに使う上級編

### この章のゴール

- Codex の「カスタムコマンド」「サブエージェント」「フック」「MCP」「Worktree」を組み合わせて、人間の介入を最小化する
- 1 つの指示で「設計 → 実装 → テスト → デプロイ準備」までが流れる構成にする
- ミスりにくく、戻りやすい運用基盤を作る

---

### 12-1. カスタムコマンド（Slash Commands）

`~/.codex/commands/` に Markdown ファイルを置くだけで、`/コマンド名` で呼び出せます。

`~/.codex/commands/lp.md`：

```md
---
description: LP を 1 本丸ごと作る
---
Codex: Next.js + Tailwind + shadcn/ui で LP を作ってください。
要件は次の通り:

$ARGUMENTS

- ヒーロー / 3 features / pricing / FAQ / CTA
- 画像は GPTImage2.0※ で生成
- 完成後 pnpm dev で起動 → スクショで確認
- 確認後、CLAUDE.md を更新し、決定事項を memory/ に追記
```

これで `/lp "AI 副業スクール、紺ベース、CTA は LINE 登録"` の 1 行から LP が立ち上がります。

---

### 12-2. サブエージェント（AGENTS.md）

`AGENTS.md` に専門家ロールを定義しておくと、Codex がタスクに応じて使い分けます。

```md
### designer
- 役割: UI / UX の最終判断
- 必ず: アクセシビリティ AA を満たす配色を選ぶ
- 禁止: アニメーション過多

### copywriter
- 役割: マーケコピー
- 文体: ですます調 / フィラー禁止 / 1 文 40 字以内
- 必ず: 数字を具体的に入れる

### qa-engineer
- 役割: テスト設計と実行
- 必ず: e2e は Playwright、unit は Vitest
- 完了条件: CI で緑になること
```

Codex に「designer の視点でこのデザインをレビューして」と頼めば、自動で designer ロールを使います。

---

### 12-3. フック（hooks）

`~/.codex/hooks.json` でイベントごとに自動実行を仕込めます。

```json
{
  "PostToolUse": [
    { "matcher": { "tool": "Write", "path": "**/*.tsx" },
      "command": "pnpm exec tsc --noEmit" },
    { "matcher": { "tool": "Bash", "command": "git commit*" },
      "command": "pnpm exec lint-staged" }
  ],
  "UserPromptSubmit": [
    { "command": "node scripts/inject-design-tokens.js" }
  ]
}
```

- TSX を書いた瞬間に型チェック
- git commit の直前に lint-staged
- ユーザー指示の直前にデザイントークン JSON を context に注入

これだけで「型エラーを放置して push」のような失敗が物理的に起きなくなります。

---

### 12-4. MCP の活用

Model Context Protocol で外部サービスを Codex の文脈に接続。本資料での主要 MCP：

| 用途 | MCP サーバー |
| -- | -- |
| 画像生成 | OpenAI Images (gpt-image-2※) |
| デザイン | Canva / Figma 公式 MCP |
| ストレージ | GitHub / Cloud Drive |
| ブラウザ | Playwright MCP |
| データ | Notion / Linear / Airtable |

「`@figma file=xxx を読んで` 〜」のように **@サーバー** 経由で構造化データを引っ張れます。

---

### 12-5. Worktree（並行作業）

複数の作業を並行する時、Git Worktree を Codex から立てます。

```
Codex: git worktree add ../wt/feature-pricing -b feature-pricing
そこで pricing セクションだけ並行作業して。
完了したら main にマージ。
```

メインの作業を止めずに別作業を進められる。LP / 動画 / スライド の同時並走に有効です。

---

### 12-6. Plan モード

「これからの作業計画を見せて」と頼むと、Codex が手順だけを表で返します。

```
Codex /plan: LP を作る
→
1. Next.js セットアップ
2. shadcn/ui 導入
3. hero セクション
4. features セクション
5. pricing セクション
6. FAQ / CTA
7. レスポンシブ調整
8. テスト
9. プレビュー
```

OK / NG / 追加 を返すと実装が始まります。**いきなりコード書き出されない**ので安全。

---

### 12-7. Worker / 並列タスク

「これらは独立だから並列でやって」と指示すると、Codex がサブプロセスを起動して並走します。

```
Codex: 以下を並列で:
- A: GPTImage2.0※ で hero 画像 5 案
- B: コピー 5 パターン（GPT-5）
- C: pricing JSON を 3 プラン分作成
完了したら 3 つを統合して LP に組み込む
```

A / B / C が同時に走り、合計時間が 1/3 になります。

---

### 12-8. メモリ（個人運用ルール）

`memory/` ディレクトリに「失敗の記録」「決定事項」「定型文」を蓄積。

- `feedback_*.md`: やらかしと回避策（×2 回詰まったら必ず書く）
- `project_*.md`: 進行中の方針
- `reference_*.md`: 外部参照用の固定知識
- `user_*.md`: 個人プロファイル

Codex はセッション開始時にこれを自動読込し、毎回同じミスを再演しないように振る舞います。

---

### 12-9. CLAUDE.md / AGENTS.md の効きどころ

```md
## プロジェクト共通ルール

- 取り消せない操作の前は確認を取る（削除 / push --force / 公開 / 課金）
- 画像生成は必ず GPTImage2.0※（モデル ID は環境変数）
- LP は Tailwind v4 固定
- 1 機能 = 1 commit
- 完成後、変えた箇所 / 触っていない箇所 / 確認してほしい箇所 を末尾に明記
```

ルールを書いておけば、Codex は毎回これを踏まえて動きます。**口頭で毎回言わなくていい**。

---

### 12-10. 公開直前のセルフレビュー

`/review` カスタムコマンドで自分のコードをレビュー：

```
Codex /review:
- 直近の diff を読む
- 個人運用ルール ([CLAUDE.md](../../CLAUDE.md)) に照らす
- 取り消せない操作の混入 / 自動公開系の有無を検出
- 問題があれば指摘、なければ「OK」と返す
```

人間のレビュー前に、Codex 自身がチェック → 通ったら人間レビュー、の二段構え。

---


---

## 13. 実践ワークフロー集

### この章のゴール

- 日次・週次・月次で実際に回せるワークフローを 6 本掲載
- 各ワークフローは「コピペ → 自分用にパラメータ調整」だけで動く形に
- 個人運用想定（YouTube → Reels / TikTok / note / X 展開）

---

### 13-1. 日次: ショート動画 1 本

**所要 40 分 / コスト約 $3**

```
1. 朝の音声メモ（録音 5 分）
   ↓
2. Whisper で文字起こし
   ↓
3. GPT-5 で台本に整形（90 秒尺・3 アクト構成）
   ↓
4. HeyGen でアバター動画生成
   ↓
5. Remotion で字幕 + テロップ焼き込み
   ↓
6. ffmpeg で BGM ミックス
   ↓
7. assets/output/YYYY-MM-DD/shorts.mp4 として保存
   ↓
8. 公開は人間が手動（YouTube は BAN 対策で全手動）
```

Codex に `/shorts "topic"` カスタムコマンドを定義しておけば、上記が 1 コマンドで走ります。

---

### 13-2. 週次: LP 1 本（キャンペーン用）

**所要 90 分 / コスト約 $5**

```
1. ChatGPT に「今週のキャンペーン軸」を音声で相談
   ↓
2. Codex に App Shots※ で取得した参考 LP を渡す
   ↓
3. Codex が Next.js + Tailwind で LP を生成
   ↓
4. ヒーロー画像は GPTImage2.0※ で 3 案
   ↓
5. Codex がレスポンシブ調整 + e2e テスト
   ↓
6. プレビュー（人間チェック）
   ↓
7. main に commit + push（公開は人間が承認）
```

---

### 13-3. 週次: スライド資料 1 本（営業 / 教材）

**所要 60 分 / コスト約 $3**

```
1. Manus に「目的 / 対象 / 構成」を 3 行で伝える
   ↓
2. Manus が目次案を返す
   ↓
3. 人間が目次レビュー
   ↓
4. Manus が各ページのテキスト + GPTImage2.0※ 画像を生成
   ↓
5. python-pptx で組み立て → PPTX + PDF 出力
   ↓
6. Google Slides にアップロード（共有 URL 生成）
   ↓
7. ChatGPT で最終文言校正
```

---

### 13-4. 週次: バナー 30 枚一括

**所要 25 分 / コスト約 $1**

```
1. titles.csv に 30 件のタイトル
   ↓
2. Codex に「全件 1280x720 と 1080x1080 の 2 サイズで生成」
   ↓
3. GPTImage2.0※ で並列生成
   ↓
4. sharp で各媒体サイズに後処理
   ↓
5. assets/output/banners_YYYY-WW/ にまとめて保存
```

---

### 13-5. 月次: コーポレートサイト改修

**所要 4 時間 / コスト約 $20**

```
1. ChatGPT に「先月の問い合わせ傾向」を要約させる（Notion / Slack を読む）
   ↓
2. Codex で「改修箇所 5 つ」を git diff レベルで提示
   ↓
3. 人間が方針確定
   ↓
4. Codex × Figma で先にデザイン更新
   ↓
5. Codex でコード反映（PR を作る）
   ↓
6. Playwright で全画面回帰テスト
   ↓
7. プレビュー URL → 人間レビュー → merge
```

---

### 13-6. 月次: 全媒体 OGP 一括差し替え

**所要 30 分 / コスト約 $2**

```
1. ブランドカラー / フォントが変わったらこれを起動
   ↓
2. HTML テンプレ 1 本を更新
   ↓
3. Codex が全記事 N 本分の OGP を Playwright で書き出し
   ↓
4. CDN へ一括アップロード（Cloudflare R2 / S3 等、選定は任意）
   ↓
5. OGP の URL を CMS の各記事に紐付け直し
```

---

### 13-7. 「人間がやること」リスト（毎回）

| いつ | やること |
| -- | -- |
| 公開前 | ファクトチェック + 誇張表現除去 |
| 公開前 | 商標・肖像・音楽ライセンスの確認 |
| 公開時 | YouTube / TikTok への手動アップロード |
| 公開後 | 数値確認（再生 / 完了率 / CV）→ memory に学びを追記 |

AI に任せる範囲と、人間が責任を持つ範囲を **明文化** しておくのが事故防止のコツです。

---

### 13-8. 1 日のタイムテーブル例

```
08:00 朝の音声メモ
08:30 Codex /shorts でショート 1 本仕込み
09:30 LP / バナー / スライドの仕込み
12:00 Codex / Manus にバッチ運転を任せて昼休み
14:00 結果レビュー + 修正指示
16:00 Codex に翌日タスクの下書きまでやらせる
17:00 メモリ更新 + 翌日の優先タスクを memory に記録
```

---

### 13-9. KPI と振り返り

毎月以下を Codex / ChatGPT に集計させます。

- 生成本数（ショート / LP / スライド / バナー / 動画）
- 人間が触った時間
- AI が動いた時間（API コスト）
- 公開後の数値（再生 / CV / フォロワー増）
- 失敗・差し戻し件数（次月の改善材料）

「**何を AI に任せたら一番リターンが大きかったか**」を毎月可視化することで、運用が進化します。

---


---

## 14. トラブルシュート & 既知の落とし穴

### この章のゴール

- 本資料の構成を運用する中で実際に起きやすい問題と回避策を、1 ページで一覧化
- 「同じ詰まり方を 2 回しない」ための記録方法
- API 制限、コスト、品質劣化、運用事故の 4 カテゴリで整理

---

### 14-1. API・モデル系

| 症状 | 原因の典型 | 回避策 |
| -- | -- | -- |
| `RateLimitError: too many requests` | Tier が低い | OpenAI Platform で Tier 上げ申請 / 並列を 3 → 1 に下げる |
| `model_not_found: gpt-image-2` | モデル ID 表記の差 | 公式 Models 一覧で実際の ID を確認、`.env` に変数化 |
| 503 が連発 | サービス側障害 | OpenAI Status を確認、リトライ間隔を 1s → 30s に |
| 出力が途中で切れる | `max_tokens` が小さい | `max_output_tokens` を明示、長文は分割生成 |
| 日本語が崩れる画像 | プロンプト言語ミスマッチ | 「文字を入れない」プロンプトで生成 → Canva で後乗せ |

---

### 14-2. コスト系

| 症状 | 原因 | 回避策 |
| -- | -- | -- |
| 月末で予算オーバー | プロジェクト分離していない | キー / プロジェクトを用途別に分割。Soft / Hard Limit 必須 |
| 不要なリトライで課金 | 失敗時の指数バックオフが弱い | 3 回まで、間隔 1s → 4s → 16s。それ以上は止めて通知 |
| 画像生成が想定の 3 倍 | n=1 のつもりが n=4 | バッチ前にコスト試算（件数 × 単価）を Codex に出させる |
| 動画 1 本に $30 | Higgsfield※ / Seedance※ の高解像度を毎回使う | プロト版は SD、本番だけ HD に分岐 |

---

### 14-3. 品質・出力系

| 症状 | 原因 | 回避策 |
| -- | -- | -- |
| 同じ画像ばかり出る | seed 固定 / variant の与え方 | プロンプトの主題以外を毎回差し替える / variants 5 〜 8 |
| AI 文体っぽい | 「個性」「自分の言葉」が抜けた | ChatGPT に「私の note 過去 10 本のトンマナで」と例示渡し |
| LP の余白がスカスカ | 「広めに」と曖昧指示 | `--space-y-12` 等、CSS の単位で指示 |
| 動画の音ズレ | fps 混在 | 30fps 統一、ffmpeg で揃える |
| PPTX が崩れる | python-pptx 直書き | テンプレ PPTX を用意し、Placeholder に流し込む |
| 改ページが汚い | `@media print` 未設定 | `break-after: page;` を明示 |

---

### 14-4. 運用・人事故系

| 症状 | 原因 | 回避策 |
| -- | -- | -- |
| `.env` を誤コミット | gitignore 不備 | `secretlint` を pre-commit 必須化 |
| 古い動画を公開した | バージョン管理なし | ファイル名に YYYY-MM-DD-HHmm を必ず付ける |
| 商標違反 | 参考スクショの内容まで流用 | 「レイアウトだけ参考」を AI に明文化、最終目検 |
| 自動公開で大失敗 | YouTube / X / TikTok に自動投稿 | 公開系の自動化は禁止。最終ボタンは人間 |
| ファクトが嘘 | LLM が事実をでっち上げ | 公開前に Web 検索 + ChatGPT に「未確認なら未確認と返せ」 |

---

### 14-5. 「同じ失敗を 2 回しない」運用

`memory/` ディレクトリに `feedback_*.md` を蓄積します。テンプレ：

```md
---
name: feedback_xxx
type: feedback
date: 2026-05-26
---

### 何が起きたか
GPTImage2.0※ で生成した画像の文字が崩れた状態のまま公開してしまった

### 原因
生成直後の OCR チェックを忘れた

### 今後のルール
- 画像生成 → 必ず OCR で文字崩れ検出
- 文字を含む画像は GPTImage 出力にせず、Canva で後乗せ
```

詰まりを 2 回しない仕組みです。AI も人間も同じ。

---

### 14-6. デバッグの心得

- **「動いた / 動かなかった」を曖昧にしない**: 実機で確認するまで「動きます」と言わない
- **小さく試す**: 1 件で動かないものは 100 件でも動かない
- **ログを取る**: API 応答の raw を必ず保存。後でモデル比較や再現に効く
- **複数モデルで再検証**: 失敗時は GPT-5 → GPT-4o → Claude などで横並びチェック

---

### 14-7. パフォーマンス改善

- バッチ並列上限を **5** に固定（API レート的に安全）
- リクエスト前後で **キャッシュ**（同プロンプト・同 seed は再利用）
- 画像は **WebP** に変換して I/O を圧縮
- HTML レンダリング系（Puppeteer / Playwright）は **single instance** を使い回す

---

### 14-8. セキュリティ

- API キーは Keychain / Credential Manager に格納
- パブリック GitHub にはサンプル `.env.example` のみ
- `secretlint` の pre-commit を全プロジェクトに展開
- 月 1 回キーローテーション
- 不審な API 呼び出しを検知する Webhook を OpenAI Usage と連携

---


---

## 15. 未確認用語リストと参考リンク

### この章の目的

本資料で使用した固有名詞のうち、**2026-05 時点で実在 / 正式名称が公式に確認できなかった** ものを一覧化します。読者が実装する際は必ず一次情報（各社公式ドキュメント）で実在性を確かめてください。

`~/.claude/CLAUDE.md` の運用ルール（「日付・数値・固有名詞・引用を出すとき → 確証がなければ『未確認』と先に明示する」）に基づき、本資料は **未確認のものは未確認として併記する** 方針で書かれています。

---

### 15-1. 未確認固有名詞

| 本資料の表記 | 想定される正式名称 / 実在の候補 | 補足 |
| -- | -- | -- |
| **GPTImage2.0** | OpenAI の画像生成 API の最新世代を指す慣用表記。実モデル ID は `gpt-image-1` 系の後継（`gpt-image-2`?）の可能性 | 「2.0」というバージョン番号で公式リリースされた事実は本資料執筆時点で未確認 |
| **OpenAI Developer プラグイン** | VS Code / Codex CLI 向け OpenAI 公式拡張の総称として使用 | 「OpenAI Developer」名称の公式プラグインの存在は未確認。VS Code Marketplace 等で要確認 |
| **Higgsfield**（ユーザー原表記「Higgesfieid」） | カメラワーク特化の動画生成サービス。Higgsfield AI 社の同名プロダクトが該当の可能性 | API エンドポイントとパラメータは本資料の例（`/v1/videos`）が正しい保証なし |
| **Seedance**（ユーザー原表記「SeeDance」） | ByteDance 系の動画生成モデル | API URL（`api.seedance.bytedance.com`）は想定値。正式 URL は公式ドキュメントで確認 |
| **Manus** | 中国系発の自律 AI エージェント。国際版あり | 機能 / プラン / 料金は本資料執筆時点の挙動。最新仕様は要確認 |
| **Canva マジックレイヤー** | Canva の生成 AI 編集機能群を指す慣用表記。実在の Magic Studio 内機能（Magic Eraser / Magic Edit / Magic Grab / Magic Switch / Magic Write / Magic Expand）の総称か個別機能の別名かは未確認 | 「マジックレイヤー」という固有名称が公式に存在するかは未確認 |
| **App Shots**（ユーザー原表記「Appshots」） | スクリーンショット系ツール群を指す慣用表記。`App Shots` / `AppShots` 等の固有プロダクトが存在する可能性あり | 同等機能を持つ実在ツール: CleanShot X / Shottr / Screenshot (macOS 標準) など |
| **HeyGen MCP サーバー / Canva MCP サーバー / Figma MCP** | 各社が MCP サーバーを公式配布している前提で記述 | 2026-05 時点でどこまで公式配布されているかは公式リポジトリで要確認。非公式実装も多数 |

---

### 15-2. 一次情報の確認先

実装前に必ず確認してほしい一次情報：

- OpenAI Platform → Models 一覧 / API Reference
- OpenAI Developer Docs（最新のプラグイン / 拡張機能）
- Canva Developer Portal（API 認可 / 利用可能エンドポイント）
- Figma Developer Docs（Personal Access Token / MCP 仕様）
- HeyGen API Docs（モデル / 価格 / レート制限）
- Remotion 公式（最新の Composition API / レンダリング）
- ByteDance / Seedance 系の公式アナウンス
- Higgsfield AI の公式 Docs
- Manus 公式（プラン / API / ブラウザ自動化の制約）

---

### 15-3. なぜ「未確認」を明示するのか

生成 AI の領域は **3 ヶ月で API が刷新される** ことが珍しくありません。

- モデル ID が変わる
- レート制限が変わる
- 料金体系が変わる
- 名称自体が変わる（旧 `dall-e-3` → 新 `gpt-image-1` 等）

「2026-05 時点ではこう書いてあった」と日付付きで残し、**読者が一次情報を参照する習慣** を強制するのが本資料の方針です。

---

### 15-4. 著者からの注意書き

- 本資料の **コードサンプル** はすべて「**書き方の骨子**」であり、コピペで動く保証はありません。必ず各 SDK の最新版に合わせて手直ししてください
- **コスト試算** は概算です。Tier / 地域 / 為替で変動します
- **API 仕様の細部**（パラメータ名、レスポンス構造）は **未確認** を含みます

---

### 15-5. 改訂方針

本資料は GitHub 上で随時更新します。改訂タイミング：

- 各社が新モデル / 新 API を発表したとき
- 著者の運用で新しい裏技が見つかったとき
- 読者からの「ここ実在しないよ」報告があったとき（Issues / PR 歓迎）

---

### 15-6. 参考になりそうな関連スキル（個人運用）

筆者が日常で使っている関連自動化スキル：

- `/srt` — 日本語音声 → 意味区切り SRT 字幕
- `/cut` — Premiere Pro XML の無音 / 雑音自動カット
- `/note-post` — note 記事の本文 + 販促文生成（台本丸写し禁止）
- `/x-post` — YouTube 完成動画 → X 投稿文生成

これらと組み合わせると、本資料の各章で生成した素材が、そのまま自分の配信フローに乗ります。

---

### おわりに

OpenAI 1 本縛りでデザイン業務を全自動化する設計は、2026 年現在「**1 人で 10 人分の制作量**」を可能にします。
ただし、**未確認の固有名詞・最新仕様・著作権・人間判断が必要な領域** は必ず残ります。

「AI に任せられる範囲」と「人間が責任を持つ範囲」を線引きし、線の上に立つ運用ルール（`CLAUDE.md` / `AGENTS.md` / `memory/`）を育てる。これが本資料を本当に役立つ形で運用する唯一の道です。



---

