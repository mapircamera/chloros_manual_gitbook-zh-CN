# Linux 概述

Chloros 1.1.0 版本为 **CLI**和**Python SDK**，支持在 Linux 工作站、服务器和 NVIDIA Jetson 边缘设备上进行无头多光谱图像处理。

{% hint style="info" %}
**Linux 上不提供图形用户界面。** Chloros 桌面图形用户界面仅在 Windows 上可用。 Linux用户可通过[CLI](../CLI.md)和[Python SDK](../api-python-sdk.md) 进行交互。
{% endhint %}

***

## 平台支持矩阵

| 功能 | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **桌面图形界面** | 是 | 不适用 | 否 | 否 |
| **CLI** | 是 | 是 | 是 | 是 |
| **Python SDK** | 是 | 是 | 是 | 是 |
| **GPU 加速 (CUDA)** | 是 | 是 | 是 | 是 (JetPack 6) |
| **纹理感知去拜耳化** | 是 (Chloros+) | 是 (Chloros+) | 是 (Chloros+) | 是 (Chloros+) |
| **动态计算适配** | 是 | 是 | 是 | 是 |***

## 支持的架构

| 架构 | 描述 | 安装方法 |
| --- | --- | --- |
| **amd64 (x86_64)** | 标准桌面/服务器处理器（Intel、AMD） | `.deb` 软件包 |
| **arm64 (aarch64)** | 基于 ARM 的处理器，主要为 NVIDIA Jetson | `.deb` 软件包 (JetPack 6) |

## 支持的 Linux 发行版

* **Ubuntu 20.04+** (amd64)
* **Debian 11+** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson 平台)***

## Linux 用户可获得的功能

* **Chloros CLI** — 用于批处理、自动化和脚本编写的完整命令行界面
* **Chloros Python SDK** — 用于集成到研究管道和自定义工具中的编程接口 (Python)
* **GPU 加速** — 在 NVIDIA GPU（台式机和 Jetson）上通过 CUDA 加速处理
* **动态计算自适应** — 自动硬件检测与处理策略优化
* **全部处理功能** — 与 Windows 相同的多光谱处理管道（校准、暗角校正、植被指数、所有导出格式）
* **Chloros+ 功能** — 多线程处理、纹理感知去拜耳化、自定义指数（需 Chloros+ 许可证）

## Linux 用户无法获得的功能

* **桌面图形界面** — 无图形界面；所有交互均通过 CLI 或 Python SDK 实现
* **图像查看器** — 无交互式图像查看器、网格视图或地图标记
* **可视化项目管理** — 项目通过 CLI 命令和 SDK 调用进行管理***

## Linux 入门指南

1. **安装 Chloros** — 有关 `.deb` 软件包的安装，请参阅 [Linux 安装](linux-installation.md)
2. **安装 Python 和 SDK**（可选） — `pip install chloros-sdk`
3. **激活许可证** — `chloros-cli login your@email.com 'password'`
4. **处理您的首个数据集** — `chloros-cli process ~/datasets/flight001`

对于 NVIDIA Jetson 用户，请参阅专门的 [NVIDIA Jetson 指南](nvidia-jetson-guide.md)，了解平台特定的设置和优化方法。

***

## 后续步骤

* [Linux 安装](linux-installation.md) — amd64 和 arm64 的详细安装说明
* [NVIDIA Jetson 指南](nvidia-jetson-guide.md) —— 针对 Jetson 的特定设置、热管理及现场部署
* [CLI : 命令行](../CLI.md) —— 完整的 CLI 参考文档
* [API : Python SDK](../api-python-sdk.md) — 完整的 SDK 参考文档
* [动态计算适配](../processing-architecture/dynamic-compute-adaptation.md) — Chloros 如何适配您的硬件
