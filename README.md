# Agent Skills

A personal collection of [WorkBuddy / Claude Code](https://www.workbuddy.cn) skills.

每个 skill 都是一个独立目录，包含一份 `SKILL.md`（YAML frontmatter + 执行说明），可直接放到 `~/.workbuddy/skills/` 下使用。

## Skills

| Skill | 说明 |
|---|---|
| [photo-sketch-brand-poster](skills/photo-sketch-brand-poster/) | 把照片做成 3:4 竖版极简品牌海报。**实拍照片 × 手绘线稿**虚实拼贴风格：照片占画面约 1/5 偏居一侧，右侧以同透视铅笔线稿自然延续，暖奶油纸底、大量留白；文字由程序叠加，保证印刷级清晰度。 |

## 安装

```bash
git clone https://github.com/haomiaomou-cmd/skills.git
cp -r skills/photo-sketch-brand-poster ~/.workbuddy/skills/
```

Windows (PowerShell)：

```powershell
git clone https://github.com/haomiaomou-cmd/skills.git
Copy-Item -Recurse .\skills\photo-sketch-brand-poster $env:USERPROFILE\.workbuddy\skills\
```

安装后重启会话，或在下一次对话中提到触发词（如"照片+手绘线稿""虚实拼贴海报""照片里长出画"）即可自动加载。

## Skill 格式

每个 `SKILL.md` 以 YAML frontmatter 开头：

```yaml
---
name: skill-name
description: 何时使用这个 skill（触发条件要写清楚、具体）
---

# 正文：执行步骤、参数、避坑要点
```

`description` 字段是自动加载的关键——写清楚「做什么」和「什么时候用」。

## License

[MIT](LICENSE)
