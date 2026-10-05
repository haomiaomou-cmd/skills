---
name: photo-sketch-brand-poster
description: 把一张照片做成 3:4 竖版极简品牌海报——"实拍照片 × 手绘线稿"虚实拼贴风格。照片占画面约 1/5 偏居一侧，右侧以同透视铅笔线稿自然延续，暖奶油纸底、大量留白；文字分三种笔迹由程序叠加：粗黑体标题 + 打字机段落 + 拙感手写批注。当用户要求"照片+手绘线稿""虚实拼贴海报""照片里长出画""极简品牌海报"时使用。
---

# 照片 × 手绘线稿 品牌海报

**两段式：AI 只出画面（无文字）→ 程序叠三层文字。永远不要让图像模型渲染文字。**

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

生成后 Read 检查：照片≈1/5 偏左、线稿同透视延续、上下留白充足。不符则修正重生成一次（上限 2 次）。

## Step 2 · 程序叠文字（三层笔迹）

坐标基数：`W, H` = 画布宽高；`MARGIN_X = 0.043W`（与照片左边缘对齐）；全部左对齐；墨色统一在 (46,43,38) 附近。

### 2.1 去水印
克隆同行左侧 ~0.42W 处的干净纸纹，用高斯模糊羽化蒙版（核心 12px 实心 + blur 8）贴到右下角。

### 2.2 主标题（粗黑体，宽字距）
取自照片里的可见文字（如墙上招牌）或主题词。

| 参数 | 值 |
|---|---|
| 字体 | `msyhbd.ttc` |
| 字号 | 0.080W |
| 位置 | (MARGIN_X, 0.104H) |
| 字距 | 0.26em，逐字绘制 |
| 颜色 | (46,43,38) |

### 2.3 打字机段落（粗体 + 紧字距）
2-3 句围绕照片主题的英文散文（不是大写短句），自然换行。

| 参数 | 值 |
|---|---|
| 字体 | `courbd.ttf`（Courier Bold） |
| 字号 | 30px |
| 起始 | (MARGIN_X, 0.190H) |
| 行距 | 21px（0.7 倍行高，紧实文字块） |
| 颜色 | (46,43,38)，与主标题同墨色 |
| 字距 | **逐字绘制，步进 = `draw.textlength(ch) × 0.90`** |

### 2.4 手写批注（拙感，1-3 句）
字体 `Inkfree.ttf`，**逐字独立随机**制造不规律感，`random.seed(7)` 可复现：

- 基线抖动 oy ±5｜单字旋转 ±6°｜单字缩放 0.93–1.08
- 墨色 alpha 200–245（下笔轻重不一）
- 步进 0.57em ± 抖动；空格 0.40em

整句再整体轻微旋转（如 -4°），RGBA 图层 `rotate` 后 `alpha_composite`。
位置放在画面/线稿上，跨接照片与线稿边界处最自然；内容取自画面物件（热水瓶、年份、器物）。

## 关键经验

- **等宽字体压缩字距**必须逐字绘制（`textlength × 系数`），anchor/kerning 无效
- **手写字距抖动方差别过大**、空格别压太窄，否则单词粘连
- **英文块不要低于 0.27H**，否则压到画面顶部边缘
- 三种笔迹共用一个墨色系，只变字体与字号，保持印刷级高级感
- 修改迭代时优先微调：亮度/位置/字重，一次只改一个变量

## 环境

- Python：`C:\Users\aasus\.workbuddy\binaries\python\envs\default\Scripts\python.exe`（Pillow 12.x）
- 字体：`C:\Windows\Fonts\` — msyhbd.ttc / courbd.ttf / Inkfree.ttf / consola.ttf
- 可复用脚本：工作区 `2026-10-05-15-40-32/compose_poster.py`
- ImageGen 单张约 5-10 积分，**先告知用户**再调用；文案贴合照片内容，用户给了确切文字就照用
