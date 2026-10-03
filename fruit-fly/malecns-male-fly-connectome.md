# MaleCNS 是什么？第一张雄性果蝇全中枢神经接线图，和 FlyWire 差在哪

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/malecns-male-fly-connectome?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/malecns-male-fly-connectome?utm_source=github&utm_medium=referral)**

如果你最近看过「果蝇大脑玩游戏」一类的开源项目，会发现 9 月以后的新项目几乎都写着同一个名字：**MaleCNS**。

它是 Male Central Nervous System 的缩写：一只**雄性**果蝇的**完整中枢神经系统**接线图。先说结论：

- **它是第一张把大脑和腹神经索放在一起的果蝇接线图。** 腹神经索相当于果蝇的脊髓，腿和翅膀的运动神经元都在这里。之前的 FlyWire 只有脑。
- **规模**：166,700 个神经元，11,710 种神经元类型，全部人工校对并注释。
- **时间线**：2025 年 10 月发布 v0.9，2026 年 6 月 8 日发布 v1.0，2026 年 9 月在 Cell 上发表论文。
- **许可证是 CC BY 4.0，署名即可商用**；FlyWire 是 CC BY-NC 4.0，不能商用。
- **它和 FlyWire 是两套完全独立的工程**：不同性别的果蝇、不同的电镜、不同的 AI 分割方法。
- **雌雄果蝇的大脑，绝大部分是一样的。** 论文发现两性差异只出现在少数细胞类型，而且集中在高级脑中枢。

## MaleCNS 和 FlyWire 对照

MaleCNS 由 Janelia 研究所的 FlyEM 团队、剑桥大学动物学系、英国 MRC 分子生物学实验室和 Google Research 合作完成。下面的数字来自它的论文（Cell 2026 与 bioRxiv 预印本）和 FlyWire 的两篇原始论文：

| | FlyWire（FAFB） | MaleCNS |
|---|---|---|
| 果蝇 | 一只雌蝇 | 一只雄蝇 |
| 覆盖范围 | 大脑（含视叶） | 大脑 + 视叶 + **腹神经索** |
| 电镜 | 连续超薄切片透射电镜 | 增强型聚焦离子束扫描电镜（eFIB-SEM） |
| 切法 | 7,062 片，每片 35～40 nm | 先用「热刀」切成 20 µm 厚的厚片，再在电镜里逐层磨削成像 |
| 体素 | 4 × 4 × 40 nm | 8 × 8 × 8 nm，三个方向一样细 |
| 成像 | 2 台定制电镜，约 16 个月 | 7 台电镜并行，13 个月，160 万亿体素 |
| 自动分割 | 普林斯顿的边界检测卷积网络 | Google 的洪水填充网络（FFN） |
| 神经元 | 139,255 | 166,700 |
| 细胞类型 | 8,453 | 11,710 |
| 人工校对 | 约 33 人年，科研社区 + 专业团队 + 公民科学家 | 约 44 人年，29 名专业校对员做了 3 年 |
| 突触 | 约 1.3 亿个（已校对神经元之间 5,450 万） | 4,600 万个突触前位点，连接着 3.12 亿个突触后位点 |
| 许可证 | CC BY-NC 4.0（不可商用） | CC BY 4.0（可商用） |

两点值得展开：

**一、成像思路完全相反。** FlyWire 是先把脑切成几千片极薄的切片，再逐片拍；MaleCNS 是只切成几十块 20 微米厚的「厚片」，然后在聚焦离子束电镜里一边用离子束磨掉表面薄薄一层、一边拍下新露出的表面。后者三个方向的分辨率都是 8 纳米，图像更规整，但每台机器很慢，所以用了 7 台同时拍。腹神经索就切成了 31 块厚片。

**二、突触的数法不一样。** 果蝇的突触常常是「一对多」：一个突触前位点同时连着好几个下游神经元的突触后位点。MaleCNS 论文报的是 4,600 万个突触前位点对应 3.12 亿个突触后位点；而 FlyWire 的「5,450 万个突触」，是把每一对「突触前—突触后」各算一个。所以这两个数字不能直接比大小。社区项目 README 里常见的「2,500 多万条连接」，指的又是有连接的神经元对数——口径的坑，我在[社区项目盘点](/zh/articles/fruit-fly-brain-community-projects)里专门列过一张表。

...

---

**[👉 继续阅读全文：MaleCNS 是什么？第一张雄性果蝇全中枢神经接线图，和 FlyWire 差在哪](https://tools.cooconsbit.com/zh/articles/malecns-male-fly-connectome?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
