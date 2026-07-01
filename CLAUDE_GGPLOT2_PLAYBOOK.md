# 将 ggplot2 Gallery 技巧“配备到 Claude Code”实操

> 目标：把 <https://exts.ggplot2.tidyverse.org/gallery/> 的图形套路，沉淀成 Claude Code 可复用的“绘图技能包”。

## 1) 建立可复用目录

```bash
mkdir -p .claude/skills/ggplot2-gallery/{references,prompts,templates,scripts}
```

建议结构：

- `references/gallery_patterns.md`：按图形类型整理套路。
- `prompts/plot_from_data.md`：Claude 的标准提问模板。
- `templates/base_theme.R`：统一主题与颜色。
- `scripts/render_plot.R`：固定渲染脚本，减少每次重复写样板。

## 2) 从 Gallery 提炼“模式库”

在 `references/gallery_patterns.md` 中至少维护以下字段（每个图一个条目）：

- **图类型**（散点/箱线/小提琴/热图/网络图等）
- **最小可运行代码**（MRE）
- **常用映射**（`aes(x, y, color, fill, size, group)`）
- **统计变换**（`stat_summary`、`geom_smooth` 等）
- **主题和配色**（`theme_minimal`、`scale_*_manual`）
- **常见坑**（离散变量排序、legend 过长、坐标轴标签重叠）

## 3) 固化一个“Claude 提示词模板”

把下面内容保存到 `prompts/plot_from_data.md`，以后直接复用：

```md
你是资深 R 可视化工程师。请使用 ggplot2 输出“可直接运行”的完整代码。

输入：
1) 数据字段：{字段说明}
2) 目标图：{图类型}
3) 业务目标：{想表达的结论}
4) 风格偏好：{简洁/论文/深色}

要求：
- 先给图形设计理由（3-5 条）
- 再给完整 R 代码（包含 library、数据预处理、作图、主题）
- 必须包含：标题、副标题、caption、图例标题
- 如果有更优替代图，再给 1 个备选方案
- 代码最后保存为 `output.png`（dpi=300）
```

## 4) 让 Claude Code 优先读取该技能

在仓库根目录放一个 `CLAUDE.md`，写明：

```md
作图任务默认遵循：
- .claude/skills/ggplot2-gallery/references/gallery_patterns.md
- .claude/skills/ggplot2-gallery/prompts/plot_from_data.md
- 输出 R 代码必须可直接运行，并保存 output.png
```

## 5) 标准化渲染脚本（推荐）

`script/render_plot.R` 示例：

```r
#!/usr/bin/env Rscript
args <- commandArgs(trailingOnly = TRUE)
input <- ifelse(length(args) >= 1, args[1], "plot.R")
source(input)
```

你让 Claude 只负责生成 `plot.R`，再统一执行：

```bash
Rscript scripts/render_plot.R plot.R
```

## 6) 质量门槛（每次都让 Claude 自检）

让 Claude 在回答末尾附一个 checklist：

- 字体和字号层级是否清晰
- 颜色是否色盲友好（如 viridis）
- 图例是否与编码一致
- 坐标轴是否可读（标签不重叠）
- 标题是否直接传达结论

## 7) 推荐你的最小闭环

1. 你提供字段说明 + 业务问题。
2. Claude 用 `plot_from_data.md` 产出 `plot.R`。
3. 本地 `Rscript scripts/render_plot.R plot.R` 生成图片。
4. 你反馈“哪里不满意”，Claude 迭代（配色/排序/标注/分面）。

---

如果你愿意，我可以下一步直接给你一个**可复制的完整技能包目录内容**（含 `gallery_patterns.md` 初始 10 个高频图模板 + `base_theme.R`），你粘贴进 Claude Code 就能立刻用。
