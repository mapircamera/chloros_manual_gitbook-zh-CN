# Linux 安装

Chloros 面向 Linux 发布的 `.deb` 软件包，用于安装 CLI 及后端服务器。Python（SDK）是一个独立的 pip 包（也作为版本匹配的 wheel 文件捆绑在 `.deb` 中）。

软件包文件名包含版本和架构信息：`chloros_1.2.0_amd64.deb` 对应 x86_64 架构，`chloros_1.2.0_arm64_jp6.deb` 对应 JetPack 6 Jetson 构建版本。请在以下命令中替换为实际下载的文件。

***

## Linux amd64 (x86_64)

### 系统要求

| 要求 | 最低 | 推荐 |
| --- | --- | --- |
| **发行版** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **处理器** | x86_64（Intel/AMD） | Intel Core i7 或更高 |
| **内存（RAM）** | 8GB | 16GB 或更多 |
| **显卡** | 无（由 CPU 处理） | 配备 4GB 及以上显存的 NVIDIA GPU（12GB 及以上可解锁 `GPU_PARALLEL`，7GB 及以上可避免“Texture Aware”功能在单图像路径中被禁用） |
| **存储空间** | 2GB可用空间 | 10GB及以上可用空间的SSD |
| **Python** | Python 3.7及以上（适用于SDK） | Python 3.10及以上 |

> **不支持 Ubuntu 20.04 和 Debian 11。** `.deb` 的依赖项列表
> 源自 Chloros 后端实际链接的库，其中包括
> `libc6 (>= 2.34)`。Focal 和 bullseye 均预装 glibc 2.31，因此 `apt` 会直接拒绝
> 安装，而非在后续运行时导致安装失败。

### 安装

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
`dpkg -i` 无法解析依赖关系。 如果报告缺少软件包，`sudo apt-get install -f`（或 `sudo apt --fix-broken install`）将完成安装——这是正常流程，并非错误。
{% endhint %}

验证安装：

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->***

## Linux arm64 (NVIDIA Jetson)

### 系统要求

| 要求 | 最低 | 推荐 |
| --- | --- | --- |
| **平台** | 搭载 JetPack 6 的 NVIDIA Jetson | Jetson Orin NX 16GB 或 AGX Orin |
| **JetPack** | JetPack 6.x | 最新版 JetPack 6 |
| **内存 (RAM)** | 8GB（GPU/CPU 共享） | 16GB+ 共享（12GB+ 是并行 GPU 工作进程的阈值） |
| **存储空间** | 2GB 可用空间 | 10GB+ 可用空间的 NVMe SSD |
| **库（Python）** | Python 3.7+（适用于SDK） | Python 3.10+ |

### 安装

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

与 amd64 版本 `.deb` 布局相同，其中 CUDA 构建版本已针对 Jetson Orin / Orin NX / Orin Nano 进行了优化。 有关 Jetson 的内存、散热及现场部署行为，请参阅 [NVIDIA Jetson 指南](nvidia-jetson-guide.md)。

***

## Python SDK 安装（所有 Linux）

SDK是后端的一个纯PythonHTTP客户端，因此同一软件包在amd64和arm64架构上均可运行。有两种获取方式：

**从PyPI获取** — 已发布的稳定版本：

```bash
pip install chloros-sdk
```

**来自捆绑的 wheel 包** —— 保证与您刚刚安装的 CLI/backend 版本匹配（当您的 `.deb` 版本比 PyPI 上发布的更新时，请使用此版本）：

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**PEP 668 发行版**（Ubuntu 23.10+、Debian 12+）不允许全局 pip 安装。请使用 `pip install --user …`、虚拟环境或 `sudo pip install --break-system-packages …`。 软件包安装程序绝不会自动将SDK安装到您的系统中 Python —— 该选择权由您自行决定。
{% endhint %}

可选附加组件：

| 附加组件 | 命令 | 添加内容 |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py`（用于实时进度流） |
| `camera` | `pip install chloros-sdk[camera]` | `bleak`（用于 BLE (DAQ-M) 传输） |

验证SDK：

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
`.deb`会安装Chloros CLI 及其后端。 Python SDK 通过本地 HTTP API（`http://127.0.0.1:5000`）与该后端通信，并在需要时自动启动它。请始终使用字面上的 IPv4 地址，而非 `localhost` ——因为 `localhost` 可能解析为 `::1`，且每次请求大约需要两秒钟。
{% endhint %}

***

## 首次设置

### 1. 登录

访问 CLI 和 SDK 需要付费的 Chloros+ 层级（**Copper** 或更高），该限制在服务器端强制执行：未登录的调用者将获得 `401 AUTH_REQUIRED`，而免费层级（Iron）的调用者将获得 `403 PLAN_UPGRADE_REQUIRED`。

```bash
chloros-cli login your@email.com 'your-password'
```

凭据缓存于 `~/.chloros/user_session.json` 中。

{% hint style="warning" %}
**每次安装或升级后，您都必须重新登录。** 该软件包的 `prerm` 脚本会刻意清除 `~/.chloros/user_session.json` 以及该机器上所有用户的缓存许可证，因此新版本安装后会重新验证许可证，而非依赖过期的缓存。
{% endhint %}

### 2. 检查许可证状态

```bash
chloros-cli status
```

`chloros-cli status` 适用于任何服务层级（包括免费版），因此您可以随时了解访问受限或可用的具体原因。

### 3. 运行系统诊断

```bash
chloros-cli selftest
```

系统将按顺序执行七项检查，若其中任何一项失败，该命令将返回非零退出代码：

| # | 检查项 | 验证内容 |
| --- | --- | --- |
| 1 | **版本** | CLI 报告其版本（`v1.2.0`）。 |
| 2 | **端口可用** | 端口 5000 处于空闲状态，*或*已被运行正常的 Chloros 后端占用（这视为通过）。 |
| 3 | **后端启动** | 后端二进制文件启动。 |
| 4 | **API 测试 (`/api/test`)** | 后端响应 `status: ok`。 |
| 5 | **系统信息** | 从 `/api/system-info` 输出 `GPU: <name>, CUDA: <bool>, PyTorch: <version>`。 |
| 6 | **去噪模型** | 找到 `*.pth.enc` 模型（在 Linux 上：`/usr/lib/chloros/models`）。 |
| 7 | **CUDA + 去噪器**| “纹理感知”功能实际上可用——需要 CUDA**且**至少一个模型文件。 |

运行以 `N/7 checks passed` 结束，并按名称列出所有失败项。

### 4. 处理您的第一个数据集

```bash
chloros-cli process ~/datasets/flight001
```

***

## 文件和目录

### 每个用户的

Chloros将其凭据和CLI配置保存在一个跨平台目录中，即**`~/.chloros/`**（在Windows上为`%USERPROFILE%\.chloros\`）。另外两个Linux专用的缓存则遵循XDG规范——这些缓存会在设置时遵循`XDG_CONFIG_HOME` / `XDG_CACHE_HOME`的规则。

| 路径 | 用途 |
| --- | --- |
| `~/.chloros/user_session.json` | 由 `chloros-cli login` 写入的登录会话缓存（每次安装/升级软件包时都会清空） |
| `~/.chloros/working_directory.txt` | 默认项目文件夹覆盖（`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`） |
| `~/.chloros/cli_language.json` | CLI 语言偏好设置（`chloros-cli language <code>`） |
| `~/.chloros/user.json` | 与Windows图形用户界面共享的语言设置——此处的`language`优先于`cli_language.json` |
| `~/.chloros/update_cache.json` | Linux/Jetson 启动更新检查的一小时缓存 |
| `~/.chloros/backend.log` | 后端由 CLI 启动时的后端日志 |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | 按序列号和捆绑包哈希值索引的、每台相机对应的 LATTICE 校准包缓存 |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | DAQ 盖片校正配置文件的可选用户覆盖设置 |
| `~/.config/chloros/system_config.json` | 来自动态计算自适应（DCA）的缓存硬件配置文件——删除它可强制进行新的硬件检测 |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | 后端服务器日志，每次启动生成一个文件 |
| `~/Chloros Projects/` | 未设置覆盖项时的默认项目文件夹 |

### 全局设置

| 路径 | 用途 |
| --- | --- |
| `/usr/bin/chloros-cli` | 封装脚本 — 为打包的原生库设置 `LD_LIBRARY_PATH`，然后执行实际二进制文件 |
| `/usr/bin/chloros-backend` | 封装脚本 — 同上，此外还设置 `CHLOROS_PRODUCTION=1`，以确保后端认证闸门绝不会在用户不知情的情况下自行禁用 |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | 编译后的二进制文件 |
| `/usr/lib/chloros/arena_runtime/` | LATTICE 相机所需的 Arena SDK 运行时 |
| `/usr/lib/chloros/models/*.pth.enc` | 纹理感知去拜耳算法所使用的加密去噪模型 |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | 与该具体构建版本完全匹配的 Python SDK 轮 |
| `/usr/lib/chloros/exiftool` | 捆绑的 exiftool（仅在系统中不存在 exiftool 时，才会通过符号链接指向 `/usr/local/bin/exiftool`） |
| `/etc/chloros/update.conf` | 由 `chloros-cli update` 读取的更新通道配置 |
| `/etc/sysctl.d/60-chloros-ptp.conf` | 设置 `net.ipv4.ip_unprivileged_port_start = 319`，以便后端可在无需 root 权限的情况下绑定 PTP 端口 |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | 将动态加载器指向 `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | 授予已登录用户访问 DAQ-U USB 串行桥接器（CP2102N，`10c4:ea60`）的权限 |
| `/lib/systemd/system/chloros-backend.service` | 启用始终运行的后端服务（已安装，**未启用**） |
| `/usr/share/applications/chloros-cli.desktop` | “Chloros CLI” 应用程序菜单项，用于打开终端 |

## 后端可执行文件位置

CLI和SDK会自动检测后端：

| 组件 | 路径 |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| 后端 | `/usr/lib/chloros/chloros-backend` |

可通过 CLI 标志（`--backend-exe`）或 SDK 构造函数参数（`backend_exe`）覆盖后端路径，并通过 `--port` （默认值为 `5000`）。

{% hint style="info" %}
`CHLOROS_BACKEND_URL` 将 **`lattice`**、**`project`**和**`daq pool-*`** 命令系列，这些命令位于远程后端。核心命令 （`process`、`login`、`logout`、`status`、 `export-status`、`time-sync`、`selftest`) 会刻意忽略该参数，并始终将目标指向 `http://127.0.0.1:<port>`。
{% endhint %}

***

## Linux 上的 LATTICE 相机和 DAQ 光传感器

所有 live-hardware 命令系列均可在 Linux（amd64 和 Jetson）上运行：

* **`chloros-cli lattice`** — 发现、连接、配置 LATTICE 相机和同步阵列，并从其捕获数据。`.deb` 捆绑了它们所需的 Arena SDK 运行时，并将其注册到动态加载器中。
* **`chloros-cli daq pool-*`** — 通过后端池连接 DAQ-U/M/E 光传感器，流式传输已校准的光谱，并记录 `.daq` 文件。 编译后的CLI仅包含`pool-*`系列：`pool-connect`、`pool-disconnect`、 `pool-list`、`pool-latest`、`pool-stream`、`pool-record`、 `pool-set-cap`。
* **`chloros-cli project`** — 无头模式下运行已保存的项目（包括其中的摄像头、传感器和处理设置）。
* **`chloros-cli time-sync`** — 检查Chloros后端为LATTICE相机和DAQ-E传感器运行的PTP主时钟。

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` 是 `pool-latest` 的必备条件， `pool-stream`、`pool-record` 以及 `pool-set-cap` 所必需；`pool-list` 显示了当前池中的 ID。

{% hint style="info" %}
**在多网卡机器上首次连接 DAQ-E 时，建议优先使用 `--eth-host`。** 自动发现会扫描 mDNS，但可能因 ARP 缓存为空而未能检测到传感器的接口，因此即使传感器状态完全正常，启动后的首次 `pool-connect --eth` 连接也可能失败。直接传入传感器的 IP 地址或主机名可完全跳过发现过程。
{% endhint %}

**DAQ-U 串行权限** 由已安装的 udev 规则处理（`uaccess` + 组 `dialout`）。如果已插入的传感器仍无法访问，请重新加载规则或将其重新插入：

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

完整的命令集请参阅 [CLI 参考文档](../CLI.md)。

### 无显示器主机的常开 PTP

首次安装时，系统会生成 systemd 单元 `chloros-backend.service`，但该单元**未启用**。 在需要为 DAQ-E 传感器和 LATTICE 摄像头持续保持 PTP 时间同步的无显示器 Jetson 或服务器上，请启用该单元：

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

若未启用该单元，PTP 仅在 Chloros 后端运行时才工作——即在 CLI / SDK 会话处于活动状态期间。

该单元将后端绑定到 `127.0.0.1:5000`（单元内部的 `CHLOROS_HOST` / `CHLOROS_PORT` 环境设置； 可通过`sudo systemctl edit chloros-backend.service`进行覆盖），并在发生故障后5秒内重启该后端。

**PTP如何获取端口。** PTP使用UDP 319/320，这两个端口均低于常规1024特权端口下限。 该软件包的 `postinst` 会将 `/etc/sysctl.d/60-chloros-ptp.conf` 写入 `net.ipv4.ip_unprivileged_port_start = 319`，从而允许后端在以您的用户身份运行时绑定这些端口。 此外，作为双重保险的措施，它还会将 `setcap cap_net_bind_service,cap_net_raw=+ep` 应用于后端二进制文件——这就是为什么 `libcap2-bin` 被声明为该软件包的依赖项。***

## Bash 脚本示例

{% hint style="info" %}
**适合脚本使用的退出代码。**`chloros-cli process` 在成功时返回 `0`，**失败时返回非零值——包括请求了图像产品但未生成任何产品的运行情况** （此时会输出 `Processing finished but wrote no image products.`，并列出项目文件夹名称及常见原因）。成功运行时会报告写入的图像产品数量（`Image products written: N`）。 退出代码：`0` 表示成功，`1` 表示失败，`2` 表示参数错误，`130` 表示运行中断。
{% endhint %}

### 处理多个数据集

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### 使用自定义设置进行处理

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

有效的 `--format` 值共有四个，且包含空格——请务必用引号括起来：

| `--format` 值 | 输出文件夹 |
| --- | --- |
| `TIFF (16-bit)` *(默认)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` 接受 `standard`（默认）或 `texture-aware`（Chloros+）。

### 使用 Cron 进行自动化处理

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK 示例

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## 故障排除

### 安装后找不到CLI

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### 权限被拒绝

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### 安装过程中出现“setcap failed”错误

`.deb` 将 `cap_net_bind_service` 应用到 `/usr/lib/chloros/chloros-backend`，以便其无需 root 权限即可绑定 PTP 端口 319/320。 如果安装时缺少 `libcap2-bin`，则该调用将被跳过。请安装该组件并重新安装软件包：

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### PTP 无法启动 / 无法绑定端口 319

确认非特权端口下限已降低；若未降低，请针对当前启动重新应用该设置：

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

然后检查主控器：

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### “未找到 LATTICE 相机驱动程序”

Arena SDK 运行时无法解析。请确认该软件包生成的加载器配置文件是否存在且已刷新：

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
ls /usr/lib/chloros/arena_runtime | head
```

### 后端启动失败

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

有关启动失败的后端日志位于 `~/.cache/chloros/logs/` 中。

### 未检测到 CUDA

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

`chloros-cli selftest` 在一行中报告了相同的问题：`GPU: <name>, CUDA: <bool>, PyTorch: <version>`。

### 缺少共享库

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### SD 卡系统启动缓慢

编译后的二进制文件会在每次启动时自动解压到临时目录中。如果存在 `/mnt/ssd/tmp`，Chloros 会自动使用它；否则请将 `TMPDIR` 设置为一个高速文件系统：

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## 在Linux上更新Chloros

`update`命令仅适用于Linux/Jetson。 该命令会检查在 `/etc/chloros/update.conf` 处配置的更新通道中发布的版本，并提供下载和安装匹配的 `.deb`：

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

在 Linux /Jetson 上，CLI 还会在每次启动时执行非阻塞式更新检查（结果在 `~/.chloros/update_cache.json` 中缓存一小时），并在存在更新版时输出 `Update available: vX.Y.Z`。 您的设置和项目在更新后仍会保留；更新完成后，您需要重新登录。

## 卸载

```bash
sudo apt remove chloros
```

卸载操作将停止 `chloros-backend.service`，恢复默认的非特权端口下限（1024），删除捆绑的 exiftool 符号链接和 Arena 加载器配置，并清除缓存的凭据。 您的项目和 `~/.chloros/` 数据文件将保持不变。

***

## 后续步骤

* [NVIDIA Jetson 指南](nvidia-jetson-guide.md) — 针对 Jetson 的优化与部署
* [CLI：命令行](../CLI.md) —— CLI 指南
* [API：Python SDK](../api-python-sdk.md) —— SDK 指南
* [CLI 参考](../reference/cli-reference.md) 和 [SDK 参考](../reference/sdk-reference.md) —— 1.2.0 版本的完整命令/API列表
* [动态计算适配](../processing-architecture/dynamic-compute-adaptation.md) —— 说明 Chloros 如何适配您的硬件
