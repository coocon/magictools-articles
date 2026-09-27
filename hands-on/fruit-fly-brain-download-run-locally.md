# 果蝇大脑怎么下载？13.9 万个神经元，我在 Mac mini 上 33 秒跑完一次实验

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/fruit-fly-brain-download-run-locally?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/fruit-fly-brain-download-run-locally?utm_source=github&utm_medium=referral)**

搜「果蝇大脑下载」的人，大多是看了「科学家把果蝇大脑上传到电脑」的新闻，想自己试试。好消息是：真的可以，而且比想象中简单。坏消息是：官方示例照着 README 改一行配置就会报错。

我在一台 Mac mini 上从零走了一遍，全程计时。先给结论：

- **「果蝇大脑」是两个文件**：138,639 个神经元的清单（3.3 MB），和一张 15,091,983 行的连接表（101 MB）。每一行写着「谁连着谁、有几个突触、是兴奋还是抑制」。
- **普通电脑就能跑**：克隆 57 秒，装依赖 21 秒，官方的糖味觉示例（30 次试验 × 1 秒脑时间）**33 秒**跑完。
- **照 README 切到公开版 v783 数据会直接 `KeyError`**，因为示例里的神经元 ID 是旧版本的。改法很简单，下文给出能直接跑的版本。
- 跑通以后，你可以做真实果蝇上很难做的实验：一次关掉一个神经元，看整只脑怎么变。

## 问题背景：下载的到底是什么

2024 年 FlyWire 联盟在 Nature 上发布了第一张完整的成年果蝇全脑接线图（连接组）。同期 Shiu 等人在 Nature 上发表了一个基于它的全脑模型：把每个神经元简化成「会漏电的积分器」（LIF），连接强度直接用突触数量，在 Brian2 模拟器里跑。

这个模型的官方仓库 [philshiu/Drosophila_brain_model](https://github.com/philshiu/Drosophila_brain_model) 把公开版数据一起放在了里面，所以「下载果蝇大脑」最直接的做法就是克隆它：

| 文件 | 大小 | 内容 |
|---|---|---|
| `Completeness_783.csv` | 3.3 MB | 138,639 个神经元的 FlyWire ID |
| `Connectivity_783.parquet` | 101 MB | 15,091,983 条连接：上游 ID、下游 ID、突触数、兴奋/抑制 |
| `model.py` / `utils.py` | 14 KB | 模型本体与读结果的工具函数 |
| 旧版 v630 数据 | 90 MB | 论文当时用的版本，示例默认还指着它 |

整个仓库克隆下来 380 MB，其中一半是 git 历史。想要更全的数据（细胞类型、神经递质、三维形态），去 FlyWire 的 Codex（codex.flywire.ai），那里是官方数据门户。**许可是 CC BY-NC 4.0：署名、不可商用。**

## 这张表里有什么

在跑模型之前，先把表拆开看看，几个数字挺有意思：

| 项 | 数值 |
|---|---|
| 神经元 | 138,639 |
| 有连接的神经元对 | 15,091,983 |
| 突触总数（把「突触数」一列加起来） | **54,492,922** |
| 兴奋性连接占比 | 60% |
| 只有 1 个突触的连接 | 49.7% |
| 5 个及以上突触的连接 | 17.9% |
| 每个神经元平均连向 | 109 个下游 |
| 两个神经元之间最多的突触 | 2,405 个 |
| 连接密度 | 0.08%（每个神经元只连了全脑的万分之八） |

第三行是一个很好的交叉验证：Nature 论文写的是「5,450 万个突触」，这张表加起来是 5,449 万，对得上。

还有两个值得一提：

- **一半的连接只有 1 个突触。** 自动检测会有误报，所以 FlyWire 在 Codex 上默认用「至少 5 个突触」才算一条连接。按这个标准，这张表只剩不到五分之一。Shiu 的模型没做这个过滤，1 个突触的连接也算进去了，只是权重小。
- **全脑连接最多的神经元只有两个，而且是同一种。** 连出最多的（9,783 个下游）和连入最多的（10,356 个上游）分别是左右视叶里的 CT1——这种细胞全脑只有 2 个，每侧一个，单个细胞的分支横跨视叶的髓质和小叶（细胞类型来自 FlyWire 官方注释，Schlegel et al. 2024）。

## 实测过程

环境：Mac mini M4（10 核、24 GB），Python 3.12，Brian2 2.10.1（代码生成后端自动选了 Cython）。我用 uv 管环境，下面给等价的 pip 命令。**Windows 我没有测。**

### 第一步：克隆和安装

```bash
git clone https://github.com/philshiu/Drosophila_brain_model.git   # 57 秒，380 MB
cd Drosophila_brain_model
python3 -m venv .venv && source .venv/bin/activate
pip install brian2 pandas pyarrow joblib                           # 21 秒（无缓存）
```

...

---

**[👉 继续阅读全文：果蝇大脑怎么下载？13.9 万个神经元，我在 Mac mini 上 33 秒跑完一次实验](https://tools.cooconsbit.com/zh/articles/fruit-fly-brain-download-run-locally?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
