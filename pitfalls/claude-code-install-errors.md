# Claude Code 安装失败实测：npm EACCES、镜像卡 600 秒、Node 20 静默装旧版、install.sh 返回地区拦截页、原生安装器顺手卸掉 npm 版——15 条报错原文与解法

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-install-errors?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-install-errors?utm_source=github&utm_medium=referral)**

搜「claude code 安装」的人，大多数不缺教程——缺的是**装到一半报错时，这句报错到底是什么意思**。安装步骤本站已经有了（[Claude Code 快速上手](/zh/articles/claude-code-quickstart-guide)），这篇不重复，只做一件事：把安装会失败的地方在本机**一个个真实复现**，逐字记下报错原文、触发条件、耗时和退出码，再给出解法。

先说最意外的三条：

1. **Node 20 装 Claude Code 不会报错**，而是**静默装到 2.1.197**。engines 从 2.1.198 起要求 `>=22`，npm 会自动挑一个旧的、满足条件的版本，没有任何警告。
2. **在国内直连 `curl -fsSL https://claude.ai/install.sh` 时，curl 退出码是 0**，下载下来的却是 447,830 字节的「App unavailable in region」HTML 页面。`| bash` 之后只会看到一句 `syntax error near unexpected token '<'`。
3. **官方原生安装器会执行 `npm uninstall -g @anthropic-ai/claude-code`**，把你已有的 npm 版删掉，终端里一个字都不提。本次实测就这样真的删掉了本机的全局安装——下面「踩坑」一节有完整经过。

![假 npm 记录到原生安装器的调用：npm uninstall -g @anthropic-ai/claude-code](https://cdn.tools.cooconsbit.com/uploads/articles/2026-09-29-cc-install/05-native-installer-npm-uninstall-zh.png)

## 问题背景

Claude Code 现在有两条官方安装路径：

- **npm**：`npm install -g @anthropic-ai/claude-code`。现在的包只是一个「壳」：真正的程序是平台对应的 optionalDependency（例如 `@anthropic-ai/claude-code-darwin-arm64`，tarball 98,952,459 字节）。postinstall 脚本 `install.cjs` 会把这个原生二进制硬链接到 `bin/claude.exe`。**`claude` 命令本身已经不是 JS 了**，而是一个 Mach-O 可执行文件（2.1.284 为 226,563,088 字节）。
- **原生安装器**：`curl -fsSL https://claude.ai/install.sh | bash`。脚本从 `downloads.claude.ai` 下载二进制、校验 sha256，然后执行 `claude install`，最终装到 `~/.local/bin/claude`。

这两条路径各有各的失败面，而国内网络环境会让其中好几个失败面的报错**看起来和真正的原因毫无关系**。

## 问题分析

把「claude code 安装失败」拆开，能出问题的环节有这些：

| 环节 | 可能出的事 | 本文对应 |
|---|---|---|
| 写入全局目录 | prefix 没有写权限 | 表中 #1 |
| npm cache | cache 目录不可写 | #2 |
| registry | 连不上、DNS 解析不了、镜像慢 | #3 #4 #6 |
| 代理 | npmrc 与环境变量冲突 | #5 |
| Node 版本 | engines 门禁 | #7 #8 |
| 原生安装器 | 地区拦截、下载服务连不上、覆盖 npm 版 | #9 #10 #12 |
| 装完之后 | PATH、postinstall 没跑、用错启动方式 | #11 #13 #14 #15 |

还有一个前提容易被忽略：**`npm install -g` 读不到项目目录里的 `.npmrc`**。实测在仓库根目录（其 `.npmrc` 写着 npmmirror），隔离 HOME 之后：

```
$ npm config get registry        → https://registry.npmmirror.com/
$ npm config get registry -g     → https://registry.npmjs.org/
```

也就是说，全局安装实际生效的只有 `~/.npmrc`、全局 npmrc 和命令行参数。排查 registry / 代理问题时，一定要用 **`npm config get xxx -g`** 看。

## 技术方案与选型

目标是「真实复现 + 不破坏本机」，方案如下：

- **所有安装都隔离**：`--prefix`、`--cache`、`--logs-dir` 全部指向 `/tmp/cc-install-0929/` 下的子目录，不写 `~/.npm` 和 `/opt/homebrew`（原生安装器那一次例外，见「踩坑」）。
- **失败面用可控的坏值触发**：registry 指向 `127.0.0.1:1` 和 `.invalid` 域名，代理指向 `127.0.0.1:1`/`:2`，cache 用 0555 目录，权限用 `/usr/local`。
- **低版本 Node 用便携版**：本机的 22 / 24 / 25 都满足 `>=22`，于是在 `/tmp` 下解压了 Node v20.19.5（npm 10.8.2），不用 nvm，也不改系统。
- **网络对照**：镜像和官方源分别用独立 cache，否则第二次会命中相同 integrity 的缓存，测不出下载时间。

**排除项**：

- **不测 Windows / Linux**：只有一台 macOS，写不出没测过的报错原文。
- **不测 `sudo npm install -g`**：会写系统目录，而且这是网上最常见的错误建议，不值得为它破坏本机。
- **不测 pnpm / yarn / bun**：报错原文里提到了「some pnpm configs」，但本次没有复现，不下结论。
- **不调用 API**：安装问题与模型无关，全程 0 次 API 调用，花费 ¥0。

...

---

**[👉 继续阅读全文：Claude Code 安装失败实测：npm EACCES、镜像卡 600 秒、Node 20 静默装旧版、install.sh 返回地区拦截页、原生安装器顺手卸掉 npm 版——15 条报错原文与解法](https://tools.cooconsbit.com/zh/articles/claude-code-install-errors?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
