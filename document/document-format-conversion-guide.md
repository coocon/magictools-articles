# 文档格式转换实战指南：Markdown、HTML、PDF 互转全解析

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/document-format-conversion-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/document-format-conversion-guide?utm_source=github&utm_medium=referral)**

每个文档都有它最适合的格式。Markdown 适合写作，HTML 适合网页展示，PDF 适合打印分发，Word 适合办公协作。麻烦在于，现实工作中你经常需要在这些格式之间穿梭——把博客文章转成 PDF 发给客户，把 Word 文档迁移到知识库，把网页内容提取成 Markdown 归档。

格式转换看似简单，真正动手时却问题频出：格式乱了、表格消失了、中文变乱码了。这篇指南梳理各种转换路径的最佳方案，帮你少走弯路。

## 四大格式特点对比

在选择转换路径之前，先理解各格式的本质特点：

| 格式 | 可读性 | 可编辑性 | 渲染效果 | 文件大小 | 适用场景 |
|------|:------:|:--------:|:--------:|:--------:|---------|
| Markdown (.md) | 极高（纯文本） | 极高 | 依赖渲染器 | 极小 | 技术文档、博客写作、README |
| HTML (.html) | 中（含标签） | 高 | 浏览器渲染 | 小~中 | 网页、邮件模板、文档展示 |
| PDF (.pdf) | 高（视觉） | 极低 | 固定排版 | 中~大 | 打印、正式文件、跨平台分发 |
| Word (.docx) | 高 | 高 | Office 渲染 | 中 | 办公协作、需要批注修订 |

核心规律：**可编辑性越高的格式，排版固定性越低；排版越固定的格式，转换损耗越大。** PDF 是单向格式，从 PDF 往外转时质量损耗最大。

## Markdown → PDF

这是最常见的转换需求，有三种方案，效果各有差异：

### 方案一：浏览器打印（推荐入门用）

1. 在支持 Markdown 预览的工具中打开文档（VS Code 预览、MagicTools、Typora）
2. 按 `Ctrl+P`（Mac：`Cmd+P`）打开打印对话框
3. 目标打印机选择「另存为 PDF」
4. 调整页面设置：去掉页眉页脚勾选，设置合适的边距

**优点**：操作简单，零学习成本，所见即所得。
**缺点**：代码高亮、字体、分页位置依赖浏览器渲染，不同机器可能有细微差异。

### 方案二：Pandoc 命令行（推荐专业用）

Pandoc 是格式转换领域的瑞士军刀，支持 40+ 种格式互转。

```bash
# 安装（Mac）
brew install pandoc
# 同时需要安装 LaTeX（用于 PDF 输出）
brew install --cask mactex-no-gui

# 基本转换
pandoc input.md -o output.pdf

# 带中文支持（必须指定字体，否则中文显示为方块）
pandoc input.md -o output.pdf \
  --pdf-engine=xelatex \
  -V mainfont="PingFang SC" \
  -V geometry:margin=2cm

# 自定义 CSS 样式（通过 HTML 中间步骤）
pandoc input.md -o output.pdf \
  --pdf-engine=wkhtmltopdf \
  --css=style.css
```

**优点**：输出质量最高，格式控制精细，支持批量处理，适合生产环境。
**缺点**：需要安装 LaTeX 环境（约 4GB），中文配置有一定学习成本。

### 方案三：在线工具（临时需求）

[MagicTools](https://tools.cooconsbit.com/tools/markdown) 内置导出功能，写好 Markdown 后一键导出 PDF，无需安装任何软件。适合偶发性需求。

## Markdown → HTML

...

---

**[👉 继续阅读全文：文档格式转换实战指南：Markdown、HTML、PDF 互转全解析](https://tools.cooconsbit.com/zh/articles/document-format-conversion-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
