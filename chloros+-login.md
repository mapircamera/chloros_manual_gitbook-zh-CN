# Chloros+ 登录

## 图形界面登录

用户 <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> 的侧边栏菜单可让您登录您的 Chloros+ 账户并解锁更多功能。

**每台机器只需登录一次。** 图形界面、CLI、Python 和 SDK 共享同一缓存会话 ——通过桌面图形用户界面登录也会激活该设备上的 CLI 和 SDK（反之亦然，可通过 `chloros-cli login` 实现）。

登录后将显示您的账户详细信息：

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.2.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->
## 套餐等级

| 套餐 | `plan_id` | 类型 |
| --- | --- | --- |
| 铁级 | `0` | 免费 |
| 铜级 | `1` | 付费（需拥有 Chloros 或更高） |
| 青铜 | `2` | 付费（Chloros+） |
| 银 | `3` | 付费 (Chloros+) |
| 金牌 | `4` | 付费（Chloros+） |

有关各付费级别的具体内容，请参阅 [套餐与定价](https://cloud.mapir.camera/pricing)。

### 访问 CLI / SDK 需要付费套餐

访问 CLI、Python 和 SDK 需拥有 **任意付费 Chloros+ 套餐（铜级或更高）**。 此规则在**服务器端**强制执行——每个 CLI/SDK 请求必须同时包含有效的会话和付费套餐：

| HTTP 状态 | `error_code` | 含义 | 解决方法 |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | 未在此设备上登录 | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | 已登录，但套餐等级过低（免费 Iron 层级） | 升级至任意付费 Chloros+ 套餐 |

在免费套餐下仍可访问 `chloros-cli status`，因此您始终可以查看当前套餐信息以及访问被拒绝的原因。

### 各套餐的连接硬件限制

每个套餐都对同时可实时连接的 LATTICE 摄像头和 DAQ 光传感器数量设有上限：

| 套餐 | LATTICE 摄像头 | DAQ 光传感器 |
| --- | --- | --- |
| Iron（免费 / 未登录） | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## CLI 登录

请使用您的 Chloros+ 凭据登录，以启用 CLI 处理功能。 在 Linux（无图形界面）上，这是激活许可证的唯一方法。

**语法：**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDK 用户**：Python SDK 还提供了一种编程方式 `logout()` 来清除缓存的凭据。 详情请参阅 [SDK 参考文档](reference/sdk-reference.md)。
{% endhint %}

**示例：**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**特殊字符**：若密码中包含如 `$`、`!` 或空格等字符，请用单引号将其括起。
{% endhint %}

**输出：**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.2.0"; capture a successful login with the current output format. -->
### 凭据存储

在**所有平台**上，缓存的凭据和配置都存储在用户主目录下的 `.chloros` 文件夹中：

| 平台 | 凭据缓存路径 |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

### 套餐到期与离线宽限期

图形用户界面（GUI）中显示的套餐到期时间即为您的许可证失效时间。对于按月续订的订阅，到期时间为当月月底；对于按年订阅，到期时间为订阅开始后一年。

Chloros 会在线验证您的许可证，但在宽限期内支持离线使用：

* 成功的服务器验证结果会被缓存 **5 分钟**，因此正常使用时几乎无需进行许可证验证请求。
* 经过签名且与设备绑定的许可证缓存可覆盖更长的离线时段：**月度套餐为 30 天**，**年度套餐则持续至订阅到期日（最长 365 天）**。
* 当宽限期届满时，套餐将自动降级为免费的 Iron 层级，直到该设备能够成功连接到许可证服务器一次；在下一次验证成功后，访问权限将恢复。

### 设备限制

每个 Chloros+ 套餐提供的注册设备数量各不相同。 使用 Chloros+ 账户登录的每台设备都会计入您的已注册设备数量。您可以在 MAPIR 云账户页面上重命名或移除设备。

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+ 套餐</th><th align="center">铜级</th><th align="center">青铜</th><th align="center">银</th><th align="center">黄金</th></tr></thead><tbody><tr><td align="right">支持的设备</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>您账户的具体设备配额显示在您的 MAPIR 云账户页面上。从设备上注销可可靠地释放其配额，且已注册的设备即使在账户达到设备上限时，也始终可以重新登录。
