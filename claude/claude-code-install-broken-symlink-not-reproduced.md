---
title: "claude install 断链、%h 未展开？Debian 容器实测装出来是好的（未复现）"
slug: claude-code-install-broken-symlink-not-reproduced
category: claude
locale: zh
translationSlug: claude-code-install-broken-symlink-not-reproduced-en
tags: [Claude Code, 安装, install.sh, symlink, Linux, claude-code-lab]
summary: "GitHub 上有 Fedora 用户报告：官方一键安装脚本装完后 ~/.local/bin/claude 是条断链，指向字面量 %h/.local/share/claude/versions/2.1.220——%h 这个占位符没被展开成家目录。我们在干净的 Debian 12 容器里按原报告的完整命令实测，装出来的符号链接是正常的（%h 已正确展开），且当前 install.sh 分发的版本已是 2.1.223（报告版本为 2.1.220）。本文是一篇「未复现实录」：完整给出实测环境、命令与输出证据，分析未复现的三种可能解释，并附上如果你正被断链卡住的一分钟手工修复。"
status: published
lab:
  testedAt: "2026-08-06"
  ccVersion: "2.1.223"
  model: "n/a（安装流程，与模型无关）"
  platform: "Linux（Docker debian:bookworm-slim）"
  status: not-reproduced
docsUrl: https://docs.claude.com/en/docs/claude-code/setup
---

> **本文性质**：未复现实录。我们没有复现出报告中的问题——这不代表报告是错的，只代表在下述环境、用下述方法测不出来。全部命令与原始输出都在文中，欢迎交叉验证。

## 原始报告说了什么

GitHub issue [#83484](https://github.com/anthropics/claude-code/issues/83484)（Fedora 44 + bash，Claude Code 2.1.220）：运行官方一键安装后，`~/.local/bin/claude` 是一条断掉的符号链接——

```console
$ curl -fsSL https://claude.ai/install.sh | bash
$ file ~/.local/bin/claude
/home/user/.local/bin/claude: broken symbolic link to %h/.local/share/claude/versions/2.1.220
```

链接目标里的 `%h` 是个未展开的占位符（systemd 风格的家目录指示符），本应是 `/home/user`。报告者定位问题不在 shell 安装脚本，而在二进制内部的 `install` 子命令；并称此前版本正常，属回归。issue 关联了一个修复 PR（#83738，报告时未合并）。

## 我们的实测：装出来是好的

环境：干净的 Docker 容器 `debian:bookworm-slim`，root 用户，2026-08-06 执行与报告完全相同的安装命令：

```console
$ curl -fsSL https://claude.ai/install.sh | bash
$ ls -la /root/.local/bin/claude
lrwxrwxrwx 1 root root 42 Aug  6 10:23 /root/.local/bin/claude -> /root/.local/share/claude/versions/2.1.223
$ file /root/.local/bin/claude
/root/.local/bin/claude: symbolic link to /root/.local/share/claude/versions/2.1.223
```

两个关键观察：

1. **符号链接是正常的**——目标路径里的家目录已正确展开为 `/root`，没有出现字面量 `%h`
2. **install.sh 当前分发的版本是 2.1.223**，比报告中的 2.1.220 新了三个版本（2.1.221/222/223 分别发布于 8 月 3/4/5 日）

## 为什么没复现：三种可能解释

按证据强度排序：

**解释一：已在 2.1.221–2.1.223 之间修复（最可能）。** 报告发生在 2.1.220，关联修复 PR 存在，而现在 install.sh 直接分发 2.1.223——如果修复已合入其中任何一版，新装用户自然不会再遇到。但我们没有找到明确写着"修复 %h 展开"的发版说明，无法坐实。

**解释二：发行版相关。** 报告环境是 Fedora 44，我们测的是 Debian 12。`%h` 是 systemd 单元文件里的家目录指示符——如果二进制的 install 子命令在某些系统上从 systemd 相关配置读取路径模板，行为可能随发行版而异。这只是机制猜想，我们没有 Fedora 环境验证。

**解释三：报告环境的残留状态。** 报告者称"此前版本正常"，旧安装的残留配置可能参与了路径生成。干净容器里没有任何残留，这或许正是测不出来的原因——但同样无法证实。

## 如果你正被断链卡住：一分钟修复

无论根因是什么，断链的修复是确定的（来自 issue 中报告者验证过的 workaround）：

```bash
# 把断链指向真实存在的版本目录（版本号按你机器上实际的改）
ls ~/.local/share/claude/versions/        # 先看有哪些版本
ln -sf ~/.local/share/claude/versions/<版本号> ~/.local/bin/claude
claude --version                          # 验证可用
```

更省事的选择：直接重跑一次官方安装命令——当前分发的 2.1.223 在我们的实测里装出来就是好的。

## 适用边界

- 本文只证明：**Debian 12 干净环境 + 2026-08-06 的 install.sh（分发 2.1.223）不出现此问题**
- 不能证明：Fedora 上已修复、2.1.220 的问题不存在、或所有环境都正常
- 如果你在其它发行版上仍能复现（尤其 Fedora + 2.1.221 及以后版本），请到 issue #83484 补充环境信息——那将直接推翻解释一、坐实解释二
