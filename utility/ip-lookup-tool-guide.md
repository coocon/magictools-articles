# IP 地址查询工具完全教程：查本机 IP、定位和网络诊断

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/ip-lookup-tool-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/ip-lookup-tool-guide?utm_source=github&utm_medium=referral)**

## IP 地址基础知识

在深入了解工具功能之前，有必要先理清几个容易混淆的概念。

### IPv4 vs IPv6

**IPv4**（Internet Protocol version 4）是目前使用最广泛的 IP 协议版本，地址由 4 组 0~255 的数字组成，用点分隔，如 `192.168.1.1`。IPv4 地址空间约为 42 亿个，随着互联网爆炸式增长，地址已接近耗尽。

**IPv6**（Internet Protocol version 6）是下一代 IP 协议，地址由 8 组十六进制数字组成，用冒号分隔，如 `2001:0db8:85a3:0000:0000:8a2e:0370:7334`。IPv6 的地址空间几乎无限（约 3.4 × 10^38 个地址），可以为地球上每一粒沙子分配一个 IP 地址。

目前大多数网络同时支持 IPv4 和 IPv6（双栈），你的设备可能同时拥有两种格式的 IP 地址。

### 公网 IP vs 内网 IP

**公网 IP**（Public IP）是互联网上可以被其他设备直接访问的 IP 地址，由你的网络运营商（ISP）分配。一个家庭或办公室通常共享一个公网 IP。

**内网 IP**（Private IP）是局域网内部使用的 IP 地址，仅在局域网范围内有效，不能被外网直接访问。常见的内网地址段：
- `10.0.0.0` ~ `10.255.255.255`
- `172.16.0.0` ~ `172.31.255.255`
- `192.168.0.0` ~ `192.168.255.255`

在 Windows 上运行 `ipconfig`，在 Mac/Linux 上运行 `ifconfig` 或 `ip addr`，看到的 IP 地址通常是内网 IP。而通过 IP 查询工具看到的，才是你的真实公网 IP。

## 本机 IP 一键查询

访问 [tools.cooconsbit.com/tools/ip-info](https://tools.cooconsbit.com/tools/ip-info)，页面加载完成后会**自动显示你当前的公网 IP 地址**，无需任何操作。

显示的信息通常包括：

- **IP 地址**：你的当前公网 IPv4/IPv6 地址
- **地理位置**：国家、省份/州、城市（精度通常到城市级别）
- **ISP（运营商）**：中国电信、中国联通、中国移动，或海外运营商
- **ASN**：自治系统编号，用于网络路由和故障排查
- **时区**：基于地理位置推断的时区信息

这些信息由 IP 地理数据库（GeoIP）提供，精度通常可以准确定位到城市，但不能精确到街道或具体地址。

## IP 归属地查询

除了查询本机 IP，你还可以输入任意 IP 地址查询其归属信息。

...

---

**[👉 继续阅读全文：IP 地址查询工具完全教程：查本机 IP、定位和网络诊断](https://tools.cooconsbit.com/zh/articles/ip-lookup-tool-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
