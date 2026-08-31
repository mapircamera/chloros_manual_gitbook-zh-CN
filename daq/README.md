# DAQ 光传感器

> **在寻找硬件信息吗？**关于传感器本身——型号、安装方式、盖帽、接口、电源以及 SCANNER 应用程序——的相关说明详见**[DAQ 用户手册](https://mapir.gitbook.io/daq)**。 本章将介绍从 Chloros 开始使用这些传感器的相关内容。

MAPIR 的 **DAQ** 光传感器将环境光测量为经过辐射计量校准的光谱。在 Chloros 中，它们发挥两种作用：

* **独立光谱仪** — 实时光谱图、色度数据以及 `.daq` 记录，均可通过 [“光传感器”选项卡](gui.md) 获取， [CLI](cli-quick-start.md) 或 Python SDK 中获取。
* **用于反射率的下行辐照度光源** — 在处理过程中，Chloros 会将您的 `.daq` 读数插值到每次拍摄的的曝光时间戳进行插值，并利用测得的下行辐照度将相机辐射度转换为反射率（`--reflectance-source daq`），校准波段无需场景内校准板。

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## 三种型号，一种数据格式

| 型号 | 传输方式 | 发现方式 |
| --- | --- | --- |
| **DAQ-U** | USB（串行） | 串行端口扫描 |
| **DAQ-M** | 蓝牙低功耗 | 按名称进行 BLE 扫描 |
| **DAQ-E** | 以太网（IPv4，PoE供电） | mDNS `_daq-e._tcp`（主机名 `daq-e-<id>.local`） |

这三款设备均采用相同的网络协议，并提供完全相同的数据：

* 每帧数据包含 **340 至 1010 nm 波长范围内、以 5 nm 为间隔的 135 个光谱点**，以及 CIE XYZ 三刺激值。
* **以 W/m²/nm 为单位的辐射校准光谱辐照度** —— 数据传送至您手中之前，已应用了每台设备的出厂校准包（及其有效的盖帽校正配置文件）。
* 统一的 **`.daq` 记录格式**（SQLite 文件）。无论文件由哪种传输方式生成，后续处理流程均完全一致。

传输堆栈（USB 串行、BLE、mDNS/zeroconf）已打包在 Chloros 后端中 ——无需安装任何组件，即可通过图形用户界面或 CLI 的 `pool-*` 命令与这三种型号中的任何一种进行通信。

***

## 校准范围：报告值为 340–1010 nm，校准值为 ~374–974 nm

传感器报告了完整的 340–1010 nm 网格，但可追溯至 NIST 的辐射增益范围约为 **374–974 nm**。 对于任何其光谱权重不足一半位于该校准范围内的相机波段，Chloros 将拒绝进行绝对反射率除法运算；被跳过的波段将报告跳过原因 `dls-uncalibrated-band-<nm>`。

在已上市的 LATTICE 滤光片 SKU 中，仅 **F988** 受此影响：

F988的反射率是使用现场反射率面板校准的：由于该波段超出了DAQ光传感器的校准范围，因此Chloros会采用您最近的面板捕获数据，并在两次面板观测之间保持该值。

如果仅使用 DAQ 数据处理 F988 捕获结果，Chloros 将以跳过原因 `dls-uncalibrated-band-988` 拒绝该波段基于 DAQ 的反射率——[反射率面板工作流](../calibration-targets.md) 是 F988 支持的处理路径。

***

## 传感器 ID

每个 DAQ 都会报告一个稳定的传感器 ID。其格式因型号而异：

| 型号 | ID 格式 | 示例 |
| --- | --- | --- |
| DAQ-U | 5 个八位组，以连字符分隔 | `CB-7C-A8-2E-5F` |
| DAQ-M | 5 个八位字节，带连字符 | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

传感器 ID 为：

* 被嵌入到它记录的每个 `.daq` 文件中，
* Chloros 用于检索该设备出厂校准包的密钥，
* 您在 CLI 和 `pool-*` 命令中传递给 `--sensor-id` 的值，以及
* 对于 DAQ-E，还包括其 mDNS 主机名（`daq-e-def330.local`）——即 `--eth-host` 接受的值。

***

## 出厂校准与云端

每台 DAQ 设备均通过可追溯至 NIST 的辐射计量链进行单独出厂校准，且 Chloros 会根据传感器 ID 为每台设备加载相应的校准包。 每台设备的校准报告（PDF）可从传感器设置中的[“光传感器”选项卡](gui.md)下载。

{% hint style="warning" %}
**DAQ-U 和 DAQ-M 需要云端访问才能进行校准。**这两种型号均不存储任何本地数据：其出厂校准包存储在 MAPIR 的云端，并通过传感器 ID 进行检索（随后缓存到本地）。 Chloros 需要互联网连接才能从 DAQ-U 或 DAQ-M 传输经过校准的 W/m²/nm 数据。**DAQ-E 是例外**——它将校准数据存储在设备上。

<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

{% endhint %}***

## 记录数据的存储位置

| 表面 | 默认的 `.daq` 存储位置 |
| --- | --- |
| 图形用户界面 — “光传感器”选项卡 | `<project folder>/light_sensor/`（完成的记录会自动添加到项目中） |
| CLI — `daq pool-record` | 运行后端程序的机器上的 `~/Documents/DAQ Live View/` |

每个 `.daq` 文件名都包含传感器 ID 和时间戳。

***

## 本章内容

* [**Chloros 中的“DAQ”选项卡**](gui.md) —— 完整的 GUI 操作指南：连接各模型、传感器级设置、光谱图、实时色度数据、双传感器反射率以及数据记录。
* [**CLI 快速入门（pool-\*）**](cli-quick-start.md) —— 从 `chloros-cli daq pool-*` 驱动 DAQ 传感器，支持的命令行路径。
* [**上限配置文件与校准范围**](caps-and-range.md) — 详细说明了各型号支持的上限类型、声明方法以及校准光谱范围。
* [**数据记录与 .daq 格式**](recording.md) — `.daq` 的 SQLite 格式及数据记录工作流。
* [**DAQ-E 网络与时间同步**](ethernet-ptp.md) —— DAQ-E 传输模式及 PTP 时间同步。
* [**反射率工作流程**](reflectance.md) ——利用 DAQ 下行数据生成反射率。
* 有关完整的标志级文档，请参阅 [CLI 参考手册](../reference/cli-reference.md)（第 `chloros-cli daq` 节）以及 [SDK 参考手册](../reference/sdk-reference.md)（`chloros_sdk.connect_daq_sensor()`），这两份文档均按可被 AI 助手直接使用的格式编写。
