# 05. Codex × HeyGen / Remotion で動画を作る

## この章のゴール

- アバター動画は **HeyGen**、プログラマブル動画は **Remotion** で作り、両者を Codex から統合制御する
- 「台本 → 音声 → 動画 → 字幕 → 書き出し」を 1 ジョブで完了させる
- 動画の差分修正（文言だけ差し替え）を自動化する

---

## 5-1. HeyGen プラグイン

### セットアップ

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

### できること

- 既存アバター（人物 / 自撮りクローン）の一覧
- 台本入力 → アバター動画生成
- 多言語ボイス（日本語 / 英語 / 韓国語等）
- 書き出し（MP4 / WebM）

### Codex への指示例

```
Codex: scripts/heygen-batch.ts を作って。
- 入力: scripts/scripts.json（[{"avatar": "fuuma_v3", "voice": "ja-male-1", "text": "..."}]）
- HeyGen API を呼んで MP4 を生成
- 完了通知は Slack Webhook に投げる（本文に * を使わない）
- 同時並列は 3 ジョブまで
```

---

## 5-2. Remotion プラグイン

Remotion は「React で動画を書く」フレームワーク。Codex と相性が最高で、JSX を書ければそのまま動画になります。

### セットアップ

```bash
npx create-video@latest my-remotion-pj --template hello-world
cd my-remotion-pj
npm install
```

### Codex に書かせる例

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

## 5-3. HeyGen + Remotion のハイブリッド

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

## 5-4. 差分修正フロー

YouTube ショート量産で頻発する「文言だけ差し替えたい」を高速化します。

```
Codex: 動画 ID heygen-abc123 の台本だけ差し替え。
- 旧: 「AI で月 100 万」
- 新: 「AI で月 300 万」
- 他の演出・尺・BGM はそのまま再利用
```

HeyGen の `script` フィールドだけ更新 → 再レンダリング。約 2〜3 分で差し替え完了。

---

## 5-5. 字幕の自動生成

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

## 5-6. 動画コスト管理

| 項目 | 単価目安 | 月運用想定 |
| -- | -- | -- |
| HeyGen 60秒動画 | 約 $0.5 〜 $2 | 30 本で $15〜$60 |
| Remotion 自前レンダ | 電気代のみ | 〜 |
| Whisper 文字起こし | $0.006/分 | 10 時間で $3.6 |
| GPT-5 字幕整形 | $0.02 / 1 万字 | $1 程度 |

トータル月 $80 〜 $150 で「短尺動画 30 本 + 字幕付き」が回ります。

---

## 5-7. 既知の落とし穴

- **HeyGen のアバター生成は本人同意必須** — 商用利用ではライセンス契約を確認
- **Remotion の `delayRender()` を忘れると Promise が壊れる** — 非同期 fetch 時は必須
- **MP4 の音ズレ** — 30fps と 29.97fps を混ぜると 1 分で 2 フレームずれる。pipeline 全体を 30 固定に統一
- **書き出し時のフォント未埋め込み** — Remotion 側で `staticFile()` 経由のフォント読み込みを徹底

---

## 次の章へ

[06. Codex 単体で LP / Web サイト / アプリ開発 →](06_codex_solo_dev.md)
