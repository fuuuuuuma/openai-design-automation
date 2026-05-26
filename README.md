# OpenAI だけでデザイン全自動化 2026 — できること & プロンプト集

> **方針**: OpenAI API キー 1 本を起点に、Codex / Manus / ChatGPT を司令塔として外部プラグイン（Canva / Figma / HeyGen / Remotion / Higgsfield / Seedance / App Shots）を呼び出す。
> 各章は **「できること」+「オススメプロンプト」** だけ。前置きはなし。
> **未確認固有名詞**は本文の文末に `※未確認` で明示（GPTImage2.0 / Higgsfield / Seedance / App Shots / Canva マジックレイヤー / OpenAI Developer プラグイン）。

---

## 目次

- [01. Codex × GPTImage2.0（画像生成）](#01-codex--gptimage20画像生成)
- [02. Codex × Canva プラグイン（デザイン修正・量産）](#02-codex--canva-プラグインデザイン修正量産)
- [03. Codex × Figma プラグイン（コンポーネント / Variables 操作）](#03-codex--figma-プラグインコンポーネント--variables-操作)
- [04. Codex × HeyGen プラグイン（アバター動画）](#04-codex--heygen-プラグインアバター動画)
- [05. Codex × Remotion プラグイン（プログラマブル動画）](#05-codex--remotion-プラグインプログラマブル動画)
- [06. Codex 単体で LP / Web サイト / アプリ開発](#06-codex-単体で-lp--web-サイト--アプリ開発)
- [07. Manus × GPTImage2.0（スライド資料を自動生成）](#07-manus--gptimage20スライド資料を自動生成)
- [08. Manus × Higgsfield × Seedance（映像生成）](#08-manus--higgsfield--seedance映像生成)
- [09. ChatGPT × Canva マジックレイヤー（チャットだけで修正）](#09-chatgpt--canva-マジックレイヤーチャットだけで修正)
- [10. 裏技: App Shots で参考サイトを丸ごと取り込む](#10-裏技-app-shots-で参考サイトを丸ごと取り込む)
- [11. 裏技: HTML 駆動 + Codex 上級機能](#11-裏技-html-駆動--codex-上級機能)
- [12. OpenAI Developer プラグインで API キーを 1 本に集約](#12-openai-developer-プラグインで-api-キーを-1-本に集約)

---

## 01. Codex × GPTImage2.0（画像生成）

### できること
- YouTube サムネ / バナー / OGP / アイキャッチを 1 コマンドで量産
- 1 トピックから 5〜10 案バリエーション
- 参照画像を渡した Image-to-Image
- 生成 → 媒体サイズへ自動リサイズ（1280×720 / 1080×1920 / 1200×630 等）
- 文字崩れを GPT-5 で OCR 検証 → 自動再生成

### オススメプロンプト（バッチ生成）
```
Codex: scripts/gen-thumb.ts を作って。
- 引数: --topic "AI 副業" --variant 5
- gpt-image-2 で 1280x720 生成
- 出力: assets/thumb-{ts}-{n}.png
- 失敗は 3 回までリトライ
- 生成後 OCR で文字崩れを検出した画像は自動再生成
- 完了時に総コスト($)を標準出力
```

### オススメプロンプト（参考トンマナで再生成）
```
Codex: assets/refs/sankou.png のトンマナで、
被写体だけ「ノートPCを開く女性」に置き換えて 4 案。
no text / no logo / no watermark を必ず指定。
出力: assets/variants/{n}.png
```

### オススメプロンプト（媒体別一括リサイズ）
```
Codex: assets/output/base.png を sharp で
1280x720 / 1080x1080 / 1080x1920 / 1200x628 / 1200x630 の
5 サイズに fit:cover で書き出して。
ファイル名: base_{youtube|insta|reel|x|ogp}.png
```

> ※GPTImage2.0 はユーザー指定の慣用表記。実モデル ID は OpenAI Platform で確認。

---

## 02. Codex × Canva プラグイン（デザイン修正・量産）

### できること
- 既存デザインの複製・テキスト差し替え・画像差し替え
- ブランドキット / カラー / フォントを JSON 一括適用
- PDF / PNG / MP4 をバッチ書き出し
- 多言語展開（GPT-5 翻訳 → そのまま流し込み）
- テンプレ 1 つから SNS 30 件量産

### オススメプロンプト（多言語展開）
```
Codex: Canva の「春LP_メイン」を 5 言語複製。
日 → 英・韓・繁中・スペイン語。
- 翻訳は GPT-5、文字数は原文の 1.1 倍以内に圧縮
- レイアウト・余白・配色は完全に維持
- 完了したら各 PDF を assets/lp_lang/ に保存
```

### オススメプロンプト（SNS 量産）
```
Codex: テンプレ「sns_basic」から SNS バナーを量産。
入力: titles.csv（1列目: タイトル文）
出力サイズ: 1080x1080 と 1080x1920 の 2 種
ブランドカラー / フォントは brand-tokens.json から取得
出力先: assets/sns/YYYY-MM-DD/
```

### オススメプロンプト(差分だけ修正)
```
Codex: LP「春キャンペーン」のヒーロー画像「だけ」差し替え。
- 新画像: GPTImage2.0 で「桜 + ノートPC を中央配置、現行トンマナ維持」を 3 案生成
- ヒーロー以外のテキスト・配置・色は一切触らない
- 完了後 PNG と PDF を両方書き出し
```

> ※Canva プラグイン / MCP は公式 / 非公式が並走。最新は Canva Developer Portal で確認。

---

## 03. Codex × Figma プラグイン（コンポーネント / Variables 操作）

### できること
- ファイル / フレーム / ノード ID の取得
- Variables（デザイントークン）の一括書き換え
- コンポーネント差し替え
- PNG / SVG / PDF 書き出し
- コメント自動投稿で関係者通知
- コードの tailwind.config と双方向同期

### オススメプロンプト（カラー一括変更）
```
Codex: design-system の Button/Primary の Variables を
brand-color: #FF3B30 → #0070F3 に変更。
影響する全フレーム（14 ファイル想定）を PNG と SVG で
out/figma_export/ に書き出して。
最後に影響範囲を Markdown 表で出力。
```

### オススメプロンプト（コード ↔ Figma 同期）
```
Codex: Figma の Variables を読み、
tailwind.config.ts の colors / spacing / typography に同期。
差分があれば PR を作成、コミットメッセージは
「sync: figma → tailwind tokens」。
```

### オススメプロンプト（コンポーネント横断置換）
```
Codex: Hero/V1 を Hero/V2 に全置換。
- 影響を受けるページを列挙
- 各ページで自動置換
- レビュー用に before/after の PNG を並べたコンタクトシートを 1 枚作成
```

---

## 04. Codex × HeyGen プラグイン（アバター動画）

### できること
- アバター一覧取得（既存 / 自撮りクローン）
- 台本 → アバター動画の MP4 生成
- 多言語ボイス（日 / 英 / 韓 / 中 等）
- バッチ生成 + 進捗管理
- 文言だけ差し替え再レンダリング

### オススメプロンプト（バッチ生成）
```
Codex: scripts/heygen-batch.ts を作って。
入力: scripts/scripts.json
   [{"avatar":"fuuma_v3","voice":"ja-male-1","text":"..."}]
- HeyGen API で MP4 を並列 3 件まで生成
- 出力: assets/heygen/YYYY-MM-DD/{idx}.mp4
- 完了後 Slack Webhook に通知（mrkdwn 太字は使わない）
```

### オススメプロンプト（文言差し替え再生成）
```
Codex: 動画 ID heygen-abc123 の台本を
「月 100 万」→「月 300 万」に差し替えて再レンダリング。
他の演出・尺・BGM はそのまま再利用。
出力: assets/heygen/abc123-v2.mp4
```

### オススメプロンプト（多言語ローカライズ）
```
Codex: scripts/heygen-localize.ts を作って。
入力: 日本語動画 1 本の台本
- GPT-5 で英・韓・繁中の 3 言語に翻訳
- 各言語の HeyGen voice を自動マッピング
- 並列レンダリング、出力は {lang}/main.mp4
```

---

## 05. Codex × Remotion プラグイン（プログラマブル動画）

### できること
- React で動画を書く（JSX → MP4）
- 字幕 / 効果 / BGM / SE の合成
- 9:16 / 16:9 / 1:1 のアスペクト比切替
- HeyGen 出力との合成（アバター + テロップ）
- Whisper + GPT-5 字幕の自動焼き込み

### オススメプロンプト（オープニング）
```
Codex: components/HookOpening.tsx を Remotion で。
- 0〜1.5s: 黒背景に "3秒で説明します" フェードイン
- 1.5〜2.5s: hero.png をフェード + 拡大 1.0→1.05
- BGM: assets/bgm-light.mp3 を音量 0.3
- フォント: Noto Sans JP 700
- 1080x1920 (Reels)、30fps
- npx remotion render で MP4 書き出し
```

### オススメプロンプト（字幕焼き込み）
```
Codex: source.mp4 を Whisper で書き起こし →
GPT-5 で日本語の意味区切り改行 →
Remotion で字幕レイヤー（黒帯 + 白文字 + 影）作成 →
ffmpeg で本体 + 字幕 + BGM 合成。
出力: output/final.mp4
```

### オススメプロンプト（HeyGen × Remotion 合成）
```
Codex: heygen-abc123.mp4 をベースに、
Remotion でテロップ・効果音・カット切替を重ねて編集。
- 開幕にフック 3 秒のテキスト挿入
- BGM は loop 再生
- 終わりに CTA カードを 3 秒
最後に ffmpeg で 1 本の MP4 に合成。
```

---

## 06. Codex 単体で LP / Web サイト / アプリ開発

### できること
- LP を 30 分で 1 本（Next.js + Tailwind + shadcn/ui）
- 1 機能 SaaS を 2 時間で MVP（DB / Auth / フォーム / 管理画面）
- ヒーロー画像は GPTImage2.0 で自動生成 → 自動差し替え
- Playwright で e2e テストまで自動生成
- GitHub Actions の CI もセットアップ
- pnpm dev 起動 → スクショで自己確認 → 修正ループ

### オススメプロンプト（LP 1 本）
```
Codex: AI 副業学習サービスの LP を作って。
- Next.js App Router + Tailwind v4 + shadcn/ui
- 構成: hero / 3 features / pricing 3 / testimonials 3 / FAQ 4 / CTA
- 色: #0F172A（背景） / #38BDF8（アクセント） / 白文字
- フォント: Inter + Noto Sans JP
- 画像は GPTImage2.0 で生成（hero / features 3 枚）
- SP は 1 カラム
- /api/contact に POST するフォーム
- 起動して localhost:3000 をスクショで見せて
```

### オススメプロンプト（1 機能 SaaS）
```
Codex: 日記ジャーナルアプリを作って。
- Next.js + Prisma + SQLite + Auth.js (Email magic link)
- 機能: 投稿 / 一覧 / 編集 / 削除
- Server Components + Route Handler
- shadcn/ui デフォルト
- Playwright e2e (/, /signin, /journal/new, /journal/[id])
- 起動して動作確認スクショを返して
```

### オススメプロンプト（社内ツール）
```
Codex: 視聴者質問フォームを作って。
- Next.js + Resend + Vercel KV
- 送信時 Slack 通知（mrkdwn 太字を使わない）
- 管理画面で一覧 + 既読管理
- main に commit 前に必ずスクショ確認
```

### オススメプロンプト（既存 LP のリブランド）
```
Codex: 現状の LP を読んで、ブランドを「AI 収益化ラボ」に切替。
- ロゴ: assets/logo-2026.svg
- カラー: #0B1426 / #F5C518
- フォント: Noto Sans JP + Inter
- コピーは私の note 過去 10 本のトンマナで全文書き直し
- 画像は GPTImage2.0 で全差し替え
```

---

## 07. Manus × GPTImage2.0（スライド資料を自動生成）

### できること
- 目次 → 各ページ本文 + キー画像までを一気通貫生成
- python-pptx で PPTX 出力 + PDF 同時書き出し
- Google Slides にアップロード + 共有 URL 発行
- 多言語展開を 1 指示で
- 失敗ページだけ再生成（全体は触らない）

### オススメプロンプト（営業資料 1 本）
```
Manus、AI 収益化スクールの営業資料 PPTX を作って。
24 ページ構成:
  1: タイトル / 2: 概要 / 3-5: 課題提起 /
  6-12: 7 つの特徴 / 13-16: 受講生の声 + 実績 /
  17-20: カリキュラム / 21: 料金 3 プラン /
  22: 入会の流れ / 23: FAQ / 24: CTA
- 各ページ右側に GPTImage2.0 で生成したイメージ画像
- 配色: 紺と金、フォント: Noto Sans JP
- 出力: output/sales_v1.pptx + output/sales_v1.pdf
```

### オススメプロンプト（教材スライド）
```
Manus、「Codex 入門 60 分講座」の教材スライド 40 ページ。
- 各セクションごとに「概念」「コード例」「演習」の 3 枚 1 組
- コード例は実際に動くサンプルを Codex に書かせて検証
- 演習ページには 5〜10 行の穴埋めスクリプトを配置
- 出力: PPTX + PDF + Notion 同期版
```

### オススメプロンプト（多言語展開）
```
Manus、output/sales_v1.pptx を英・韓・繁中・西の 4 言語に展開。
- ファイル名: sales_en.pptx / sales_ko.pptx / sales_zh.pptx / sales_es.pptx
- 文字数は日本語版の 1.1 倍以内
- 画像のキャプションだけ言語別に再生成、画像本体は流用
```

### オススメプロンプト（Google Slides 連携）
```
Manus、output/sales_v1.pptx を Google Drive にアップロード。
Google Slides に変換し、閲覧専用の共有 URL を発行して返して。
権限: リンクを知っている人 → 閲覧者
```

> ※Manus は中国系発の自律 AI エージェント（国際版あり）。UI / プラン名は変更されやすい。

---

## 08. Manus × Higgsfield × Seedance（映像生成）

### できること
- Higgsfield: カメラワーク（ドリー / クレーン / オービット）が得意
- Seedance: 人物動作 / 物理表現（水・布）が得意
- Manus が指揮系統で 2 系統を組み合わせる
- 静止画 → 短尺動画 → 連結 → BGM + 字幕 までを 1 ジョブで
- 失敗ショットだけ差し戻し再生成

### オススメプロンプト（30 秒 PR 動画）
```
Manus、30 秒尺の AI スクール PR 動画を作って。
ショット設計:
1) Higgsfield: カフェで PC を開く女性、slow_zoom_in、5s
2) Higgsfield: ノート画面に AI アイコン、top-down 30deg、5s
3) Seedance: 笑顔で電話に出る男性、natural smile、5s
4) Seedance: 教材ロゴをホログラム表示、logo rotate、5s
5) Higgsfield: 暮らしが変わる瞬間、orbit、5s
6) Seedance: CTA「無料説明会へ」テキスト、subtle pulse、5s

- 画像ベースは GPTImage2.0 で全 6 枚先に生成
- BGM: assets/bgm/uplift_120bpm.mp3
- 字幕: captions/promo.srt（事前に Whisper + GPT-5 で整形）
- 出力: 9:16 vertical / 30fps / 1080x1920
- ffmpeg で結合 + BGM ミックス + 字幕焼き込み
```

### オススメプロンプト（差し戻し 1 ショットだけ）
```
Manus、ショット 3 の表情が硬い。
"warm smile, looking off-camera, slight head tilt" で
Seedance を再生成、ショット 3 だけ差し替えて全体を再結合。
他のショットは触らない。
```

### オススメプロンプト（縦横両対応）
```
Manus、同じ素材から 9:16（Reels / Shorts / TikTok）と
16:9（YouTube）の 2 本を同時書き出し。
9:16 では字幕を画面下 18% に、16:9 では下 8% に配置。
出力: output/promo_vertical.mp4 / promo_horizontal.mp4
```

> ※Higgsfield / Seedance はユーザー原表記の正式名称候補。API 仕様は各公式 Docs を要確認。

---

## 09. ChatGPT × Canva マジックレイヤー（チャットだけで修正）

### できること
- ブラウザだけで完結（スマホでも可）
- 既存 Canva デザインの文字差し替え
- 不要オブジェクト除去 + 背景補完
- 構図の再配置（左右入れ替え / 上下入れ替え）
- 音声入力で曖昧な日本語指示 → Canva の正確指示に翻訳

### オススメプロンプト（文字差し替え）
```
@Canva 「春キャンペーン LP」のヒーローテキストを
「新春 50% OFF」から「春の応援キャンペーン」に変更。
配置・フォント・色はそのまま。
完了したら PNG ダウンロード URL を出して。
```

### オススメプロンプト（オブジェクト除去）
```
@Canva 「営業資料 1 ページ」の左下の人物を
Magic Eraser 相当で削除。
背景を自然に補完して、削除後の PNG を返して。
```

### オススメプロンプト（構図入れ替え）
```
@Canva このスライドのテキストを左 50%・画像を右 50% に並べ替え。
余白は均等、タイトルだけ上部固定。
変更後のスクショを返して。
```

### オススメプロンプト（曖昧指示の通訳）
```
私: 「画像をふんわり、もっと暖色」を Canva の正確な指示に翻訳して
ChatGPT: → "Apply soft gaussian blur radius 4,
   reduce saturation by 15%, warm temperature +10"
→ そのまま @Canva に貼り付け
```

> ※「マジックレイヤー」はユーザー指定の慣用表記。実機能は Canva Magic Studio 群（Magic Edit / Magic Eraser / Magic Grab 等）に対応。

---

## 10. 裏技: App Shots で参考サイトを丸ごと取り込む

### できること
- URL を入れるだけで PC / SP / タブレット 3 サイズを自動キャプチャ
- セクション単位の自動切り抜き
- ZIP 化して Codex / Manus に丸ごと渡せる
- 動画もキーフレーム 6 枚で構造抽出
- 過去の自分の制作物もスクショ → 再生成の土台にできる

### オススメプロンプト（参考 LP → 自社 LP）
```
Codex: assets/refs/lp_ref.png を読み込み、構造を解析。
- セクション分割
- 各セクションの主要見出し / サブ文言 / CTA / 画像位置を抽出
- lp_structure.json として保存
- その JSON を元に Next.js + Tailwind で別ブランド LP を生成
- 文章 / 画像は私のブランドに合わせて新規生成
- レイアウト・余白だけ参考、ロゴ・文言・写真は完全置換
```

### オススメプロンプト（参考動画 → 別映像）
```
Codex: assets/refs/short_ref.mp4 のキーフレーム 6 枚を抽出。
GPT-5 にカット割り（編集の切替位置）と
各カットの構図 / 被写体 / 動きを分析させて JSON 化。
その JSON を Remotion / Manus に渡して
別映像を 30 秒尺で生成。
```

### オススメプロンプト（過去作の超え方）
```
Codex: 私の過去の YouTube サムネ 20 枚を assets/past/ に集約。
- 各画像の構図・色・文字配置の共通点を分析
- 「ベスト 3 の共通要素」を Markdown でまとめる
- それを上回るデザインの方向性を 3 案提示
- 各案について GPTImage2.0 でサンプル画像を 2 枚ずつ生成
```

> ※「App Shots」はユーザー指定の慣用表記。同等機能は CleanShot X / Shottr / OS 標準のスクショでも代替可。

---

## 11. 裏技: HTML 駆動 + Codex 上級機能

### HTML 駆動でできること
- 1 つの HTML ソースから LP / PDF / PPTX / OGP / メルマガまで派生
- CSS Variables でブランドを一括管理
- Puppeteer / Playwright で自動スクショ / PDF 化
- reveal.js でスライド化 → pdf2pptx で PowerPoint 化
- ブログ記事 100 本分の OGP を 5 分で量産

### オススメプロンプト（HTML から全派生）
```
Codex: content/q2_review.md を読み、
templates/slide.html に流し込んで out/slides/*.html を生成。
- 各 H2 を 1 スライドとして展開
- 画像は GPTImage2.0 で自動生成（プロンプトは H2 から派生）
- Puppeteer で out/q2.pdf に結合
- Playwright で各スライドを PNG にも書き出し
- ZIP で public/q2_review.zip
```

### オススメプロンプト（OGP 量産）
```
Codex: posts/ 配下のブログ記事 100 本分の OGP を量産。
- templates/ogp.html に title / excerpt を流し込み
- Playwright で 1200x630 PNG をスクショ
- 出力: out/ogp/{slug}.png
- 完了後、各記事の frontmatter に og_image パスを書き戻す
```

### Codex 上級機能 — オススメプロンプト集

**カスタムコマンド（`~/.codex/commands/lp.md`）**
```
/lp "AI 副業スクール、紺ベース、CTA は LINE 登録"
→ LP 1 本が立ち上がる
```

**サブエージェント定義（`AGENTS.md`）**
```
designer / copywriter / qa-engineer を AGENTS.md に定義し、
「designer の視点でレビューして」
「copywriter として文言を書き直して」
「qa-engineer として e2e を追加して」
で役割を切替
```

**フック（`~/.codex/hooks.json`）**
```
- TSX 保存時 → tsc --noEmit を自動実行
- git commit 前 → lint-staged + secretlint
- ユーザー指示時 → design-tokens.json を context に自動 inject
```

**並列タスク**
```
Codex: 以下を並列実行。
A: GPTImage2.0 で hero 画像 5 案
B: GPT-5 でコピー 5 パターン
C: pricing JSON を 3 プラン分作成
3 つ揃ったら統合して LP に組み込み、pnpm dev で起動。
```

**Plan モード**
```
Codex /plan: LP を作る
→ 実装前に手順だけテーブルで提示。
OK / NG / 追加 を返してから実装が始まる。
```

**Worktree（並行作業）**
```
Codex: git worktree add ../wt/feature-pricing -b feature-pricing
そこで pricing セクションだけ並行作業。
メイン作業は止めずに、完成したら main にマージ。
```

**MCP ブラウザ操作**
```
Codex: Playwright MCP で reference-site.com を開き、
ヒーロー / フィーチャー / 価格セクションのスクショを取得。
そのまま structure.json に変換して LP テンプレに流し込む。
```

**Codex セルフレビュー**
```
Codex /review:
- 直近の git diff を読む
- CLAUDE.md / AGENTS.md のルールに照らす
- 取り消せない操作 / 自動公開系の混入を検出
- 問題があれば指摘、なければ OK と返す
```

---

## 12. OpenAI Developer プラグインで API キーを 1 本に集約

### できること
- API キーを Keychain / Credential Manager に格納（環境変数に書かない）
- VS Code / Codex CLI / JetBrains でキーを共有
- 用途別プロジェクト分離（画像用 / 動画用 / 開発用）
- Soft / Hard リミットで自動課金停止
- secretlint で誤コミット防止

### オススメプロンプト（初期セットアップ）
```
Codex: 新規 OpenAI プロジェクト「codex-design-auto-2026」用に
API キーを 1 本発行（Permissions: Restricted, 有効期限 90 日）。
OpenAI Developer プラグインに登録し、
Storage backend は macOS Keychain。
.env への書き込みは禁止、.gitignore に .env* を追記。
secretlint を pre-commit にセットアップ。
```

### オススメプロンプト（コスト上限）
```
Codex: ~/.codex/config.toml にコスト制限を追加。
- max_tokens_per_request: 100000
- max_cost_per_session_usd: 5.0
- warn_at_usd: 1.0
さらに OpenAI Platform の Soft Limit を $100、Hard Limit を $200 に。
メール通知 ON。
```

### オススメプロンプト（用途別キー分離）
```
Codex: 用途別に API キーを 3 本発行。
- key-image: 画像生成専用
- key-video: 動画生成専用
- key-dev: 開発・コード生成専用
各キーは別々のプロジェクトに紐付け、月次の利用を分離集計できるよう設定。
.codex/config.toml で用途ごとに切替。
```

> ※「OpenAI Developer プラグイン」はユーザー指定の慣用表記。実体は VS Code Marketplace / JetBrains Plugins / OpenAI 公式拡張のいずれか。最新の正式名称は OpenAI 公式 Docs で確認。

---

## おわりに

- **司令塔は OpenAI（Codex / Manus / ChatGPT）**、実行隊は外部プラグイン（Canva / Figma / HeyGen / Remotion / Higgsfield / Seedance / App Shots）
- **公開ボタンは人間が押す**（YouTube / TikTok / X の自動投稿は BAN リスクで原則禁止）
- **「未確認」固有名詞** は実装前に必ず一次情報で実在性確認

---

ライセンス: CC0 1.0 Universal
