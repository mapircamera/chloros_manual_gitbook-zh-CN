---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# 常见问题

<details>

<summary>我可以使用 Chloros 处理非 MAPIR 品牌的摄像头图像吗？</summary>

不可以，Chloros仅支持处理MAPIR相机的图像。 请参阅[支持的相机型号列表](supported-cameras.md)以获取更多信息。我们确实在MAPIR云端提供其他相机的图像处理服务，完整列表请见[此处](https://mapir.gitbook.io/mapir-cloud/supported-cameras)。

</details>

<details>

<summary>如果没有校准靶，我能对图像进行反射率校准吗？</summary>

不可以。如果在拍摄非校准目标图像时未同时拍摄校准目标图像，您将无法将图像的像素值与已知的反射率百分比建立关联。此外，若未包含来自 MAPIR 光传感器的日志数据，则无法测量环境光谱，反射率结果也将不准确。

</details>

<details>

<summary>在 Chloros 中处理前，我可以编辑图像吗？</summary>

不可以。Chloros 默认输入数据未被修改。请勿更改文件名。

</details>

<details>

<summary>我可以将 MAPIR 和 Survey3 相机设置为自动曝光，然后在 Chloros 中处理图像吗？</summary>

不可以。Survey3 图像数据集必须采用固定/锁定的曝光设置，因此不支持自动快门速度或自动 ISO。同一型号相机的所有图像必须具有相同的快门速度和 ISO（曝光）。

</details>

<details>

<summary>Chloros能否处理或分析正射镶嵌图像？</summary>

不可以。仅支持单独的 MAPIR 相机图像，不支持正射镶嵌图等拼接图像。

</details>

<details>

<summary>如何加快 Chloros 的目标检测步骤？</summary>

在文件浏览器表格中，预先在右侧列中选中目标图像，将指示 Chloros 仅在这些图像中查找校准目标，从而大大加快处理速度。

</details>

<details>

<summary>如果我要将图像上传到 <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud</a>，是否应在上传前使用 Chloros 进行处理？</summary>

如果您计划上传至我们的在线处理平台 [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription)，请勿在上传前编辑图像。Cloud 平台将执行所有相同的处理步骤，并提供更多功能。

</details>

<details>

<summary>MAPIR 将来会支持 X 功能吗？我真的很希望 MAPIR 能提供 X 功能。</summary>

我们始终乐于听取您对产品的反馈。如果您发现产品存在问题，或有改进建议，请通过 [联系我们](https://www.mapir.camera/community/contact) 分享您的想法。我们大部分的研发工作都以倾听客户最迫切的需求为导向。

</details>

<details>

<summary>Chloros 是否支持 Linux？</summary>

是的！Chloros 1.1.0 通过 `.deb` 软件包支持 Linux amd64（x86_64）和 arm64（NVIDIA Jetson JetPack 6）。 CLI 和 Python SDK 在 Linux 上得到全面支持。 Linux 没有图形用户界面（GUI）——所有交互均通过 [CLI](CLI.md) 或 [Python SDK](api-python-sdk.md)。详情请参阅 [Linux 概述](linux/linux-overview.md)。

</details>

<details>

<summary>我可以在 NVIDIA Jetson 上运行 Chloros 吗？</summary>

可以！Chloros 1.1.0 支持运行 JetPack 6 的 NVIDIA Jetson 平台，包括 Jetson Nano、Orin Nano、Orin NX 和 AGX Orin。Chloros 会自动检测您的 Jetson 型号并优化其处理策略。 有关设置和部署说明，请参阅 [NVIDIA Jetson 指南](linux/nvidia-jetson-guide.md)。

</details>

<details>

<summary>Chloros 会自动针对我的硬件进行优化吗？</summary>

是的！Chloros 1.1.0 包含 [动态计算自适应](processing-architecture/dynamic-compute-adaptation.md) 功能，可自动检测您的 CPU、GPU、RAM 以及（在 Jetson 上）温度传感器。 随后，它将选择最佳的处理策略——从高内存系统上的 `GPU_PARALLEL`，到资源受限设备上的 `GPU_SINGLE`，再到不带 NVIDIA GPU 的系统上的 `CPU_PARALLEL`。无需手动配置。

</details>

<details>

<summary>什么是 4 线程处理管道？</summary>

Chloros 1.1.0 为 Chloros+ 用户采用了 4 线程流水线架构： 线程 1（检测）加载图像并检测校准目标，线程 2（校准）计算反射率校准，线程 3（处理）执行 GPU 加速的去拜耳化及索引计算，线程 4（导出）写入输出文件。 为实现最大吞吐量，多个图像可同时在不同线程中处理。详情请参阅 [处理管道](processing-architecture/processing-pipeline.md)。

</details>

<details>

<summary>如何对我的 Chloros 安装运行诊断？</summary>

使用 `selftest` 命令运行 7 项系统诊断，包括版本检查、端口可用性、后端启动、API 连接性、系统信息、去噪模型以及 CUDA 可用性：

```bash
chloros-cli selftest
```

这在 Linux/Jetson 系统上特别有用，可用于验证 GPU 和 CUDA 的配置。

</details>
