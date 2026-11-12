# DAQ-E 网络连接与时间同步

> 传感器物理网络的设置——包括布线、PoE、IP 地址分配以及设备本身的网络设置——详见 **[DAQ 用户手册](https://mapir.gitbook.io/daq/daq-e/network-setup)**。 本页面主要介绍Chloros 方面的内容：连接、时间同步，以及发现操作未返回结果时的处理方法。

DAQ-E 是 DAQ 系列中的以太网设备：通过 PoE 供电，通过 mDNS（服务 `_daq-e._tcp`）进行发现，并可通过由其传感器 ID 派生而来的主机名进行寻址 ——例如 `daq-e-<6 hex>.local`、`daq-e-def330.local`。本页介绍该设备如何在网络上传输数据以及如何参与 PTP 时间同步。

## 传输模式

| 模式 | 端点 | 消费者 | 备注 |
| --- | --- | --- | --- |
| **组播**（默认） | UDP `239.10.10.10:5002` | 同一局域网内的任意数量设备均可接收同一数据流 | 每个数据报均经过 CRC-16/CCITT 校验 |
| **原始模式** | TCP 端口 `5000` | 仅限一个客户端（排他） | 与 DAQ-U 字节级兼容 |

Chloros 默认使用组播，这使得 GUI、CLI 和 SDK 能够同时监视同一传感器。

## 网络要求

* **同一广播域。** 运行 Chloros 的机器必须与传感器位于同一 L2 网络段——mDNS 发现无法穿越路由器。
* **防火墙。** 在 Windows 系统上，安装程序会在每个网络配置文件中为后端添加 **Chloros 后端 (UDP-In)**防火墙规则，该规则涵盖 DAQ-E 数据（UDP 5002）、 mDNS（UDP 5353）和 PTP（UDP 319/320）；此外，还会添加一条仅限于本地子网的**“Chloros 后端（TCP-In）”** 规则，用于 DAQ-E 在固件更新期间与计算机建立回连。 在Linux 系统上，如果安装时该防火墙处于活动状态，`.deb`会为`ufw`或`firewalld`打开UDP端口。 第三方防火墙或后续启用的防火墙，也需要允许相同的端口。
* **PoE供电，无状态指示灯。** DAQ-E本身没有指示灯——请通过交换机或注入器端口的链路/PoE指示灯确认供电情况，并在通电后等待几秒钟，以便设备启动并加入网络。

## 连接

**图形界面（GUI）：** “光传感器”选项卡 → “连接传感器” → 设备类型“DAQ-E (以太网)”。设备发现功能仅在“连接”对话框显示期间运行（Windows 系统中通过mDNS浏览配合ARP扫描），每15秒重复一次；点击“刷新”按钮可立即重新扫描。 发现的传感器会显示在下拉列表中；系统会自动选中第一个检测到的传感器。

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI**（后端正在运行）：

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

池中已存在的 DAQ-E 既可通过主机名也可通过 IP 地址识别：若使用其他地址再次连接，系统将返回现有的池条目，而非再次初始化该设备。

### 多网卡主机及启动后的首次连接

在拥有多个活动网络接口的主机上，即使传感器状态正常，启动后的**首次** `pool-connect --eth` 连接也可能返回空结果——当 ARP 缓存尚未加载时，发现扫描可能会遗漏传感器所在的接口。 可靠的解决方法是跳过发现过程，并显式传入地址：

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` 接受 mDNS 主机名或 IP 地址，始终能定位到正确的传感器，是脚本和无头安装的推荐形式。在图形界面中，请使用连接对话框中的“刷新”按钮，并等待一次重新扫描周期。

## 设备设置与固件

传感器本身保存着其网络设置：DHCP（出厂设置，若无 DHCP 服务器响应则回退到 169.254.x.x 的链路本地地址）或静态地址。它还保存着开机时是否开始流媒体传输的设置。 可在图形界面中进行更改（传感器齿轮图标下的设置，**网络**：**更改…**、**开机时流式传输**、**重启传感器**；参见 [DAQ 选项卡](gui.md))，或通过 CLI 和 SDK 使用 `pool-set-network`、 `pool-set-auto-stream` 和 `pool-reboot` (`DAQSensorSession.set_network()`、`set_auto_stream()`、`reboot()`；参见[CLI 参考](../reference/cli-reference.md#daq-e-device-settings))。 传感器将重启以应用网络变更，Chloros 会重新与其建立连接。静态地址必须位于运行 Chloros 的计算机所在网络中：若重启后 Chloros 无法访问该地址，则会拒绝该地址。传感器的 ID 及其 `daq-e-<id>.local` 主机名永远不会改变。 1.3.0 版本新增功能。

**固件更新功能已集成到图形用户界面中。** 当已连接的 DAQ-E 运行的固件版本早于您的 Chloros 构建版本所捆绑的镜像时，其传感器行会显示一个琥珀色的 **有更新可用** 图标，且齿轮设置模态框中会提供一个“更新至 \<version\>”按钮。 更新通过网络传输约需 30 秒；传感器将自动重启并重新连接，且若传输中断，当前固件将保持不变。 在CLI或SDK中，以及在没有图形界面的Linux系统上，请运行 `chloros-cli daq pool-update-firmware --sensor-id <id>`（`DAQSensorSession.update_firmware()`）。

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## PTP 时间同步

DAQ-E 固件 v1.2.0+ 以普通（仅从属）时钟的身份参与 IEEE 1588 PTPv2 协议。 **Chloros主机的后端即为PTP主时钟**——局域网（LAN）上的每台DAQ-E和每台LATTICE相机均在域0中对其进行从属同步，确保所有设备的时间戳误差控制在约1毫秒以内。 正是这一共享时钟，使得 DAQ 读数的时间戳能够与相机曝光时间相匹配（参见 [录制与 .daq 格式](recording.md)）。

通过CLI检查同步状态：

| 命令 | 显示内容 |
| --- | --- |
| `chloros-cli time-sync status` | 主机主时钟状态、BMCA 优先级、时钟标识，以及 `health`：`OK` 或 `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT`，并附带一个 `health_detail` 说明其数据包为何无法到达网络 |
| `chloros-cli time-sync peers` | 检测到的所有从设备（DAQ-E 传感器 + LATTICE 摄像头） |
| `chloros-cli time-sync cameras` | 每台摄像头的 PTP 健康状态（`PtpStatus`、`PtpOffsetFromMaster`、`PtpMeanPathDelay`） |
| `chloros-cli time-sync restart` | 重启主控进程 |

在图形用户界面（GUI）中，DAQ-E 设置模态框会显示一个实时更新的 **PTP 同步** 行，其中包含传感器当前的 PTP 状态。**PTP 已关闭的 DAQ-E 将保持关闭状态。** 该设备将其 PTP 开/关设置保存在自身内存中，因此，PTP 已被关闭的设备在重新连接或重启后仍保持关闭状态，其时间戳不会与 PTP 同步。 `chloros-cli daq pool-list`会标记此类设备（`!! PTP is switched OFF on this unit …`），并输出修复该问题的命令：

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Chloros绝不会自动将设备重新启用，因为关闭状态可能是用户有意为之。

针对严格对齐用户的详细说明：

* 每个流式数据报都携带一个标志字段；**时间戳经过 PTP 同步的帧中，第 2 位会被设置**。需要严格相机/数据采集（DAQ）对齐的管道应以此位作为触发条件。
* 在进行同步采集之前，请确认传感器出现在 `chloros-cli time-sync peers` 中，且 `chloros-cli daq pool-list` 未标记其 PTP 处于关闭状态。 （MAPIR 的内部直接硬件工具也可通过 `--wait-ptp` 标志基于 PTP 锁定进行录制门控，该标志会等待传感器进入 SLAVE 状态，最长等待时间为 15 秒；该工具不属于随附的 CLI 的一部分。）
* 当 PTP 处于主动从属状态时，传感器会拒绝手动时钟推送（“PTP 正在提供时钟”）。这是设计使然——请信任 PTP。

## Linux 注意事项

* **无需 root 权限的 PTP 端口。** PTP 绑定 UDP 319/320 端口。 `.deb` 会写入 `/etc/sysctl.d/60-chloros-ptp.conf`（`net.ipv4.ip_unprivileged_port_start = 319`），因此后端会以您的用户身份绑定这些端口。 从 1.3.0 版本开始，该软件包不再为后端设置文件权限（`setcap`），也不需要 `libcap2-bin`；请勿自行添加权限，否则解压后的后端将无法启动。 如果 PTP 无法绑定端口 319，请参阅 [Linux 安装](../linux/linux-installation.md#ptp-will-not-start--cannot-bind-port-319)。

* **无显示器的 Jetson / Linux 主机：** 该软件包会安装后端服务，但不会启用任何服务。若要实现无需手动执行命令即可始终保持 PTP（及 DAQ）运行，请以您的用户身份启用该服务：

  ```bash
  sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
  ```

  若未启用，PTP 仅在后端被启动时运行——例如通过 `chloros-cli` 命令或 SDK 脚本。 有关 root `chloros-backend.service` 以及如何从该用户切换，请参阅 [Linux 安装指南](../linux/linux-installation.md#always-on-ptp-for-headless-hosts)。

## 故障排除：“未找到 DAQ-E 设备”

| 检查项 | 详细信息 |
| --- | --- |
| 电源 | 传感器上无 LED 指示灯 — 检查交换机/注入器端口的 PoE 和链路指示灯；通电后等待几秒钟 |
| 广播域 | 主机和传感器位于同一 L2 段；mDNS 无法路由 |
| 防火墙 | Windows：安装程序的 **Chloros后端（UDP-In）** 规则允许 DAQ-E 数据流（UDP 5002）、mDNS（5353）和 PTP（319/320）进入；第三方防火墙也需要相同的设置。 Linux：`.deb` 仅在安装时防火墙处于活动状态时，才会在 `ufw` / `firewalld` 中打开这些端口——若防火墙在安装后启用，请手动允许这些端口 |
| 多网卡主机 | 发现功能会通过 mDNS 进行浏览，并以 ARP 作为备用方案；`--eth-host <ip-or-hostname>` 跳过发现过程并直接连接 |
| GUI 重新扫描 | 发现功能仅在“连接”对话框打开时运行；请使用“刷新”功能 |
