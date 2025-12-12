# 支持语言

Chloros 提供全球**38种语言**的完整界面支持，让世界各地的用户都能轻松使用。 您可在所有界面即时切换语言：桌面端、浏览器端、CLI及Python SDK。

Chloros支持以下语言：

| # | 语言 | 原生名称 | CLI 代码 |
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

### 在 Chloros 桌面/浏览器中

1. 打开应用程序设置
2. 导航至语言选择菜单
3. 从列表中选择首选语言
4. 界面将即时更新

### 在 Chloros CLI 中

使用`language`命令查看或更改CLI界面语言：

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

更多详情请参阅[CLI文档](CLI.md)。

### 在 Chloros Python SDK 中

初始化 SDK 时设置语言参数，即可按首选语言获取消息和输出内容。

## 覆盖范围

全部 38 种语言在以下场景均获得完整支持：

* **Chloros 桌面版** - 完整图形界面翻译
* **Chloros 浏览器版** - 全语言网页界面
* **Chloros CLI** - 命令行界面及输出消息
* **Chloros Python SDK** - API 消息与文档

语言支持确保全球用户能以母语高效工作，无障碍沟通。
