# AI 文本隐形水印检测指南：零宽字符、Tag 隐写与统计水印的边界

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/ai-text-watermark-detection-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/ai-text-watermark-detection-guide?utm_source=github&utm_medium=referral)**

复制一段 AI 生成的文字，粘贴进十六进制编辑器，你可能会看到一些"不存在"的东西：几十个不占显示宽度、不发出声音、但实实在在占据码点的 Unicode 字符。它们随复制粘贴一起旅行，可以用来标记文本来源、追踪泄露渠道，甚至编码一段完整的隐藏明文。

这篇文章把字符层隐写的六类手法逐一说清：每类字符是什么、正常用途是什么、被滥用成水印时长什么样、怎么检测和清除。也会明确一件多数同类文章不说的事——**这类工具的能力边界在哪里，什么样的水印是字符扫描永远查不出来的**。

你可以随时用 [MagicTools 的 AI 水印检测器](https://tools.cooconsbit.com/tools/ai-watermark-checker) 对照实验：粘贴文本即可逐字符扫描，全部在浏览器本地完成，文本不会上传。

## 六类隐形字符：从"经典水印"到"隐写信道"

### 1. 零宽字符（zero-width）

最经典的一类，五个成员：

| 码点 | 名称 | 缩写 |
|------|------|------|
| U+200B | 零宽空格 | ZWSP |
| U+200C | 零宽非连接符 | ZWNJ |
| U+200D | 零宽连接符 | ZWJ |
| U+2060 | 词连接符 | WJ |
| U+FEFF | 零宽不换行空格（BOM） | BOM |

它们的正当用途是排版控制：ZWSP 提示浏览器"这里可以换行"，ZWNJ/ZWJ 控制阿拉伯文、印地文等连写文字的字形连接，BOM 标记文件字节序。被滥用时，最简单的玩法是把每个零宽字符当一个比特——ZWSP 是 0，ZWNJ 是 1——一段 8 字符的序列就能藏一个字节。文本看起来毫无变化，但"指纹"随复制粘贴完整保留。

### 2. 方向控制符（direction）

U+200E/200F（LRM/RLM）、U+202A–202E（嵌入与强制覆盖）、U+2066–2069（隔离符）。设计给阿拉伯文、希伯来文等从右往左书写的文字与拉丁文混排时用。

这一类的危险性超出水印范畴：U+202E（RLO，从右往左强制覆盖）可以让文件名 `invoice_exe.pdf` 实际是 `invoice_fdp.exe`，是钓鱼攻击的经典手法；2021 年公开的 "Trojan Source" 攻击（CVE-2021-42574）就是用方向控制符让源代码在编辑器里的显示顺序与编译器读到的逻辑顺序不一致。出现在普通中英文文本里的方向控制符，几乎都值得怀疑。

### 3. 变体选择符（variation-selector）

U+FE00–FE0F（VS1–VS16）和补充区 U+E0100–E01EF（VS17–VS256）。正当用途是指定同一字符的不同字形——最常见的是 VS16（U+FE0F）把一个符号切换成 emoji 彩色形态。

被滥用时它是高容量隐写信道：256 个变体选择符正好能编码一个字节，把它们附在任意可见字符后面，就能在一句普通的话里藏进任意二进制数据。2024 年初 Paul Butler 的文章《Smuggling arbitrary data through an emoji》让这个手法广为人知。

### 4. Tag 字符（tag）——能藏完整明文的一类

U+E0000–E007F，Unicode 里最"名存实亡"的区块：原设计用于语言标记，1999 年就被废弃，但字符还在。其中 U+E0020–U+E007E 与 ASCII 可打印区一一对应——**把任意 ASCII 字符的码点加上 0xE0000，就得到它的隐形版本**。

...

---

**[👉 继续阅读全文：AI 文本隐形水印检测指南：零宽字符、Tag 隐写与统计水印的边界](https://tools.cooconsbit.com/zh/articles/ai-text-watermark-detection-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
