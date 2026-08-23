# Markdown 写作完全指南：2026 年最佳实践与高级技巧

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/markdown-best-practices-2026?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/markdown-best-practices-2026?utm_source=github&utm_medium=referral)**

Markdown 诞生于 2004 年，John Gruber 的初衷很简单：让纯文本文件在不渲染的情况下也具备良好的可读性。二十年过去，Markdown 已成为技术写作的事实标准，从 GitHub README 到技术博客，从 API 文档到知识库，无处不在。

但很多人只会用几个基本语法，遇到复杂场景就手足无措。这篇指南将从历史脉络讲到实战技巧，帮你真正掌握 Markdown。

## Markdown 的主流变体

Markdown 规范并不统一，这是它最大的"历史遗留问题"。

**CommonMark**：2014 年由 Jeff Atwood 等人发起，目标是建立一套无歧义的 Markdown 规范。现在大多数渲染引擎（包括 GitHub、GitLab）都以 CommonMark 为基础。

**GFM（GitHub Flavored Markdown）**：GitHub 在 CommonMark 基础上扩展的变体，增加了表格、任务列表、删除线、自动链接等特性。是目前使用最广泛的 Markdown 变体。

**MultiMarkdown**：扩展了脚注、表格、引用等功能，主要在学术写作圈流行。

**MDX**：在 Markdown 中嵌入 JSX 组件，专为 React 技术栈设计，是现代文档站点（如 Docusaurus、Nextra）的首选。

实际写作时，**优先以 GFM 为准**——它的兼容性最广，绝大多数平台都支持。

## 基础语法速查

### 标题

```markdown
# 一级标题（每篇文章只用一次，通常作为文章标题）
## 二级标题（主要章节）
### 三级标题（子章节）
#### 四级标题（尽量避免，层级太深影响可读性）
```

**核心原则**：一篇文档只有一个 `h1`。如果你在博客发布，文章标题通常由平台自动生成 h1，正文就从 `##` 开始写。

### 文本格式

```markdown
**粗体文本** 或 __粗体文本__
*斜体文本* 或 _斜体文本_
~~删除线文本~~（GFM 支持）
`行内代码`
> 引用块，适合摘录或注意事项
```

### 列表

```markdown
- 无序列表项（推荐用 - 而不是 * 或 +，统一风格）
  - 缩进两个空格表示子项
  - 子项二

1. 有序列表项
2. 第二项
3. 第三项（序号可以全写 1.，渲染时自动排序）
```

### 链接与图片

```markdown
[链接文字](https://example.com "可选标题")
![图片 alt 文字](https://example.com/image.png "可选标题")

<!-- 引用式链接，适合长文中重复引用同一链接 -->
[链接文字][ref-id]
[ref-id]: https://example.com
```

## 进阶语法

### 代码围栏：始终指定语言

代码块的正确写法是用三个反引号包裹，**并且必须指定语言**：

````markdown
```javascript
const greeting = (name) => `Hello, ${name}!`;
console.log(greeting("World"));
```

```python
def greeting(name: str) -> str:
    return f"Hello, {name}!"
```

```bash
# 安装依赖
npm install && npm run build
```
````

...

---

**[👉 继续阅读全文：Markdown 写作完全指南：2026 年最佳实践与高级技巧](https://tools.cooconsbit.com/zh/articles/markdown-best-practices-2026?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
