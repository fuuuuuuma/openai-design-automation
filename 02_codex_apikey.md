# 02. Codex 環境と API キー管理（OpenAI Developer プラグイン）

## この章のゴール

- Codex CLI を導入し、OpenAI API キーを「**コードから見えない場所**」に置く
- 「OpenAI Developer プラグイン」※経由で API キーを統一管理する
- コスト上限と利用ログを最初の 10 分で仕込む

> **※未確認**: 「OpenAI Developer プラグイン」は本資料では「VS Code / Codex CLI 向けに OpenAI が配布する公式拡張」を指す呼称として用います。2026-05 時点で複数の派生（GitHub Marketplace 版・VS Code 拡張版・JetBrains 版）が並走しているため、最新の正式名称は OpenAI 公式ドキュメントで確認してください。

---

## ステップ 1. Codex CLI のインストール

```bash
# 公式 CLI（OpenAI が配布する Codex CLI）
npm i -g @openai/codex-cli
codex --version
```

代替として、JetBrains IDE 用プラグイン、VS Code 拡張版、Cursor 同梱版があります。本ガイドでは macOS の Terminal + Codex CLI を前提に進めます。

---

## ステップ 2. API キーの取得とスコープ

OpenAI Platform にログイン → **Settings → API keys → Create new secret key** で発行。発行時に次を必ず設定します。

- **Name**: `codex-design-auto-2026`（用途が分かる名前）
- **Permissions**: `Restricted`（read/write を最小化）
- **Project**: 専用プロジェクトを切る（コスト分離）
- **Expiration**: 90 日（更新を強制してローテーション）

> 1 つのキーに全責務を持たせない。画像生成専用、動画生成専用、開発専用で 3 本に分けるとローテーションが楽です。

---

## ステップ 3. OpenAI Developer プラグインに登録

OpenAI Developer プラグイン側で次の設定を行います（VS Code 拡張版を例に）。

1. コマンドパレット → `OpenAI: Sign in with API Key`
2. プロジェクト選択 → `codex-design-auto-2026`
3. **Storage backend**: macOS Keychain（Windows は Credential Manager）
4. **Default model**: `gpt-5-codex`（コード）/ `gpt-image-2`※ （画像）

これで `OPENAI_API_KEY` を環境変数に置かなくても、Codex / プラグイン経由の生成は Keychain から自動取得されます。

> **※未確認**: `gpt-image-2` というモデル ID は本資料の慣用表記。正確なモデル ID は Platform 上で確認すること。

---

## ステップ 4. シークレットを「絶対にコミットしない」設定

```bash
# プロジェクトルートで
echo ".env*"            >> .gitignore
echo "*.key"            >> .gitignore
echo "credentials.json" >> .gitignore
echo ".openai/"         >> .gitignore

# pre-commit フックで secret 検出
npm i -D @secretlint/secretlint @secretlint/secretlint-rule-preset-recommend
npx secretlint --init
```

`secretlint` を pre-commit で走らせると、`sk-` 始まりの文字列が含まれた瞬間に commit がブロックされます。

---

## ステップ 5. コスト上限と利用ログ

API 暴走対策は必須です。OpenAI Platform → **Settings → Limits** で次を設定します。

- **Hard limit**: 月 $200（個人運用の目安）
- **Soft limit**: 月 $100（メールで警告）
- **Email alerts**: ON
- **Usage tracking**: プロジェクト別で集計

加えて、Codex CLI 側でも 1 リクエスト単位の上限を入れます。

```bash
# ~/.codex/config.toml
[limits]
max_tokens_per_request = 100000
max_cost_per_session_usd = 5.0
warn_at_usd = 1.0
```

---

## ステップ 6. プロジェクトの基本構成

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

## ステップ 7. 動作確認

```bash
codex "Hello から動作確認。OPENAI_API_KEY 経由ではなくプラグインから読めているか、最初の応答に keychain と入れて返答してください"
```

応答に `keychain` が含まれていれば、シークレット経路が正しく通っています。

---

## 次の章へ

[03. Codex × GPTImage2.0 で画像を量産する →](03_codex_gptimage.md)
