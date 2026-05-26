# 04. Codex × Canva / Figma プラグイン連携

## この章のゴール

- Codex から Canva と Figma を MCP / プラグイン経由で操作する
- 既存テンプレに対して「差分修正だけ AI に投げる」運用に切り替える
- 1 つの素材を Canva と Figma 両方に同期させる

---

## 4-1. Canva プラグイン

### セットアップ

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

### できること

- デザイン検索 → ID 取得
- 既存デザインの複製と編集
- 画像・テキスト要素の差し替え
- PDF / PNG / MP4 でエクスポート

### Codex への依頼例

```
Codex: 「2026春LP_メイン」というデザインを 5 言語に複製して。
日本語 → 英語 / 韓国語 / 繁体字 / 簡体字 / スペイン語。
テキスト翻訳は GPT-5 を経由。レイアウト崩れを避けるため文字数を 1.1 倍以内に圧縮。
完了したら各 PDF を assets/output/lp_lang/ に保存。
```

Codex が翻訳 → Canva API での差し替え → エクスポート、までを一括実行します。

---

## 4-2. Figma プラグイン（公式 MCP）

### セットアップ

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

### できること

- ファイル / フレーム / ノード ID の取得
- Variables の読み書き（デザイントークン）
- コンポーネント差し替え
- 画像書き出し（PNG / SVG / PDF）
- コメント投稿

### 「コードからデザインを動かす」例

```
Codex: design-system ライブラリの「Button/Primary」コンポーネントを、
Variables の brand-color = #FF3B30 → #0070F3 に差し替えて。
影響する 14 ファイルすべて plays 用に書き出して、output/figma_export/ に保存。
```

これで色変更とエクスポートが 1 コマンドで終わります。手作業なら半日コース。

---

## 4-3. Canva ↔ Figma 同期

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

## 4-4. 差分修正 (Patch Mode)

毎回全生成しないために「差分指示」だけ送る運用にします。

```
Codex: 既存 LP「春キャンペーン」のヒーロー画像のみ差し替え。
- 元: 桜のイラスト
- 新: 桜 + ノートPC を中央配置、トンマナそのまま
- それ以外の要素は触らない
```

GPTImage2.0※ で新画像生成 → Canva プラグインで該当ノードのみ replace_image → PDF 書き出し。**全体は触らない** が鍵。

---

## 4-5. 「修正だけ Canva」「初稿は Figma」の役割分担

経験則ですが次が機能します。

| フェーズ | ツール | 理由 |
| -- | -- | -- |
| ワイヤー設計 | Figma | コンポーネント化と Variables が強い |
| 初稿〜中稿 | Figma | 履歴管理・複数人レビュー |
| 最終調整・量産 | Canva | テンプレ流用・媒体別書き出しが速い |
| 緊急修正 | ChatGPT × Canva マジックレイヤー※ | ブラウザだけで完結 |

---

## 4-6. 既知の罠

- **Canva 無料プランは API が制限される** — Pro / Teams 契約推奨
- **Figma のフレーム ID は URL から抜くのが速い** — `?node-id=` の値
- **画像差し替え後にキャッシュが残る** — Canva はブラウザ側でリロード必須
- **トークン名の衝突** — Figma Variables と Canva ブランドキットで命名規約を同じにする

---

## 次の章へ

[05. Codex × HeyGen / Remotion で動画を作る →](05_codex_video_plugins.md)
