# 支持的语言

Chloros 提供 **全球 38 种语言** 的完整界面支持，让世界各地的用户都能轻松使用。 您可以在所有界面（桌面版、浏览器版、CLI、Python 及 SDK）之间即时切换语言。

Chloros 支持以下语言：

| # | 语言 | 本地名称 | CLI 代码 |
|---|----------|-------------|----------|
| 1 | 🇺🇸 英语 | English | `en` |
| 2 | 🇪🇸 西班牙语 | Español | `es` |
| 3 | 🇵🇹 葡萄牙语 | Português | `pt` |
| 4 | 🇫🇷 法语 | Français | `fr` |
| 5 | 🇩🇪 德语 | Deutsch | `de` |
| 6 | 🇮🇹 意大利语 | Italiano | `it` |
| 7 | 🇯🇵 日语 | 日本語 | `ja` |
| 8 | 🇰🇷 韩语 | 한국어 | `ko` |
| 9 | 🇨🇳 简体中文 | 简体中文 | `zh` |
| 10 | 🇹🇼 繁体中文 | 繁體中文 | `zh-TW` |
| 11 | 🇷🇺 俄语 | Русский | `ru` |
| 12 | 🇳🇱 荷兰语 | Nederlands | `nl` |
| 13 | 🇸🇦 阿拉伯语 | العربية | `ar` |
| 14 | 🇵🇱 波兰语 | Polski | `pl` |
| 15 | 🇹🇷 土耳其语 | Türkçe | `tr` |
| 16 | 🇮🇳 印地语 | हिंदी | `hi` |
| 17 | 🇮🇩 印尼语 | Bahasa Indonesia | `id` |
| 18 | 🇻🇳 越南语 | Tiếng Việt | `vi` |
| 19 | 🇹🇭 泰语 | ไทย | `th` |
| 20 | 🇸🇪 瑞典语 | Svenska | `sv` |
| 21 | 🇩🇰 丹麦语 | Dansk | `da` |
| 22 | 🇳🇴 挪威语 | Norsk | `no` |
| 23 | 🇫🇮 芬兰语 | Suomi | `fi` |
| 24 | 🇬🇷 希腊语 | Ελληνικά | `el` |
| 25 | 🇨🇿 捷克语 | Čeština | `cs` |
| 26 | 🇭🇺 匈牙利语 | Magyar | `hu` |
| 27 | 🇷🇴 罗马尼亚语 | Română | `ro` |
| 28 | 🇺🇦 乌克兰语 | Українська | `uk` |
| 29 | 🇧🇷 巴西葡萄牙语 | Português Brasileiro | `pt-BR` |
| 30 | 🇭🇰 粤语 | 粵語 | `zh-HK` |
| 31 | 🇲🇾 马来语 | Bahasa Melayu | `ms` |
| 32 | 🇸🇰 斯洛伐克语 | Slovenčina | `sk` |
| 33 | 🇧🇬 保加利亚语 | Български | `bg` |
| 34 | 🇭🇷 克罗地亚语 | Hrvatski | `hr` |
| 35 | 🇱🇹 立陶宛语 | Lietuvių | `lt` |
| 36 | 🇱🇻 拉脱维亚语 | Latviešu | `lv` |
| 37 | 🇪🇪 爱沙尼亚语 | Eesti | `et` |
| 38 | 🇸🇮 斯洛文尼亚语 | Slovenščina | `sl` |

## 如何更改语言

### 在 Chloros 桌面版/浏览器版

1. 打开应用程序设置
2. 进入语言选择菜单
3. 从列表中选择您偏好的语言
4. 界面将立即更新

### 在 Chloros CLI

使用 `language` 命令查看或更改 CLI 界面语言：

```bash
# View current language
chloros-cli language

# Change to Spanish
chloros-cli language es

# Change to Chinese (Simplified)
chloros-cli language zh

# Change to Brazilian Portuguese
chloros-cli language pt-BR

# List all available languages
chloros-cli language --list
```

更多详情，请参阅 [CLI 文档](CLI.md)。

### 在 Chloros Python SDK

初始化 SDK 时设置语言参数，即可将消息和输出显示为您首选的语言。

## 支持范围

以下所有 38 种语言均得到全面支持：

* **Chloros 桌面版** - 完整的图形用户界面（GUI）翻译
* **Chloros 浏览器版** - 支持所有语言的网页界面
* **Chloros CLI** - 命令行界面及输出消息
* **Chloros Python SDK** - API 消息及文档

语言支持确保全球用户能够使用母语高效工作，不受任何障碍影响。
