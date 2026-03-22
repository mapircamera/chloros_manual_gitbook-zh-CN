---
metaLinks: {}
---

# 入门指南

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros 是由 [MAPIR](https://www.mapir.camera) 开发的一款用于处理图像及其他传感器数据的软件应用程序。

***{% hint style="success" %}**Chloros 1.1.0 的新功能**：原生 Linux 支持（amd64 和 arm64）、NVIDIA Jetson 边缘计算、动态计算自适应、4 线程处理管道、新的 CLI 命令和选项。 完整更新日志请参见 [下载](download.md)。
{% endhint %}

Chloros 提供 3 种应用模式：

## Chloros：桌面图形界面应用程序

具备所有功能的独立窗口。_仅限 Windows 系统。_

## [Chloros CLI：命令行界面](CLI.md)

命令行批处理。非常适合自动化、脚本编写和无头操作。 可在 **Windows、Linux amd64 及 Linux arm64 (NVIDIA Jetson)** 上使用。_使用 CLI 需持有 Chloros 或更高版本的许可证。_

## [Chloros API: Python SDK](api-python-sdk.md)

用于自动化和自定义工作流的 Python 编程接口。非常适合研究流程、与现有 Python 应用程序的集成以及构建自定义工具。可通过 `pip install chloros-sdk` 在 **所有平台** 上使用。 _访问该 API 需要 Chloros 或更高版本的许可证。_***

## 支持的平台

| 平台 | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11** | 是 | 是 | 是 |
| **Linux amd64 (x86_64)** | 否 | 是 | 是 |
| **Linux arm64 (NVIDIA Jetson)** | 否 | 是 | 是 |

有关 Linux 的安装说明，请参阅 [Linux 与边缘计算](linux/linux-overview.md) 部分。

***

## Chloros+

虽然 Chloros 对于大多数任务都是免费的，但您可能会发现需要更多功能。此时，购买 Chloros+ 的付费许可证将为您带来益处。 拥有 Chloros+ 许可证，您可以解锁以下新功能：

* **多线程处理**：通过管道同时处理图像，大幅加快大型项目的图像处理速度。
* **GPU（CUDA）加速**：利用当今更高规格的 GPU 显存选项，进一步加速图像处理管道。建议使用 4GB 或更大的显存以获得最佳效果。
* **Chloros+**[**CLI**](CLI.md)**访问**：通过命令行运行 Chloros+，以实现自动化并集成到您自己的软件中。
* **Chloros+**[**API**](api-python-sdk.md)**访问方式：**通过 Python 运行 Chloros+ 进行程序化控制，实现与您的研究流程、数据分析工作流及自定义应用程序的无缝集成。
* **多设备使用**：每份 Chloros+ 许可证支持注册 2 台及以上设备。使用您的 MAPIR 云账户管理已注册设备。通过升级 Chloros+ 许可证，可增加对更多设备的支持。
* **先进的纹理感知去拜耳化方法：** 高质量的边缘感知去拜耳化技术结合 AI/ML 降噪模型，可消除几乎所有的去拜耳化噪声。
* **自定义多光谱指数公式：** 在 Chloros 栅格计算器中输入自定义多光谱指数，适用于图像处理和图像查看沙盒。
* **Linux 与边缘计算：** 在 Linux x86_64 和 ARM64 平台（包括 NVIDIA Jetson）上运行 Chloros，以实现现场和边缘处理。 参见 [Linux 概述](linux/linux-overview.md)。

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ 定价与注册</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
