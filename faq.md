---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# 常见问题

<details>

<summary>我能否使用 Chloros 处理非 MAPIR 品牌的摄像头图像？</summary>

不可以，Chloros仅支持处理MAPIR摄像头的图像——即Survey3和LATTICE系列。 更多信息请参阅[支持的相机型号列表](supported-cameras.md)。 我们确实在 MAPIR Cloud 上提供其他摄像头的处理服务，完整列表请参见[此处](https://mapir.gitbook.io/mapir-cloud/supported-cameras)。

</details>

<details>

<summary>Chloros 是否支持 LATTICE 摄像头？</summary>

是的。Chloros 1.2.0 端到端支持 LATTICE M3C 和 M3M 摄像头模块：**实时控制**——可通过 GUI 的“摄像头”选项卡进行发现、连接、预览和捕获， `chloros-cli lattice` 或 Python SDK，包括采用 PTP 时间同步的同步多相机阵列——以及对捕获图像的**完整辐射度处理** （原始数据 → 去拜耳化 → 辐射度 → 反射率 → 指数）。请参阅 [支持的相机](supported-cameras.md) 和 [LATTICE 指南](lattice/README.md)。

</details>

<details>

<summary>如果没有校准靶，我能对图像进行反射率校准吗？</summary>

**Survey3：** 不行。如果在拍摄非校准目标图像时未同时拍摄校准目标图像，您将无法将图像的像素值与已知的反射率百分比建立关联。 此外，若未包含来自 MAPIR 光传感器的日志数据，则无法测量环境光谱，反射率结果也将不准确。**LATTICE：** 是的。反射率可以参照由 DAQ 光传感器（而非面板）测得的下行辐照度（ρ = π·L/E）。 当帧内存在通过质量保证（QA）检测的目标时，该目标将默认作为绝对参考（`--reflectance-source auto`）。 有一个例外：“F988 的反射率使用场景内的反射率面板进行校准：该波段超出了 DAQ 光传感器的校准范围，因此 Chloros 会应用您最近的面板捕获数据，并在两次面板观测之间保持该值。” 参见 [校准标靶](calibration-targets.md)。

</details>

<details>

<summary>我需要 DAQ 光传感器吗？</summary>

若不涉及辐射度：LATTICE 的辐射度导出数据源自每台相机的出厂辐射校准，既不需要 DAQ 光传感器，也不需要校准靶标。对于 **反射率**，您需要一个环境光的参考——可以是 DAQ 光传感器的下行辐射测量值，也可以是画面内的校准靶标。 DAQ传感器可让您**无需在场景中放置任何面板**即可生成校准后的反射率数据。记录的`.daq`文件会根据时间戳自动与您的图像进行匹配。 请参阅 [校准目标](calibration-targets.md) 和 [CLI 参考资料](reference/cli-reference.md)。

</details>

<details>

<summary>我可以将 Chloros 与 AI 助手（如 Claude、ChatGPT 等）配合使用吗？</summary>

可以——本手册以及 CLI/SDK 均为此设计：

* 完整的手册索引托管在 `https://mapir.gitbook.io/chloros/llms.txt`，以便 AI 助手发现每一页。
* 每页的原始 Markdown 代码均可在其小写页面 URL 后附加 `.md` 的链接中获取（例如 `https://mapir.gitbook.io/chloros/reference/cli-reference.md`）。
* [CLI 参考文档](reference/cli-reference.md) 和 [SDK 参考](reference/sdk-reference.md) 专为大型语言模型（LLM）设计：包含精确的标志、默认值、退出语义以及可复制粘贴的命令。

有关如何将您的助手指向 Chloros 的方法，请参阅 [AI 助手](ai-assistants.md)。

</details>

<details>

<summary>处理后的输出文件保存在哪里？</summary>

生成文件保存在项目文件夹下，按相机分类，再按文件格式分组：

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **相机文件夹** — LATTICE 相机对应 `LATT-<sensor>-<lens>-F<filter>`，Survey3 对应的文件格式文件夹为 `<model>_<filter>`（例如 `Survey3N_RGN`）
* **格式文件夹** — `tiff16`、`tiff8`、`png8`、 `jpg8` 或 `tiff32`
* **产品文件夹** — `Reflectance_Calibrated_Images/`、`Debayered_Images/`、 `Preview_Images/`、`Radiance_Images/`（始终位于 `tiff32` 之下）、`<INDEX>_Index_Images/`**导出文件保留源文件名——文件夹用于标识产品，而非文件名后缀。**对于 CLI，除非您传入 `-o`，否则项目文件夹将创建在输入文件夹旁边。 请注意，若运行 `chloros-cli process` 时请求了产品但未生成任何输出，则会输出 `Processing finished but wrote no image products.` 并**以非零状态退出**，因此脚本可以检测到此情况。 请参阅 [输出图像格式](output-image-formats.md) 和 [CLI 参考文档](reference/cli-reference.md)。

</details>

<details>

<summary>在 Chloros 中处理前，我可以编辑我的图像吗？</summary>

不可以。Chloros 假设输入数据未被修改。请勿更改文件名。

</details>

<details>

<summary>我可以将我的 MAPIR 和 Survey3 相机设置为自动曝光，然后在 Chloros 中处理图像吗？</summary>

不可以。Survey3图像数据集必须采用固定/锁定的曝光参数，因此不支持自动快门速度或自动ISO。同一型号相机的所有图像必须具有相同的快门速度和ISO（曝光）值。

LATTICE相机没有此限制：Chloros会实时控制其曝光（智能自动曝光），且每次拍摄都会记录实际使用的曝光值和增益，辐射测量处理流程会对此进行补偿。

</details>

<details>

<summary>Chloros 能否处理或分析正射镶嵌图像？</summary>

不可以。仅支持单独的 MAPIR 相机图像，不支持正射镶嵌图等拼接图像。

</details>

<details>

<summary>如何加快 Chloros 的目标检测步骤？</summary>

在文件浏览器表格中，预先在右侧列中选中目标图像，将指示 Chloros 仅在这些图像中查找校准目标，从而大大加快处理速度。

</details>

<details>

<summary>如果我要将图像上传到 <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud</a>，是否应该在上传前先在 Chloros 中进行处理？</summary>

如果您计划将图像上传至我们的在线处理平台 [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription)，请勿在上传前编辑图像。Cloud 平台将执行所有相同的处理操作，并提供更多功能。

</details>

<details>

<summary>MAPIR 将来会支持 X 功能吗？我真的很希望 MAPIR 能提供 X 功能。</summary>

我们始终乐于听取您对产品的反馈。如果您发现产品存在问题，或对如何改进产品有任何建议，请[联系我们](https://www.mapir.camera/community/contact)分享您的想法。 我们的大部分研发工作都以倾听客户最迫切的需求为指导。

</details>

<details>

<summary>Chloros 是否支持 Linux？</summary>

是的！Chloros 1.2.0 通过 `.deb` 软件包支持 Linux amd64 (x86_64) 和 arm64 （NVIDIA Jetson JetPack 6）的支持。 CLI 和 Python SDK 在 Linux 上得到全面支持，包括对 LATTICE 实时摄像头和 DAQ 传感器的控制。 Linux 没有图形用户界面 ——所有交互均通过 [CLI](CLI.md) 或 [Python SDK](api-python-sdk.md) 进行。详情请参阅 [Linux 概述](linux/linux-overview.md)。

</details>

<details>

<summary>我可以在 NVIDIA Jetson 上运行 Chloros 吗？</summary>

可以！Chloros 支持运行 JetPack 6 的 NVIDIA Jetson 平台，包括 Jetson Nano、Orin Nano、Orin NX 和 AGX Orin。Chloros 会自动检测您的 Jetson 型号并优化其处理策略。 有关设置和部署说明，请参阅 [NVIDIA Jetson 指南](linux/nvidia-jetson-guide.md)。

</details>

<details>

<summary>Chloros 会自动针对我的硬件进行优化吗？</summary>

是的！Chloros 包含 [动态计算适配](processing-architecture/dynamic-compute-adaptation.md) 功能，可自动检测您的 CPU、GPU、内存以及（在 Jetson 上）温度传感器。 随后，它会选择最佳的处理策略——从高内存系统上的 `GPU_PARALLEL`，到资源受限设备上的 `GPU_SINGLE`，再到不带 NVIDIA GPU 的系统上的 `CPU_PARALLEL`。 无需手动配置。

</details>

<details>

<summary>什么是 4 线程处理管道？</summary>

Chloros 为 Chloros+ 用户采用了 4 线程流水线架构： 线程 1（检测）加载图像并检测校准目标；线程 2（校准）计算反射率校准；线程 3（处理）执行 GPU 加速的去拜耳化及色差校正；线程 4（导出）写入输出文件。 多张图像可同时在不同线程中处理，以实现最大吞吐量。详情请参阅 [处理管道](processing-architecture/processing-pipeline.md)。

</details>

<details>

<summary>如何对我的 Chloros 安装运行诊断测试？</summary>

使用 `selftest` 命令运行 7 步烟雾测试：版本、端口可用性、后端启动、API 连接性（`/api/test`）、系统信息 （`/api/system-info` — GPU/CUDA/PyTorch）、去噪模型是否存在，以及 CUDA + 去噪模型就绪状态：

```bash
chloros-cli selftest
```

这在 Linux/Jetson 系统上特别有用，可用于验证 GPU 和 CUDA 的配置。

</details>
