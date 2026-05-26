# OpenAI だけでデザイン業務を全自動化する 2026 年完全ガイド

> **対象**: クリエイター・YouTuber・LP制作者・スライド量産担当・1人マーケター
> **方針**: OpenAI API キー 1 本を起点に、Codex / Manus / ChatGPT / 外部プラグイン（Canva / Figma / HeyGen / Remotion / Higgsfield / Seedance）まで横断し、画像・動画・スライド・LP・Webサイト・アプリ生成までを全自動化する
> **更新**: 2026-05 時点

---

## 目次

| #  | 章 | 内容 |
| -- | -- | -- |
| 01 | [全体像と OpenAI 1 本縛りの設計思想](01_overview.md) | なぜ OpenAI 起点で完結できるのか、外部ツール接続の考え方 |
| 02 | [Codex 環境と API キー管理（OpenAI Developer プラグイン）](02_codex_apikey.md) | API キーの取り回し、秘匿管理、コスト管理 |
| 03 | [Codex × GPTImage2.0 で画像を量産する](03_codex_gptimage.md) | Codex から直接画像生成、バッチ処理、リサイズ |
| 04 | [Codex × Canva / Figma プラグイン連携](04_codex_design_plugins.md) | デザイン編集の自動化、テンプレ流し込み |
| 05 | [Codex × HeyGen / Remotion で動画を作る](05_codex_video_plugins.md) | アバター動画 + プログラマブル動画の合成 |
| 06 | [Codex 単体で LP / Web サイト / アプリ開発](06_codex_solo_dev.md) | ゼロからの 1 人 SaaS 量産 |
| 07 | [Manus × GPTImage2.0 でスライド資料を生成](07_manus_slides.md) | プレゼン資料・営業資料・教材の量産 |
| 08 | [Manus × Higgsfield × Seedance で映像生成](08_manus_higgsfield_seedance.md) | 物理ベース映像と中華系動画モデルの組合せ |
| 09 | [ChatGPT × Canva「マジックレイヤー」修正フロー](09_chatgpt_canva_magic.md) | チャットだけでデザイン修正完結 |
| 10 | [裏技: App Shots で参考サイトを呼び出す](10_appshots_advanced.md) | 参照デザインを高速取り込み |
| 11 | [HTML 駆動の上級ワークフロー](11_html_workflow.md) | HTML を中間言語にして全媒体に展開 |
| 12 | [Codex の機能をふんだんに使う上級編](12_codex_power_features.md) | カスタムコマンド、サブエージェント、フック |
| 13 | [実践ワークフロー集](13_workflows_real.md) | 1 日 1 LP / 週次スライド / 月次レポート |
| 14 | [トラブルシュート & 既知の落とし穴](14_troubleshooting.md) | API 制限、レート、課金、品質劣化対策 |
| 15 | [未確認用語リストと参考リンク](15_unverified_terms.md) | 2026-05 時点で実在/正式名称未確認の固有名詞まとめ |

---

## 5 分で掴む全体像

```
[OpenAI API キー]
       │
       ├──► [Codex CLI]
       │       ├──► GPTImage2.0  ※未確認 → 画像生成
       │       ├──► Canva プラグイン  → デザイン修正
       │       ├──► Figma プラグイン → コンポーネント自動配置
       │       ├──► HeyGen プラグイン → アバター動画
       │       └──► Remotion プラグイン → プログラマブル動画
       │
       ├──► [Manus]
       │       ├──► GPTImage2.0  → スライド画像
       │       └──► Higgsfield × Seedance → 動画生成
       │
       └──► [ChatGPT UI]
               └──► Canva マジックレイヤー  ※未確認 → 即時修正
```

---

## 前提環境

- **macOS** または **Windows 11**（本資料の例は macOS で記述）
- **Node.js 22 LTS 以上**（Codex CLI / Remotion 共通要件）
- **OpenAI API 課金アカウント**（Tier 1 以上推奨）
- **Codex CLI**（OpenAI 公式）または互換 CLI
- **Manus アカウント**（自律エージェント。中国系発・国際版あり）
- **Canva Pro / Figma 有料プラン**（プラグイン API 経由は有料プラン要件のことが多い）

---

## 本ガイドの読み方

1. **まず 01_overview.md を読む** — 全体構造を頭に入れる
2. **02 → 06 を順に試す** — Codex 環境の構築 → 単機能 → 統合
3. **07 → 09 で別エージェントを経由する選択肢を学ぶ** — Codex で足りない場面の補完手段
4. **10 → 12 を裏技として参照** — 量と速度を出すための上級テクニック
5. **13 でレシピを真似る** — 実際の日次/週次ワークフローを写経する

---

## ライセンス

CC0 1.0 Universal（自由に転用可。ただし本資料内の「未確認」マーク付き情報は読者側で実在性を確認すること）。
