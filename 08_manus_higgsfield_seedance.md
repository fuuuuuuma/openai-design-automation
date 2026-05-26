# 08. Manus × Higgsfield × Seedance で映像生成

## この章のゴール

- Manus を指揮系統として、Higgsfield※ と Seedance※ の 2 系統の動画生成を組み合わせる
- カメラワーク（Higgsfield 強み）× 物理表現・人物動作（Seedance 強み）の役割分担
- 「画像 → 短尺動画 → 連結 → BGM・字幕 → 公開用 MP4」までの全自動パイプライン

> **※未確認**:
> - 「Higgsfield」はカメラ制御に強いとされる動画生成サービスの慣用表記（ユーザー原表記「Higgesfieid」を正式名称候補に修正併記）。
> - 「Seedance」は ByteDance 系の動画生成モデル（ユーザー原表記「SeeDance」も同義として扱う）。
> どちらも正確な API 仕様は 2026-05 時点の公式情報を要確認。

---

## 8-1. なぜ「OpenAI 縛り」なのに外部モデルを使うのか

[01_overview.md](01_overview.md) の通り、OpenAI Sora 系で大半は賄えますが、

- **カメラワーク**（ドリー / クレーン / オービット）の自由度は Higgsfield※ の方が現時点で高い
- **人物の自然な動き**（ダンス / 演技）と背景物理（水・布）は Seedance※ がリードする日がある

そのため「OpenAI モデルを脳として、外部モデルを画作りに使う」役割分担を採用します。**指揮は Manus**、つまり OpenAI 系のエージェントが投げる構図を守ります。

---

## 8-2. パイプライン全景

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

## 8-3. ショット設計

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

## 8-4. Higgsfield※ 呼び出し例

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

## 8-5. Seedance※ 呼び出し例

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

## 8-6. Manus が組む編集スクリプト

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

## 8-7. 失敗カットの差し戻し

Manus に「ショット 3 の人物の表情が硬い」と伝えるだけで、

1. 該当 shot の prompt を微修正（"warm smile, looking off-camera"）
2. Seedance※ で再生成
3. 該当 shot だけ差し替え
4. 結合を再実行

を自動で回します。**全体再生成しない**のがコスト・時短の鍵。

---

## 8-8. 公開直前の人間チェック

- 商標 / 肖像権 / 音楽ライセンス
- 字幕の意味区切り（[srt スキル](../README.md) と同じ要領）
- 16:9 / 9:16 の切替時のクロップ
- 終了画面の CTA URL タイポ

ここだけは必ず人間が見ます。公開系の不可逆操作（YouTube / TikTok 投稿）は手動運用が原則。

---

## 8-9. コスト目安

| 項目 | 単価 | 30 秒尺 |
| -- | -- | -- |
| Higgsfield※ 5 秒尺 | 約 $0.4 〜 $1 | 3 ショットで $1.2 〜 $3 |
| Seedance※ 5 秒尺 | 約 $0.3 〜 $0.8 | 3 ショットで $0.9 〜 $2.4 |
| GPTImage2.0※ 静止画 6 枚 | $0.24 | $0.24 |
| Manus 実行時間 | プラン内 | プラン内 |
| 合計 | — | **$3 〜 $6 / 1 本** |

---

## 次の章へ

[09. ChatGPT × Canva「マジックレイヤー」修正フロー →](09_chatgpt_canva_magic.md)
