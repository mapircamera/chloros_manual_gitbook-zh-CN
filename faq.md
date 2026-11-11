---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# 常见问题

<details>

<summary>我可以使用Chloros处理非MAPIR品牌的相机拍摄的图像吗？</summary>

不可以，Chloros仅支持处理MAPIR品牌的相机图像——即Survey3和LATTICE系列。 更多信息请参阅[支持的相机型号列表](supported-cameras.md)。我们确实在MAPIR云平台上提供其他相机的图像处理服务，完整列表请参见[此处](https://mapir.gitbook.io/mapir-cloud/supported-cameras)。

</details>

<details>

<summary>Chloros是否支持LATTICE相机？</summary>

是的。Chloros 1.3.0 端到端支持 LATTICE M3C 和 M3M 相机模块： **实时控制**—— 通过 GUI 的“相机”选项卡（`chloros-cli lattice`）或 Python SDK 进行设备发现、连接、预览和抓取，包括支持 PTP 时间同步的多相机阵列——以及对抓取数据的**完整辐射度处理** （原始数据 → 去拜耳化 → 辐射度 → 反射率 → 指数）。请参阅 [支持的相机](supported-cameras.md) 和 [LATTICE 指南](lattice/README.md)。

</details>

<details>

<summary>如果没有校准目标，我能对图像进行反射率校准吗？</summary>

**Survey3

：** 不行。如果在拍摄非校准目标图像时未同时拍摄校准目标图像，您将无法将图像的像素值与已知的反射率百分比建立关联。 如果您也没有包含MAPIR

光传感器记录的数据，则无法测量环境光谱，反射率结果将不准确。

**LATTICE：** 可以。反射率可以参照由DAQ光传感器（而非面板）测得的下行辐照度来确定（ρ = π·L/E）。 当帧内存在通过质量保证（QA）检测的目标时，它默认将成为绝对参考（`--reflectance-source auto`），除非该目标通过时带有标记，此时目标与DAQ链将根据测量分数进行比较。 有一个例外：“F988反射率使用场景内的反射率面板进行校准：该波段超出了DAQ光传感器的校准范围，因此Chloros

会应用您最近的面板捕获数据，并在两次面板观测之间保持该值。” 参见 [校准目标](calibration-targets.md)。

</details>

<details>

<summary>我需要 DAQ 光传感器吗？</summary>

对于辐射度而言不需要：LATTICE 辐射度导出数据源自每台相机的出厂辐射校准，既不需要 DAQ 传感器，也不需要校准靶标。对于 **反射率**，您需要一个环境光的参考——可以是 DAQ 光传感器测得的下行辐射值，也可以是画面内的校准靶标。 DAQ传感器可让您**无需在场景中放置任何面板**即可生成校准后的反射率。记录的`.daq`文件会根据时间戳自动与您的图像进行匹配。 请参阅 [校准靶](calibration-targets.md) 和 [CLI

参考文档](reference/cli-reference.md)。

</details>

<details>

<summary>我可以将 Chloros 与 AI 助手（如 Claude、ChatGPT 等）配合使用吗？</summary>

可以——本手册以及 CLI/SDK 都是为此设计的：

* 完整的手册索引托管在 `https://mapir.gitbook.io/chloros/llms.txt`，以便 AI 助手能够检索到每一页。
* 每个页面的原始 Markdown 代码可通过其小写页面 URL 并附加 `.md` 获取（例如 `https://mapir.gitbook.io/chloros/reference/cli-reference.md`）。
* [CLI参考](reference/cli-reference.md) 和 [SDK参考](reference/sdk-reference.md) 专为大型语言模型（LLM）设计：包含精确的参数、默认值、退出语义以及可直接复制粘贴的命令。

有关如何将您的助手指向 Chloros，请参阅 [AI 助手](ai-assistants.md)。

</details>

<details>

<summary>处理后的输出文件保存在哪里？</summary>

生成文件保存在项目文件夹下，按相机分组，再按文件格式分类：

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **相机文件夹** — LATTICE 格式为 `LATT-<sensor>-<lens>-F<filter>`，Survey3 格式为 `<model>_<filter>`（例如 `Survey3N_RGN`）
* **格式文件夹** — `tiff16`、`tiff8`、`png8`、 `jpg8` 或 `tiff32`
* **产品文件夹** — `Reflectance_Calibrated_Images/`、`Debayered_Images/`、 `Preview_Images/`、`Radiance_Images/`（始终位于 `tiff32` 之下）、`<INDEX>_Index_Images/`
* **`clamp_masks/`** — 仅限 LATTICE 多光谱数据：产品文件夹旁有逐像素掩膜，详见下文**导出文件保留源文件名 — 项目文件夹用于标识产品，而非文件名后缀。**使用CLI时，项目文件夹将创建在您的默认项目文件夹下（`Documents/MAPIR/Chloros Projects`，除非您已更改），除非您传入参数`-o`； 相对路径 `-o` 则以您运行命令的文件夹为基准。 请注意，如果运行 `chloros-cli process` 时请求了产品但未生成任何输出，则会打印 `Processing finished but wrote no image products.` 并**以非零状态退出**，因此脚本可以检测到此情况。 请参阅 [输出图像格式](output-image-formats.md) 和 [CLI 参考](reference/cli-reference.md)。

</details>

<details>

<summary>clamp_masks 文件夹和 *_clampmask.tif 文件是什么？</summary>

1.3.0 版本新增功能。多光谱 LATTICE M3C 相机的辐射度和反射率产品会进行 3×3 波段解混，当某个波段产生负值时，Chloros 会将该值裁剪为零。 如果某个产品中至少有一个像素被裁剪，则会生成一个单波段 `uint8` 掩膜，用于标记哪些像素和波段被裁剪（位 0 = 产品的 R 通道，位 1 = G，位 2 = B）。 掩码文件保存在产品文件夹**旁边的** `clamp_masks/` 文件夹中——例如 `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif` ——而该产品的 `Chloros:UnmixClampMask` XMP 标签会标明其掩码名称（或直接标注为 `none`）。

它们并非场景的图像：Chloros 自身的导入功能会跳过它们，而且由于它们从未位于产品文件夹内，因此您可以将产品文件夹原样提供给 Pix4D 或 Metashape。参见 [夹具遮罩](output-image-formats.md#clamp-masks)。

</details>

<details>

<summary>为什么Chloros会提示“该项目文件夹层级过深，不适用于这台计算机”？</summary>

此问题仅出现在Windows系统中。Windows默认禁用长路径功能，因此任何文件路径的长度均不得超过260个字符。 Chloros 产品通常位于项目文件夹下方约 130 个字符处，因此 Chloros 在处理前会拒绝更深的项目文件夹——而不是像 1.3.0 版本之前那样运行却什么也不写。该消息会显示最长的符合要求的项目文件夹路径。

可通过以下任一方式解决：

* 使用更短的项目文件夹（在“项目设置”中设置的**工作目录**，或CLI中使用 `-o` / `chloros-cli set-project-folder`）。
* 启用 Windows 长路径并重启 Chloros。Chloros 安装程序中的 **启用 Windows 长路径** 选项（默认勾选）会实现此功能；若您曾取消勾选该选项，请重新运行安装程序。

请参阅 [开始处理](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer)。

</details>

<details>

<summary>在Chloros中处理图像前，我可以编辑图像吗？</summary>

不可以。Chloros 默认假设输入数据未被修改。请勿更改文件名。

</details>

<details>

<summary>我可以将MAPIR Survey3相机设置为自动曝光，然后在Chloros中处理图像吗？</summary>

不可以。Survey3图像数据集必须采用固定/锁定的曝光参数，因此不允许使用自动快门速度或自动ISO。同一相机型号的所有图像必须具有相同的快门速度和ISO（曝光）值。

LATTICE相机没有此限制：Chloros会实时控制其曝光（智能自动曝光），且每次拍摄都会记录实际使用的曝光值和增益，辐射校正处理流程会对此进行补偿。

</details>

<details>

<summary>Chloros能否处理或分析正射镶嵌图像？</summary>

不可以。Chloros仅支持MAPIR相机的单张图像，不支持正射镶嵌图等拼接图像。

</details>

<details>

<summary>如何加快Chloros的目标检测步骤？</summary>

在文件浏览器表格中，预先在右侧列中选中目标图像，将指示Chloros仅在这些图像中查找校准目标，从而大大加快处理速度。

</details>

<details>

<summary>如果我要将图像上传到 <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud</a>，是否应该在上传前先在 Chloros 中进行处理？</summary>

如果您计划将图像上传至我们的在线处理平台 [MAPIR

Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription)，请勿在上传前编辑图像。Cloud 平台将执行所有相同的处理步骤，甚至更多。

</details>

<details>

<summary>MAPIR将来会支持X功能吗？我真的很希望MAPIR能提供X功能。</summary>

我们始终乐于听取您对产品的反馈。如果您发现产品存在问题，或对如何改进产品有任何建议，请[联系我们](https://www.mapir.camera/community/contact)分享您的想法。 我们的大部分研发工作都以倾听客户最迫切的需求为导向。

</details>

<details>

<summary>Chloros 是否支持 Linux？</summary>

是的！Chloros 1.3.0 通过 `.deb` 软件包支持 Linux amd64（x86_64）和 arm64（NVIDIA Jetson JetPack 6）。 CLI 和 Python SDK 在 Linux 上得到全面支持，包括对 LATTICE 摄像头和 DAQ 传感器的实时控制。Linux 版本不提供图形界面（GUI）——所有交互均通过 [CLI](CLI.md) 或 [Python SDK](api-python-sdk.md) 进行。 详情请参阅 [Linux 概述](linux/linux-overview.md)。

</details>

<details>

<summary>我可以在 NVIDIA Jetson 上运行 Chloros 吗？</summary>

可以！Chloros 支持运行 JetPack 6 的 NVIDIA Jetson **Orin 系列**——包括 Orin Nano、Orin NX 和 AGX Orin。 较旧的机型（Jetson Nano、TX2、Xavier NX）无法运行 JetPack 6，且不受当前软件包支持。Chloros 会自动检测您的 Jetson 机型并优化其处理策略。 Jetson 软件包使用 JetPack 6 的 CUDA 版本 PyTorch，因此处理将在 Jetson 的 GPU 上运行。有关设置和部署说明，请参阅 [NVIDIA Jetson 指南](linux/nvidia-jetson-guide.md)。

</details>

<details>

<summary>Chloros 会自动针对我的硬件进行优化吗？</summary>

是的！Chloros 包含 [动态计算适配](processing-architecture/dynamic-compute-adaptation.md) 功能，可自动检测您的 CPU、GPU、内存以及（在 Jetson 上）温度传感器。随后，它会选择最佳的处理策略 ——从适用于高内存 GPU 的 `GPU_PARALLEL`，到适用于较小内存 GPU 的 `GPU_SINGLE`，再到适用于没有 PyTorch 可用的 CUDA GPU 的系统的 `CPU_PARALLEL`。 无需手动配置。

</details>

<details>

<summary>什么是 4 线程处理管道？</summary>

Chloros
为 Chloros
+ 用户采用了 4 线程流水线架构： 线程 1（检测）加载图像并检测校准目标；线程 2（校准）计算反射率校准；线程 3（处理）执行 GPU 加速的去拜耳化及索引计算；线程 4（导出）写入输出文件。 多张图像可同时在不同线程中处理，以实现最大吞吐量。详情请参阅 [处理管道](processing-architecture/processing-pipeline.md)。

</details>

<details>

<summary>如何对我的 Chloros 安装进行诊断？</summary>

使用 `selftest` 命令运行 7 步烟雾测试：版本、端口可用性、后端启动、API 连接性 (`/api/test`)、系统信息 (`/api/system-info` — GPU/CUDA/PyTorch)、去噪模型是否存在，以及 CUDA + 去噪模型就绪状态：

```bash
chloros-cli selftest
```

这在 Linux 系统上特别有用，可用于验证 GPU 和 CUDA 的配置。在 Jetson 上，`CUDA: False` 通常表示缺少 JetPack 6 的 CUDA 和 cuDNN 库：请使用 `sudo apt install nvidia-jetpack` 安装它们。

</details>
