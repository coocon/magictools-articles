# 果蝇大脑真的被「上传」到电脑了吗？把 Eon 的演示拆成四层来看

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/fruit-fly-brain-upload-explained?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/fruit-fly-brain-upload-explained?utm_source=github&utm_medium=referral)**

2026 年 3 月 8 日，旧金山的初创公司 Eon Systems 发了一条帖子：「We've uploaded a fruit fly.」视频里，一只虚拟果蝇循着看不见的味觉线索走向香蕉片，身上落了灰就停下来梳理，梳完继续走，最后开吃。很多人第一次知道：一只动物的大脑，可以被完整地复制进电脑。

真是这样吗？我用他们用到的开源组件在一台 Mac mini 上把这套东西拼了一遍，又逐条读了 Eon 的两篇原文和背后的三篇 Nature 论文。先给结论：

- **被复制的是一张接线图**：一只雌果蝇大脑里 139,255 个神经元之间谁连着谁、连了几个突触。这部分是扎实的，2024 年发表在 Nature 上，数据公开。
- **「让它动起来」的那部分，大多不来自这张图。** 走路、梳理、进食的动作由身体模型里现成的控制器完成，脑和身体之间的对应关系是人手工选的。这是 Eon 自己在技术文章里写明的。
- **流传最广的「91% 行为准确率」被误读了。** 它的出处是 2024 年的一篇论文：模型对进食和梳理回路做了 164 条可检验的预测，91% 与实验一致。那是回路层面的预测，不是这只虚拟果蝇的行为评分。

下面一层一层拆。

## 「上传」由四层拼成

Eon 在帖子里说，他们只用了四样东西：连接图、按突触数定的权重、兴奋/抑制神经元的划分，以及「漏电积分—发放」（LIF）神经元模型。技术文章里则写得更完整：这是一次**集成**，把几项已经发表的成果接在了一起。

| 层 | 是什么 | 谁做的 | 开源情况 |
|---|---|---|---|
| ① 接线图 | 13.9 万个神经元、5,450 万个突触的连接组 | FlyWire 联盟，Nature 2024 | 数据公开，CC BY-NC 4.0 |
| ② 神经元模型 | 每个神经元简化成 LIF 单元，全脑在 Brian2 里跑 | Shiu 等，Nature 2024（2023 年预印本） | 代码 MIT |
| ③ 身体 | 按 X 光显微 CT 扫描建模的果蝇身体，87 个独立关节，跑在 MuJoCo 物理引擎里 | NeuroMechFly v2，Wang-Chen 等 2024 | Apache-2.0 |
| ④ 脑—体接口 | 把少数下行神经元的放电，翻译成转向、前进、梳理、进食指令 | Eon | 未见公开代码 |

另外还有一个视觉模型（Lappalainen 等 2024，flyvis）把复眼画面转成视觉神经元的活动，再喂进全脑模型。但 Eon 自己写道，这部分目前「有点装饰性」，对行为输出影响不大。

前三层都是学术界多年的积累，Eon 的新贡献主要在第四层，以及把四层接成闭环：感觉输入 → 脑 → 动作 → 新的感觉输入，每 15 毫秒同步一次。

## 第一层：接线图是怎么来的

这张图来自一只成年雌果蝇的大脑（数据集名叫 FAFB，Female Adult Fly Brain）。大脑被切成几千片超薄切片，每片用电子显微镜成像，AI 把每根神经纤维描出来，再由科研人员和世界各地的公民科学家逐个纠错。Nature 论文估计，人工校对一共花了约 **33 人年**。

结果是 139,255 个神经元、5,450 万个突触。果蝇的突触密度是每立方微米 7.4 个，哺乳动物皮层不到 1 个。

我把公开数据下载下来核对过：连接表有 15,091,983 行，把每行的突触数加起来是 54,492,922，和论文对得上（完整过程见[果蝇大脑怎么下载](/zh/hands-on/fruit-fly-brain-download-run-locally)）。

...

---

**[👉 继续阅读全文：果蝇大脑真的被「上传」到电脑了吗？把 Eon 的演示拆成四层来看](https://tools.cooconsbit.com/zh/articles/fruit-fly-brain-upload-explained?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
