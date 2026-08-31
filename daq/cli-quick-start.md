# CLI 快速入门（pool-*）

出厂配置的 `chloros-cli` 驱动器通过 **`daq pool-*`** 命令族——这些轻量级的 HTTP 客户端通过 Chloros 后端持久的传感器池来操作传感器。 后端拥有传输通道，因此图形用户界面、CLI 以及 SDK 脚本均共享一个活动句柄，而非争夺端口。 客户所需的一切功能均可通过 `pool-*` 实现：连接、流式传输、记录经过校准的 `.daq` 文件以及切换电极配置文件。

`pool-*` 也是已发布版本中**唯一的**数据采集（DAQ）界面。`chloros-cli daq --help` 列出了 `pool-*` 的子命令， 若在已发布的构建版本中调用直接硬件数据采集子命令，系统会以显式错误退出，明确指出缺失的包并引导您返回 `pool-*` —— 绝不会出现静默失败的情况。 （直接硬件命令仅可在从 MAPIR 源代码检出后运行；`pip install chloros-sdk` 同样不提供这些命令。）

***

## 先决条件

* **Chloros 后端必须正在运行** —— `pool-*` 命令是 HTTP 的客户端，而非硬件驱动程序。 在 Windows 上，启动 Chloros 桌面应用程序（它会启动后端）。 在无显示器的 Linux/Jetson 上，启用服务：`sudo systemctl enable --now chloros-backend.service`。
* **Chloros+（付费层级）登录**：请先运行 `chloros-cli login`。 该限制在服务器端执行——若未登录，使用 `401 AUTH_REQUIRED` 时命令将失败；在免费（Iron）层级中，使用 `403 PLAN_UPGRADE_REQUIRED` 时命令将失败。
* 这些命令默认针对 `http://127.0.0.1:5000`；若您的后端在其他位置运行，`daq pool-*` 系列将遵循 `CHLOROS_BACKEND_URL` 环境变量。

***

## 五分钟会话

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)

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

## `pool-connect` — 在传感器池中打开一个传感器

| 变体 | 含义 |
| --- | --- |
| `daq pool-connect` | 智能检测：查找该机器上的任意 DAQ。 |
| `daq pool-connect --port PORT` | 位于特定串行端口的 DAQ-U（例如 `COM3`、`/dev/ttyUSB0`）。 |
| `daq pool-connect --ble` | 通过 BLE 连接的 DAQ-M，MAC 地址自动扫描。 |
| `daq pool-connect --mac MAC` | 位于已知 BLE MAC 地址的 DAQ-M（隐含 `--ble`）。 |
| `daq pool-connect --eth-host HOST` | 通过已知主机名或 IP 地址连接 DAQ-E —— **可靠路径**。 |
| `daq pool-connect --eth` | 通过自动发现连接 DAQ-E（mDNS，带 ARP 备用方案）。请参阅下方的注意事项。 |

调优标志，均为可选：

| 标志 | 含义 |
| --- | --- |
| `--integration-time MS` / `-t MS` | 手动积分时间（单位：毫秒）。 |
| `--frame-avg N` / `-f N` | 每个报告频谱的帧平均数。 |
| `--no-ae` | 禁用自动曝光（默认开启）。 |
| `--no-stream` | 连接但不启动数据流（稍后可通过 `pool-stream --start` 恢复）。 |
| `--cap-id CAP` | 峰值校正配置文件；后端默认值为 `sunshine_cosine`。 参见 [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap)。 |

{% hint style="warning" %}
**`--eth` 自动发现注意事项。** 在多网卡主机（拥有多个活动网络接口）上，即使传感器状态正常，启动后执行的*首次* `pool-connect --eth` 操作也可能返回空结果——这是因为在 ARP 缓存处于冷状态时，发现扫描可能无法检测到传感器的接口。 如果 `--eth` 未发现任何内容，请重试，或使用 `--eth-host <ip-or-hostname>` 完全跳过发现过程——这是多网卡机器上的可靠路径。 DAQ-E 的主机名为 `daq-e-<id>.local`（例如 `daq-e-def330.local`）；其纯 IP 地址同样有效。
{% endhint %}

## `pool-list` — 查看已连接设备

显示后端池中的所有传感器，包括其他所有命令所需的 `sensor_id`：

| 型号 | `sensor_id` 格式 | 示例 |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5 个八位组带连字符 | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

## `pool-latest` — 读取频谱帧

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

返回最新的一帧，或最近的 `--recent N` 帧；`--json` 生成适用于脚本编写的机器可读输出。 帧数据为在 135 点、340–1010 nm 网格上经过辐射校准的光谱辐照度（W/m²/nm），并已应用传感器的盖板轮廓。 若需获得定量辐照度数值，请对至少 15 秒的帧数据进行平均处理——这是仪器特性，并非缺陷。

## `pool-stream` —— 暂停或恢复流式传输

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
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | 输出目录，**在运行后端的机器上**解析。 |
| `--device-name NAME` | — | 与录制数据一同存储的标签。 |
| `--stop` | — | 停止正在进行的录制。 |

{% hint style="info" %}
录制在后端进行， 因此 `.daq` 文件会保存在 **后端机器** 的文件系统中——默认情况下保存在 `~/Documents/DAQ Live View/` 位置，而不一定是在您运行 CLI 的位置。 文件名包含传感器 ID 和时间戳。
{% endhint %}

## `pool-set-cap` — 声明安装的盖子

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

盖子 ID 用于选择应用于每个光谱的出厂测量校正曲线，且**必须与传感器上实际安装的盖子相匹配**——传感器和软件均无法自行检测盖子，该选择信息会被记录到每个 `.daq` 文件中。 所有情况下的默认值均为 `sunshine_cosine`（每台 DAQ 出厂时均已安装 Sunshine 余弦校正盖，设计衰减系数约为 12×——若未声明更换校正盖，光谱校正误差将大致与此系数相当）。

| `--cap-id` | 适用型号 |
| --- | --- |
| `sunshine_cosine`（默认） | DAQ-U、DAQ-M、DAQ-E |
| `fov_15`、`fov_45`、`fov_90` | DAQ-U、DAQ-E |
| `fov_30`, `fov_60` | 仅限 DAQ-U |
| `none` | 仅限 DAQ-E — 参见注释 |

连接时，若盖帽ID超出传感器设定范围，系统将返回明确错误并拒绝连接。 `none`（DAQ-E）表示盖帽已被物理移除——它仍会为 DAQ-E 的内嵌式玻璃扩散器应用出厂几何轮廓，因此并非无操作；而裸露的 DAQ-E 属于台架配置，不属于受支持的现场配置。 （裸机 DAQ-U 属于真正意义上的裸机，完全不需要校正轮廓；DAQ-M 需配合其 Sunshine 保护盖使用。）

## `pool-disconnect` — 释放传感器

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

***

## 命令摘要

| 命令 | 用途 |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | 打开后端池中的传感器。 |
| `daq pool-list` | 显示后端池中所有传感器及其 `sensor_id` 信息。 |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | 最近的 N 个校准光谱帧。 |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | 恢复/暂停流式传输。 |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--stop]` | 开始/停止 `.daq` 录制（后端侧）。 |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | 运行时切换电容校正配置文件。 |
| `daq pool-disconnect --sensor-id ID [--all]` | 释放一个传感器或所有传感器。 |

***

## DAQ-E 首次连接的故障排除

1. DAQ-E 没有状态指示灯——请通过交换机或注入器端口的 PoE/链路指示灯确认电源是否正常，并请在通电后等待几秒钟，以便设备启动并加入网络。
2. 后端机器必须与传感器位于**同一广播域**内——mDNS 无法穿越路由器。
3. 在 Windows 上，首次运行时请接受 Defender 防火墙的提示（mDNS UDP 5353，DAQ-E 数据 UDP 5002，PTP UDP 319/320）。
4. `--eth` 仍无响应？请使用 `--eth-host` 并配合该设备的主机名（`daq-e-<id>.local`）或 IP 地址——这是更可靠的连接路径，特别是在多网卡主机上。

***{% hint style="info" %}**给 AI 助手的提示。** 本手册的每一页均以原始 Markdown 格式提供——在页面的小写别名 URL 后附加 `.md`（例如本页：`https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`）； 机器可读的索引即为 `https://mapir.gitbook.io/chloros/llms.txt`。 如需获取 `chloros-cli daq` 及其他所有命令家族的完整参数级文档，请查阅 [CLI 参考手册](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`)； 在 [SDK 参考](../reference/sdk-reference.md) 中，Python 的路径为 `chloros_sdk.connect_daq_sensor()`。
{% endhint %}
