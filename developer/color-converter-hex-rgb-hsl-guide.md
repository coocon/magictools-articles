# 颜色转换完全指南：HEX、RGB、HSL 与现代 CSS 配色（2026）

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/color-converter-hex-rgb-hsl-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/color-converter-hex-rgb-hsl-guide?utm_source=github&utm_medium=referral)**

## 颜色转换：前端工程师每天都在做的事

打开浏览器 DevTools 看任意一个网站的样式，颜色值通常是这样混搭的：

```css
.button {
  background: #3b82f6;             /* HEX */
  color: rgb(255 255 255);         /* RGB */
  border-color: hsl(217 91% 60%);  /* HSL */
  box-shadow: 0 4px 12px oklch(0.7 0.15 230 / 0.4);  /* OKLCH */
}
```

每种颜色表示法都有适用场景：HEX 用于设计稿对接、RGB 用于程序计算、HSL 适合做主题色调整、OKLCH 是 2026 年现代设计系统的首选。本指南帮你彻底搞懂这些格式之间的关系，并掌握高效的转换方法。

---

## 主流颜色格式对比

### HEX：设计师的母语

最古老也最普及的格式：`#RRGGBB`，每两位是一个十六进制数（00-FF），分别代表红绿蓝。

```css
#3b82f6  /* 蓝色 */
#3B82F6  /* 同上，大小写不影响 */
#3b82f6cc  /* 8 位 = RGB + Alpha 透明度 */
#3bf  /* 简写，等价于 #33bbff */
```

**优点**：紧凑、可复制、Photoshop/Figma 直接用
**缺点**：无法直观看出颜色关系，调浅/调深需要重新计算

### RGB / RGBA：浏览器的内部表示

```css
rgb(59 130 246)         /* 现代语法（推荐） */
rgb(59, 130, 246)       /* 老语法，逗号分隔 */
rgba(59 130 246 / 0.8)  /* 带透明度，用斜杠分隔 */
```

**优点**：直接对应像素值、可程序化生成、易做线性混色
**缺点**：人类很难凭数字判断颜色

### HSL：调色板的瑞士军刀

```css
hsl(217 91% 60%)  /* 色相 / 饱和度 / 亮度 */
```

- **H（Hue 色相）**：0-360 度，红橙黄绿青蓝紫绕一圈
- **S（Saturation 饱和度）**：0%（灰）到 100%（最艳）
- **L（Lightness 亮度）**：0%（黑）到 100%（白），50% 是最饱和的纯色

**杀手锏**：固定 H/S，只调 L 就能批量生成主题色阶（这就是 Tailwind blue-50 → blue-900 的原理）。

### OKLCH：2026 年的新标准

CSS Color Module Level 4 引入的现代格式，**感知均匀**——参数变化和视觉变化成正比：

```css
oklch(0.7 0.15 230)  /* 亮度 / 色度 / 色相 */
```

- **L**：感知亮度，0（黑）到 1（白）
- **C**：色度（接近饱和度），0 到 0.4 左右
- **H**：色相，0-360 度

**为什么值得切换？** HSL 的「亮度」其实是数学定义，不符合人眼感知。HSL `lightness: 50%` 的黄色看起来亮、紫色看起来暗。OKLCH 真正做到「数值相同，视觉一样亮」，是设计系统的福音。

---

## 在线一键转换：3 步完成

不用记复杂公式，打开 MagicTools 颜色转换工具：

1. **输入任意格式**：粘贴 HEX、RGB、HSL、颜色名（red/blue/...）都可以
2. **自动转换**：所有格式同时显示，复制即可
3. **可视化预览**：右侧实时显示色块，肉眼校对

工具支持 8 种主流格式互转，包括 HEX、RGB、HSL、HSV、OKLCH、LAB、CMYK 和颜色名。

---

## 常用转换规则速查

### HEX ↔ RGB

每两位 HEX 转十进制即可：

...

---

**[👉 继续阅读全文：颜色转换完全指南：HEX、RGB、HSL 与现代 CSS 配色（2026）](https://tools.cooconsbit.com/zh/articles/color-converter-hex-rgb-hsl-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
