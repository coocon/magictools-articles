# claude install 断链、%h 未展开？Debian 容器实测装出来是好的（未复现）

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-install-broken-symlink-not-reproduced?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-install-broken-symlink-not-reproduced?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：claude install 断链、%h 未展开？Debian 容器实测装出来是好的（未复现）](https://tools.cooconsbit.com/zh/articles/claude-code-install-broken-symlink-not-reproduced?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
