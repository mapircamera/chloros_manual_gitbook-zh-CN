# Linux 概述

Chloros 1.3.0 为 **CLI**和**Python SDK** 提供了原生 Linux 支持——可在 Linux 工作站、服务器和 NVIDIA Jetson 边缘设备上进行无头多光谱图像处理，以及对 LATTICE 相机和 DAQ 光传感器的实时控制。

{% hint style="info" %}
**Linux 系统不提供桌面 GUI。**Chloros 桌面 GUI 仅限 Windows 系统使用。Linux 用户可通过 [CLI](../CLI.md) 和 [Python SDK](../api-python-sdk.md) 与 Chloros 进行交互。 `.deb` 确实会在您的应用程序菜单中添加一个**Chloros CLI** 条目——它只是打开一个运行 `chloros-cli` 的终端模拟器。
{% endhint %}

***

## 平台支持对照表

| 功能 | Windows（GUI） | Windows（CLI/SDK） | Linux amd64（CLI/SDK） | Linux arm64 / Jetson（CLI/SDK） |
| --- | --- | --- | --- | --- |
| **桌面 GUI** | 是 | 不适用 | 否 | 否 |
| **命令行界面 CLI** (`chloros-cli`) | 是 | 是 | 是 | 是 |
| **Python SDK** (`chloros-sdk`) | 是 | 是 | 是 | 是 |
| **图像处理管道** | 是 | 是 | 是 | 是 |
| **LATTICE 相机控制（实时）** | 是（“相机”选项卡） | 是 (`chloros-cli lattice`，SDK) | 是 | 是 |
| **DAQ 光传感器（实时）** | 是（“光传感器”选项卡） | 是 (`chloros-cli daq pool-*`，SDK) | 是 | 是 |
| **PTP 时间同步（主机为主时钟）** | 是 | 是（`chloros-cli time-sync`） | 是 | 是 |
| **GPU 加速（CUDA）** | 是 | 是 | 是 | 是（JetPack 6） |
| **纹理感知去拜耳算法**（需要 CUDA GPU） | 是（Chloros+） | 是（Chloros+） | 是（Chloros+） | 是（Chloros+） |
| **动态计算自适应** | 是 | 是 | 是 | 是 |
| **后端作为系统服务** (`chloros-backend@<user>.service`) | 否 | 否 | 是（需手动启用） | 是（需手动启用） |
| **就地更新程序** (`chloros-cli update`) | 不支持（需运行安装程序） | 不支持（需运行安装程序） | 是 | 是 |

***

## 支持的架构

| 架构 | 描述 | 软件包 |
| --- | --- | --- |
| **amd64 (x86_64)** | 标准桌面/服务器处理器（Intel、AMD） | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | ARM 处理器 — NVIDIA Jetson Orin 系列 (JetPack 6) | `chloros_<version>_arm64_jp6.deb` |

## 支持的Linux发行版

* **Ubuntu 22.04 LTS 或更高版本** (amd64)
* **Debian 12 或更高版本** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson Orin 平台)

***

## Linux 用户可获得的功能

* **Chloros CLI** — 用于批处理、自动化和脚本编写的完整命令行界面
* **Chloros Python SDK** — 用于研究管道和自定义工具的 Python 编程接口，由 `.deb` 安装到系统 `python3` 中 （`/usr/lib/chloros/sdk/`中包含适用于虚拟环境的版本匹配wheel包；PyPI适用于未安装Chloros包的机器）
* **LATTICE 相机控制** — 通过 `chloros-cli lattice` 和 SDK 发现、连接、配置并从 LATTICE 相机及同步的多相机阵列中捕获图像；`.deb` 捆绑了相机所需的 Arena SDK 运行时
* **DAQ光传感器控制** — 通过`chloros-cli daq pool-*`和SDK连接DAQ-U/M/E传感器、流式传输校准光谱并记录`.daq`文件
* **PTP 时间同步** — Chloros 后端运行 PTP 主时钟，LATTICE 相机和 DAQ-E 传感器以此为从； 可通过 `chloros-cli time-sync` 对其进行监控，并使用 `chloros-backend@<user>.service` systemd 单元以无头模式保持其运行（参见 [Linux 安装](linux-installation.md#always-on-ptp-for-headless-hosts))
* **项目自动化** — 使用 `chloros-cli project` 和 SDK 的 `open_project` 无头运行已保存的项目
* **GPU 加速** — 在 NVIDIA GPU 上进行 CUDA 加速处理（amd64 架构上的桌面 GPU，以及搭载 JetPack 6 的 Jetson GPU）
* **动态计算自适应** — 自动检测硬件并选择处理策略，同时提供 `CHLOROS_STRATEGY` 覆盖选项作为专家级应急方案
* **全部处理功能** — 与 Windows 版本相同的处理流程：校准、暗角校正、植被指数计算以及所有导出格式
* **Chloros+ 功能** —— 多线程（流水线）处理、基于纹理的去拜耳算法（在 CUDA GPU 上）以及自定义指数，需订阅付费版 Chloros+ 计划

## Linux用户无法获得的功能

* **桌面图形界面** —— 无图形界面；所有交互均通过CLI或Python SDK实现
* **图像查看器** —— 无交互式图像查看器、网格视图或地图标记
* **可视化项目管理** — 项目通过 CLI 命令和 SDK 调用创建和驱动（硬件本身——相机、传感器、采集设备——仍可通过终端完全控制）***

## 许可要求

访问 CLI 和 SDK 需要 **付费的 Chloros+ 套餐——Copper 及以上级别**（Copper、Bronze、Silver、Gold）。免费的**Iron** 套餐不提供 CLI/SDK 访问权限。此限制由后端强制执行，而不仅仅是 CLI：

| 情况 | 后端响应 |
| --- | --- |
| 未登录 | `401` 伴随 `error_code: AUTH_REQUIRED` |
| 在免费的铁级登录 | `403` 伴随 `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` 在任何层级均可正常工作——这是唯一不受门控限制的路由——因此拒绝的原因始终可见。

***

## 在 Linux

1. **安装 Chloros** —— 有关 `.deb` 的安装，请参阅 [Linux 安装指南](linux-installation.md)
2. **验证** — `chloros-cli --version` 会输出 `Chloros CLI 1.3.0`；`chloros-cli selftest` 会运行 7 步诊断
3. **检查 Python SDK** —— `.deb` 会安装它：`python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"`（在虚拟环境中，`pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`）
4. **登录** — `chloros-cli login your@email.com 'your-password'`（每台机器仅需登录一次；软件包升级会保留登录信息，卸载软件包则会清除登录信息）
5. **处理您的第一个数据集** — `chloros-cli process ~/datasets/flight001`

对于 NVIDIA Jetson，请参阅专门的 [NVIDIA Jetson 指南](nvidia-jetson-guide.md)，了解平台特定的设置、散热行为和现场部署信息。

***

## 后续步骤

* [Linux 安装](linux-installation.md) —— 针对 amd64 和 arm64 的详细安装说明、文件位置及故障排除
* [NVIDIA Jetson 指南](nvidia-jetson-guide.md) — 针对 Jetson 的特定设置、内存与热行为以及现场部署
* [CLI：命令行](../CLI.md) — CLI 指南
* [API：Python SDK](../api-python-sdk.md) — SDK 指南
* [CLI参考](../reference/cli-reference.md) 和 [SDK参考](../reference/sdk-reference.md) — 1.3.0 版本的完整命令/API列表
* [动态计算适配](../processing-architecture/dynamic-compute-adaptation.md) —— Chloros 如何适配您的硬件

{% hint style="info" %}
**通过编程方式读取本手册。** 每页内容均以原始 Markdown 格式发布，其URL 由该页URL 加上 `.md` 组成（例如 `https://mapir.gitbook.io/chloros/linux/linux-installation.md`），且整本手册的索引发布在 [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt)。
{% endhint %}
