# npm install 会改你的 package-lock.json：CI 必须用 npm ci 的 3 个理由

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/npm-ci-vs-npm-install?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/npm-ci-vs-npm-install?utm_source=github&utm_medium=referral)**

问一个问题：`npm install` 和 `npm ci` 有什么区别？

大多数人的答案是「ci 快一点，CI 里用」。这个答案不算错，但漏掉了最关键的一条：**`npm install` 有权改写 `package-lock.json`，`npm ci` 永远不会。** 一个是「按 lock 装，装不上就改 lock」，一个是「按 lock 装，装不上就报错」。

这一字之差，在你本机上几乎感觉不到。在 CI 里，它决定了你的构建到底是不是可复现的。

## 先说清楚 npm ci 做了什么

`npm ci` 里的 ci 不是 continuous integration，官方叫它 clean install。它的行为是固定的四条：

1. **必须有 `package-lock.json`**（或 `npm-shrinkwrap.json`），没有直接报错。
2. **`package.json` 和 lock 不一致就报错退出**，不会替你修。
3. **先删掉整个 `node_modules`**，再从 lock 里逐条安装。
4. **绝不写 `package-lock.json`。**

对比 `npm install`：没有 lock 就生成一份；lock 和 `package.json` 对不上就重新解析、把结果写回 lock；`node_modules` 里已有的包能复用就复用。

所以 `npm install` 是「让项目能跑起来」，`npm ci` 是「验证 lock 文件描述的就是能跑起来的项目」。前者适合你写代码，后者适合机器验收。

下面三个场景，每一个我都见过团队因此浪费半天。

## 理由一：lock 文件会被镜像源来回翻转

打开你的 `package-lock.json`，每个包下面有一行 `resolved`：

```json
"resolved": "https://registry.npmjs.org/lodash/-/lodash-4.17.21.tgz"
```

这个 URL 是从当时安装用的 registry 来的。国内团队常见的情况是：A 同学 `.npmrc` 配了 npmmirror，B 同学用官方源，C 同学公司内网有私有仓库。三个人各自 `npm install` 一次，lock 文件里几百行 `resolved` 就换一次域名，diff 里全是无意义的 URL 变更，真正的版本变化淹没在里面。

`npm ci` 在这一步不会写 lock，所以 CI 不会制造这种 diff。但本地开发还是会。彻底的解法是把 registry 写进项目根目录的 `.npmrc` 并提交：

```ini
# .npmrc（提交到仓库）
registry=https://registry.npmmirror.com
```

所有人、包括 CI，用同一个源，`resolved` 就不再翻转。较新版本的 npm 还有一个 `replace-registry-host` 配置项，可以在安装时把 lock 里的 registry 域名替换成当前配置的源，适合已经翻乱了的老仓库。

...

---

**[👉 继续阅读全文：npm install 会改你的 package-lock.json：CI 必须用 npm ci 的 3 个理由](https://tools.cooconsbit.com/zh/articles/npm-ci-vs-npm-install?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
