---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# 下载

下载最新版本的 Chloros，开始进行多光谱图像处理。

### 系统要求

#### Windows


| 要求          | 最低配置                                              | 推荐配置                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **操作系统** | Windows 10（64 位）                                  | Windows 11（64 位）                                  |
| **处理器**        | Intel Core i5 或同等性能                          | Intel Core i7 或更高                              |
| **内存 (RAM)**     | 8GB                                                  | 16GB 或更多                                         |
| **显卡**    | 兼容 DirectX 11                                | 配备 4GB 及以上显存的 NVIDIA 显卡                            |
| **存储空间**          | 6GB 可用空间                                       | 配备 10GB 及以上可用空间的 SSD                            |
| **显示器**          | 1920x1080                                            | 2560x1440 或更高                                  |
| **网络连接**         | 用于\[可选\]Chloros+许可证激活                                | 用于\[可选\]Chloros+许可证激活                                |

#### Linux amd64 (x86_64)

| 要求             | 最低配置                    | 推荐配置               |
| ----------------- | -------------------------- | ------------------------- |
| **发行版**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **处理器**     | x86\_64 (Intel/AMD)        | Intel Core i7 或更高   |
| **内存 (RAM)**  | 8GB                        | 16GB 或更高              |
| **显卡**        | 无（由 CPU 处理）      | 配备 4GB+ 显存的 NVIDIA GPU |
| **存储空间**       | 2GB 可用空间             | 10GB+ 可用空间的 SSD       |
| **Python**        | Python 3.7+（用于 SDK）      | Python 3.10+              |

#### Linux arm64（NVIDIA Jetson）

| 要求      | 最低要求                      | 推荐配置                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **平台**     | 搭载 JetPack 6 的 NVIDIA Jetson | Jetson Orin NX 16GB 或 AGX Orin |
| **内存 (RAM)** | 8GB（GPU/CPU 共享）         | 16GB+ 共享                    |
| **存储空间**      | 2GB 可用空间               | 10GB+ 可用空间的 NVMe SSD        |
| **Python**       | Python 3.7+（用于 SDK）        | Python 3.10+                    |

{% hint style="info" %}
**GPU 加速**：配备 NVIDIA GPU 的 Chloros+ 用户可使用 CUDA 加速，从而显著提升处理速度。该功能支持 Windows 和 Linux amd64 系统上的 NVIDIA 台式机 GPU，以及搭载 JetPack 6 的 NVIDIA Jetson 设备。Chloros+ 用户还可通过多线程处理实现最高速度。
{% endhint %}

***

## 下载 Chloros


### 最新稳定版：1.3.0 版（2026年11月11日）

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">下载适用于 Windows 的 Chloros（.exe）</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">下载适用于 Linux amd64 的 Chloros（.deb）</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">下载适用于 Linux arm64 / Jetson 的 Chloros（.deb）</a>


#### Windows 安装程序（GUI + CLI + 后端）

* **文件类型**：.exe（Windows 安装程序）

**安装步骤：**

1. 下载上述 .exe 文件
2. 双击安装程序开始安装
3. 按照安装向导的提示操作
4. 选择安装目录（默认：`C:\Program Files\Chloros\`）
5. 在 **“选择其他任务”**页面上，请保持两个**“系统设置”** 选项均被勾选（推荐 — 详见下文）
6. 完成安装并启动 Chloros 或 Chloros CLI
7. 使用您的 [MAPIR Cloud Chloros+ 账户](https://cloud.mapir.camera/pricing) 登录 （或继续使用免费版本）

**系统设置选项**（默认均已勾选）：

| 安装程序中的选项名称 | 功能说明 | 是否退出静默安装 |
| --- | --- | --- |
| **启用Windows 长路径**（允许项目文件夹路径长度超过260个字符；推荐） | 将 Windows 的全局设置 `LongPathsEnabled` 设为 1。Windows 默认禁用此设置，此时任何文件路径长度均不得超过 260 个字符；Chloros 产品通常位于项目文件夹下方约 130 个字符处，因此若禁用长路径功能，系统将拒绝更深层的项目文件夹。 该设置对安装后启动的程序生效。即使卸载后，该设置仍会保持启用状态，因为其他程序可能依赖此设置。 | `/MERGETASKS="!longpaths"` |
| **将Chloros 后端排除在Microsoft Defender扫描范围之外**（相机捕获速度大幅提升；推荐） | 在安装文件夹中为`chloros-backend.exe`添加Microsoft Defender进程排除项，从而确保实时保护不会扫描后端写入的每个文件，无论其写入位置在哪里。 在高捕获速率下，扫描开销约占捕获速率的 30%（在 9 台摄像头连续捕获时，每秒捕获帧数分别为 77.7 帧和 52.7 帧）。 不会排除其他程序，且该排除规则会在卸载时被移除。若系统未安装 Defender 或 Defender 被策略锁定，此步骤将静默失败，安装将继续进行。 | `/MERGETASKS="!defenderexclusion"` |

静默安装（`/SILENT` 或 `/VERYSILENT`）将同时应用这两项选项，除非您使用所示的 `/MERGETASKS` 值选择退出。

{% hint style="success" %}
安装程序会将CLI 文件夹（`C:\Program Files\Chloros\cli\`）添加到您的系统中一次——重新安装或升级时不再追加重复条目，并会移除先前安装程序添加的重复条目。 安装完成后，请打开一个新的终端以使用 `chloros-cli`。安装程序还会在每个网络配置文件中为后端添加两条 Windows 防火墙规则： **Chloros 后端 (UDP-In)**，用于摄像头流、PTP、mDNS 以及 DAQ-E 数据流；以及**Chloros 后端 (TCP-In)**，仅限于本地子网，用于 DAQ-E 固件更新——在此过程中，传感器会连接回这台计算机以采集图像。 卸载时，这两条规则均会被移除。
{% endhint %}

#### Linux amd64（.deb 软件包 — CLI + 后端）

* **文件类型**：.deb（Debian/Ubuntu 软件包）
* **架构**：x86_64 (amd64)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

#### Linux arm64 — NVIDIA Jetson (.deb 软件包 — CLI + 后端)

* **文件类型**：.deb（JetPack 6）
* **架构**：aarch64（arm64）

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

请使用您实际下载的文件名，并保留 `./`，以便 `apt` 安装该文件，而不是按名称查找软件包。使用 `apt install` 进行安装， 而非 `dpkg -i`：apt 还会安装 Python SDK 的依赖项，而该软件包仅推荐而非强制要求这些依赖，且 `dpkg -i` 会跳过这些依赖。 自 1.3.0 版本起，后端和 CLI 已解压安装在 `/usr/lib/chloros/` 目录下，因此它们启动时无需先自行解压。

Jetson 软件包使用 JetPack 6 的 CUDA 版本 PyTorch，因此处理过程（包括内置的 Texture Aware 去拜耳算法）将在 Jetson 的 GPU 上运行。 它使用 JetPack 6 安装的 CUDA 和 cuDNN 库（在缺少这些库的镜像中为 `sudo apt install nvidia-jetpack`）。

有关详细的设置说明，请参阅 [Linux 安装指南](linux/linux-installation.md)；有关 Jetson 的具体指导，请参阅 [NVIDIA Jetson 指南](linux/nvidia-jetson-guide.md)。

#### Python SDK（所有平台）

每个安装程序都捆绑了一个对应的 `chloros_sdk` wheel 包，因此 SDK 版本始终与已安装的 GUI/CLI/后端版本一致。在 Windows 系统上，安装程序会自动将其安装到系统 Python 中。 在 Linux 系统上，`.deb` 会将 SDK 安装到系统 `python3` 本身中（位于 `/usr/lib/python3/dist-packages` 目录下）， 因此 `import chloros_sdk` 在安装完成后即可正常运行，无需 pip 也无需网络。只有虚拟环境（其无法访问系统包）才需要将捆绑的 wheel 安装到其中：

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

对于仅支持 pip 的主机（未安装 Chloros 包；此时 SDK 会驱动另一台机器上的后端），SDK 也可在 PyPI 上获取：

```bash
pip install chloros-sdk
```

请参阅 [API：Python SDK](api-python-sdk.md) 和 [SDK 参考](reference/sdk-reference.md) 以获取文档。

{% hint style="info" %}
**Linux 用户**：`.deb` 软件包会安装 CLI、后端以及 Python SDK。Linux 版本不提供图形界面（GUI）——所有交互均通过 CLI 或 SDK 进行。
{% endhint %}

***

## 其他资源

### Python SDK


针对开发人员和自动化工作流，每个安装程序均附带 Chloros Python SDK：Windows 安装程序会将其安装到您的 Python 环境中，而 Linux `.deb` 则会将其安装到系统 `python3` 中。 在未安装 Chloros 软件包的机器上（即在另一台机器上驱动后端的机器），请从 PyPI 安装：

```bash
pip install chloros-sdk
```

**文档**：[API：Python SDK](api-python-sdk.md)

**要求**：一个待驱动的 Chloros 后端（Windows 安装程序或 Linux `.deb` 软件包会安装一个），以及付费层级的 Chloros+ 登录账号

***

## 包含内容

### Windows 安装程序

* ✅ **Chloros GUI** - 功能齐全的图形用户界面
* ✅ **Chloros CLI** - 命令行界面（需要 Chloros+ 许可证）
* ✅ **Chloros 后端** - 处理引擎
* ✅ **Python SDK** - 若系统已安装 Python，则自动安装（需 Chloros+ 许可证）
* ✅ **摄像头配置文件** - 预配置的 MAPIR 摄像头模板

### Linux.deb 软件包

* ✅ **Chloros CLI** - 命令行界面（需要 Chloros+ 许可证）
* ✅ **Chloros 后端** - 处理引擎
* ✅ **Python SDK** - 已安装在系统 `python3` 上（需要 Chloros+ 许可证）
* ✅ **摄像头配置文件** - 预配置的 MAPIR 摄像头模板
* ❌ 无图形界面 — Linux 仅支持无头模式下的 CLI/SDK


### Python SDK（PyPI，适用于未安装 Chloros 软件包的机器）

* ✅ **Chloros SDK** - Python API（需要 Chloros+ 许可证）

***

## 升级至 Chloros+

订阅 Chloros+ 以解锁高级功能：

* 🚀 **多线程处理** - 并行处理图像
* ⚡ **GPU（CUDA）加速** - 利用 NVIDIA GPU 算力
* 💻 **命令行界面（CLI）访问** - 通过命令行工具实现自动化
* 🐍 **Python SDK** - 通过编程方式访问 API

* 📱 **多设备支持** - 可在 2 至 10 台及以上设备上使用（视套餐而定）
* **🐻 先进的纹理感知去拜耳算法** - 一种高质量的边缘感知去拜耳算法，结合 AI/ML 降噪模型，可消除几乎所有的去拜耳噪声。
* 🧮 **自定义公式** - 创建自定义多光谱指数

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">查看 Chloros+ 套餐与定价</a></p>***

## 安装帮助

### 故障排除

**安装失败并显示错误信息：**

* 确保您拥有管理员权限
* 暂时禁用杀毒软件
* 检查是否满足最低系统要求

**应用程序无法启动（Windows）：**

* 确认已安装 Windows 10/11（64 位）
* 更新显卡驱动程序
* 在 Windows 事件查看器中查看错误详情
* 附上错误日志联系技术支持

**CLI 无法启动（Linux）：**

* 确认 `.deb` 软件包已正确安装：`dpkg -l | grep chloros`
* 检查权限：`sudo chmod +x /usr/bin/chloros-cli`
* 运行诊断：`chloros-cli selftest`
* 检查是否缺少库文件：`ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"`（自 1.3.0 版本起，`/usr/lib/chloros/chloros-backend` 是一个小型启动脚本；程序本体位于 `/usr/lib/chloros/backend/` 中）

**许可证激活问题：**

* 确保网络连接正常
* 在 [https://cloud.mapir.camera](https://cloud.mapir.camera) 处验证凭据
* 检查防火墙是否阻挡了 Chloros

* 详细说明请参阅 [Chloros+ 登录](chloros+-login.md)

### 获取支持

需要安装或设置方面的帮助吗？

* 📧 **电子邮件**：info@mapir.camera
* 🌐 **网站**：[https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **文档**：[入门指南](./)
* ❓ **常见问题**：[常见问题解答](faq.md)***

## 软件更新

Chloros 会检查更新，并在有新版本可用时发出通知，并提供此下载页面的链接——您只需运行新的签名安装程序即可完成更新。您的设置、项目和登录信息在更新后仍会保留。 在 Linux 和 Jetson 系统上，`chloros-cli update` 会检查是否有更新版本，并提供下载和安装对应的 `.deb`（此命令仅限 Linux 系统）。

***

## 更新日志**版本 1.3.0（最新版，2026 年 11 月 11 日）**—— 完整列表请参阅 [入门指南](./) 页面中的**Chloros

1.3.0 新增内容**。**版本 1.2.0**— 请参阅 [入门指南](./) 页面中的**Chloros 1.2.0 新增功能**。

<details>

<summary>1.0.5 版</summary>

**发布日期：2026年2月10日**

**新功能*** **纹理感知去拜耳算法 \[仅限 Chloros+] -** 纹理感知算法采用高质量的边缘感知去拜耳算法，并结合 AI/ML 降噪模型，可消除几乎所有的去拜耳噪声。
* **支持 T4P 校准目标*** **Chloros+ GPU 处理速度更快，内存管理更优化**

**错误修复*** 全新前端（GUI），现应可在所有 Windows 计算机上正常运行。

</details>

<details>

<summary>版本 1.0.4</summary>

**发布日期：2026年1月5日**

**新功能*** **图像/元数据切换**：在文件浏览器中新增切换功能，可将所选图像的元数据以表格形式显示，而非图像网格形式
* **图像网格缩放滑块**：新增 UI 滑块，用于调整缩略图大小 （也支持 CTRL + 鼠标滚轮）
* **图像网格导出按钮**：顶部行中的按钮可将缩略图从 JPG 切换为处理后的导出格式（目标值、反射率、指数、LUT）
* **地图标签页**：新增交互式 2D 地图，显示图像的 GPS 位置标记
  * 支持 Google 地图和 ESRI 地图瓦片（根据可用缩放级别自动选择最佳瓦片服务）
  * 将鼠标悬停在地图标记上可预览缩略图

**错误修复*** 改进了在非英语系统上安装 Chloros 的支持

</details>

<details>

<summary>版本 1.0.3</summary>

**发布日期：2025年12月20日**

**新功能*** 首次发布

**改进*** 首次发布

**错误修复*** 首次发布

**已知问题*** 首次发布

</details>

***

## 许可协议**专有软件** - 版权所有 (c) 2026 MAPIR Inc.

禁止未经授权的使用、分发或修改。

**免费版**：可供个人和商业使用，但功能受限**Chloros+**：基于订阅的许可证，提供高级功能和商业部署支持
