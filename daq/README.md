# DAQ 光传感器

> **在寻找硬件信息吗？**传感器本身——包括型号、安装方式、防护盖、接口、电源以及 SCANNER 应用程序——的相关说明详见**[DAQ 用户手册](https://mapir.gitbook.io/daq)**。 本章介绍如何在 Chloros 中使用这些传感器。

MAPIR 的 **DAQ** 光传感器将环境光测量为经辐射校准的光谱。在 Chloros 中，它们具有两种功能：

* **独立光谱仪** —— 实时光谱图、色度数据以及 `.daq` 记录，均可通过 [光传感器选项卡](gui.md)、[CLI](cli-quick-start.md) 或 Python SDK 获取。
* **用于反射率计算的向下辐照度源** — 在处理过程中， Chloros会将您的`.daq`读数插值到每次拍摄的曝光时间戳，并利用测得的下行光将相机辐射度转换为反射率（`--reflectance-source daq`），对于已校准的波段无需场景内校准面板。

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## 三种型号，一种数据格式

| 型号 | 传输方式 | 发现机制 |
| --- | --- | --- |
| **DAQ-U** | USB（串行） | 串行端口扫描 |
| **DAQ-M** | 蓝牙低功耗 | 按名称进行 BLE 扫描 |
| **DAQ-E** | 以太网（IPv4，PoE供电） | mDNS `_daq-e._tcp`（主机名 `daq-e-<id>.local`） |**DAQ-E-S（1.3.0 版本新增）。** 一款配备固定式漫射器和倾斜传感器的 DAQ-E。Chloros 将其列为 `DAQ-E-S`，其运行方式与 DAQ-E 完全相同，唯一区别在于它不进行帽角校正，且每次读数都会包含设备的倾斜、横滚和俯仰角度。 参见 [DAQ-E-S：固定扩散板和倾斜](caps-and-range.md#daq-e-s-fixed-diffuser-and-tilt)。

这三款设备均采用相同的通信协议，并提供完全一致的数据：

* 每帧包含 **340 至 1010 nm 波长范围内、以 5 nm 为步长的 135 点光谱**，以及 CIE XYZ 三刺激值。
* **以 W/m²/nm 为单位的辐射计量校准光谱辐照度** —— 数据传送至您手中之前，已应用了每台设备的出厂校准包（及其有效的遮光罩校正曲线）。
* 统一的 **`.daq` 记录格式**（SQLite 文件）。无论文件由哪种传输方式生成，后续处理流程均完全一致。

传输协议栈（USB 串行、BLE、mDNS/zeroconf）已集成在 Chloros 后端中——无需安装任何软件，即可通过 GUI 或 CLI 的 `pool-*` 命令与这三种型号中的任何一种进行通信。

***

## 校准范围：报告值为 340–1010 nm，校准值为 ~374–974 nm

传感器报告了完整的 340–1010 nm 网格，但可追溯至 NIST 的辐射增益范围约为 **374–974 nm**。 Chloros 拒绝将任何其光谱权重不足一半位于该校准范围内波段的相机波段进行绝对反射率除法计算；被跳过的波段将报告跳过原因 `dls-uncalibrated-band-<nm>`。

在已上市的 LATTICE 滤光片型号中，仅 **F988** 受此影响：

F988 的反射率是使用现场反射率面板进行校准的：由于该波段超出了 DAQ 光传感器的校准范围，因此 Chloros 会采用您最近一次的面板捕获数据，并在两次面板观测之间保持该值不变。

如果仅使用 DAQ 数据处理 F988 采集数据，Chloros 将拒绝该波段基于 DAQ 的反射率，跳过原因代码为 `dls-uncalibrated-band-988` ——[反射率面板工作流](../calibration-targets.md) 是 F988 支持的处理路径。

***

## 传感器 ID

每个 DAQ 都会报告一个稳定的传感器 ID。其格式因型号而异：

| 型号 | ID 格式 | 示例 |
| --- | --- | --- |
| DAQ-U | 5 个八位组，用连字符分隔 | `CB-7C-A8-2E-5F` |
| DAQ-M | 5 个八位字节，带连字符 | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

传感器 ID 为：

* 被嵌入到它记录的每个 `.daq` 文件中，
* Chloros 用于获取该设备出厂校准包的密钥，
* 您在CLI命令中传递给`--sensor-id`的值，以及
* 对于 DAQ-E，还包括其 mDNS 主机名（`daq-e-def330.local`）——即 `--eth-host` 接受的值。

***

## 出厂校准与云端

每台 DAQ 设备均通过可追溯至 NIST 的辐射测量链进行单独出厂校准，Chloros 会为每台设备加载以传感器 ID 为键的校准包。 每台设备的校准报告（PDF）可从传感器设置中的 [光传感器选项卡](gui.md) 下载。

{% hint style="warning" %}
**DAQ-U 和 DAQ-M 需要连接云端才能进行校准。**这两种型号均不存储任何数据在设备本地：其出厂校准包存储在 MAPIR 的云端，并通过传感器 ID 进行检索（随后缓存到本地）。 Chloros 需要互联网连接，才能从 DAQ-U 或 DAQ-M 传输经过校准的 W/m²/nm 数据。**DAQ-E 是例外**——它将校准数据存储在设备本地。
{% endhint %}
<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

***

## 记录数据的存储位置

| 界面 | 默认 `.daq` 存储位置 |
| --- | --- |
| GUI — “光传感器”选项卡 | `<project folder>/light_sensor/`（完成的记录会自动添加到项目中） |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/`（位于运行后端的机器上） （当后端位于同一台计算机上时，相对路径 `-o` 将解析为运行该命令的文件夹） |

每个 `.daq` 文件名都包含传感器 ID 和时间戳。

***

## 本章内容

* [**Chloros 中的“DAQ”选项卡**](gui.md) ——完整的图形用户界面（GUI）操作指南：连接各型号设备、单传感器设置、光谱图、实时色度数据、双传感器反射率以及数据记录。
* [** CLI 快速入门 (pool-\*)**](cli-quick-start.md) —— 通过 `chloros-cli daq pool-*` 驱动 DAQ 传感器，即受支持的命令行路径。
* [**限幅配置文件与校准范围**](caps-and-range.md) — 详细说明各型号支持的限幅类型、如何声明限幅，以及校准光谱范围。
* [**数据记录与 .daq 格式**](recording.md) — 介绍 `.daq` 的 SQLite 格式及数据记录工作流。
* [**DAQ-E 网络连接与时间同步**](ethernet-ptp.md) —— DAQ-E 传输模式和 PTP 时间同步。
* [**反射率工作流程**](reflectance.md) ——利用 DAQ 下行数据生成反射率。
* 有关完整的标志级文档，请参阅 [CLI 参考](../reference/cli-reference.md)（第 `chloros-cli daq` 节）和 [SDK 参考](../reference/sdk-reference.md)（`chloros_sdk.connect_daq_sensor()`），这两份文档均采用可被 AI 助手直接调用的格式编写。
