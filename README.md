---
metaLinks: {}
---

# 入门指南

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>

Chloros 是 [MAPIR](https://www.mapir.camera) 推出的一款软件应用程序，用于处理多光谱图像、实时控制 MAPIR 硬件以及记录传感器数据。Chloros 1.3.0 支持完整的 MAPIR 产品系列：

* **Survey3 相机** — 将RAW+JPG格式图像处理为经过校准的反射率和植被指数图。请参阅[支持的相机](supported-cameras.md)。
* **LATTICE 相机** — 实时连接 GigE 多光谱相机模块，可单独使用或作为同步的多相机阵列：预览、采集并处理为经过校准的辐射度和反射率产品。请参阅 [LATTICE 章节](lattice/README.md)。
* **DAQ 光传感器** — DAQ-U（USB）、DAQ-M（蓝牙）以及 DAQ-E 和 DAQ-E-S（以太网）光谱传感器：实时校准光谱、 `.daq` 记录，以及用于反射率处理的向下照度。请参阅 [DAQ 章节](daq/README.md)。

{% hint style="success" %}
**Chloros 1.3.0 的新功能**：AI 助手可通过 `chloros-mcp` 驱动 Chloros，且 CLI、SDK 和 `chloros-mcp` 均提供相同功能。 LATTICE 索引产品已得到改进，全传感器数组运行速度更快且可在任何网络上工作，后端在命令之间持续运行。请参阅下方的“新功能”列表，并[下载](download.md)安装程序。
{% endhint %}

{% hint style="info" %}
**正在使用 AI 助手操作 Chloros？** 本手册专为此设计。请让您的助手访问：

* `https://mapir.gitbook.io/chloros/llms.txt` — 所有页面的机器可读索引。
* 任何页面的原始 Markdown 格式 — 在其 URL 末尾添加 `.md`（例如 `https://mapir.gitbook.io/chloros/reference/cli-reference.md`）。
* [CLI 参考](reference/cli-reference.md) 和 [SDK 参考](reference/sdk-reference.md) —— 专为大型语言模型（LLM）处理而编写的完整、精确值参考页面。

提示词示例：*“阅读 https://mapir.gitbook.io/chloros/reference/cli-reference.md,，然后编写一个脚本，用于登录并处理 ~/flights/flight_001 文件夹中的数据，将其转换为反射率 + NDVI GeoTIFF 格式。”*

或者让它直接运行 Chloros：`chloros-mcp` 是一个随 SDK 安装的 MCP 服务器，可与 Claude Desktop、Claude Code、Cursor 及其他 MCP 客户端配合使用。

完整指南：[如何将Chloros 与AI助手结合使用](ai-assistants.md)。
{% endhint %}

***

## Chloros 1.3.0 的新功能

* **AI 助手可驱动 Chloros。** `chloros-mcp` 是一款安装了 Python SDK 的 MCP 服务器，可让 Claude Desktop、Claude Code、Cursor 及其他 MCP 客户端处理图像，并运行 LATTICE 相机、阵列和 DAQ 光传感器。请参阅 [如何将 Chloros 与 AI 助手配合使用](ai-assistants.md) 中的说明。
* **CLI、SDK 和 `chloros-mcp` 提供相同的功能。** 出于安全考虑，仅登录、DAQ-E 配置以及直接 DAQ 服务器功能保留在 CLI 中。 部分 1.2.0 脚本需要修改：请参阅 [CLI 参考](reference/cli-reference.md) 和 [SDK 参考](reference/sdk-reference.md) 顶部的 **

1.3.0 版本变更**。
* **改进了 LATTICE 指数产品。** 在所有处理模式下，从 LATTICE 捕获数据中导出的 NDVI 及其他指数产品均已得到改进。
* **Windows 安装程序选项。**在**系统设置**中，安装程序提供了**启用 Windows 长路径**和**将 Chloros 后端排除在 Microsoft Defender 扫描之外** 两个选项，两者默认均已勾选。 前者允许项目文件夹路径长度超过 260 个字符；后者可防止实时扫描导致相机采集速度变慢（在高采集速率下，实时扫描会使采集速率降低约 30%）。安装程序现在还会仅向 `PATH` 添加一次 CLI 文件夹，并移除早期安装程序留下的重复条目。请参见 [下载](download.md)。
* **过深的项目文件夹会被直接拒绝。** 当Windows 长路径功能处于关闭状态（Windows 默认设置）时，若项目文件夹路径深度超出260个字符的限制，系统会在任何操作执行前直接拒绝该文件夹，并显示符合路径限制的最长项目文件夹名称以及如何开启长路径功能。此前，此类运行将无法生成任何输出文件。 启用长路径后，位于非常深层文件夹中的产品现在可以保留其元数据（GPS 和 XMP 标签）。
* **剪切掩码。** 当 LATTICE 辐射度或反射率产品的波段解混产生负波段值时，该值将被剪切为零，并且在产品文件夹旁边的 `clamp_masks/` 文件夹中，会通过逐像素掩码记录哪些像素和波段受到了影响。 产品在 `Chloros:UnmixClampMask` 标签中为该掩膜命名。实时预览的裁剪效果现与导出结果完全一致。参见 [输出图像格式](output-image-formats.md)。
* **产品具有自我描述性。** 索引产品携带自身的处理标签（`Chloros:ProcessingLevel=index`、索引表达式，以及是否基于辐射度或反射率计算），而非源原始数据的标签。Survey3 产品在所有处理模式下均携带Chloros 处理标签。 LATTICE 原始文件和去拜耳化文件保持传感器原生方向，并在 XMP 中记录其方向；图像查看器仅在显示时对其进行旋转。
* **Linux 软件包以未解压形式发布。** 后端和 CLI 作为文件夹安装在 `/usr/lib/chloros/` 目录下，因此不再在每次启动时自动解压。 在 x86_64 平台上，后端启动时间不再耗时约 100 秒，且 `.deb` 的文件大小约为之前的二分之一。 捆绑的 ExifTool 现可在纯净的 Ubuntu 系统上运行。`.deb` 现还会为系统安装 Python SDK（通过 `python3` （请使用 `sudo apt install ./chloros_1.3.0_<arch>.deb` 进行安装），并提高了相机流所需的内核套接字接收缓冲区大小，同时新增了一个以您的用户身份运行的后端服务。 Jetson 软件包 `chloros_1.3.0_arm64_jp6.deb` 是为 JetPack 6 构建的，使用 JetPack 的 CUDA 版本 PyTorch 在 Jetson 的 GPU 上进行处理。 在 Linux 系统上，每张导出的图像现在仅需一次写入磁盘（先在内存中写入并标记，然后复制到目标位置）：而在 Jetson 设备上（磁盘性能是主要瓶颈），一个包含 288 张图像的 LATTICE 运行仅需 14 分钟，而非 24 分钟。参见 [Linux 安装指南](linux/linux-installation.md)。
* **LATTICE OCN 相机在阳光下的读数更准确。** 波段解混后，辐射度、反射率和指数会进行快速校正，因此灰度目标在阳光下测得的NDVI 值接近中性。RGN 相机保持不变。
* **应用内错误报告。**“调试日志”选项卡顶部的错误图标可向 MAPIR 发送错误描述、最多 10 张图像、日志及系统详细信息。参见 [GUI：导航](navigation.md)。
* **LATTICE 阵列。** 默认情况下，阵列保留完整传感器数据；采用压缩的 12 位传输方式（`--packed`，默认 `auto`）可将完整传感器帧率提高约三分之一； 数组可在任何网络上以该网络允许的帧率运行（低速链路会降低帧率，而非拒绝数组或缩减帧大小）；数组中的相机重启后会自动恢复，PTP 协议随即重新启用；在光线昏暗的环境下，自动曝光不再降低帧率（它会先填满周期，然后提高增益）； 连续捕获功能可跟上高速阵列的节奏；RGB 相机新增了**高光**控制；SDK 中的`ArraySession`新增了`grab()`、 `frame_stream()` 和 `continuous()`。请参阅 [LATTICE 章节](lattice/README.md)。
* **更稳定的自动曝光。** 在光线恒定的情况下，曝光值保持不变；光线变化后，曝光值会在几秒内稳定下来；微小的高光区域不再导致整个场景变暗；手动曝光时增益仍保持在“自动”模式；在强光下停止响应的相机，其曝光值将根据光线传感器测得的光照水平保持稳定，若未连接光线传感器，则由相机自身的帧数据决定。 参见 [相机设置](lattice/camera-settings.md)。
* **光传感器。** **DAQ-E-S 支持：**其固定式漫射器无需镜头校正，且每次读数均包含设备的俯仰、滚转和偏航角度。 记录文件会标明其遮光罩是手动选择还是系统默认的，处理过程会对默认情况发出警告：DAQ-U、DAQ-M 和 DAQ-E 出厂时未配备遮光罩，因此请为已安装的遮光罩命名。 `.daq` 的实时记录可在断电后保留（请保留包含其 `.daq-journal` 的 `.daq` 文件）， 当 DAQ-E 停止传输时，重新连接后，其记录中会标记传输中断的间隔。 仅包含光传感器记录的项目可在应用程序和CLI 中列出并处理这些记录，使用 `chloros_scripts` 生成的记录将显示为由该 DAQ 生成，且其名称后缀可由您设置。每次捕获都会保存所使用的 DAQ 读数。请参阅 [DAQ 章节](daq/README.md)。
* **处理。**LATTICE帧在目标检测期间开始处理。目标和DAQ参考值按波段分别确定。移动项目后，其文件会重新建立链接。8位捕获数据的导出格式为8位，采用新的**TIFF（8位）**格式。 未使用校准目标的Survey3 运行将作为其备用产品（`Vignette_Corrected_Images` 或 `Sensor_Response_Images`）保存，而非反射率图像。反射率图像的各波段 DLS 标签始终属于其自身帧。 LATTICE 采集不支持“纹理感知”功能，此类采集始终采用标准去拜耳处理。
* **CLI 和 SDK。** 相对输出路径现位于您运行命令或脚本的文件夹中（此前已安装的后端会尝试将其解析到自身的安装文件夹中，从而导致失败）。 后端会在命令和脚本之间持续运行，并在闲置 30 分钟后自动停止；`chloros-cli backend status` 和 `backend stop` 负责管理此过程。 捕获命令会列出所有无法写入的文件并以代码 1 退出，且当磁盘空间已满时，重复捕获操作将停止。 `process` 运行成功时会报告 `Image products written: N (plus M clamp masks)`。在 Windows 上，SDK 的 `ChlorosLocal` / `process_folder` 命令可查找已安装的后端（并使用已运行的后端），而 `read_image_audit_tags` 可直接从普通 `pip install` 运行。

***

## Chloros 1.2.0 的新功能

* **实时相机控制 — 新增“相机”选项卡。** 可逐台连接 LATTICE 相机，或作为同步的多相机阵列（PTP 时间同步、硬件触发捕获）使用，支持实时预览叠加、各波段直方图、智能自动曝光、实时指数计算器以及应用内相机固件更新。
* **光传感器 — 新增“光传感器”选项卡。** 可连接 DAQ-U（USB）、DAQ-M（蓝牙）和 DAQ-E（以太网）传感器； 查看实时校准光谱（W/m²/nm），将`.daq`文件记录到项目中，选择顶部校正曲线，并通过网络更新DAQ-E固件。
* **采集模式与记录器。** 单次/连续/间隔采集，以及仅支持原始数据的“最快采集”模式；可按项目选择“全量采集”生成的相机和导出类型；阵列记录器支持监控级索引视频和分析级原始帧数据，并可离线生成视频。
* **LATTICE 处理管道。** 导入 LATTICE 采集文件夹，并将每个原始帧拆分为去拜耳化、预览、float32 辐射度 (W/m²/sr/nm) 和反射率产物，并支持按产物单独启用/禁用。 反射率数据可来自帧内校准目标或数据采集（DAQ）的下行辐射；导出时将应用阵列对齐；若缺少出厂校准数据，系统将根据相机序列号自动下载。
* **项目会记住硬件配置。** 已连接的相机和光传感器会随项目（`cameras.json` / `sensors.json`）一起保存，当您重新打开项目时，它们将使用保存的设置重新连接。 参见 [GUI：项目](projects.md)。
* **图像查看器升级。** 支持光标像素/索引读数（带正确的按文件反射率缩放）、图层直方图、GSD 分桶滑块、按触发/按相机网格模式、LATTICE 产品视图，以及将索引/LUT 沙盒导出到磁盘。
* **CLI 和 SDK 功能大幅扩展。** 新增 `lattice`、`daq pool-*`、`project` 和 `time-sync` 命令系列； 新增 `process` 选项（`--input-level`，按产品切换开关；`--reflectance-source`，数组对齐标志）； SDK 智能连接句柄（`connect_camera` / `connect_array` / `connect_daq_sensor`），可自动启动后端； `open_project()` 自动化；SDK 轮子已与安装程序打包在一起，并作为 `chloros-sdk` 发布到 PyPI。
* **透明的失败语义。** 此前，若 `chloros-cli process` 运行时请求了产品但未生成任何产品，会明确报错并返回非零退出代码；而成功的运行会报告生成了多少个图像产品。
* **新的输出布局。** 产品存储在 `<project>/<camera>/<format>/<Product>_Images/` 文件夹中，并保留源文件名——通过文件夹（而非文件名后缀）来标识产品。参见 [输出图像格式](output-image-formats.md)。
* **更多输入源、方案和语言支持。** 支持 `.dng` 输入源；全部 38 种界面语言均已完整支持；按方案设置的硬件限制中，免费（无需登录）使用最多支持 4 台摄像头和 2 个光传感器。
* **可靠性。**“停止处理”功能可干净利落地结束，并提供真实的运行摘要；多摄像头项目会导出每台摄像头的数据；安装程序升级时不再导致您被强制注销。***

Chloros 提供 3 种应用界面：

## Chloros：桌面图形用户界面（GUI）应用程序

独立窗口，具备所有功能，包括“实时摄像头”和“光传感器”选项卡。_仅限 Windows 系统。_

## [Chloros CLI：命令行界面](CLI.md)

支持命令行批处理，并提供实时 `lattice`、`daq pool-*`、`project` 和 `time-sync` 命令。 非常适合自动化、脚本编写和无头操作。支持 **Windows、Linux amd64 以及 Linux arm64（NVIDIA Jetson）**。_访问 CLI 需要付费的 Chloros+ 套餐。_

## [Chloros API：Python SDK](api-python-sdk.md)

用于自动化和自定义工作流的 Python 编程接口：全流程处理、实时摄像头/阵列会话、DAQ 传感器会话以及保存项目的自动化。 随桌面版/CLI 软件包一同安装，也以 `pip install chloros-sdk` 的名称发布。_使用 API 需订阅付费的 Chloros+ 套餐。_

***

## 支持的平台

| 平台 | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | 是 | 是 | 是 |
| **Linux amd64 (x86_64)** | 否 | 是 | 是 |
| **Linux arm64 (NVIDIA Jetson)** | 否 | 是 | 是 |

有关 Linux 的安装说明，请参阅 [Linux 与边缘计算](linux/linux-overview.md) 部分。

***

## 三个步骤快速入门

1. **安装** — 下载并运行适用于您平台的安装程序。请参阅 [下载](download.md)。
2. **登录（GUI 模式可选）** — 即使没有账户，GUI 也能免费处理图像。通过 [Chloros+ 登录](chloros+-login.md) 可解锁并行处理、GPU 加速、更高的设备限制，以及 CLI/SDK 访问权限。
3. **创建您的第一个项目** — 打开 Chloros，创建一个 [新项目](projects.md)，[添加您的图像](processing-images-gui/adding-files-to-a-project.md)，然后 [开始处理](processing-images-gui/starting-the-processing.md)。若要直接控制实时硬件，请打开“摄像头”或“光传感器”选项卡——请参阅 [GUI：导航](navigation.md)。

***

## Chloros+

虽然 Chloros 对于大多数任务都是免费的，但您可能会发现自己需要更多功能。这时，购买 Chloros+ 的付费许可证将对您大有裨益。拥有 Chloros+ 许可证后，您可以解锁以下新功能：

* **多线程处理**：通过在处理管道中同时处理图像，大幅加快大型项目的图像处理速度。
* **GPU（CUDA）加速**：利用当今更高容量的 GPU 显存选项，进一步加快图像处理管道的速度。建议使用 4GB 或以上的显存以获得最佳效果。
* **Chloros+**[**CLI**](CLI.md) **访问**：通过命令行运行 Chloros+，实现自动化并集成到您自己的软件中。所有付费套餐均可使用；该功能在服务器端强制执行。
* **Chloros+**[**API**](api-python-sdk.md) **访问方式：**通过 Python 运行 Chloros+ 以实现程序化控制，从而与您的研究流程、数据分析工作流及自定义应用程序无缝集成。适用于任何付费套餐；在服务器端强制执行。
* **更高的硬件限制**：可同时连接更多摄像头和光传感器。未登录时，图形用户界面（GUI）最多可连接 4 台摄像头和 2 个 DAQ 光传感器；付费套餐将同时提高这两项上限：

| 套餐 | 摄像头 | DAQ 光传感器 |
| --- | --- | --- |
| 铁 (免费，无需登录) | 4 | 2 |
| 铜 / 青铜 | 6 | 3 |
| 银 | 10 | 6 |
| 金 | 20 | 12 |

* **多设备使用**：每份 Chloros+ 许可证允许注册 2 台及以上设备。请使用您的 MAPIR 云账户管理已注册设备。通过升级 Chloros+ 许可证可增加对更多设备的支持。
* **高级纹理感知去拜耳化方法：** 高质量的边缘感知去拜耳化技术结合 AI/ML 降噪模型，可消除几乎所有的去拜耳化噪声。
* **自定义多光谱指数公式：** 在 Chloros 栅格计算器中输入自定义多光谱指数，适用于图像处理和图像查看沙盒。
* **Linux 与边缘计算：** 在 Linux x86_64 和 ARM64 平台（包括 NVIDIA Jetson）上运行 Chloros，以实现现场和边缘处理。请参阅 [Linux 概述](linux/linux-overview.md)。

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ 定价与注册</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.3.0 build so the banner prints "Chloros CLI 1.3.0". -->
