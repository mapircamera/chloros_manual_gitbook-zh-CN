# Linux 概述

Chloros 1.2.0 为 **CLI**和**Python SDK** ——无头多光谱图像处理，以及 LATTICE 相机和 DAQ 光传感器实时控制——在 Linux 工作站、服务器和 NVIDIA Jetson 边缘设备上提供原生支持。

{% hint style="info" %}
**Linux 上不提供桌面图形用户界面。**Chloros 桌面图形用户界面仅适用于 Windows。 Linux用户可通过[CLI](../CLI.md) 和 [Python SDK](../api-python-sdk.md) 与 Chloros 进行交互。 `.deb` 确实会在您的应用程序菜单中添加一个**Chloros CLI** 条目——它只是打开一个运行 `chloros-cli` 的终端模拟器。
{% endhint %}

***

## 平台支持对照表

| 功能 | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **桌面图形界面** | 是 | 不适用 | 否 | 否 |
| **CLI** (`chloros-cli`) | 是 | 是 | 是 | 是 |
| **Python SDK** (`chloros-sdk`) | 是 | 是 | 是 | 是 |
| **图像处理管道** | 是 | 是 | 是 | 是 |
| **LATTICE 相机控制（实时）** | 是（“相机”选项卡） | 是（`chloros-cli lattice`、SDK） | 是 | 是 |
| **DAQ 光传感器（实时）** | 是（“光传感器”选项卡） | 是（`chloros-cli daq pool-*`、SDK） | 是 | 是 |
| **PTP 时间同步（主机为主时钟）** | 是 | 是（`chloros-cli time-sync`） | 是 | 是 |
| **GPU 加速（CUDA）** | 是 | 是 | 是 | 是（JetPack 6） |
| **纹理感知去拜耳算法** | 是 (Chloros+) | 是 (Chloros+) | 是 (Chloros+) | 是 (Chloros+) |
| **动态计算自适应** | 是 | 是 | 是 | 是 |
| **后端作为系统服务** (`chloros-backend.service`) | 否 | 否 | 是（需手动启用） | 是（需手动启用） |
| **就地更新程序** (`chloros-cli update`) | 否（运行安装程序） | 否（运行安装程序） | 是 | 是 |***

## 支持的架构

| 架构 | 描述 | 软件包 |
| --- | --- | --- |
| **amd64 (x86_64)** | 标准桌面/服务器处理器（Intel、AMD） | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | ARM 处理器 — NVIDIA Jetson Orin 系列 | `chloros_<version>_arm64_jp6.deb` (JetPack 6 构建版) |

## 支持的 Linux 发行版

* **Ubuntu 22.04 LTS 或更新版本** (amd64)
* **Debian 12 或更新版本** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson Orin 平台)***

## Linux 用户可获得的功能

* **Chloros CLI** — 用于批处理、自动化和脚本编写的完整命令行界面
* **Chloros Python SDK** — 用于研究流程和自定义工具的 Python 编程接口（可从 PyPI 安装，也作为版本匹配的 wheel 包捆绑在 `.deb` 中）
* **LATTICE 相机控制** — 通过 `chloros-cli lattice` 和 SDK 发现、连接、配置并从 LATTICE 相机及同步的多相机阵列中捕获图像； `.deb` 打包了相机所需的 Arena SDK 运行时
* **DAQ 光传感器控制** — 通过 `chloros-cli daq pool-*` 和 SDK 连接 DAQ-U/M/E 传感器、流式传输校准光谱并记录 `.daq` 文件
* **PTP 时间同步** — Chloros 后端运行 PTP 主时钟，LATTICE 相机和 DAQ-E 传感器均以此为主时钟进行同步；可通过 `chloros-cli time-sync` 进行检查， 并使用 `chloros-backend.service` systemd 单元使其以无头模式持续运行（参见 [Linux 安装指南](linux-installation.md#always-on-ptp-for-headless-hosts))
* **项目自动化** — 使用 `chloros-cli project` 以及 SDK 中的 `open_project` 无头运行已保存的项目
* **GPU 加速** — 在 NVIDIA GPU（台式机和 Jetson）上进行 CUDA 加速处理
* **动态计算自适应** — 自动检测硬件并选择处理策略，同时提供 `CHLOROS_STRATEGY` 作为专家级应急方案
* **所有处理功能** — 与 Windows 采用相同的处理管道：校准、暗角校正、植被指数计算以及所有导出格式
* **Chloros+ 功能** —— 多线程（流水线）处理、纹理感知去拜耳算法以及自定义指数，需订阅付费的 Chloros+ 计划

## Linux 用户无法获得的功能

* **桌面图形界面** — 无图形界面；所有交互均通过 CLI 或 Python SDK 进行
* **图像查看器** — 没有交互式图像查看器、网格视图或地图标记
* **可视化项目管理** — 项目通过 CLI 命令和 SDK 调用创建和驱动（硬件本身——摄像头、传感器、采集设备——仍可通过终端完全控制）***

## 许可要求

访问 CLI 和 SDK 功能需拥有 **付费 Chloros+ 层级 — 铜级或更高**（铜级、青铜级、银级、金级）。 免费的**Iron** 套餐不支持访问 CLI/SDK。该限制由后端强制执行，而不仅仅是 CLI：

| 情况 | 后端响应 |
| --- | --- |
| 未登录 | `401` 并附带 `error_code: AUTH_REQUIRED` |
| 在免费 Iron 层登录 | 包含 `401` 和 `error_code: AUTH_REQUIRED` |

`chloros-cli status` 在任何层级均可运行——这是唯一不受网关限制的路径——因此拒绝原因始终可见。

***

## Linux 入门指南

1. **安装 Chloros** —— 有关 `.deb` 的安装，请参阅 [Linux 安装指南](linux-installation.md)
2. **验证** — `chloros-cli --version` 会打印 `Chloros CLI 1.2.0`；`chloros-cli selftest` 会运行 7 步诊断
3. **安装 Python 和 SDK**（可选） — `pip install chloros-sdk`
4. **登录** — `chloros-cli login your@email.com 'your-password'`（每台机器仅需登录一次，每次软件包升级后需重新登录）
5. **处理您的首个数据集** — `chloros-cli process ~/datasets/flight001`

对于 NVIDIA Jetson，请参阅专门的 [NVIDIA Jetson 指南](nvidia-jetson-guide.md)，了解平台特定的设置、散热行为和现场部署信息。

***

## 后续步骤

* [Linux 安装指南](linux-installation.md) — 针对 amd64 和 arm64 的详细安装说明、文件位置及故障排除
* [NVIDIA Jetson 指南](nvidia-jetson-guide.md) —— 针对 Jetson 的特定设置、内存和散热特性以及现场部署
* [CLI：命令行](../CLI.md) —— CLI 指南
* [API： Python SDK](../api-python-sdk.md) — SDK指南
* [CLI 参考](../reference/cli-reference.md) 和 [SDK 参考](../reference/sdk-reference.md) — 1.2.0 版本的完整命令/API 列表
* [动态计算适配](../processing-architecture/dynamic-compute-adaptation.md) —— Chloros 如何适配您的硬件

{% hint style="info" %}
**通过编程方式阅读本手册。** 每页内容均以原始 Markdown 格式发布在各自的 URL 及 `.md` （例如 `https://mapir.gitbook.io/chloros/linux/linux-installation.md`）上以原始 Markdown 格式提供，且整本手册的索引发布在 [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt)。
{% endhint %}
