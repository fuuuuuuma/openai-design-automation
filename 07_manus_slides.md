# 07. Manus × GPTImage2.0 でスライド資料を生成

## この章のゴール

- Manus（自律 AI エージェント）に GPTImage2.0※ を呼ばせて、PPTX / PDF / Slides 用の素材を量産する
- 「目次 → 各スライド画像 → ノート → 最終 PPTX 出力」を 1 セッションで完了させる
- 大量の営業資料・社内教材を「テーマだけ与える」で生産する

> **※未確認**: Manus は 2025〜2026 にかけて急速に機能拡張中。本資料の手順は 2026-05 時点の挙動を前提に書きます。UI 名や設定パスは変わる可能性あり。

---

## 7-1. Manus の何がスライドに向いているか

Manus は「**ブラウザ・OS 操作・コード実行・ファイル管理**」を 1 エージェントで完結します。スライド作成においては：

- Markdown → PPTX の変換
- HTML → PDF の変換
- ブラウザで参照サイトを巡回（外部素材の収集）
- 画像生成 API（GPTImage2.0※ 等）を Manus から呼ぶ
- ZIP でまとめて納品

までを「1 つのタスク内」でやり切ります。**人間は最初の指示 1 回だけ**。

---

## 7-2. 基本フロー

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

## 7-3. Manus への指示テンプレ

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

## 7-4. GPTImage2.0※ 呼び出しの自動化

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

## 7-5. python-pptx での組み立て

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

## 7-6. 多言語展開

「同じ資料を 5 言語に展開して」と Manus に追加指示すれば、

- 各ページの text を GPT-5 で翻訳
- 画像のキャプションだけ言語別に再生成
- ファイル名を `sales_en.pptx` `sales_zh.pptx` のように分けて保存

までを自動で行います。

---

## 7-7. Google Slides 経由の場合

PPTX を Manus に Google Drive へアップロード → Slides に変換 → 編集権限を共有、までやらせます。

```
Manus: 出力した sales_v1.pptx を Google Drive にアップロードし、
Google Slides に変換、リンクは閲覧者:閲覧のみで発行。
URL を返して。
```

OAuth 連携が必要なので、初回だけ人間がブラウザでログイン承認します。

---

## 7-8. 大量量産のコツ

- **テンプレを 3 種類用意**: コーポレート / カジュアル / 教材。最初の指示で選ばせる
- **ブランドキットを JSON 化**: 色・フォント・余白を 1 JSON にしてテンプレ別に切替
- **画像生成を非同期で並列化**: 24 ページ分の画像を逐次にすると 10 分かかるが、並列なら 2 分
- **失敗ページのみ再生成**: 「Manus、ページ 12 だけ画像作り直して」で済むよう、ページ ID を必ず付与

---

## 7-9. コスト感

24 ページ資料 1 本：

| 項目 | コスト |
| -- | -- |
| GPT-5 文章生成 24 ページ分 | $0.5 〜 $1 |
| GPTImage2.0※ 24 枚 | $1 前後 |
| Manus 実行時間 8 分 | プラン内（要 Manus 課金） |
| 合計 | **$2 〜 $3 / 1 本** |

---

## 次の章へ

[08. Manus × Higgsfield × Seedance で映像生成 →](08_manus_higgsfield_seedance.md)
