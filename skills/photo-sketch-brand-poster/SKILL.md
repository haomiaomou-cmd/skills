---
name: photo-sketch-brand-poster
description: 把一张照片做成 3:4 竖版极简品牌海报——"实拍照片 × 手绘线稿"虚实拼贴风格。照片占画面约 1/5 偏居一侧，右侧以同透视铅笔线稿自然延续，暖奶油纸底、大量留白；文字由程序叠加保证印刷级清晰。当用户要求"照片+手绘线稿""虚实拼贴海报""照片里长出画""极简品牌海报"时使用。
---

# 照片 × 手绘线稿 品牌海报

**两段式：AI 只出画面（无文字）→ 程序叠文字。永远不要让图像模型渲染中文。**

## Step 1 · 生成画面

ImageGen 图生图：`image1`=原图，`input_fidelity=high`，`quality=high`，`size=1152x1536`。

```
Vertical 3:4 aspect ratio minimalist editorial brand poster artwork, absolutely no text/letters/numbers.
Background: flat warm cream ivory paper (#F4EEE1), subtle fine paper grain, low saturation.
Top 35% almost entirely empty cream paper (reserved type zone); bottom 20% also empty.
Middle of canvas, shifted slightly left: one wide horizontal band (~80% width, ~1/3 height), air above and below.
- LEFT: the reference photograph itself kept as a real film photo (warm muted colors, film grain), clean straight edges.
- RIGHT: starting exactly at the photo's right edge, the SAME scene continues in the SAME perspective as a loose naive hand-drawn pencil line sketch: [列出主体物件] keep extending into the drawing, casual childlike designer doodle linework, thin dark graphite strokes, no color/fill/shading, lines gradually sparser and dissolving into the cream paper. The drawing grows out of the photograph.
Style: quiet cafe-brand editorial aesthetic, asymmetric, refined print quality. No frame, border, watermark, signature, typography.
```

生成后用 Read 检查一次：照片≈1/5 偏左、线稿同透视延续、上下留白充足。不符则修正提示词重生成一次（最多 2 次）。

## Step 2 · 程序叠文字

1. **去水印**：从同行左侧 ~0.42W 处克隆干净纸纹，用高斯模糊羽化蒙版（核心 12px 实心 + blur 8）贴到右下角。
2. **排版**（全部左对齐，x 与照片左边缘对齐 ≈0.043W，深灰近黑）：

| 元素 | 字体 | 字号 | 位置 | 颜色 |
|---|---|---|---|---|
| 主标题（照片主题词，中英均可） | `msyhbd.ttc` | 0.080W | y 0.112H | (46,43,38) |
| 等宽英文两行（主题相关） | `consola.ttf` | 0.0235W | y 0.840H / 0.869H | (59,56,51) |
| 地址小字（中文） | `msyh.ttc` | 0.0215W | y 0.912H | (80,76,69) |

标题字距 ≈0.26em，逐字绘制。

3. Read 检查成片，交付 PNG。

## 环境与参考

- Python：`C:\Users\aasus\.workbuddy\binaries\python\envs\default\Scripts\python.exe`（Pillow 12.x 已装）
- 字体：`C:\Windows\Fonts\` 下 msyh.ttc / msyhbd.ttc / simhei.ttf / consola.ttf
- 可复用脚本：WorkBuddy 工作区 `2026-10-05-15-40-32/compose_poster.py`
- ImageGen 单张约 5-10 积分，**先告知用户**再调用
- 文案需贴合照片内容与主题；用户给了确切文字就照用
