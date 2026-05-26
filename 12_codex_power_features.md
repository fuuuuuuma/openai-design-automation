# 12. Codex の機能をふんだんに使う上級編

## この章のゴール

- Codex の「カスタムコマンド」「サブエージェント」「フック」「MCP」「Worktree」を組み合わせて、人間の介入を最小化する
- 1 つの指示で「設計 → 実装 → テスト → デプロイ準備」までが流れる構成にする
- ミスりにくく、戻りやすい運用基盤を作る

---

## 12-1. カスタムコマンド（Slash Commands）

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

## 12-2. サブエージェント（AGENTS.md）

`AGENTS.md` に専門家ロールを定義しておくと、Codex がタスクに応じて使い分けます。

```md
## designer
- 役割: UI / UX の最終判断
- 必ず: アクセシビリティ AA を満たす配色を選ぶ
- 禁止: アニメーション過多

## copywriter
- 役割: マーケコピー
- 文体: ですます調 / フィラー禁止 / 1 文 40 字以内
- 必ず: 数字を具体的に入れる

## qa-engineer
- 役割: テスト設計と実行
- 必ず: e2e は Playwright、unit は Vitest
- 完了条件: CI で緑になること
```

Codex に「designer の視点でこのデザインをレビューして」と頼めば、自動で designer ロールを使います。

---

## 12-3. フック（hooks）

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

## 12-4. MCP の活用

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

## 12-5. Worktree（並行作業）

複数の作業を並行する時、Git Worktree を Codex から立てます。

```
Codex: git worktree add ../wt/feature-pricing -b feature-pricing
そこで pricing セクションだけ並行作業して。
完了したら main にマージ。
```

メインの作業を止めずに別作業を進められる。LP / 動画 / スライド の同時並走に有効です。

---

## 12-6. Plan モード

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

## 12-7. Worker / 並列タスク

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

## 12-8. メモリ（個人運用ルール）

`memory/` ディレクトリに「失敗の記録」「決定事項」「定型文」を蓄積。

- `feedback_*.md`: やらかしと回避策（×2 回詰まったら必ず書く）
- `project_*.md`: 進行中の方針
- `reference_*.md`: 外部参照用の固定知識
- `user_*.md`: 個人プロファイル

Codex はセッション開始時にこれを自動読込し、毎回同じミスを再演しないように振る舞います。

---

## 12-9. CLAUDE.md / AGENTS.md の効きどころ

```md
# プロジェクト共通ルール

- 取り消せない操作の前は確認を取る（削除 / push --force / 公開 / 課金）
- 画像生成は必ず GPTImage2.0※（モデル ID は環境変数）
- LP は Tailwind v4 固定
- 1 機能 = 1 commit
- 完成後、変えた箇所 / 触っていない箇所 / 確認してほしい箇所 を末尾に明記
```

ルールを書いておけば、Codex は毎回これを踏まえて動きます。**口頭で毎回言わなくていい**。

---

## 12-10. 公開直前のセルフレビュー

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

## 次の章へ

[13. 実践ワークフロー集 →](13_workflows_real.md)
