# 果蝇、小鼠、人：全脑模拟还差多远？用 Mac mini 实测算一笔账

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/whole-brain-emulation-fly-mouse-human?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/whole-brain-emulation-fly-mouse-human?utm_source=github&utm_medium=referral)**

「科学家上传了一只果蝇的大脑」之后，下一个问题几乎总是：**那人脑呢？**

这篇用一个具体的办法回答：拿我们在 Mac mini 上真实跑过的果蝇全脑模拟做基准，照同样的模型和算法，把规模放大到小鼠和人，看看会撞上哪几堵墙。先说结论：

- **神经元数量差了五六个数量级。** 果蝇脑 139,255 个，小鼠 7,089 万个（约 500 倍），人 861 亿个（约 62 万倍）。
- **第一堵墙是测绘。** 迄今完整画出来的最大神经系统是果蝇；哺乳动物只画完了小鼠 1 立方毫米的皮层，约占整只小鼠脑的四百分之一。
- **第二堵墙是校对。** 果蝇全脑的人工校对花了约 33 人年。线性外推，小鼠要约 1.7 万人年。
- **第三堵墙是算力和内存。** 果蝇在 Mac mini 上模拟 1 秒要 12.6 秒；外推到小鼠约 41 小时、1.4 TB 内存，人脑约 5.7 年、1.7 PB 内存。
- **第四堵墙最根本：接线图本身不够。** 我们实测，现有的果蝇全脑模型不给输入时一个脉冲都不发；它没有学习、记忆，也没有神经调质。

## 三种大脑有多大

| 物种 | 神经元 | 完整接线图 | 出处 |
|---|---|---|---|
| 秀丽隐杆线虫 | 约 300 | 1986 年画完 | Dorkenwald 2024 引 White 1986 |
| 果蝇幼虫 | 约 3,000 | 2023 年画完 | 同上，引 Winding 2023 |
| 成年果蝇（脑） | 139,255 | 2024 年，FlyWire | Dorkenwald et al. 2024, Nature |
| 成年果蝇（脑 + 腹神经索） | 166,700 | 2026 年，MaleCNS | Berg et al. 2026, Cell |
| 小鼠 | 7,089 万 | 只画完 1 mm³ 皮层 | Herculano-Houzel et al. 2006, PNAS |
| 人 | 861 亿 | 远未开始 | Azevedo et al. 2009 |

人脑「1,000 亿个神经元」是个流传很广的说法。2009 年 Azevedo 等人用「各向同性分离法」（把脑组织打成均匀的细胞核悬液再计数）实际数了一遍，得到的是 861 亿 ± 81 亿。

## 第一堵墙：测绘

画接线图的第一步是用电镜把整块组织以纳米精度拍下来。看看已经拍过的体积：

| 数据集 | 成像体积 | 规模 |
|---|---|---|
| 果蝇脑（FlyWire 的神经纤维网） | 0.0175 mm³ | 13.9 万个神经元 |
| 果蝇全中枢（MaleCNS） | 0.082 mm³ | 16.7 万个神经元，7 台电镜拍了 13 个月 |
| 小鼠视觉皮层（MICrONS） | 1 mm³ | 20 多万个细胞、约 5 亿个突触 |
| 整只小鼠脑 | 约 400 mm³（脑重 0.42 克） | 7,089 万个神经元 |
| 人脑 | 约 120 万 mm³ | 861 亿个神经元 |

MICrONS（2025 年发表在 Nature）是目前最大的哺乳动物连接组之一，但它只是整只小鼠脑的约四百分之一。

如果用 MaleCNS 的成像速度（7 台电镜 13 个月拍 0.082 mm³）去拍一整只小鼠脑，要大约 5,000 年。这当然是个过于朴素的算法——电镜在变快，可以开更多台，MICrONS 用的也是另一套成像方案——但它说明了量级：**小鼠脑的体积是果蝇全中枢的约 5,000 倍。**

## 第二堵墙：校对

拍下来只是开始。AI 自动分割会犯两类错：把两个神经元粘在一起，或把一个神经元断成几截。这些错只能靠人改：

- FlyWire（果蝇脑）：3,013,513 次人工修改，约 **33 人年**。
- MaleCNS（果蝇全中枢）：29 名专业校对员做了 3 年，约 **44 人年**。

...

---

**[👉 继续阅读全文：果蝇、小鼠、人：全脑模拟还差多远？用 Mac mini 实测算一笔账](https://tools.cooconsbit.com/zh/articles/whole-brain-emulation-fly-mouse-human?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
