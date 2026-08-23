# 文本对比工具使用说明：快速找出两个版本哪里改了

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/diff-checker-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/diff-checker-guide?utm_source=github&utm_medium=referral)**

## 为什么要用文本对比工具？

很多时候你拿到的是两个版本的文字，但不知道到底改了哪里。比如领导改过的方案、客户回传的合同、同事修改后的配置文件，肉眼逐行看非常费时间，也很容易漏掉细节。

这类场景最适合用 Diff 工具。MagicTools 的文本对比工具可以把新增、删除和未变化的内容直接标出来。

## 这个工具能做什么？

打开 [tools.cooconsbit.com/tools/diff](https://tools.cooconsbit.com/tools/diff) 后，左边放原文，右边放新版本。工具会按**行**进行对比，并显示：

- 新增了多少行
- 删除了多少行
- 有多少行没有变化
- 统一视图 `unified`
- 左右分栏视图 `split`

如果你只是想快速看改动，统一视图更适合；如果你要逐行核对原文和修改稿，分栏视图更清楚。

## 怎么使用最省事？

### 场景一：对比两版文案

1. 左侧粘贴旧版本。
2. 右侧粘贴新版本。
3. 先看顶部统计，确认改动规模。
4. 再从上往下看高亮行，重点关注新增和删除。

### 场景二：检查配置文件变更

像 `.env`、JSON、YAML、SQL 片段这类文本，只要内容是一行一行排列的，都适合直接放进来比较。这样能快速看出是不是改错了参数、删掉了某一行配置。

## 什么时候特别有用？

### 审核合同或制度文件

法务或运营常常会收到“只改了一点点”的新版文件。用 Diff 工具能马上定位具体改动，避免在长文里来回找。

### 校对文章改稿

编辑或作者回看修改稿时，最怕“好像改了，但不知道改了哪”。这个工具适合先快速看结构性变化，再决定是否继续细修。

### 没有 Git 时临时比对代码

不是所有代码片段都在仓库里。有时只是别人发来两段脚本，这时候直接粘贴比打开 IDE 更快。

...

---

**[👉 继续阅读全文：文本对比工具使用说明：快速找出两个版本哪里改了](https://tools.cooconsbit.com/zh/articles/diff-checker-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
