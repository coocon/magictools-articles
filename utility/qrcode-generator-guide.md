# 二维码生成器完全教程：创建、定制和使用 QR 码

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/qrcode-generator-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/qrcode-generator-guide?utm_source=github&utm_medium=referral)**

## 什么是二维码？

**QR Code**（Quick Response Code，快速响应码）由日本电装波动公司（Denso Wave）的原昌宏团队于 **1994 年** 发明，最初用于汽车零部件的快速追踪管理。QR 这个名字来源于其设计目标——能够被快速读取。

与传统一维条形码只能在水平方向编码信息不同，二维码在**水平和垂直两个方向**同时编码，信息存储密度大幅提升：

| 类型 | 数字最大容量 | 字母最大容量 |
|------|------------|------------|
| 一维条形码（EAN-13）| 13 位数字 | — |
| QR Code（Version 40）| 7,089 个数字 | 4,296 个字母 |

二维码右上角的三个正方形（**定位符**）是其标志性特征，让扫描器能在任意角度快速识别码的位置和方向。

---

## 二维码能存储哪些内容？

二维码只是一个信息载体，理论上可以存储任何文本。但不同内容类型有对应的标准格式，扫描后手机应用会智能识别并执行对应操作：

### 1. 纯文本 / URL 网址

最常见的用途。扫码后直接跳转到网页，无需手动输入网址。

```
https://tools.cooconsbit.com/tools/qrcode
```

### 2. 联系人信息（vCard 格式）

按 vCard 标准格式编码，扫码后手机自动弹出"添加联系人"界面：

```
BEGIN:VCARD
VERSION:3.0
FN:张三
ORG:某科技有限公司
TEL:+86-138-0000-0000
EMAIL:zhangsan@example.com
URL:https://example.com
END:VCARD
```

### 3. WiFi 连接信息

访客扫码自动连接 WiFi，无需说出密码：

```
WIFI:T:WPA;S:MyNetworkName;P:MyPassword;;
```

参数说明：`T` 为加密类型（WPA/WEP/无）、`S` 为 SSID（网络名称）、`P` 为密码。

### 4. 电子邮件 / 电话号码

```
// 发送邮件（预填收件人和主题）
mailto:contact@example.com?subject=询盘&body=你好

// 拨打电话
tel:+86-138-0000-0000

// 发送短信
smsto:+86-138-0000-0000:你好
```

### 5. 地理位置坐标

```
geo:39.9042,116.4074?q=北京天安门广场
```

...

---

**[👉 继续阅读全文：二维码生成器完全教程：创建、定制和使用 QR 码](https://tools.cooconsbit.com/zh/articles/qrcode-generator-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
