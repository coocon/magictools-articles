# 网友拿果蝇大脑干了什么？马里奥、我的世界、DOOM……9 个项目逐个拆，复测了 2 个

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/fruit-fly-brain-community-projects?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/fruit-fly-brain-community-projects?utm_source=github&utm_medium=referral)**

2026 年 3 月 Eon Systems 宣布「上传了一只果蝇」之后，果蝇大脑成了开发者的新玩具。光是 9 月份，GitHub 上就冒出了一串项目：让果蝇大脑打 DOOM、玩马里奥、玩 Flappy Bird、在《我的世界》里当 NPC、跑在鸿蒙手机上、住在 Mac 桌面上，甚至还有人拿它做股票交易信号。

标题一个比一个唬人。我把 9 个项目的 README 和关键代码逐个读完，对每个都问三个问题：

1. **用了多少个神经元？** 全脑 13.9 万 / 16.6 万，还是从里面抽一小块，还是自己搭的几十个？
2. **动作到底是谁决定的？** 连接组本身、训练出来的读出层、状态机，还是手调的权重？
3. **有没有对照？** 把某条通路切断，行为会不会跟着变？

然后挑了两个能在 Mac 上跑的项目亲手复测。先说结论：

- **这批项目里最「诚实」的，恰恰是最火的那几个。** DOOMFLY（417 星）在 README 第一段就写明「目前没有证明学会了生存」，DesktopFly（1,063 星）单列了一节「哪些是真的」。
- **「用了全脑」不等于「全脑在做决定」。** 好几个项目全量加载了十几万个神经元，但真正决定动作的，是一个训练出来的读出层，或者一个状态机。
- **21 个神经元、手调权重的项目也能通关马里奥，但很脆弱。** 我把它的所有权重整体缩到 0.8 倍，果蝇一步也不走了；放大到 1.3 倍，跑到一半摔死。

## 9 个项目一览

星数截至 2026-10-03。

| 项目 | 干了什么 | 用的数据 | 神经元规模 | 动作由谁决定 |
|---|---|---|---|---|
| [DOOMFLY](https://github.com/nftechie/doomfly)（417★） | 果蝇大脑打 DOOM | MaleCNS v1.0 | 全量 166,700 | 固定的神经元→按键映射，外加实验性多巴胺可塑性 |
| [DesktopFly](https://github.com/DenisSergeevitch/desktop-fly)（1,063★） | Mac 桌面上的 3D 果蝇宠物 | FlyWire + MaleCNS | 抽取 668 + 1,045 | 子回路 LIF + 建模的身体与状态 |
| [fly-flappy](https://github.com/ns2250225/fly-flappy)（28★） | 果蝇大脑玩 Flappy Bird | MaleCNS v1.0 | 全量 166,700 | 训练出来的逻辑回归读出层 + 安全护栏 |
| [FlyCraft](https://github.com/Yi-111-a/FlyCraft) | 《我的世界》里的战斗 NPC | MaleCNS | 子图约 8,000 节点 | 子图动力学 → 打 / 逃 / 游荡 |
| [赛博果蝇 HarmonyOS](https://github.com/zhuhaozoo/FlyBrain-HarmonyOS) | 鸿蒙手机上的 3D 果蝇生态 | MaleCNS v1.0 | 全量 165,122 | 状态机为主，连接组做六路调制 |
| [Fly Mario](https://github.com/FuChen1649/fly-mario) | 果蝇回路自动通关马里奥 | 只借用了细胞名和 ID | **21 个** | 手调权重的 LIF 小回路 |
| [@chnak/fly](https://github.com/chnak/fly) | TypeScript 训练库，README 里提到做交易信号 | MaleCNS | README 未说清 | 逻辑回归读出层 |
| [果蝇的每一天](https://github.com/SlimeBoyOwO/LingChat/pull/825)（LingChat PR） | Rust 写的 3D 果蝇生活小游戏 | FlyWire v783 | 全量 139,255 | LIF + 奖惩可塑性 + 感觉运动映射 |
| [fly-brain（Rojas）](https://github.com/erojasoficial-byte/fly-brain)（65★） | 两只相同连接组的果蝇「长出个性」 | FlyWire v783 | 全量 138,639 | LIF + 赫布可塑性 + NeuroMechFly 身体 |

下面挑最有意思的几个细说。

## DOOMFLY：最认真，也最坦白

这是目前「果蝇大脑玩游戏」里工程量最大的一个。每一帧 DOOM 画面被转成 **3,335 个感光细胞的亮度输入和 811 个颜色输入**，灌进 MaleCNS 的 166,700 个神经元、25,582,938 条连接，不裁剪任何回路。果蝇受伤时，会给两个多巴胺神经元一个 200 毫秒的「惩罚」信号，驱动蘑菇体里 4,184 条连接的可塑性变化——也就是说，它在尝试真的「学」。

但作者在 README 第一段就写了：

...

---

**[👉 继续阅读全文：网友拿果蝇大脑干了什么？马里奥、我的世界、DOOM……9 个项目逐个拆，复测了 2 个](https://tools.cooconsbit.com/zh/articles/fruit-fly-brain-community-projects?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
