# 01. 全体像と OpenAI 1 本縛りの設計思想

## なぜ「OpenAI だけで」完結できるのか

2026 年現在のデザイン業務自動化は、もはや 1 ツールでは完結しません。画像生成、動画生成、デザイン編集、Webサイト構築、それぞれに最適化された専用サービスが乱立しています。
そこで本ガイドは「**OpenAI の API キー 1 本を起点として、外部ツール群を呼び出して全自動化する**」というアーキテクチャを採用します。OpenAI の API キーで生成系（GPTImage2.0※ / GPT-5 / Realtime API）を駆動し、Codex や Manus のエージェント能力で外部 SaaS のプラグイン API を叩く構成です。

> **※未確認**: 「GPTImage2.0」は本資料では「OpenAI が提供する最新画像生成 API の総称」として扱います。2026-05 時点での正式モデル ID は OpenAI Platform の Models 一覧で都度確認してください（例: `gpt-image-1` 系の後継）。詳細は [15_unverified_terms.md](15_unverified_terms.md) を参照。

---

## 設計原則 5 つ

### 1. 「主語」を OpenAI にする

外部ツールを使うときも、必ず Codex / Manus / ChatGPT のいずれかの OpenAI エージェントが「主語」として実行ループを回します。プラグインや MCP サーバーは「呼ばれる側」。これにより、人間が触るインターフェースが OpenAI 系に統一され、操作学習コストが圧倒的に減ります。

### 2. API キーは 1 本に集約

OpenAI Developer プラグイン（VS Code / Codex CLI 向け）で API キーを 1 ヶ所に集約管理します。`OPENAI_API_KEY` を環境変数にせず、専用のシークレットストアに入れることで、誤コミットを防ぎます。詳しくは [02_codex_apikey.md](02_codex_apikey.md)。

### 3. 中間表現は Markdown / HTML / JSON のいずれか

ツール間でデータを受け渡す中間表現は、人間が読めて LLM が再編集できる Markdown / HTML / JSON のいずれかに統一します。Figma の独自バイナリや Canva の独自フォーマットを最終成果物としてだけ使い、編集途中はテキストで保つ設計です。

### 4. 「やり直し可能性」を最優先

生成 AI は失敗します。必ず「生成 → プレビュー → 修正」のループを 2〜3 周回す前提で組みます。1 発で完成を目指さない。Codex のチェックポイント機能、Manus のスナップショット、ChatGPT のチャット履歴を「セーブポイント」として使い倒します。

### 5. 公開系の操作は自動化しない

YouTube への投稿、X への自動投稿、課金アクションの実行は、本ガイドではすべて「生成・保存まで」で止めます。BAN リスクとアカウント停止の被害が大きすぎるため、最終ボタンは人間が押す原則です。

---

## カバーする生成物の種類

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

## 「OpenAI 縛り」が崩れる例外

外部生成系を使う場面が 2 つだけあります。

1. **動画生成の Seedance**（中国 ByteDance 系・物理表現に強い）  
2. **Higgsfield**（カメラワーク制御が得意なモーション系）

これらは OpenAI Sora 系で代替可能な日もありますが、2026-05 時点では「画作りの幅」を優先して併用します。ただし呼び出し元の指揮は Manus（OpenAI モデルが頭脳）にすることで、運用主体は OpenAI に保ちます。

---

## 章間の依存関係

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

## 次の章へ

[02. Codex 環境と API キー管理 →](02_codex_apikey.md)
