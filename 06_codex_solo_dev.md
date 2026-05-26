# 06. Codex 単体で LP / Web サイト / アプリ開発

## この章のゴール

- 外部プラグインを使わず Codex CLI 1 本だけで LP / コーポレートサイト / 1 機能 SaaS を完成させる
- 「対話 1 回 → アプリ起動」のテンポを掴む
- 公開（Cloudflare Pages / Netlify など、未確認のサービス名は読者側で選定）まで自動化する

---

## 6-1. なぜ Codex 単体で完結できるか

Codex は「コードを書く」だけでなく **Bash 実行・ファイル操作・パッケージ導入・テスト実行・サーバー起動** までを 1 セッションで行えます。
つまり「LP を作って」と命令すると、以下を自分で完結します：

1. `pnpm create next-app@latest` でプロジェクト生成
2. Tailwind v4 のセットアップ
3. ヒーロー / フィーチャー / 価格 / FAQ / CTA セクション作成
4. `pnpm dev` で起動 → スクショ確認
5. レビュー → 微修正 → 完成

人間がやることは「方針の指示」と「ブランドカラー渡し」だけです。

---

## 6-2. LP 1 本を 30 分で

### プロンプト例

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

## 6-3. ヒーロー画像の差し替え

LP の hero 画像は [03_codex_gptimage.md](03_codex_gptimage.md) のスクリプトで GPTImage2.0※ から生成。

```
Codex: hero 画像を gen-thumbnail.ts で生成して。
- topic: "AI を学ぶ社会人イメージ"
- size: 1920x1080
- 出力後、components/Hero.tsx の bgImage を差し替えて再起動
```

差し替え後は HMR で自動反映。

---

## 6-4. SaaS（1 機能アプリ）

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

## 6-5. 公開（デプロイ）

> 公開系はユーザー手動の確認を必ず挟む方針（環境による）

Codex に「Cloudflare Pages にデプロイして」と指示すると、

1. `wrangler login`（ブラウザが開く → 人間が承認）
2. `pnpm build` + `wrangler pages deploy`
3. デプロイ URL を取得

までを行います。`wrangler` の認証ステップだけ人間が踏めば、それ以降は完全自動。**ただし課金・公開のような取り消せない操作は事前に必ず確認** が運用ルール（個人運用方針）。

---

## 6-6. テストと CI

```
Codex: e2e テストを Playwright で書いて。
- /, /signin, /journal/new, /journal/[id] を回って 200 を確認
- フォーム送信が API に届くこと
- スクショは tests/screenshots/ に保存
- GitHub Actions で push 時に自動実行
```

`.github/workflows/ci.yml` まで自動生成されます。

---

## 6-7. 「失敗を許す」設計

Codex は時々ミスります。回避策：

- **小さい commit を強制**: AGENTS.md に「1 機能 = 1 commit」と明記
- **dev サーバーの起動確認を必須に**: 「最後に必ず http GET で 200 を確認」
- **テスト無しの完了報告を禁止**: 「テストを書いてから完了と言う」

これだけで Codex の「動かないコードの納品」が激減します。

---

## 6-8. 例: 個人運用で実際に作るもの（参考）

| 用途 | スタック | 所要時間 |
| -- | -- | -- |
| 教材販売 LP | Next.js + Stripe Checkout | 60 分 |
| 視聴者の質問フォーム | Next.js + Resend + KV | 30 分 |
| YouTube 字幕生成ツール | Next.js + Whisper + Remotion | 4 時間 |
| Notion 連携の予約表 | Next.js + Notion API | 90 分 |
| ChatGPT のラッパー（社内向け） | Next.js + OpenAI SDK | 120 分 |

---

## 次の章へ

[07. Manus × GPTImage2.0 でスライド資料を生成 →](07_manus_slides.md)
