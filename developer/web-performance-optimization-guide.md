# Web 性能优化完全指南：让网站加载速度提升 50% 的 15 个技巧

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/web-performance-optimization-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/web-performance-optimization-guide?utm_source=github&utm_medium=referral)**

Google 在 2023 年的研究数据揭示了一个残酷的现实：**页面加载时间每增加 1 秒，移动端转化率下降 20%**。BBC 发现每多花 1 秒加载时间，就额外失去 10% 的用户。Amazon 估算延迟 100 毫秒会导致销售额下降 1%。

性能不是"锦上添花"，而是直接影响业务的核心指标。本文系统梳理 15 个经过实践验证的优化技巧，从最大收益的图片优化，到精细的运行时调优，帮你构建真正快速的 Web 应用。

---

## Core Web Vitals：Google 定义的性能标准

在优化之前，先了解 Google 用来评估用户体验的三个核心指标（Core Web Vitals）：

| 指标 | 全称 | 含义 | 良好标准 |
|------|------|------|---------|
| **LCP** | Largest Contentful Paint | 最大内容元素的渲染时间 | ≤ 2.5 秒 |
| **INP** | Interaction to Next Paint | 用户交互到下一帧渲染的延迟（2024年替代FID） | ≤ 200ms |
| **CLS** | Cumulative Layout Shift | 累积布局偏移（页面元素意外移动） | ≤ 0.1 |

这三个指标直接影响 Google 搜索排名。优化它们既是性能工程，也是 SEO 策略。

---

## 图片优化：最大的性能收益点

图片通常占网页总字节数的 50%~70%，是性能优化中性价比最高的方向。

### 技巧 1：使用现代图片格式

| 格式 | 相比 JPEG | 相比 PNG | 浏览器支持 |
|------|---------|---------|-----------|
| WebP | 小 25~35% | 小 26% | 97%+ |
| AVIF | 小 50% | 小 50%+ | 90%+（2024年） |
| JPEG XL | 小 35% | - | 仍在推进中 |

```html
<!-- 使用 picture 元素提供多格式回退 -->
<picture>
  <source srcset="hero.avif" type="image/avif">
  <source srcset="hero.webp" type="image/webp">
  <img src="hero.jpg" alt="Hero image" width="1200" height="600">
</picture>
```

### 技巧 2：响应式图片（srcset）

```html
<img
  src="photo-800w.jpg"
  srcset="photo-400w.jpg 400w,
          photo-800w.jpg 800w,
          photo-1600w.jpg 1600w"
  sizes="(max-width: 600px) 400px,
         (max-width: 1200px) 800px,
         1600px"
  alt="响应式图片示例"
>
```

浏览器根据设备屏幕宽度和分辨率自动选择最合适的图片，避免手机下载 2MB 的桌面图片。

### 技巧 3：图片懒加载

```html
<!-- 原生懒加载，现代浏览器支持 -->
<img src="below-fold.jpg" loading="lazy" alt="延迟加载图片">

<!-- 视口内的关键图片：显式声明 eager（默认行为）-->
<img src="hero.jpg" loading="eager" alt="首屏图片">
```

懒加载可将初始页面数据量减少 50% 以上，显著提升 LCP。

### 技巧 4：设置明确的图片尺寸

```html
<!-- ✅ 正确：声明宽高，浏览器预留空间，避免布局偏移（CLS） -->
<img src="avatar.jpg" width="120" height="120" alt="用户头像">

<!-- ❌ 错误：图片加载后撑开布局，导致 CLS 分数飙升 -->
<img src="avatar.jpg" alt="用户头像">
```

---

## 资源加载优化

### 技巧 5：CSS/JS 压缩与代码分割

现代构建工具（Vite、webpack、Next.js）均自动完成压缩（Minify）。代码分割（Code Splitting）则需要主动配置：

```javascript
// React 动态导入：按需加载组件
import { lazy, Suspense } from 'react';

const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<div>加载中...</div>}>
      <HeavyComponent />
    </Suspense>
  );
}
```

...

---

**[👉 继续阅读全文：Web 性能优化完全指南：让网站加载速度提升 50% 的 15 个技巧](https://tools.cooconsbit.com/zh/articles/web-performance-optimization-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
