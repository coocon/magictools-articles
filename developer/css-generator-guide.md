# CSS 生成器怎么用：圆角、阴影、渐变和 transform 可视化生成

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/css-generator-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/css-generator-guide?utm_source=github&utm_medium=referral)**

## 为什么前端写 CSS 时还需要生成器？

像 `border-radius`、`box-shadow`、`linear-gradient`、`transform` 这些属性，单独写并不难，但参数一多，调试就会变得很烦。尤其是你还想边改边看效果时，手敲代码效率并不高。

MagicTools 的 CSS 生成器适合做“先可视化调出来，再复制代码”的工作流。

## 支持哪些 CSS 类型？

打开 [tools.cooconsbit.com/tools/css](https://tools.cooconsbit.com/tools/css) 后，可以在顶部切换：

- Border Radius
- Box Shadow
- Text Shadow
- Background
- Gradient
- Transform

页面左侧调参数，右侧会生成对应 CSS，并提供预览和复制、下载功能。

## 最常见的几种用法

### 1. 做卡片圆角

如果你只需要统一圆角，直接用 `border-radius` 模式即可；如果四个角需要分别设置，也可以关闭统一模式后单独调整。

### 2. 调阴影层次

`box-shadow` 模式适合做按钮、卡片、悬浮面板的阴影效果。你可以直接控制偏移、模糊、扩散和颜色，比反复改代码再刷新页面更快。

### 3. 生成渐变背景

做 Banner、按钮、卡片背景时，很多人最常用的是渐变。这里可以选择线性或径向渐变，调好颜色后直接复制。

### 4. 调整 transform

需要旋转、缩放、位移时，`transform` 模式会把常见参数组合成一条完整 CSS，省得自己拼接。

## 适合哪些人？

- 前端开发
- 设计转代码人员
- 想快速试视觉效果的产品或运营
- 不想记太多 CSS 参数细节的新手

## 使用建议

**第一，先在生成器里把效果调顺眼，再复制到项目里。**  
这样比在代码里来回试更直观。

...

---

**[👉 继续阅读全文：CSS 生成器怎么用：圆角、阴影、渐变和 transform 可视化生成](https://tools.cooconsbit.com/zh/articles/css-generator-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
