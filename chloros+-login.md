# Chloros+ 登录

## Chloros 和 Chloros（浏览器）登录

用户 <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> 侧边栏菜单可让您登录 Chloros+ 账户并解锁更多功能。

登录后将显示您的账户详情：

<figure><img src=".gitbook/assets/user_account.JPG" alt="" data-size="line"><figcaption></figcaption></figure>## CLI登录

使用您的Chloros+凭证登录以启用CLI处理功能。

**语法：**

```bash
chloros-cli login <email> <password>
```

{% hint style=&quot;info&quot; %}
**SDK 用户**：Python 还提供程序化 `logout()` 方法清除缓存凭证。 详情请参阅[Python SDK文档](api-python-sdk.md#logout)。
{% endhint %}

**示例：**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**特殊字符**：若密码包含`$`、`!`等特殊字符或空格，请使用单引号包裹。
{% endhint %}

**输出：**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### 方案到期

图形界面显示的方案到期时间即为许可证失效日期。月度订阅方案到期于当月末，年度订阅方案到期于订阅启动后一年。许可证验证需每月连接互联网进行核验，并设有30天宽限期。

### 设备数量限制

每种 Chloros+ 方案支持注册的设备数量各不相同。使用 Chloros+ 账户登录的每台设备均计入注册设备数量。您可在 MAPIR 云账户页面重命名或移除设备。

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+ 套餐</th><th align="center">铜级</th><th align="center">青铜</th><th align="center">银级</th><th align="center">黄金</th></tr></thead><tbody><tr><td align="right">支持设备</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
