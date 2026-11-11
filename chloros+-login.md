# Chloros+ 登录

## 图形界面登录

用户侧边栏菜单 <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> 允许您登录 Chloros+ 账户并解锁更多功能。

**每台机器只需登录一次。** 图形界面（GUI）、命令行界面（CLI）和 Python SDK 共享同一缓存会话——通过桌面图形界面登录后，该设备上的 CLI 和 SDK 也会自动激活（反之亦然，通过 `chloros-cli login` 实现）。

登录后将显示您的账户详情：

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.3.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->

## 套餐等级

| 套餐 | `plan_id` | 类型 |
| --- | --- | --- |
| 铁 | `0` | 免费 |
| 铜级 | `1` | 付费（Chloros

+） |
| 青铜级 | `2` | 付费（Chloros

+） |
| 银级 | `3` | 付费（Chloros

+） |
| 黄金 | `4` | 付费（Chloros

+） |

有关各付费层级的具体内容，请参阅 [套餐与定价](https://cloud.mapir.camera/pricing)。

### 访问 CLI

/SDK

需订阅付费层级

访问 CLI

和 Python

SDK

需要 **任何付费 Chloros

+ 套餐（铜级或更高）**。此要求在**服务器端** 强制执行——每个 CLI

/SDK

请求都必须同时包含有效会话和付费套餐：

| HTTP

状态 | `error_code` | 含义 | 解决方法 |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | 未在此设备上登录 | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | 已登录，但套餐等级过低（免费 Iron 等级） | 升级至任意付费的 Chloros

+ 套餐 |

`chloros-cli status` 在免费套餐下仍可访问，因此您可以随时查看当前套餐及访问被拒绝的原因。

### 各套餐的连接硬件限制

每个套餐都限制了可同时实时连接的 LATTICE 摄像头和 DAQ 光传感器数量：

| 套餐 | LATTICE 摄像头 | DAQ 光传感器 |
| --- | --- | --- |
| Iron（免费 / 未登录） | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## 命令行界面（CLI

）登录

请使用您的 Chloros

+ 凭据登录以启用 CLI

处理。在 Linux

（无图形界面）环境下，这是激活许可证的唯一方式。

**语法：**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDK

用户**：Python

SDK

还提供了一个用于清除缓存凭据的编程方法 `logout()`。详情请参阅 [SDK

参考文档](reference/sdk-reference.md)。
{% endhint %}

**示例：**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**特殊字符**：若密码中包含 `$`、`!` 或空格等字符，请使用单引号将其括起来。
{% endhint %}

**输出：**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.3.0"; capture a successful login with the current output format. -->

### 凭据存储

在**所有平台**上，缓存的凭据和配置都存储在用户主目录下的 `.chloros` 文件夹中：

| 平台 | 凭据缓存路径 |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

在现有版本上安装新版本将保留您的登录信息，这在 Windows

（安装程序）和 Linux

（`.deb`，包括 `chloros-cli update`）中均适用。 卸载 Chloros

—— 或删除 Linux

软件包 —— 会清除缓存的登录信息，因此重新安装后您需要再次登录。

### 套餐到期与离线宽限期

图形用户界面（GUI）中显示的套餐到期时间表示您的许可证何时失效。 对于按月续订的订阅，到期日为当月月底；对于按年订阅，到期日为订阅开始后一年。

Chloros

会在线验证您的许可证，但在宽限期内支持离线使用：

* 成功的服务器验证结果会缓存 **5 分钟**，因此正常使用时几乎不会发起许可证验证请求。
* 经过签名且与设备绑定的许可证缓存可覆盖更长的离线时段：**月度套餐为 30 天**，**年度套餐则持续至订阅到期日（最长 365 天）**。
* 当宽限期届满时，套餐将自动降级为免费的 Iron 层级，直到该设备能够成功连接到许可证服务器一次；在下一次验证成功后，访问权限将恢复。

### 设备限制

每种 Chloros

+ 套餐支持的注册设备数量各不相同。您使用 Chloros

+ 账户登录的每台设备都会计入您的注册设备数量。您可以在 MAPIR

云账户页面上重命名或移除设备。

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+ 套餐</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">银</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">支持的设备</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>

您账户的具体设备配额显示在您的MAPIR云账户页面上。从设备上注销可可靠地释放该设备的配额，且已注册的设备即使在账户达到设备上限时，也始终可以重新登录。
