# Chloros+ 登录

## Chloros 和 Chloros（浏览器版）登录

用户 <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> 侧边栏菜单可让您登录您的 Chloros+ 账户并解锁更多功能。

登录后将显示您的账户详情：

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## CLI 登录

使用您的 Chloros+ 凭据登录以启用 CLI 处理功能。在 Linux（无图形界面）上，这是激活许可证的唯一方式。

**语法：**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDK 用户**：Python SDK 还提供了一种程序化 `logout()` 方法来清除缓存的凭据。 详情请参阅 [Python SDK 文档](api-python-sdk.md#logout)。
{% endhint %}

**示例：**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**特殊字符**：若密码包含 `$`、`!` 或空格等字符，请使用单引号将其括起来。
{% endhint %}

**输出：**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### 凭据存储

缓存的凭据存储在特定于平台的路径中：

| 平台 | 凭据缓存路径 |
| --- | --- |
| **Windows** | `%APPDATA%\Chloros\cache\` |
| **Linux** | `~/.cache/chloros/` |

### 套餐到期

GUI 中的套餐到期时间显示了您的许可证何时失效。对于按月续订的订阅，到期时间为当月月底。对于按年订阅，到期时间为订阅开始后一年。许可证验证需要每月连接互联网进行确认，并设有 30 天的宽限期。

### 设备限制

每种 Chloros+ 套餐提供的注册设备数量各不相同。使用 Chloros+ 账户登录的每台设备都将计入您的注册设备数量。您可以在 MAPIR 云账户页面上重命名或移除设备。

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+ 套餐</th><th align="center">铜级</th><th align="center">青铜</th><th align="center">银</th><th align="center">金</th></tr></thead><tbody><tr><td align="right">支持的设备</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
