# 文本清洗工具使用说明：去空格、删空行、去 HTML 标签一页搞定

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/text-cleaner-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/text-cleaner-guide?utm_source=github&utm_medium=referral)**

## 为什么文本经常会“看着正常，用起来很乱”？

从 PDF、网页、聊天记录、表格里复制出来的文字，经常带着各种隐形问题：多余空格、连续空行、HTML 标签、弯引号、制表符、奇怪符号。肉眼看似乎没问题，一粘到系统里就开始出错。

这时候最省事的做法，不是手动一点点删，而是先做一次文本清洗。

## 这个工具可以清洗什么？

在 [tools.cooconsbit.com/tools/text-cleaner](https://tools.cooconsbit.com/tools/text-cleaner) 里，你可以按需勾选规则，比如：

- 去掉每行首尾空格
- 合并多余空格
- 合并多余空白行
- 删除全部空行
- 去除 HTML 标签
- 把弯引号改成普通引号
- 删除特殊字符
- 删除数字
- 删除标点
- 统一换行格式
- 把 Tab 替换成空格
- 转小写或大写
- 解码常见 HTML 实体

## 推荐怎么用？

### 处理从网页复制出来的内容

如果原文里有 `<p>`、`<div>`、`<span>` 这类标签，可以勾选：

- `Strip HTML tags`
- `Decode HTML entities`
- `Collapse spaces`

这样能比较快地得到纯文本。

### 处理 PDF 复制后的乱格式

PDF 文本最常见的问题是空格乱、空行多、Tab 混进来。通常可以先勾选：

- `Trim each line`
- `Collapse spaces`
- `Remove extra blank lines`
- `Replace tabs with spaces`

### 处理要导入系统的数据文本

如果目标系统对字符很敏感，可以进一步勾选：

- `Normalize line endings`
- `Remove special characters`

...

---

**[👉 继续阅读全文：文本清洗工具使用说明：去空格、删空行、去 HTML 标签一页搞定](https://tools.cooconsbit.com/zh/articles/text-cleaner-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
