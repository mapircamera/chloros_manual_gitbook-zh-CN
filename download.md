---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# 下载

下载 Chloros 的最新版本，开始进行多光谱图像处理。

### 系统要求

#### Windows

| 要求                        | 最低配置                                              | 推荐配置                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **操作系统** | Windows 10（64 位）                                  | Windows 11（64 位）                                  |
| **处理器**        | Intel Core i5 或同等性能                          | Intel Core i7 或更高                              |
| **内存 (RAM)**     | 8GB                                                  | 16GB 或更高                                         |
| **显卡**    | 兼容 DirectX 11                                | NVIDIA 显卡，配备 4GB+ 显存                            |
| **存储空间**          | 6GB 可用空间                                       | SSD，配备 10GB+ 可用空间                            |
| **显示器**          | 1920x1080                                            | 2560x1440 或更高                                  |
| **互联网**         | 用于 \[可选] Chloros+ 许可证激活                        | 用于 \[可选] Chloros+ 许可证激活                        |

#### Linux amd64 (x86\_64)

| 要求                        | 最低配置                    | 推荐配置               |
| ----------------- | -------------------------- | ------------------------- |
| **发行版**  | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+             |
| **处理器**     | x86_64 (Intel/AMD)        | Intel Core i7 或更高   |
| **内存 (RAM)**  | 8GB                        | 16GB 或更多              |
| **显卡**        | 无（CPU 处理）      | 配备 4GB+ 显存的 NVIDIA GPU |
| **存储空间**    | 2GB 可用空间             | 配备 10GB+ 可用空间的 SSD       |
| **Python**        | Python 3.7+（适用于 SDK）      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| 要求      | 最低要求                      | 推荐配置                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **平台**     | 搭载 JetPack 6 的 NVIDIA Jetson | Jetson Orin NX 16GB 或 AGX Orin |
| **内存 (RAM)** | 8GB（GPU/CPU 共享）         | 16GB+ 共享                    |
| **存储空间**      | 2GB 可用空间               | 10GB+ 可用空间的 NVMe SSD        |
| **Python**       | Python 3.7+（适用于 SDK）        | Python 3.10+                    |

{% hint style="info" %}
**GPU 加速**：配备 NVIDIA GPU 的 Chloros+ 用户可使用 CUDA 加速，显著提升处理速度。此功能同时适用于 Windows（桌面 GPU）和 Linux（桌面 GPU 及 NVIDIA Jetson）。 Chloros+ 用户还可通过多线程处理实现最高速度。
{% endhint %}

***

## 下载 Chloros

### 最新稳定版（2026年3月23日）：版本 1.1.0

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">下载适用于 Windows 的 Chloros（.exe）</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">下载适用于 Linux amd64 的 Chloros (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">下载适用于 Linux arm64 / Jetson 的 Chloros (.deb)</a>

#### Windows 安装程序（GUI + CLI + 后端）

* **文件类型**：.exe（Windows 安装程序）**安装步骤：**

1. 下载上述 .exe 文件
2. 双击安装程序开始安装
3. 按照安装向导的提示操作
4. 选择安装目录（默认：`C:\Program Files\[USER]\Chloros\`）
5. 完成安装并启动 Chloros 或 Chloros CLI
6. 使用您的 [MAPIR Cloud Chloros+ 账户](https://cloud.mapir.camera/pricing) 登录（或继续使用免费版本）

{% hint style="success" %}
安装程序会自动将 `chloros-cli` 添加到系统 PATH 中，以便通过命令行访问。
{% endhint %}

#### Linux amd64 (.deb 软件包 — CLI + 后端)

* **文件类型**：.deb（Debian/Ubuntu 软件包）
* **架构**：x86_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (.deb 软件包 — CLI + 后端)

* **文件类型**: .deb (JetPack 6)
* **架构**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

有关详细的设置说明，请参阅 [Linux 安装指南](linux/linux-installation.md)；有关 Jetson 的具体指导，请参阅 [NVIDIA Jetson 指南](linux/nvidia-jetson-guide.md)。

#### Python SDK (所有平台)

```bash
pip install chloros-sdk
```

请参阅 [API : Python SDK](api-python-sdk.md) 获取文档。

{% hint style="info" %}
**Linux 用户**：`.deb` 包会安装 CLI 及后端。Python 和 SDK 需通过 pip 单独安装。 Linux 没有图形界面（GUI）——所有交互均通过 CLI 或 SDK 进行。
{% endhint %}

***

## 其他资源

### Python SDK

对于开发人员和自动化工作流，请安装 Chloros Python SDK：

```bash
pip install chloros-sdk
```

**文档**：[API: Python SDK](api-python-sdk.md)**系统要求**：必须已安装 Chloros（通过 Windows 安装程序或 Linux `.deb` 软件包），且需使用 Chloros+ 许可证登录***

## 包含内容

### Windows 安装程序

* ✅ **Chloros 图形界面** - 功能齐全的图形界面
* ✅ **Chloros CLI** - 命令行界面（需 Chloros+ 许可证）
* ✅ **Chloros 后端** - 处理引擎
* ✅ **相机配置文件** - 预配置的 MAPIR 相机模板

### Linux .deb 软件包

* ✅ **Chloros CLI** - 命令行界面 （需 Chloros+ 许可证）
* ✅ **Chloros 后端** - 处理引擎
* ✅ **相机配置文件** - 预配置的 MAPIR 相机模板
* ❌ 无图形界面 — Linux 仅支持无头模式 CLI/SDK

### Python SDK (画中画，所有平台)

* ✅ **Chloros SDK** - Python API（需 Chloros+ 许可证）***

## 升级至 Chloros+

订阅 Chloros+ 以解锁高级功能：

* 🚀 **多线程处理** - 并行处理图像
* ⚡ **GPU (CUDA) 加速** - 利用 NVIDIA GPU 性能
* 💻 **CLI 访问** - 通过命令行工具实现自动化
* 🐍 **Python SDK** - 程序化访问 API
* 📱 **多设备支持** - 可在 2 至 10 台以上设备上使用（视套餐而定）
* **🐻 高级纹理感知去拜耳化方法** - 结合 AI/ML 降噪模型的高质量边缘感知去拜耳化技术，可消除几乎所有的去拜耳化噪点。
* 🧮 **自定义公式** - 创建自定义多光谱指数

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">查看 Chloros+ 套餐与定价</a></p>***

## 安装帮助

### 故障排除

**安装失败并显示错误信息：**

* 确保您拥有管理员权限
* 暂时禁用杀毒软件
* 检查是否满足最低系统要求

**应用程序无法启动 (Windows)：**

* 确认已安装 Windows 10/11（64 位）
* 更新显卡驱动程序
* 检查 Windows 事件查看器中的错误详情
* 附上错误日志联系支持团队

**CLI 无法启动 (Linux)：**

* 确认 `.deb` 软件包已正确安装：`dpkg -l | grep chloros`
* 检查权限：`sudo chmod +x /usr/bin/chloros-cli`
* 运行诊断：`chloros-cli selftest`
* 检查是否缺少库文件：`ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**许可证激活问题：**

* 确保网络连接正常
* 在 [https://cloud.mapir.camera](https://cloud.mapir.camera) 处验证凭据
* 检查防火墙是否阻止了 Chloros
* 请参阅 [Chloros+ 登录](chloros+-login.md) 获取详细说明

### 获取支持

需要安装或设置方面的帮助吗？

* 📧 **电子邮件**：info@mapir.camera
* 🌐 **网站**：[https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **文档**：[入门指南](./)
* ❓ **常见问题**：[常见问题解答](faq.md)***

## 更新日志

<details>

<summary>版本 1.1.0 (最新)</summary>

**发布日期：2026年3月**

**新功能*** **Linux 支持** — 针对 Linux amd64 (x86_64) 和 arm64 (NVIDIA Jetson JetPack 6) 提供原生 CLI 和 SDK 支持。 通过 `.deb` 软件包进行安装。
* **NVIDIA Jetson 支持** — 针对 Jetson Nano、Orin Nano、Orin NX 和 AGX Orin 边缘设备进行了优化处理。
* **动态计算适配** — 自动硬件检测与处理策略优化。Chloros 可适配从 Jetson Nano 到多 GPU 工作站的各类硬件。
* **4 线程处理管道** — 支持检测、校准、处理和导出线程的并行运行，并具备动态 GPU 内存分配功能。
* **新增 CLI 命令** — `selftest`（系统诊断）和 `update`（Linux 更新管理）。
* **新增 CLI 进程标志** — `--debayer`（标准/纹理感知）、`--indices`（指定索引）、`--target`（优先搜索目标子文件夹以加快检测速度）。
* **新增 GUI 菜单项** — 现在可通过主菜单下拉菜单访问“添加文件”、“添加文件夹”以及“开始/停止处理”功能。**改进**

* 跨平台后端自动检测（Windows 和 Linux 路径）
* 增强了 SDK 和 `get_status()`，支持按线程跟踪进度
* 新增 SDK 异常：`ChlorosConfigurationError`、`ChlorosAuthenticationError`
* 针对 NVIDIA Jetson 的热管理与自适应限速
* 自动内存管理，内存不足时回退至分块 GPU 处理

</details>

<details>

<summary>版本 1.0.5</summary>

**发布日期：2026年2月10日**

**新功能*** **纹理感知去拜耳化方法 \[仅限 Chloros+] -** 纹理感知采用高质量的边缘感知去拜耳化算法，结合 AI/ML 降噪模型，可消除几乎所有的去拜耳化噪声。
* **支持 T4P 校准目标*** **Chloros+ GPU 处理速度更快，内存管理更优**

**错误修复*** 全新前端（GUI），现应可在所有 Windows 计算机上运行。

</details>

<details>

<summary>版本 1.0.4</summary>

**发布日期：2026年1月5日**

**新功能*** **图像/元数据切换**：在文件浏览器中新增切换功能，可将选定图像的元数据以表格形式显示，而非图像网格
* **图像网格缩放滑块**：新增UI滑块，用于调整缩略图大小 （也支持 CTRL + 鼠标滚轮）
* **图像网格导出按钮**：顶部行中的按钮可将缩略图从 JPG 切换为处理后的导出格式（目标、反射率、指数、LUT）
* **地图标签页**：新增交互式 2D 地图，显示图像的 GPS 位置标记
  * 支持 Google 地图和 ESRI 地图瓦片（根据可用缩放级别自动选择最佳瓦片服务）
  * 鼠标悬停时可在地图标记上预览缩略图

**错误修复*** 改进了对非英语系统安装 Chloros 的支持

</details>

<details>

<summary>版本 1.0.3</summary>

**发布日期：2025年12月20日**

**新功能*** 首次发布

**改进*** 首次发布

**错误修复*** 首次发布

**已知问题*** 首次发布

</details>***

## 许可协议**专有软件** - 版权所有 (c) 2026 MAPIR Inc.

禁止未经授权的使用、分发或修改。

**免费版**：可供个人及商业使用，但功能受限**Chloros+**：订阅制许可，提供高级功能及商业部署支持
