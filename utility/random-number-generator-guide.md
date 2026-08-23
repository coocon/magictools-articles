# 随机数生成器怎么用：区间取数、批量生成、去重抽取都可以

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/random-number-generator-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/random-number-generator-guide?utm_source=github&utm_medium=referral)**

## 随机数生成器不只是“抽一个数字”

很多人第一次想到随机数工具，是抽奖或者猜数字。但在实际使用里，它还有很多更具体的场景：批量生成测试号码、从一个区间里随机取样、分配编号、做名单抽取等。

MagicTools 的随机数生成器除了能设最小值和最大值，还支持批量生成和去重。

## 这个工具能做什么？

在 [tools.cooconsbit.com/tools/random-number](https://tools.cooconsbit.com/tools/random-number) 页面中，你可以设置：

- 最小值 `Min`
- 最大值 `Max`
- 生成数量 `How many numbers?`
- 是否去重 `Unique numbers`

生成结果后可以一键复制，批量结果会按换行排列，适合继续粘贴到表格或文档里。

## 最常见的几个用法

### 课堂点名或抽奖

如果名单已经按数字编号，直接设置区间后生成一个随机数即可。想一次抽多个且不能重复，就开启 `Unique numbers`。

### 测试数据准备

开发或测试经常会需要一批随机整数，比如订单号尾号、模拟评分、随机索引等。这里可以一次生成多组，不用自己写脚本。

### 随机分组或取样

做调查、活动、实验时，常常需要从一个编号区间里随机抽几个不重复的值，这就是 `Unique numbers` 的典型场景。

## 使用时要注意什么？

**第一，最小值必须小于最大值。**  
如果区间本身不成立，工具会直接提示错误。

**第二，去重模式下，生成数量不能超过区间总数。**  
比如 1 到 10 一共只有 10 个数，就不能要求去重生成 20 个。

**第三，适合整数随机。**  
当前页面是整数随机，不是小数随机。

## 推荐使用步骤

1. 先确定随机区间。
2. 再决定是抽 1 个还是批量生成。
3. 如果不能重复，就打开 `Unique numbers`。
4. 生成后直接复制结果。

...

---

**[👉 继续阅读全文：随机数生成器怎么用：区间取数、批量生成、去重抽取都可以](https://tools.cooconsbit.com/zh/articles/random-number-generator-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
