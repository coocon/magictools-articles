# HTML 转 Markdown 实战教程：3 种方式一步到位

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/html-to-markdown-tutorial?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/html-to-markdown-tutorial?utm_source=github&utm_medium=referral)**

## 为什么需要 HTML 转 Markdown？

随着静态博客（Hugo、Hexo、Jekyll）和基于 Markdown 的内容管理系统（Notion、Obsidian、Ghost）的普及，越来越多的人需要将现有的 HTML 内容转换为 Markdown 格式。以下是几个典型场景：

**博客平台迁移**：从 WordPress、CSDN、简书等平台搬迁文章到自建博客时，这些平台导出的通常是 HTML 格式，需要转换才能用于 Markdown 驱动的静态站点。

**内容清洗**：网页抓取的内容夹杂大量 `<div>`、`<span>`、广告标签和 inline 样式，转换为 Markdown 后可以得到干净的纯文字内容，方便二次加工。

**版本控制需求**：HTML 文件在 Git 中 diff 查看非常困难，而 Markdown 是纯文本，每次修改都清晰可见。

**跨平台发布**：同一篇文章需要发布到不同平台，Markdown 作为中间格式，比 HTML 更容易转换为其他目标格式。

## 三种输入方式详解

MagicTools 的 HTML 转 Markdown 工具提供三种灵活的输入方式，覆盖了实际工作中绝大多数场景。

### 方式一：粘贴 HTML 代码

这是最直接的方式。将 HTML 源码粘贴到左侧输入框，工具会自动解析并在右侧显示转换后的 Markdown。

**适用场景**：
- 从 CMS 后台导出的 HTML 内容
- 网页"查看源代码"复制的片段
- 邮件客户端导出的 HTML

**操作步骤**：
1. 打开 [tools.cooconsbit.com/tools/html2md](https://tools.cooconsbit.com/tools/html2md)
2. 在左侧输入框粘贴 HTML 代码
3. 右侧实时显示 Markdown 转换结果
4. 点击「复制」或「下载 .md 文件」保存结果

**示例**：输入以下 HTML：

```html
<h2>欢迎使用 MagicTools</h2>
<p>这是一段<strong>加粗</strong>的文字，还有<a href="https://example.com">一个链接</a>。</p>
<ul>
  <li>功能一</li>
  <li>功能二</li>
</ul>
```

转换后得到：

```markdown
## 欢迎使用 MagicTools

这是一段**加粗**的文字，还有[一个链接](https://example.com)。

- 功能一
- 功能二
```

### 方式二：输入 URL 自动抓取

输入任意网页 URL，工具会通过服务端代理自动抓取页面内容并提取正文，转换为 Markdown。这一功能由 `/api/fetch-url` 接口实现，支持最大 5MB 的页面，超时限制 10 秒。

**适用场景**：
- 批量收集网络文章并整理为本地笔记
- 将在线教程保存为离线 Markdown 文档
- 从竞品网站采集内容进行分析研究

...

---

**[👉 继续阅读全文：HTML 转 Markdown 实战教程：3 种方式一步到位](https://tools.cooconsbit.com/zh/articles/html-to-markdown-tutorial?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
