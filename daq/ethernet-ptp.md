# DAQ-E 网络连接与时间同步

> 传感器物理网络的设置——包括布线、PoE、IP 地址分配以及设备自身的网络设置——详见 **[DAQ 用户手册](https://mapir.gitbook.io/daq/daq-e/network-setup)**。 本页面主要介绍 Chloros 相关内容：连接、时间同步，以及发现操作未返回结果时的处理方法。

DAQ-E 是 DAQ 系列中的以太网设备：通过 PoE 供电，通过 mDNS（服务 `_daq-e._tcp`）进行发现，并可通过由其传感器 ID 派生而来的主机名进行寻址 ——例如 `daq-e-<6 hex>.local`、`daq-e-def330.local`。本页介绍其网络数据传输方式以及参与 PTP 时间同步的机制。

## 传输模式

| 模式 | 端点 | 消费者 | 备注 |
| --- | --- | --- | --- |
| **组播**（默认） | UDP `239.10.10.10:5002` | 同一局域网内的任意数量设备均可接收同一数据流 | 每个数据报均经过 CRC-16/CCITT 校验 |
| **原始模式** | TCP 端口 `5000` | 仅限一个客户端（排他） | 与 DAQ-U 完全字节兼容 |

Chloros 默认使用多播，这使得 GUI、CLI 和 SDK 能够同时监视同一传感器。

## 网络要求

* **同一广播域。** 运行 Chloros 的机器必须与传感器位于同一 L2 网络段——mDNS 发现无法穿过路由器。
* **Windows 防火墙提示：请允许。** 当 Chloros 首次绑定多播套接字时，Windows Defender 会提示一次。 允许该请求可覆盖 DAQ-E 数据（UDP 5002）、mDNS（UDP 5353）和 PTP（UDP 319/320）。在 Linux 上，此过程不会显示提示。
* **PoE供电，无状态指示灯。** DAQ-E本身没有指示灯——请通过交换机或注入器端口的链路/PoE指示灯确认供电情况，并在通电后等待几秒钟，以便设备启动并加入网络。

## 连接

**图形界面：** “光传感器”选项卡 → “连接传感器” → 设备类型“DAQ-E (以太网)”。 仅当“连接”对话框显示在屏幕上时才会进行设备发现（包括 mDNS 浏览以及对 Windows 的 ARP 扫描），每 15 秒重复一次；点击“刷新”按钮将立即重新扫描。发现的传感器会显示在下拉列表中；系统会自动选中第一个检测到的传感器。

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI**（后端正在运行）：

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

### 多网卡主机及开机后的首次连接

在拥有多个活动网络接口的主机上，即使传感器状态正常，启动后的**首次** `pool-connect --eth` 扫描结果也可能为空——这是因为在 ARP 缓存尚未预热时，发现扫描可能会遗漏传感器所在的接口。 可靠的解决方法是跳过发现过程，并显式指定地址：

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` 接受 mDNS 主机名或 IP 地址，始终能定位到正确的传感器，是脚本和无头安装的推荐形式。 在图形界面中，请使用“连接”对话框中的“刷新”按钮，并等待一次重新扫描周期。

## 设备设置与固件

传感器本身保存着网络设置——静态 IP 与 DHCP + 链路本地地址、设备名称、启动时自动流传输、OTA 密码。 这些设备端的设置在随附的 CLI 中未作为命令提供；它们可通过显示这些设置的 Chloros 图形界面进行管理，或借助 MAPIR 支持进行管理。

**固件更新功能已集成到图形用户界面中。**当连接的 DAQ-E 运行的固件版本早于您 Chloros 构建版本中捆绑的镜像时，其传感器行会显示一个琥珀色的**有更新可用** 图标，且齿轮设置弹出窗口中会提供一个“更新至<version>

”按钮。 更新通过网络传输约需 30 秒；传感器将自动重启并重新连接，若传输过程中断，当前固件将保持不变。

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## PTP 时间同步

DAQ-E 固件 v1.2.0 及以上版本可作为普通（仅从属）时钟参与 IEEE 1588 PTPv2 协议。 **Chloros主机的后端即为PTP主时钟**——局域网内的每台DAQ-E和每台LATTICE相机均在域0中对其进行从属同步，确保所有设备的时间戳误差控制在约1毫秒以内。 正是这个共享时钟使得 DAQ 读数的时间戳能够与相机曝光时间匹配（参见 [录制与 .daq 格式](recording.md)）。

通过 CLI 检查同步状态：

| 命令 | 显示内容 |
| --- | --- |
| `chloros-cli time-sync status` | 主机主时钟状态、BMCA 优先级、时钟标识 |
| `chloros-cli time-sync peers` | 检测到的所有从设备（DAQ-E 传感器 + LATTICE 相机） |
| `chloros-cli time-sync cameras` | 每台摄像头的 PTP 健康状态（`PtpStatus`、`PtpOffsetFromMaster`、`PtpMeanPathDelay`） |
| `chloros-cli time-sync restart` | 重启主控进程 |

在图形用户界面（GUI）中，DAQ-E 设置模态窗口会显示一个实时更新的 **PTP 同步** 行，其中包含传感器当前的 PTP 状态。

关于严格对齐消费者的详细说明：

* 每个流式数据报都包含一个标志字段；**时间戳与 PTP 同步的帧中，第 2 位会被设置**。需要严格相机/DAQ 对齐的管道应根据该位进行门控。
* 在进行同步捕获之前，请确认传感器出现在 `chloros-cli time-sync peers` 中。 （MAPIR 的内部直接硬件工具也可通过 `--wait-ptp` 标志在 PTP 锁定时触发录制，该标志最多等待 15 秒直至传感器进入 SLAVE 状态； 该工具功能不包含在已发布的 CLI 中。)
* 当 PTP 处于主动从属状态时，传感器会拒绝手动时钟推送（“PTP 正在提供时钟”）。这是设计使然——请信任 PTP。

## Linux 注意事项

* **PTP 在安装时需要 `libcap2-bin`。** `.deb` 的 postinst 脚本会向 `cap_net_bind_service=+ep` 授予 `/usr/lib/chloros/chloros-backend` 的权限，以便其无需 root 权限即可绑定 PTP 端口 319/320。 如果缺少 `libcap2-bin`，该步骤将被跳过，PTP 将无法启动。解决方法：

  ```bash
  sudo apt install libcap2-bin
  sudo apt reinstall chloros
  ```

* **无显示器的 Jetson / Raspberry Pi：**首次安装时，系统会生成 systemd 单元 `chloros-backend.service`，但该单元未启用。若要在不使用图形界面的情况下实现 PTP（及 DAQ 功能）的持续运行：

  ```bash
  sudo systemctl enable --now chloros-backend.service
  ```

  若未启用该单元，PTP 仅在 Chloros GUI 打开时运行。

## 故障排除：“未找到 DAQ-E 设备”

| 检查项 | 详细信息 |
| --- | --- |
| 电源 | 传感器上无 LED 指示灯 — 检查交换机/注入器端口的 PoE 和链路指示灯；通电后等待几秒钟 |
| 广播域 | 主机和传感器位于同一 L2 段；mDNS 无法路由 |
| Windows 防火墙 | 首次运行时接受 Defender 提示（UDP 5002、5353、319/320） |
| 多网卡主机 | 开机后的首次发现可能无法检测到传感器 — 请通过 `--eth-host <ip-or-hostname>` 建立连接 |
| GUI 重新扫描 | 仅在连接对话框打开时才会运行发现功能；请使用“刷新”按钮 |</version>
