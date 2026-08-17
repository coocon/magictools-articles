---
title: "Protobuf 终于有官方 LSP 了：VS Code / Neovim 配置实操指南"
slug: protobuf-lsp-setup-guide
summary: "Buf 把 production-grade 的 Protobuf LSP 直接打包进了 buf CLI：跳转定义、补全、找引用、重命名、organize imports、与 buf lint 同源的实时诊断，全都有了。本文给出 VS Code、Neovim（0.11 新写法 + lspconfig 旧写法）、JetBrains 三条配置路径，一个两文件的最小验证工程，以及已知的坑（buf-config filetype 要手动注册、旧插件要卸载、别再用 buf beta lsp）。所有命令基于 buf v1.72.0 实测。"
category: developer
tags: [Protobuf, LSP, Buf, VS Code, Neovim, gRPC, 开发工具, 编辑器]
coverImage: ""
status: published
locale: zh
source: authored
---

写 `.proto` 文件的编辑器体验，长期停留在"语法高亮 + 保存后跑 protoc 看报错"的石器时代：补全靠猜、跳转靠 grep、字段名拼错要编译才知道。这个局面已经结束了——**Buf 把一个完整的 LSP server 直接打包进了 `buf` CLI**（命令 `buf lsp serve`），跳转定义、补全、找引用、重命名、语义高亮、与 `buf lint` 同源的实时诊断，一次配齐。

先交代背景：Buf 的这篇发布博客写于 2026 年 1 月，最近才在 Hacker News 上翻红（163 分、115 评论）。所以它不是"新闻"，而是很多人还没用上的"半年前就该配的东西"——LSP 已 GA、wire protocol 稳定，本文所有命令基于当前最新的 **buf v1.72.0** 实测。

## 为什么选官方 LSP，而不是社区方案

Protobuf 的社区 LSP 一直有，但各有硬伤：

| 方案 | 现状 |
|------|------|
| **Buf LSP（官方）** | 打包在 buf CLI 内，由 Buf 的全规范编译器前端 protocompile 驱动（Google 内部也在部分场景使用它）；随 CLI 月更 |
| protols | Rust + tree-sitter 社区方案，活跃，但解析非全规范 |
| protobuf-language-server | Go 老牌社区方案，README 自认"不做完整校验" |
| pbls | 个人项目，极小众 |
| vscode-proto3 | 不是 LSP，只是语法高亮 + 调 protoc，2026-03 后基本停更 |
| buf-language-server | Buf 自己的旧尝试，已归档——功能并进了 buf CLI |

官方方案的核心优势不是功能列表，而是**一致性**：LSP 的诊断遵循你 `buf.yaml` 里配的 lint 规则，格式化和 `buf format` 输出一致，跳转能理解 BSR 模块依赖。编辑器里看到的和 CI 里跑出来的是同一套结论。

底层还有个值得一提的细节：Buf 为 LSP 新开发了 query-driven 的编译器前端（借鉴 rustc 的查询式架构），支持增量编译，诊断精度比 protoc 高——比如 `repeated repeated M x = 4;` 这种重复修饰符，它能精确指出第二个 `repeated` 多余并给出修复建议，protoc 做不到。

## 第一步：安装 Buf CLI（≥ 1.72.0）

```bash
# macOS / Linux（官方 tap，不是 homebrew-core）
brew install bufbuild/buf/buf

# Windows
scoop install buf        # 或 winget install bufbuild.buf

# 项目内 npm 安装
npm install @bufbuild/buf   # 之后用 npx buf 调用

# 直接下二进制
curl -sSL "https://github.com/bufbuild/buf/releases/download/v1.72.0/buf-$(uname -s)-$(uname -m)" \
  -o /usr/local/bin/buf && chmod +x /usr/local/bin/buf
```

装完确认：`buf --version` 输出 1.72.0（或更新）。LSP server 不需要单独安装，`buf lsp serve --help` 能看到帮助就说明就绪。

> 一个老坑：早期资料里的命令是 `buf beta lsp`，**已过时**。GA 后的命令是 `buf lsp serve`，默认走 stdio。

## VS Code：装扩展就完事

1. 安装扩展 **`bufbuild.vscode-buf`**（Marketplace 搜 "Buf"，要求 VS Code 1.95+）
2. **先禁用其他 Protobuf 扩展**（如 vscode-proto3）——官方文档明确建议，避免高亮和诊断打架
3. 没了。扩展默认用你 `$PATH` 里的 buf；**如果找不到会自动下载最新版 CLI** 到自己的存储目录。想指定二进制路径用设置项 `buf.commandLine.path`

额外福利：命令面板里直接有 `buf generate`、`buf build`、依赖更新等命令。

## Neovim：两种写法

**Neovim 0.11+ 原生写法**（官方文档推荐）：

```lua
vim.lsp.config('buf_ls', {
  cmd = { 'buf', 'lsp', 'serve' },
  filetypes = { 'proto', 'buf-config' },
  root_markers = { 'buf.yaml', '.git' },
})
vim.lsp.enable('buf_ls')
```

**nvim-lspconfig 旧写法**（等价可替换）：

```lua
require('lspconfig').buf_ls.setup({})
```

有一个**必做的额外步骤**，官方文档写了但极容易漏：`buf-config` 这个 filetype 需要手动注册，否则 LSP 不会附着到 buf.yaml 等配置文件上：

```lua
vim.filetype.add({ filename = {
  ['buf.yaml'] = 'buf-config',
  ['buf.gen.yaml'] = 'buf-config',
  ['buf.policy.yaml'] = 'buf-config',
  ['buf.lock'] = 'buf-config',
}})
```

同时卸载已弃用的旧插件：`uarun/vim-protobuf`、`bufbuild/vim-buf`，以及 ALE 里的 buf-lint/buf-format（职责已被 LSP 覆盖）。

## JetBrains / 其他编辑器

- **IntelliJ 系**：官方插件 "Buf for Protocol Buffers"（插件市场 ID 19147），同样由 Buf LSP server 驱动
- **Zed**：装 Proto 扩展，settings.json 里 `"language_servers": ["buf"]`
- **Emacs**：无第一方插件，lsp-mode / eglot 直连 `buf lsp serve` 即可

## 两分钟验证：一个最小工程

建一个两文件工程，验证跳转和诊断真的在工作：

```
demo/
├── buf.yaml
└── proto/acme/v1/
    ├── user.proto
    └── order.proto
```

```yaml
# buf.yaml
version: v2
modules:
  - path: proto
lint:
  use:
    - STANDARD
```

```protobuf
// proto/acme/v1/user.proto
syntax = "proto3";
package acme.v1;

message User {
  string id = 1;
  string name = 2;
}
```

```protobuf
// proto/acme/v1/order.proto
syntax = "proto3";
package acme.v1;

import "acme/v1/user.proto";

message Order {
  string order_id = 1;
  acme.v1.User buyer = 2;
  int64 amount = 2;  // 故意写错：字段号重复
}
```

配好 LSP 后你应该立刻看到：

1. **实时诊断**：`amount = 2` 处出现波浪线，报 `field number "2" used more than once`——不用保存、不用编译。改成 `= 3` 后消失
2. **跳转定义**：光标放在 `acme.v1.User` 上按跳转键，落到 user.proto 的 message 定义
3. **补全**：输入 `acme.` 出符号补全；写 `import "` 出路径补全
4. **找引用**：在 `User` 上反查，能列出 order.proto 里的使用处
5. **重命名 / Organize imports / 整文件格式化**（格式化结果与 `buf format` 一致）

命令行交叉验证：`buf lint` 应输出同一个字段号错误——编辑器里的波浪线和 CI 里的报错自此是同一套东西。

## 已知的坑

- **保持 buf CLI 为新版**。曾有 nvim 0.12 + 新版 lspconfig 组合触发 LSP 崩溃的 issue（已修复），月更的 CLI 修 bug 很快，别锁老版本
- **格式化只有整文件模式**，不支持格式化选中片段（vscode-buf 的已知限制）
- Windows 上拉取 BSR 公开模块曾有 401 鉴权问题（已修复），依赖 BSR deps 的工程遇到拉不下来先升级 CLI
- VS Code 里 Buf 扩展曾干扰其他 YAML 插件的格式化，如果 buf.yaml 之外的 YAML 文件行为异常，检查扩展冲突

## 常见问题 FAQ

### Protobuf LSP 需要单独安装吗？

不需要。LSP server 打包在 buf CLI 二进制内，装好 buf（`brew install bufbuild/buf/buf`）就有了，编辑器通过 `buf lsp serve` 启动它。VS Code 的 Buf 扩展甚至会在找不到 buf 时自动下载。

### buf lsp 和 protoc 报错不一致怎么办？

以 buf 为准并统一工具链。Buf 的编译器前端 protocompile 是全规范实现且诊断更精确；如果 CI 还在用 protoc 而编辑器用 buf LSP，建议把 lint/build 也迁到 buf，让编辑器和 CI 同源。

### Neovim 里 buf.yaml 文件没有 LSP 功能？

大概率是漏了 filetype 注册。`buf-config` 不是 Neovim 内置 filetype，需要用 `vim.filetype.add` 把 buf.yaml / buf.gen.yaml / buf.lock 映射过去，LSP 才会附着。

### 已经在用 vscode-proto3 或社区 LSP，有必要换吗？

有。vscode-proto3 只有高亮和保存后诊断，无跳转/重命名/引用查找，且已基本停更；社区 LSP 的解析器均非全规范。官方 LSP 的诊断与 buf lint 规则同源，这是"编辑器所见即 CI 所得"的唯一方案。

## 参考链接

- [Protobuf finally has LSP support — Buf Blog](https://buf.build/blog/protobuf-lsp)
- [Buf CLI 安装文档](https://buf.build/docs/cli/installation/)
- [编辑器集成官方文档（VS Code / Neovim / IntelliJ / Zed）](https://buf.build/docs/cli/editor-integration/)
- [vscode-buf 扩展](https://marketplace.visualstudio.com/items?itemName=bufbuild.vscode-buf)
- [protocompile — Buf 的 Protobuf 编译器前端](https://github.com/bufbuild/protocompile)
- [Hacker News 讨论（2026-08 翻红帖）](https://news.ycombinator.com/item?id=49322573)
