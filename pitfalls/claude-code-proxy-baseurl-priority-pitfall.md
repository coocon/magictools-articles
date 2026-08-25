# ANTHROPIC_BASE_URL 设了却不生效

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-proxy-baseurl-priority-pitfall?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-proxy-baseurl-priority-pitfall?utm_source=github&utm_medium=referral)**

## 现象

给 Claude Code 挂第三方中转站（one-api / new-api 那类 API 网关），配置写得规规矩矩，`~/.zshrc` 里三行齐全：

```bash
export ANTHROPIC_BASE_URL="https://xxxx.example.com"
export ANTHROPIC_AUTH_TOKEN="sk-..."
export CLAUDE_CODE_USE_VERTEX=0
```

新开终端，`claude` 启动，顶部写着：

```
Fable 5 · Google Vertex AI
```

Vertex。中转站的 base URL 像是根本没被看见。

同一天还有第二件事：CloudCLI（claudecodeui 改名后的 Web UI，用来远程控制本机 claude）由 macOS launchd 拉起，登进去一直让选 provider，提示未认证。可命令行里 `claude` 明明已经能走中转站了——同一个用户，同一台机器。

两个现象，两个独立的坑，共同点是：**环境变量你以为设上了，进程里其实不是那么回事。**

## 坑一：settings.json 的 env 比 shell export 大

先查环境变量本身有没有生效。`echo $ANTHROPIC_BASE_URL` 是对的，`echo $CLAUDE_CODE_USE_VERTEX` 也是 0。shell 这层没问题。

那就翻 Claude Code 自己的配置。`~/.claude/settings.json` 的 `env` 块里躺着三行测试残留：

```json
{
  "env": {
    "CLAUDE_CODE_USE_VERTEX": "1",
    "ANTHROPIC_VERTEX_PROJECT_ID": "hello",
    "CLOUD_ML_REGION": "global"
  }
}
```

`ANTHROPIC_VERTEX_PROJECT_ID` 值是 `hello`——一眼就知道是当初试 Vertex 时随手填的，试完忘了删。

根因清楚了：**`settings.json` 的 `env` 字段是「强制注入」，优先级高于 shell 里的 export。** 你在 zshrc 里写的 `CLAUDE_CODE_USE_VERTEX=0`，进程启动时被这里的 `"1"` 盖掉。Claude Code 检测到 Vertex 配置齐活，就走 Vertex 通道去了，`ANTHROPIC_BASE_URL` 从头到尾没人问津——它压根不在那条代码路径上。

...

---

**[👉 继续阅读全文：ANTHROPIC_BASE_URL 设了却不生效](https://tools.cooconsbit.com/zh/articles/claude-code-proxy-baseurl-priority-pitfall?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
