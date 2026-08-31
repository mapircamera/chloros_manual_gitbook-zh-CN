---
metaLinks: {}
---

# 入门指南

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros 是一款由 [MAPIR](https://www.mapir.camera) 开发的一款软件应用程序，用于处理多光谱图像、实时控制 MAPIR 硬件以及记录传感器数据。 Chloros 1.2.0 支持完整的 MAPIR 产品系列：

* **Survey3 相机** — 将 RAW+JPG 原始图像处理为经过校准的反射率和植被指数图。请参阅 [支持的相机](supported-cameras.md)。
* **LATTICE 相机** — 实时连接 GigE 多光谱相机模块，可单独使用或作为同步多相机阵列：预览、采集并处理为经过校准的辐射度和反射率产品。请参阅 [LATTICE 章节](lattice/README.md)。
* **DAQ 光传感器** — DAQ-U（USB）、DAQ-M（蓝牙）和 DAQ-E（以太网）光谱传感器：提供实时校准光谱、`.daq` 记录，以及用于反射率处理的下行照度数据。 请参阅[DAQ章节](daq/README.md)。

{% hint style="success" %}
**Chloros 1.2.0 版本更新**： 实时 LATTICE 相机和阵列控制、DAQ 光传感器集成、采集模式和记录器、完整的 LATTICE 辐射测量处理管道、基于 CLI/SDK 的项目自动化，以及更多功能。 请参阅下方的“新功能”列表，并[下载](download.md)查看更改日志。
{% endhint %}

{% hint style="info" %}
**正在使用 Chloros 配合 AI 助手吗？** 本手册正是为此而设计。请让您的助手访问：

* `https://mapir.gitbook.io/chloros/llms.txt` — 所有页面的机器可读索引。
* 任何页面的原始 Markdown 格式 — 在其 URL 后缀处添加 `.md`（例如 `https://mapir.gitbook.io/chloros/reference/cli-reference.md`）。
* [CLI 参考指南](reference/cli-reference.md) 以及 [SDK 参考](reference/sdk-reference.md)——专为 LLM 处理而编写的完整、精确值参考页面。

提示词示例：*“阅读 https://mapir.gitbook.io/chloros/reference/cli-reference.md,，然后编写一个脚本，用于登录并处理 ~/flights/flight_001 文件夹中的数据，将其转换为反射率 + NDVI GeoTIFF 格式。”*

完整指南：[如何在 AI 助手中使用 Chloros](ai-assistants.md)。
{% endhint %}

***

## Chloros 1.2.0 的新功能

* **实时相机控制 — 新增“相机”选项卡。** 可单独连接 LATTICE 相机，或作为同步的多相机阵列（PTP 时间同步、硬件触发捕获）使用，支持实时预览叠加、分波段直方图、智能自动曝光、实时指数计算器以及应用内相机固件更新。
* **光传感器 — 新增“光传感器”选项卡。**可连接 DAQ-U（USB）、DAQ-M（蓝牙）和 DAQ-E（以太网）传感器； 查看实时校准光谱（W/m²/nm），将`.daq`文件记录到项目中，选择电容校正曲线，并通过网络更新DAQ-E固件。
* **采集模式与记录器。** 单次/连续/间隔采集，以及仅限原始数据的“最快采集”模式；可按项目选择“全量采集”生成的摄像头和导出类型；阵列记录器支持监控级索引视频和分析级原始突发数据，并可离线生成视频片段。
* **LATTICE 处理管道。** 导入 LATTICE 采集文件夹，并将每个原始帧展开为去拜耳化、预览、float32 辐射度 (W/m²/sr/nm) 和反射率产物，并支持按产物逐项切换。 反射率数据可来自帧内校准目标或数据采集（DAQ）的下行光；导出时将应用阵列对齐；若缺少出厂校准数据，系统会根据相机序列号自动下载。
* **项目会记住硬件配置。** 已连接的相机和光传感器会随项目（`cameras.json` / `sensors.json`）一同保存，当您重新打开项目时，这些设备将使用其保存的设置重新连接。 参见 [GUI：项目](projects.md)。
* **图像查看器升级。** 支持光标像素/索引读数（带有正确的按文件反射率缩放）、图层直方图、GSD 分桶滑块、按触发/按相机网格模式、LATTICE 产品视图，以及将索引/LUT 沙盒导出到磁盘。
* **CLI 和 SDK，功能大幅扩展。** 新增 `lattice`、`daq pool-*`、`project` 和 `time-sync` 命令族； 新增 `process` 选项（`--input-level`，按产品设置的开关；`--reflectance-source`，数组对齐标志）； SDK 智能连接句柄（`connect_camera` / `connect_array` / `connect_daq_sensor`），可自动启动后端； `open_project()` 自动化；SDK 包已随安装程序捆绑，并以 `chloros-sdk` 的名称发布到 PyPI。
* **诚实的失败语义。** 此前，`chloros-cli process` 运行时若请求生成产品但未实际生成任何产品，会明确报错并以非零状态退出；而成功运行时则会报告生成的图像产品数量。
* **新的输出布局。** 生成结果保存在 `<project>/<camera>/<format>/<Product>_Images/` 文件夹中，并保留源文件名——通过文件夹（而非文件名后缀）来标识生成结果。参见 [输出图像格式](output-image-formats.md)。
* **更多输入支持、方案和语言。** 支持 `.dng` 输入；38 种界面语言均已完整支持；按方案设置的硬件限制，免费（无需登录）使用时最多支持 4 台摄像头和 2 个光传感器。
* **可靠性。**“停止处理”功能可干净利落地终止操作并提供真实的运行摘要，多摄像头项目会导出每台摄像头的数据，且安装程序升级时不再导致您被自动注销。***

Chloros 提供 3 种应用界面：

## Chloros：桌面图形用户界面（GUI）应用程序

独立窗口，具备全部功能，包括“实时摄像头”和“光传感器”选项卡。_仅限 Windows 系统。_

## [Chloros CLI：命令行界面](CLI.md)

支持命令行批处理，并包含实时命令 `lattice`、`daq pool-*`、`project` 和 `time-sync`。 非常适合自动化、脚本编写和无头操作。 可在 **Windows、Linux amd64 和 Linux arm64（NVIDIA Jetson）** 上使用。 _访问 CLI 需要付费的 Chloros+ 服务等级。_

## [Chloros API：Python SDK](api-python-sdk.md)

用于自动化和自定义工作流的编程接口 Python：全流程处理、实时摄像头/阵列会话、DAQ 传感器会话以及保存项目的自动化。 随桌面版/CLI 软件包安装，同时以 `pip install chloros-sdk` 形式发布。_访问该 API 需要付费的 Chloros+ 服务等级。_

***

## 支持的平台

| 平台 | 图形界面 | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | 是 | 是 | 是 |
| **Linux amd64 (x86_64)** | 否 | 是 | 是 |
| **Linux arm64 (NVIDIA Jetson)** | 否 | 是 | 是 |

有关 Linux 的安装说明，请参阅 [Linux 与边缘计算](linux/linux-overview.md) 部分。

***

## 三个步骤开始使用

1. **安装** — 下载并运行适用于您平台的安装程序。请参阅 [下载](download.md)。
2. **登录（GUI 版可选）** — 即使没有账户，GUI 版也可免费处理图像。通过 [Chloros+ 登录](chloros+-login.md) 可解锁并行处理、 GPU 加速、更高的设备限制，以及 CLI/SDK 访问权限。
3. **创建您的第一个项目** — 打开 Chloros，创建 [新项目](projects.md)，[添加图像](processing-images-gui/adding-files-to-a-project.md)，并[开始处理](processing-images-gui/starting-the-processing.md)。 若要驱动实时硬件，请打开“摄像头”或“光传感器”选项卡——参见 [GUI：导航](navigation.md)。***

## Chloros+

虽然 Chloros 对于大多数任务可免费使用，但您可能会发现还需要更多功能。这时，购买 Chloros+ 的付费许可证将对您大有裨益。 拥有 Chloros+ 许可证，您可以解锁以下新功能：

* **多线程处理**：通过在处理管道中同时处理图像，大幅加快大型项目的图像处理速度。
* **GPU（CUDA）加速**：利用当今更高容量的 GPU 内存选项，进一步加快图像处理管道的速度。建议使用 4GB 或更大的显存以获得最佳效果。
* **Chloros+**[**CLI**](CLI.md)**访问**：通过命令行运行 Chloros+，以实现自动化并集成到您自己的软件中。 适用于任何付费套餐；在服务器端强制执行。
* **Chloros+**[**API**](api-python-sdk.md)**访问方式：**从 Python 运行 Chloros+ 以实现编程控制，从而与您的研究流程、数据分析工作流以及自定义应用程序无缝集成。适用于任何付费套餐；在服务器端强制执行。
* **更高的硬件限制**：可同时连接更多摄像头和光传感器。未登录时，图形用户界面（GUI）最多可连接 4 台摄像头和 2 个 DAQ 光传感器；付费套餐将同时提高这两项上限：

| 套餐 | 摄像头 | DAQ 光传感器 |
| --- | --- | --- |
| 铁（免费，无需登录） | 4 | 2 |
| 铜 / 青铜 | 6 | 3 |
| 银 | 10 | 6 |
| 金 | 20 | 12 |

* **多设备使用**：每份 Chloros+ 许可证允许注册 2 台及以上设备。请使用您的 MAPIR 云账户管理已注册设备。 通过升级您的 Chloros+ 许可证，可增加对更多设备的支持。
* **高级纹理感知去拜耳算法：** 一种高质量的边缘感知去拜耳算法，结合 AI/ML 降噪模型，可消除几乎所有的去拜耳噪声。
* **自定义多光谱指数公式：** 在 Chloros 栅格计算器中输入自定义多光谱指数，适用于图像处理和图像查看沙盒。
* **Linux 与边缘计算：** 在 Linux x86_64 和 ARM64 平台（包括 NVIDIA Jetson）上运行 Chloros，以实现现场和边缘处理。 请参阅 [Linux 概述](linux/linux-overview.md)。

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ 定价与注册</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.2.0 build so the banner prints "Chloros CLI 1.2.0". -->
