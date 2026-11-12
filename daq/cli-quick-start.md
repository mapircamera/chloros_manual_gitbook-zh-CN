# CLI 快速入门（pool-*）

随附的 `chloros-cli` 通过 **`daq pool-*`** 命令集驱动 DAQ 传感器——这些命令集是轻量级 HTTP 客户端，通过 Chloros 后端的持久化传感器池来操作传感器。 由于后端负责传输管理，因此图形界面、CLI 和 SDK 脚本均可共享一个活动句柄，而无需争夺端口。 客户所需的一切功能均可通过 `pool-*` 实现：连接、流式传输、记录校准后的 `.daq` 文件以及切换 cap 配置文件。

`pool-*` 是已发布版本中支持的 DAQ 界面。`chloros-cli daq --help` 列出了 `pool-*` 的子命令，而 `daq discover` / `daq list` 执行与 `pool-discover` 相同的扫描。 如果某个直接硬件 DAQ 子命令不可用，程序会以明确的错误信息退出，指出缺失的软件包并引导您使用 `pool-*` —— 绝不会出现静默失败的情况。 （`pip install chloros-sdk` 同样不提供直接硬件命令。）

***

## 先决条件

* **Chloros后端必须正在运行** —— `pool-*`命令是HTTP客户端，而非硬件驱动程序。在Windows系统上，请启动Chloros桌面应用（该应用会启动后端）。 在无显示器的 Linux/Jetson 系统上，请以您的用户身份启用该服务：`sudo systemctl enable --now chloros-backend@$USER.service`（参见 [Linux 安装指南](../linux/linux-installation.md#always-on-ptp-for-headless-hosts)）。
* **Chloros+（付费套餐）登录**：请先运行 `chloros-cli login`。 权限验证在服务器端进行——若未登录，使用 `401 AUTH_REQUIRED` 时命令将失败；在免费（Iron）层级中，使用 `403 PLAN_UPGRADE_REQUIRED` 时命令将失败。
* 这些命令默认针对 `http://127.0.0.1:5000`；如果您的后端在其他地方运行，`daq pool-*` 系列会遵循 `CHLOROS_BACKEND_URL` 环境变量。

***

## 五分钟会话

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)
#    (a DAQ-M's BLE MAC is not printed anywhere: find it with `chloros-cli daq pool-discover --only ble`)

# 2. List the pool — this shows the sensor_id used by every command below
chloros-cli daq pool-list

# 3. Read the most recent calibrated spectrum frame (add --json for scripting)
chloros-cli daq pool-latest --sensor-id daq-e-def330 --json

# 4. Record a calibrated .daq file for 60 seconds
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 60 \
  --device-name "field-A"

# 5. Release the sensor when done
chloros-cli daq pool-disconnect --sensor-id daq-e-def330
```

***

## `pool-connect` — 在池中打开一个传感器

| 变体 | 含义 |
| --- | --- |
| `daq pool-connect` | 智能检测：查找该机器上的任意 DAQ。 |
| `daq pool-connect --port PORT` | 位于特定串行端口的 DAQ-U（例如 `COM3`、`/dev/ttyUSB0`）。 |
| `daq pool-connect --ble` | 通过 BLE 连接的 DAQ-M，MAC 地址自动扫描。 |
| `daq pool-connect --mac MAC` | 通过已知 BLE MAC 地址连接 DAQ-M（即 `--ble`）。 |
| `daq pool-connect --eth-host HOST` | 通过已知主机名或 IP 地址连接 DAQ-E —— **可靠路径**。 |
| `daq pool-connect --eth` | 带自动发现功能的 DAQ-E（mDNS，支持 ARP 备用方案）。请参阅下方的注意事项。 |

调优标志，均为可选：

| 标志 | 含义 |
| --- | --- |
| `--integration-time MS` / `-t MS` | 手动积分时间（单位：毫秒）。 |
| `--frame-avg N` / `-f N` | 每个报告频谱的帧平均数。 |
| `--no-ae` | 禁用自动曝光（默认开启 AE）。 |
| `--no-stream` | 连接但不启动数据流（稍后可通过 `pool-stream --start` 恢复）。 |
| `--cap-id CAP` | 峰值校正配置文件；后端默认值为 `sunshine_cosine`。 参见 [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap)。 |

若连接的 DAQ-E 已通过其另一个地址（即其 `daq-e-<id>.local` 主机名或 IP 地址）加入池中，则会返回现有的池条目，而非再次打开该设备。

{% hint style="warning" %}
**`--eth` 自动发现注意事项。** 在多网卡主机（拥有多个活动网络接口）上，即使传感器状态正常，启动后*首次*执行的 `pool-connect --eth` 操作也可能返回空结果——这是因为在 ARP 缓存处于冷状态时，发现扫描可能会遗漏传感器的接口。 如果 `--eth` 未发现任何内容，请重试，或者使用 `--eth-host <ip-or-hostname>` 完全跳过发现过程——这是在多网卡机器上的可靠路径。 DAQ-E 的主机名为 `daq-e-<id>.local`（例如 `daq-e-def330.local`）；其纯 IP 地址同样有效。
{% endhint %}

## `pool-discover` — 不建立连接的扫描

```bash
chloros-cli daq pool-discover                  # USB, BLE and Ethernet
chloros-cli daq pool-discover --only ble       # BLE only: this is how you find a DAQ-M's MAC
chloros-cli daq pool-discover --json           # machine-readable
```

扫描可连接的传感器，并显示连接地址，但不会实际建立连接。`--only` 采用 `usb,ble,eth` 中以逗号分隔的子集； `--timeout SEC` 用于设置每种传输协议的扫描时间（默认 5 秒，限制在 1–20 秒之间）。 池中已打开的传感器不会列出（已连接的 DAQ-M 会停止广播）——对于这些传感器，请使用 `pool-list`。 在正式发布版本中，`chloros-cli daq discover` 和 `daq list` 会执行相同的扫描操作。

## `pool-list` — 查看已连接设备

显示后端池中的所有传感器，包括其他所有命令所需的 `sensor_id`：

| 型号 | `sensor_id` 格式 | 示例 |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5 个八位字节，以连字符分隔 | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

PTP 已关闭的 DAQ-E（该设置保存在设备上）在 `pool-list` 中通过 `!! PTP is switched OFF on this unit (saved in its flash): its timestamps are not PTP-synced.` 进行标记，并附有将其重新启用的命令——参见 [`pool-enable-ptp`](#pool-enable-ptp-switch-a-daq-es-ptp-back-on)。

## `pool-latest` — 读取频谱帧

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

返回最新的一帧，或最近的 `--recent N` 帧；`--json` 生成适用于脚本编写的机器可读输出。 帧数据为基于 135 个点、340–1010 nm 网格的辐射计量校准光谱辐照度（W/m²/nm），并已应用传感器的盖板轮廓。 若需获得定量辐照度数值，请对至少 15 秒的帧数据进行平均处理——这是仪器特性，并非缺陷。

## `pool-stream` —— 暂停或恢复数据流

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` — 记录 `.daq` 文件

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| 标志 | 默认值 | 含义 |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | 录制时长（以秒为单位）； `0` 表示运行直至您发出 `--stop` 命令。 |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | 输出目录。 当后端位于本机时，相对路径将以执行命令的文件夹为基准进行解析；当后端位于远程时，路径将在**运行后端的机器**上进行解析。 |
| `--device-name NAME` | — | 与录制数据一同存储的标签。 |
| `--stop` | — | 停止正在进行的录制。 |
| `--confirm-after SEC` | `20` | 启动后，等待最长此时间以写入第一行数据；若未收到数据，则发出响亮的警告（`!! WARNING: no rows written after …s. This recording is producing an EMPTY file.`）。 `0` 禁用此检查。 |
| `--heartbeat SEC` | `30` | 启用 `--duration` 后，每 SEC 秒打印一行计数，若计数停止增长则发出警告。 `0` 会禁用此功能。 |

{% hint style="info" %}
记录操作在后端进行，因此 `.daq` 文件会保存在 **后端机器** 的文件系统中——默认情况下位于 `~/Documents/DAQ Live View/` 处，而不一定是在您运行 CLI 的位置。 文件名包含传感器 ID 和时间戳。
{% endhint %}

## `pool-set-cap` — 声明已安装的盖帽

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

该盖子 ID 用于选择应用于每个光谱的出厂测量校正曲线，且**必须与传感器上实际安装的盖子相匹配**——无论是传感器还是软件都无法自行检测盖子，且该选择信息会被记录在每个 `.daq` 文件中。 所有情况下的默认值均为 `sunshine_cosine`（每台 DAQ 出厂时均安装有 Sunshine 余弦校正盖，设计衰减系数约为 12×——若未声明更换校正盖，光谱校正误差将大致与此系数相当）。

| `--cap-id` | 适用型号 |
| --- | --- |
| `sunshine_cosine`（默认） | DAQ-U、DAQ-M、DAQ-E |
| `fov_15`、`fov_45`、`fov_90` | DAQ-U、DAQ-E |
| `fov_30`, `fov_60` | 仅限 DAQ-U |
| `none` | 仅限 DAQ-E — 参见注释 |

连接时，若盖帽 ID 超出传感器设定范围，系统将返回明确错误并拒绝连接。 `none`（DAQ-E）表示盖子已被物理移除——它仍会为 DAQ-E 的凹陷式玻璃扩散器应用出厂几何轮廓，因此并非无操作；而裸露的 DAQ-E 属于台架配置，不属于受支持的现场配置。 （裸机 DAQ-U 属于真正的裸机，完全不需要校正轮廓；DAQ-M 需配合其 Sunshine 盖使用。）

## `pool-disconnect` — 释放传感器

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

`pool-disconnect` 会释放任何已打开的传感器，因此它也会释放因脚本崩溃而留在池中的传感器（前提是该传感器上的记录已先完成）。 若传感器不在池中，则无传感器可释放：该命令将输出 `<sensor-id>: already released (not in the pool)` 并退出 `0`。若后端拒绝释放或未确认释放，则退出 `1`。

## `pool-enable-ptp` — 重新启用 DAQ-E 的 PTP

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

DAQ-E 将其 PTP 开启/关闭设置保存在自身内存中，因此 PTP 已被关闭的设备将保持关闭状态——其时间戳不会与 PTP 同步——直到重新开启为止。 `pool-list` 会标记此类设备，并为其打印此命令。Chloros 绝不会自行重新启用设备，因为关闭可能是故意的。1.3.0 版新增功能。

## DAQ-E 设备设置

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto
chloros-cli daq pool-set-auto-stream --sensor-id daq-e-def330 --on
chloros-cli daq pool-reboot --sensor-id daq-e-def330
chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

`pool-info` 显示传感器的固件版本（以及该Chloros版本是否包含更新版本）、其地址及获取方式、与当前运行设置不同的已保存网络设置、开机流传输和PTP信息。 `pool-set-network` 会重启传感器以应用更改，并使用其原有设置重新连接；`--no-reboot` 仅保存设置。 静态地址必须位于该计算机的网络中，否则Chloros将拒绝该地址；`--force`会覆盖此限制，但仅适用于将被重新分配地址或路由到该地址的计算机。 在传感器记录数据期间，重启请求将被拒绝。`pool-update-firmware` 是图形界面中 **“更新至 \<version\>**” 命令的命令行对应版本，也是在没有图形界面的 Linux 系统上更新 DAQ-E 的方法。1.3.0 版本新增。***

## 命令摘要

| 命令 | 用途 |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | 在后端池中打开一个传感器。 |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | 扫描传感器而不建立连接（查找 DAQ-M 的 BLE MAC 地址）。 |
| `daq pool-list` | 显示后端池中所有传感器及其 `sensor_id`。 |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | 最近的 N 个校准频谱帧。 |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | 恢复/暂停数据流传输。 |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--confirm-after SEC] [--heartbeat SEC] [--stop]` | 开始/停止 `.daq` 录制（后端操作）。 |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | 在运行时切换电容校正配置文件。 |
| `daq pool-disconnect --sensor-id ID [--all]` | 释放一个传感器或所有传感器。 |
| `daq pool-enable-ptp --sensor-id ID` | 为 PTP 已关闭的 DAQ-E 重新启用 PTP。 |
| `daq pool-info --sensor-id ID [--json]` | DAQ-E 的固件、地址、已保存的网络设置、开机流传输和 PTP。 |
| `daq pool-set-network --sensor-id ID (--auto \| --static IP [--mask M] [--gateway G]) [--no-reboot] [--force]` | DHCP 或静态地址；重启并重新连接传感器。 |
| `daq pool-set-auto-stream --sensor-id ID (--on \| --off)` | 上电时开始流式传输。 |
| `daq pool-reboot --sensor-id ID` | 重启 DAQ-E 并重新连接。 |
| `daq pool-update-firmware --sensor-id ID` | 将此版本的 DAQ-E 固件刷写到旧款传感器上。 |

***

## DAQ-E 首次连接故障排除

1. DAQ-E 没有状态指示灯——请通过交换机或注入器端口的 PoE/链路指示灯确认供电情况，并在通电后等待几秒钟，以便设备启动并加入网络。
2. 后端机器必须与传感器位于**同一广播域**内——mDNS 无法穿透路由器。
3. 防火墙必须允许 mDNS（UDP 5353）、DAQ-E 数据（UDP 5002）和 PTP（UDP 319/320）通过。 Windows安装程序会为此添加一条规则，而Linux的`.deb`则会在安装时已启用的`ufw` / `firewalld`中打开这些端口； 第三方防火墙或后续启用的防火墙需要允许这些端口。
4. 仍未收到来自 `--eth` 的响应？ 请使用 `--eth-host` 并配合设备的主机名（`daq-e-<id>.local`）或 IP 地址，这样可以跳过发现过程。

***
{% hint style="info" %}
**AI 助手提示。** 本手册的每一页均以原始 Markdown 格式提供——在页面的小写 URL 片段后附加 `.md`（本页：`https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`）； 机器可读的索引为 `https://mapir.gitbook.io/chloros/llms.txt`。 如需查看 `chloros-cli daq` 及其他所有命令家族的完整参数级文档，请查阅 [CLI 参考](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`)； 在 [SDK参考](../reference/sdk-reference.md) 中，Python路径为 `chloros_sdk.connect_daq_sensor()`。
{% endhint %}
