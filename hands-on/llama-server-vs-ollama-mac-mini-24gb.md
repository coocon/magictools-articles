# llama.cpp 部署 llama-server 还是用 ollama？Mac mini 24GB 同一个 GGUF 实测：ollama 底下跑的就是 llama-server

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/llama-server-vs-ollama-mac-mini-24gb?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/llama-server-vs-ollama-mac-mini-24gb?utm_source=github&utm_medium=referral)**

## 问题背景

在 Mac 上跑本地模型，绕不开一个选择：直接用 llama.cpp 的 `llama-server`，还是装 ollama？

网上的对比大多停留在「ollama 简单，llama.cpp 灵活」。真部署过就会发现，影响结果的往往不是哪个更快，而是一些默认值：上下文有多长、超长了会怎样、几个请求能同时跑、内存占多少。这些默认值不同，同一个模型在两边的表现就会完全不同。

这次在一台 Mac mini M4 24GB 上，让两边加载**同一个 GGUF 文件**，用同一套脚本、走同一个接口（OpenAI 兼容的 `/v1/chat/completions`，流式）逐项测：部署、接口、速度、默认上下文、长文、并发、内存，以及 27B 模型在 24GB 机器上的表现。

## 问题分析

开测前先看了一眼进程，结论在第一步就出来了：**ollama 0.35.1 的推理进程，就是它安装包里自带的 `llama-server`**。

![ollama serve 的子进程是 bin/ollama/llama-server，参数里有 -c 4096 -np 1](https://cdn.tools.cooconsbit.com/uploads/articles/2026-10-03-llamacpp-vs-ollama/02-ollama-runner-is-llama-server.png)
*图：测速脚本在 ollama 运行时采到的进程表（路径已缩写）。上面是 ollama 官方库里的 qwen3:4b，下面是我用 `ollama create` 导入的 Qwen3.8 GGUF。*

ollama 解压后的目录里能看到 `llama-server`（`--version` 显示 `0.5.0-dev (build 1, commit 6f767fe96)`），还有 `mlx_metal_v3/v4` 等 MLX 相关文件。加载 GGUF 模型时，`ollama serve` 会拉起这个 llama-server 当 runner，传给它的关键参数是：

| ollama 传的参数 | 含义 |
|---|---|
| `-c 4096 -np 1` | 上下文 4096，只有 1 个并发槽 |
| `--no-jinja --chat-template chatml` | 官方库模型由 ollama 自己套模板（导入的 GGUF 没有这两个参数） |
| `--context-shift --keep 4` | 生成写满上下文时丢掉旧 token 继续生成 |
| `-b 512 -ub 512` | 逻辑批大小 512，llama-server 默认是 `-b 2048 -ub 512` |
| `--no-webui` | 关掉 llama-server 自带的网页界面 |

所以这道题不是「两个引擎谁快」，而是**同一个 llama-server，ollama 替你填了哪套参数、包了哪层管理**。下面的实测基本都在验证这件事。

## 技术方案与选型

- **版本**：llama.cpp 官方 release `b11376`（2026-10-03，macos-arm64 预编译包）；ollama 官方 release `v0.35.1`（2026-09-29，`ollama-darwin.tgz`）。两个都解压到临时目录运行，没有动本机原来装的 brew ollama 0.19。
- **模型**，两边都是同一个文件：
  - 4B：ollama 官方库 `qwen3:4b` 的 GGUF blob（2.5GB，Q4_K_M，元数据名为 Qwen3 4B Thinking 2507）。llama-server 直接 `-m` 加载这个 blob（硬链接，不另占空间）。
  - 27B：本机已有的 `Qwen3.8-27B-UD-Q4_K_S.gguf`（15.4GB）。ollama 用 `ollama create` 从这个文件导入，llama-server 直接加载。
- **测法**（Python 脚本，标准库实现）：每个 case 起一个服务，依次测冷启动、顺序解码（长文写作，max_tokens 300 / 27B 为 200，各 5 轮 / 3 轮）、预填充（约 2,500 token 的值班记录，各 5 / 3 轮）、长文找针（开头埋一句「暗号是青鸟-7421」，后面堆满无关文字，结尾提问；4B 约 15.7k token，27B 约 7.6k token）、并发（4B 4 路 / 27B 2 路同时发）、内存（macOS `footprint`）。每个 prompt 前加随机 UUID，避免命中前缀缓存。`temperature 0`。
- **读数口径**：解码速度 =（completion_tokens − 1）÷（最后一个流式片段时间 − 第一个片段时间）；预填充速度 = prompt_tokens ÷ 首 token 时间，属于客户端计时，含 HTTP 开销。内存用的是 footprint，**不包含 mmap 映射的权重文件**，看的是 KV 缓存、计算缓冲区这类额外开销；ollama 计入 serve 和 runner 两个进程之和。

## 实测过程

### 1. 部署：都是一条命令，差别在默认值

```bash
# llama.cpp：下载 release 包解压即可运行；模型加载完才开始监听
./llama-server -m Qwen3.8-27B-UD-Q4_K_S.gguf --host 127.0.0.1 --port 8080

# ollama：先起服务，模型在第一次请求时才加载
ollama serve                         # 默认监听 127.0.0.1:11434
ollama create qwen38-ud -f Modelfile # Modelfile 只有一行：FROM /path/to/Qwen3.8-27B-UD-Q4_K_S.gguf
```

...

---

**[👉 继续阅读全文：llama.cpp 部署 llama-server 还是用 ollama？Mac mini 24GB 同一个 GGUF 实测：ollama 底下跑的就是 llama-server](https://tools.cooconsbit.com/zh/articles/llama-server-vs-ollama-mac-mini-24gb?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
