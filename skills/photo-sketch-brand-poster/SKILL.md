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
字体 `Inkfree.ttf`，**字号 0.030W**，**逐字独立随机**制造不规律感，`random.seed(7)` 可复现：

- 基线抖动 oy ±5｜单字旋转 ±6°｜单字缩放 0.93–1.08
- 墨色 alpha 200–245（下笔轻重不一）
- 步进 0.57em ± 抖动；空格 0.40em

整句再整体轻微旋转（如 -4°），RGBA 图层 `rotate` 后 `alpha_composite`。
位置默认 `(0.520W, 0.400H)`——跨接照片与线稿边界处最自然；内容取自画面物件（热水瓶、年份、器物）。

### 2.5 照片×线稿撕裂边缘（可选：用户要"撕扯/破碎/斑驳/慢慢过渡"时）
在去水印后、**文字前**执行（手写批注会自然压在撕口上方）。先定位边界，取处理条带 x∈[边界−47, 边界+66]，numpy 实现：

**边界检测（⚠️ 不要用固定阈值扫列）**：照片右侧线稿的排线密度因图而异，任何固定阈值（无论是 `frac>0.22` 还是 `sat>25`）都会在部分图上误判——实测 `frac>0.22` 把密集排线误判成照片（x=808，撕口打在线稿中间），而 `sat>25` 在灰蓝冷调照片上又全部漏检。**唯一稳健的做法是找最大落差**：照片侧列的非纸率接近 1、线稿侧骤降，两侧的落差不依赖绝对量级。用落差法粗定位，再在邻域内精修到像素级：

```python
arr = np.asarray(img).astype(np.float64); H, W, _ = arr.shape
g = arr.mean(2)
paper = np.median(arr[int(.05*H):int(.15*H), int(.60*W):int(.90*W)].reshape(-1, 3), axis=0)
frac = (np.abs(g - paper.mean()) > 20).mean(0)        # 每列非纸占比（整幅，不依赖行范围）

k = int(0.04*W); sm = np.convolve(frac, np.ones(k)/k, mode='same')
hi = int(.80*W); drop = sm[:hi-k] - sm[k:hi]          # 左侧非纸率 − 右侧非纸率
i = int(np.argmax(drop)) + k                          # 粗定位（平滑后）
lo, hh = max(int(.25*W), i-40), min(hi, i+40)         # 邻域精修
x_edge = lo + int(np.argmax([frac[n] - frac[n+6] for n in range(lo, hh-6)]))
# 校验：边界两侧应有明显落差（照片内的绝对非纸率因图而异，浅色物件图可能只有 ~0.5）
assert frac[x_edge] - frac[x_edge+12] > 0.10, \
    f"edge check failed: {frac[x_edge]:.2f} -> {frac[x_edge+12]:.2f}"
```

校验 **只比较两侧落差**，不要要求照片侧非纸率≈1——照片里若有浅色物件（木架、白标签、浅色墙面）会接近纸色，实测某图照片侧仅 0.47，硬性要求≈1 会误报。内容行范围仍用 `nonpaper` 的整幅行均值（`>0.25`）确定 `y0/y1`。

- 逐行边缘 x = 基准 + 一维平滑噪声（±10px，cell 120/38，条带两端 26px 渐弱）
- alpha = clip((d − 12·n1 − 8·n2)/20)，d=像素x−行边缘；n1=fbm[72,26,10] 大锯齿、n2=fbm[6,3] 细碎
- **蚀孔**（斑驳掉块）：照片侧 d∈[−34,6] 且 m>0.62（m=fbm[12,5]），强度 ×2.8
- **残留**（照片色斑留在纸上）：纸侧 d∈[4,30] 且 m<0.30，减 alpha
- d>30 渐出（×clip((30−d)/9)），不伤线稿起点；A 高斯 blur 0.5 防锯齿
- 替换色 = 逐行纸色（条带右侧 60px 中值）+ fbm 纸纹颗粒 ±7
- 再叠（**有机喷溅 v2**，替代旧版均匀点阵"30咬入+70残留+150纤维"）：碎屑沿撕口分布，密度约 0.85 颗/像素高——距离 = |N(0,5.5)| 截断 30px（贴边密、远端疏）；55% 落纸侧（照片色残留，取样 x_edge−4~34），45% 落照片侧（行纸色 ×0.92~1.10 蛀孔）；尺寸基线 0.6–1.6px，10% ×1.8–3.2 中片、3% ×3.5–6.0 大片；**每颗 = 2–5 个错位子圆叠成的不规则簇**（轮廓参差，不是单椭圆）；30% 画到独立软层整体 blur 1.2px 做晕染，其余画实层；最后沿波状边缘逐行叠细窄暗带（照片边缘色 ×0.80，alpha 30–80）作纸边投影
- seed：numpy 11 / random 7，可复现

**教训**：过渡带宽 26px + 低频噪声会呈"喷枪云雾感"；宽度收到 20 + 细碎噪声权重加大 + 蚀孔硬阈值，才是"纸茬"的干脆感。

**流程提示**：2.5 是**可选**效果，用户未点名"撕扯/破碎/斑驳/慢慢过渡"时走标准直边，不要默认套用。做撕纸变奏版时**复制底图另存新源文件**（如 `base7_torn.png`），直边版成品保留，两版并存供对比。撕纸版等于同底图 + 不同后处理，**无需重新调用 ImageGen**（省积分）。

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
