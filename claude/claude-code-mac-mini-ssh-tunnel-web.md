# 把家里的 Mac mini 变成 24 小时在线的 Claude Code 工作站：claudecodeui + SSH 反向隧道，手机浏览器随时接管

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-mac-mini-ssh-tunnel-web?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-mac-mini-ssh-tunnel-web?utm_source=github&utm_medium=referral)**

家里的 Mac mini 是常年开机的：Claude Code 在上面同时管着五六个项目的工作区，长任务一跑就是几十分钟。问题是人不总在家——在外面想看一眼任务跑到哪了、批准一个权限请求、给它派个新活，怎么办？

目标很具体：**在任何有浏览器的设备上**打开一个网址，登录，就能看到并操作 Mac mini 上的 Claude Code 会话。不装客户端、不连 VPN，公司电脑和手机都能用。

这套方案上线一周，每天在用。本文按真实决策顺序写：先讲两个选型（Web 界面选什么、通道选什么），再给完整落地配置，然后是实际效果截图和运行数据，最后是已经踩过的坑。

## 选型一：Web 界面用什么

「浏览器里操作 Claude Code」有四条路，我逐一排除到最后一个：

**官方 Web 版（claude.ai/code）**：最先排除。它跑在 Anthropic 的云沙箱里，操作的是沙箱里的代码副本——而我要的是操作 **Mac mini 本地的工作区**：本地的仓库、本地的 `.env`、本地已经登录好的各种 CLI。两者根本不是一个东西。

**ttyd + tmux（终端 Web 化）**：架构上完全可行，把 tmux 会话通过 ttyd 挂到网页上就是一个远程终端。但在手机上用过五分钟就知道问题在哪：触屏敲终端是折磨，方向键、Ctrl 组合键、滚动回看历史输出，每一样都很别扭。它给你的是「一块远程屏幕」，而不是「为移动端设计的操作界面」。

**code-server（VS Code Web 版）**：能用，但绕。它本体是编辑器，Claude Code 还得在它的集成终端里跑——等于套了一层重壳（内存占用以 GB 计）来获得一个更差的终端体验，移动端布局也基本不可用。

**claudecodeui（开源，13.5k stars）**：最终选择。决定性的一条是它的数据模型——

> claudecodeui **不维护自己的会话状态**，它直接读 `~/.claude/projects/` 下的会话 JSONL 文件。这意味着你在终端里开的 Claude Code 会话，网页上看到的是**同一份**；反过来在网页上发起的会话，回到电脑上 `claude --resume` 也能接着聊。Web 界面只是同一份数据的另一个视图，不产生「网页会话」和「终端会话」两套账。

这一条直接命中我的核心场景：外出时接管的就是**出门前在终端里开的那个会话**，而不是另起炉灶。其余加分项：

- **移动端优先的 UI**：项目列表抽屉、会话流、底部输入框，是按手机屏幕设计的，不是桌面布局硬缩
- **自带认证**：JWT 登录 + bcrypt 密码，登录接口天然适合在 nginx 层再加限流（后文有配置）
- **不止聊天**：内置文件浏览器（带编辑）、git 面板、真·终端（WebSocket），聊天解决不了的时候还有兜底
- **开源可改**：这点一周内就兑现了——上线第二天撞上一个上游 bug，fork 下来自己修了（见「踩过的坑」）

实测足迹很轻：Node 单进程,RSS 稳定在 **170MB 左右**，对一台还要跑正经任务的 Mac mini 来说无感。

## 选型二：为什么 SSH 反向隧道，而不是 Tailscale

家庭宽带没有公网 IP，这是前提。「从外面够到家里的机器」教科书答案是 Tailscale，但对**这个具体需求**——任何浏览器可达的 Web 入口——SSH 反向隧道在四个维度上更合适：

**1. 访问端零安装（决定性）。** Tailscale 的模型是「设备入网」：每台访问设备都要装客户端、登录账号。手机可以装，**公司电脑往往不让装**，临时借用的设备更不可能。SSH 隧道 + nginx 的产出是一个普通 HTTPS 网址，任何有浏览器的东西都能用。「零安装」不是锦上添花，它决定了这个入口在多少真实场景下可用。

**2. 链路自己可控。** Tailscale 靠 NAT 打洞直连，打不通就回落 DERP 中继——国内运营商级 NAT 加上对端网络限制，打洞成功率并不乐观，回落后流量绕 Tailscale 的海外节点，延迟和稳定性都不受你控制。SSH 隧道的链路是固定且自选的：Mac mini → 你挑的 VPS → 你。线路好不好买 VPS 之前就知道，出问题 `ssh -v` 一眼看穿，没有打洞、中继、第三方控制面这些「今天怎么又慢了」的玄学层。

**3. 暴露面是一个端口，不是一台整机。** Tailscale 入网后整台机器对网内可达（所有监听端口），信任边界是「账号安全 + ACL 写对」。反向隧道暴露的是恰好一个端口上的恰好一个服务，且该端口在 VPS 上只监听回环——公网唯一能碰的是 nginx 上那一个 URL。审计是「三道门锁好了没」，不是「整张网的 ACL 有没有漏」。

**4. 复用已有资产。** 我的 VPS 上 nginx、证书自动续期、日志体系都是现成的，这套方案只新增一个 vhost 和一条隧道；Tailscale 则要引入新账号体系 + 每台设备一个常驻 agent + 一套 ACL 配置语言。

**反过来，什么时候该选 Tailscale**：只从自己的设备访问且每台都能装客户端；要的是整机能力（远程桌面、SMB、直接 SSH）；没有 VPS 也不想维护。以及一条要诚实说的——本方案的入口是**公网可达**的，安全完全依赖认证那几道门；Tailscale 的服务只在私有网络内可见，天然少一类风险。Web 界面没有可靠认证的话，不要用本方案裸奔。

顺带一句 Cloudflare Tunnel：思路与本方案相同（从内网拨出、边缘做入口）且免费，没有 VPS 的话是最好替代；有 VPS 的话，自己的 nginx 在超时策略、限流、日志上都更可控。

## 架构

```
[Mac mini 家中]                      [VPS 公网]                      [任意浏览器]
claudecodeui                SSH 反向隧道        nginx (TLS+限流)
127.0.0.1:18300   ────────►  127.0.0.1:18300  ────────►  https://code.example.com
（只监听回环）      ssh -R       （只监听回环）      反向代理
```

...

---

**[👉 继续阅读全文：把家里的 Mac mini 变成 24 小时在线的 Claude Code 工作站：claudecodeui + SSH 反向隧道，手机浏览器随时接管](https://tools.cooconsbit.com/zh/articles/claude-code-mac-mini-ssh-tunnel-web?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
