# Linux 安装

Chloros 在 Linux 上的发行包为 `.deb`，该包会安装 CLI、后端服务器以及 Python SDK（对于系统 `python3`，还包含一个与之版本匹配的 wheel 包，用于虚拟环境）。

软件包文件名包含版本号和架构信息：`chloros_1.3.0_amd64.deb` 对应 x86_64 架构， 而 `chloros_1.3.0_arm64_jp6.deb` 代表 arm64（NVIDIA Jetson、JetPack 6）。请在以下命令中替换为实际下载的文件名。

{% hint style="info" %}
**

1.3.0 版本新增：程序以未解压形式发布。** 后端和CLI将作为文件夹安装，即 `/usr/lib/chloros/backend/` 和 `/usr/lib/chloros/cli/`， 而非单个自解压文件。它们不再在每次启动时自动解压到临时文件夹中，因此启动速度大大提升（在 x86_64 平台上，后端以前每次启动都需要花费约 100 秒进行解压） ，也不再需要在 `/tmp` 目录中预留数 GB 的可用空间。amd64 版本的 `.deb` 大小也仅为以前的一半左右。
{% endhint %}

***

## Linux amd64 (x86_64)

### 系统要求

| 要求 | 最低 | 推荐 |
| --- | --- | --- |
| **发行版** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **处理器** | x86_64 (Intel/AMD) | Intel Core i7 或更高 |
| **内存 (RAM)** | 8GB | 16GB 或更多 |
| **显卡** | 无（由 CPU 处理） | 配备 4GB+ 显存的 NVIDIA GPU（12GB+ 可解锁 `GPU_PARALLEL`，7GB+ 可使“纹理感知”功能在单图像路径中保持关闭） |
| **存储空间** | 2GB可用空间 | 10GB+可用空间的SSD |
| **Python** | Python 3.7+（用于SDK） | Python 3.10+ |

> **不支持 Ubuntu 20.04 和 Debian 11。** `.deb` 的依赖项列表
> 源自 Chloros 后端实际链接的库，其中包括一个
> `libc6` 版本，该版本比 focal 和 bullseye 发行版中自带的 glibc 2.31 更新，因此 `apt` 会直接拒绝
> 安装，而非在后续运行时导致安装失败。

### 安装

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**请使用 `apt install ./…`，而非 `dpkg -i`。** `apt` 既能解决该软件包的依赖关系，也会安装其推荐的组件：Python SDK 的依赖项， `python3-numpy` 和 `python3-requests`。而 `dpkg -i` 则不安装这两者。 CLI和后端是自包含的，无需这些组件即可运行，但 `import chloros_sdk` 需要它们；安装输出会列出任何缺失的组件。请保留 `./`， 以便 `apt` 能直接安装该文件，而非搜索同名软件包。该软件包还声明了 `libgl1`，捆绑的 OpenCV 会加载该文件，而精简版服务器镜像可能不包含此文件。
{% endhint %}

验证安装：

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### 系统要求

| 要求 | 最低 | 推荐 |
| --- | --- | --- |
| **平台** | 搭载 JetPack 6 的 NVIDIA Jetson | Jetson Orin NX 16GB 或 AGX Orin |
| **JetPack** | JetPack 6.x | 最新版 JetPack 6 |
| **内存 (RAM)** | 8GB（GPU/CPU 共享） | 16GB+ 共享（12GB+ 是并行处理的阈值） |
| **存储** | 2GB可用空间 | 10GB+可用空间的NVMe SSD |
| **Python** | Python 3.7+（用于SDK） | Python 3.10+ |

### 安装

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

与 amd64 版本 `.deb`（包含 Python SDK、接收缓冲区和后端服务）布局相同，适用于运行 **JetPack 6**（L4T R36）的 Jetson Orin / Orin NX / Orin Nano。 其 PyTorch 基于 JetPack 6 为 Orin 构建的 CUDA 版本，因此 GPU 处理（包括 Texture Aware 去拜耳算法）将在 Jetson 的 GPU 上运行。 该软件包本身不包含 CUDA：它使用 JetPack 6 安装的 CUDA 和 cuDNN 库（在缺少这些库的镜像上，请使用 `sudo apt install nvidia-jetpack`）。 当无法使用 CUDA GPU 时，处理将在 CPU 上运行，且 `--debayer texture-aware` 将使用标准去拜耳算法（后端会记录一行 `[DEBAYER] … needs a CUDA GPU …` 日志说明原因）。有关 Jetson 的内存、散热及现场部署行为，请参阅 [NVIDIA Jetson 指南](nvidia-jetson-guide.md)。***

## Python SDK 安装（所有 Linux 系统）**该 `.deb` 会为您自动安装 SDK。** 两个软件包（amd64 和 Jetson）均会为系统安装该 SDK `python3`，因此 `import chloros_sdk` 在安装完成后即可直接使用，无需 pip 也无需网络。 其依赖包（`chloros_sdk`、`lattice_sdk`、`lattice_cam`、 `arena_api` 以及项目加载器）都位于 `/usr/lib/python3/dist-packages` 中，并归该软件包所有：升级时会替换这些文件，移除软件包时则会移除它们。 该版本始终与您安装的CLI和后端相匹配。

SDK自身的依赖项 `python3-numpy` 和 `python3-requests` 是 **推荐的，而非必需的**：`sudo apt install ./chloros_<version>_<arch>.deb` 会安装它们，而 `dpkg -i` 则不会。安装输出会报告 SDK 导入的版本 `python3`、任何缺失的依赖项， 以及 `python3` 路径中其他位置存在的、会覆盖该软件包副本的 `chloros_sdk`（例如之前通过 `pip install --user` 安装的版本）。**在虚拟环境**中（该环境无法访问系统中的包），请将捆绑的 wheel 包安装到其中（完全离线）：

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**从 PyPI 安装** —— 仅适用于未安装 Chloros 包的机器，且其 SDK 在另一台机器上驱动后端（参见 [远程后端模式](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)）：

```bash
pip install chloros-sdk
```

可选附加内容，适用于 pip 安装（虚拟环境或 PyPI）：

| 附加内容 | 命令 | 添加内容 |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py`（用于实时进度流） |
| `camera` | `pip install chloros-sdk[camera]` | 用于 BLE（DAQ-M）传输的 `bleak` |

验证 SDK：

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Python SDK 通过本地 HTTP API（`http://127.0.0.1:5000`）与 `.deb` 安装的后端进行通信。 `ChlorosLocal` 和连接辅助程序会在需要时自动启动它；而已保存项目的 `connect_all()` 则永远不会这样做， 因此在无头机器上，请将后端作为 [服务](#always-on-ptp-for-headless-hosts) 运行。请务必使用字面上的 IPv4 地址，而非 `localhost` —— `localhost` 可能解析为 `::1`，且每次请求大约需要两秒钟。
{% endhint %}

***

## 首次设置

### 1. 登录

访问 CLI 和 SDK 需要付费的 Chloros+ 套餐（**Copper** 或更高），该限制在服务器端强制执行：未登录的调用者将获得 `401 AUTH_REQUIRED`，而免费套餐（Iron）的调用者将获得 `403 PLAN_UPGRADE_REQUIRED`。

```bash
chloros-cli login your@email.com 'your-password'
```

凭据缓存于 `~/.chloros/user_session.json` 中。

{% hint style="info" %}
**升级后登录信息将保留；卸载则会清除登录信息。** 在现有安装（包括通过 `chloros-cli update` 进行的安装）上安装较新版本的 `.deb` 会保留 `~/.chloros/user_session.json` 及缓存的许可证，因此无界面 CLI/SDK 任务仍可正常运行。 卸载该软件包（`sudo apt remove chloros`）将清除该机器上所有用户的已保存登录信息和许可证缓存，因此全新安装时需要再次安装 `chloros-cli login`。
{% endhint %}

### 2. 检查许可证状态

```bash
chloros-cli status
```

`chloros-cli status` 适用于任何服务等级（包括免费版），因此您可以随时查看访问成功或失败的原因。

### 3. 运行系统诊断

```bash
chloros-cli selftest
```

系统将按顺序执行七项检查，若其中任何一项失败，命令将返回非零状态码：

| # | 检查项 | 验证内容 |
| --- | --- | --- |
| 1 | **版本** | CLI 报告其版本（`v1.3.0`）。 |
| 2 | **端口可用** | 端口 5000 处于空闲状态，*或*已被状态正常的 Chloros 后端响应（这被视为通过）。 |
| 3 | **后端启动** | 后端二进制文件成功启动。 |
| 4 | **API测试 (`/api/test`)** | 后端返回 `status: ok`。 |
| 5 | **系统信息** | 从 `/api/system-info` 输出 `GPU: <name>, CUDA: <bool>, PyTorch: <version>`。 |
| 6 | **去噪模型** | 找到 `*.pth.enc` 模型 （在 Linux 上：`/usr/lib/chloros/models`）。 |
| 7 | **CUDA + 去噪器**| “Texture Aware” 功能实际上可以正常使用——需要 CUDA**以及** 至少一个模型文件。 |

运行以 `N/7 checks passed` 结束，并按名称列出所有失败项。

### 4. 处理您的第一个数据集

```bash
chloros-cli process ~/datasets/flight001
```

***

## 文件与目录

### 用户级

Chloros 将其凭据和 CLI 配置保存在一个跨平台目录中，即 **`~/.chloros/`**（在 Windows 上为 `%USERPROFILE%\.chloros\`）。另外两个Linux专用的缓存则遵循XDG规范——若已设置，则使用`XDG_CONFIG_HOME` / `XDG_CACHE_HOME`。

| 路径 | 用途 |
| --- | --- |
| `~/.chloros/user_session.json` | 由 `chloros-cli login` 写入的登录会话缓存（在软件包升级后保留；卸载软件包时清空） |
| `~/.chloros/working_directory.txt` | 默认项目文件夹覆盖设置（`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`） |
| `~/.chloros/cli_language.json` | 命令行界面（CLI）语言偏好设置（`chloros-cli language <code>`） |
| `~/.chloros/user.json` | 与Windows图形界面共享的语言设置 —— 此处的 `language` 优先于 `cli_language.json` |
| `~/.chloros/update_cache.json` | Linux/Jetson 启动时更新检查的一小时缓存 |
| `~/.chloros/backend.log` | 后端由 CLI 启动时的后端日志 |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | 按摄像头缓存的 LATTICE 校准包，以序列号和捆绑包哈希值作为键值 |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | DAQ 容差校正配置文件的可选用户覆盖设置 |
| `~/.config/chloros/system_config.json` | 来自动态计算自适应（DCA）的缓存硬件配置文件——删除它可强制进行新的硬件检测 |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | 后端服务器日志，每次启动生成一个文件 |
| `~/Documents/MAPIR/Chloros Projects/` | 未设置覆盖时的默认项目文件夹 |

### 全局设置

| 路径 | 用途 |
| --- | --- |
| `/usr/bin/chloros-cli` | 封装脚本 — 为捆绑的原生库设置 `LD_LIBRARY_PATH` （若该文件夹存在，则将 `TMPDIR` 设置为 `/mnt/ssd/tmp`），随后执行实际二进制文件 |
| `/usr/bin/chloros-backend` | 封装脚本 — 针对后端执行相同操作，并将其标记为已安装的生产构建 |
| `/usr/lib/chloros/cli/`, `/usr/lib/chloros/backend/` | 已编译的CLI和后端，解压后安装（1.3.0版本新增）； 程序分别为 `cli/chloros-cli` 和 `backend/chloros-backend` |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | 两行启动脚本，用于启动上述实际程序——保留在这些路径下，因为SDK、CLI和旧版脚本会在此处查找它们 |
| `/usr/lib/chloros/arena_runtime/` | LATTICE 相机所需的 Arena SDK 运行时 |
| `/usr/lib/chloros/models/*.pth.enc` | 纹理感知去拜耳算法所使用的加密去噪模型 |
| `/usr/lib/python3/dist-packages/chloros_sdk/`（以及 `lattice_sdk`、`lattice_cam`、`arena_api`，即项目加载器） | Python SDK，为系统 `python3` 安装且归该软件包所有 |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | 与该确切构建版本完全匹配的 Python SDK wheel 包，用于虚拟环境 |
| `/usr/lib/chloros/exiftool` | 捆绑的 ExifTool 启动器（仅在系统中不存在 exiftool 时，才会通过符号链接指向 `/usr/local/bin/exiftool`）。从 1.3.0 版本开始，ExifTool 脚本随附了位于 `/usr/lib/chloros/exiftool-dist/` 中的自有 Perl 库，因此无需系统 `libimage-exiftool-perl` 包即可运行 |
| `/etc/chloros/update.conf` | 由 `chloros-cli update` 读取的更新通道配置 |
| `/etc/sysctl.d/60-chloros-ptp.conf` | 设置 `net.ipv4.ip_unprivileged_port_start = 319`，以便后端无需 root 权限即可绑定 PTP 端口 |
| `/etc/sysctl.d/60-chloros-gige.conf` | 将内核的套接字接收缓冲区 (`net.core.rmem_max`, `net.core.rmem_default`) 至 32 MB，并在安装时应用这些设置 （若已设置更高的值则保持不变）。在内核默认设置下，摄像头数据包会在内核中被丢弃，导致数组丢失帧并降低帧率；当缓冲区空间不足时，`chloros-cli lattice network` 会报告缓冲区状态。 参见 [Linux 主机 — 套接字接收缓冲区](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | 将动态加载器指向 `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | 授予已登录用户访问 DAQ-U USB 串行桥接器的权限 (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | 可选的始终在线后端服务，该服务将以后端用户（由您指定）的身份运行（已安装，**未启用**）——需启用此服务，请参阅 [始终在线 PTP](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | 相同的服务，以 **root**身份运行后端（已安装，**未启用**），保留该服务是为了确保启用它的机器能持续运行 |
| `/var/log/chloros/` | root 服务的详细日志，可由 `adm` 组读取 （在彻底清除软件包时被删除） |
| `/usr/share/applications/chloros-cli.desktop` | “Chloros CLI” 应用程序菜单项，用于打开终端 |

**防火墙（1.3.0 版新增）。** 若在安装时 `ufw` 或 `firewalld` 处于活动状态，该软件包将打开 UDP 端口 `5002` （DAQ-E 数据）、`5353`（mDNS 发现）、`319` 和 `320`（PTP），并在卸载该软件包时移除这些规则。 如果已安装防火墙但未启用，则不会添加任何规则，并会打印出应允许的端口，以便您日后启用时使用。它不会打开任何 TCP 端口： 当传感器通过 TCP 连接回此计算机以收集图像时，若防火墙处于活动状态并阻断了该连接，DAQ-E 固件更新将因 `… device never connected back on TCP <port> …` 错误而失败。

## 后端可执行文件位置

CLI 和 SDK 会自动检测后端：

| 组件 | 路径 |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| 后端 | `/usr/lib/chloros/chloros-backend`（`/usr/lib/chloros/backend/chloros-backend`的启动器） |

可通过 `--backend-exe` CLI 标志或 `backend_exe` SDK 构造函数参数覆盖后端路径， 并使用 `--port` 设置端口（默认值为 `5000`）。

{% hint style="info" %}
`CHLOROS_BACKEND_URL` 将 **`lattice`**、**`project`**以及**`daq pool-*`** 命令族。 核心命令（`process`、`login`、`logout`、`status`、 `export-status`、`time-sync`、`selftest`）会刻意忽略该参数，并始终将目标指向 `http://127.0.0.1:<port>`。
{% endhint %}

***

## Linux 上的 LATTICE 相机和 DAQ 光传感器

所有 live-hardware 命令系列均可在 Linux（amd64 和 Jetson）上运行：

* **`chloros-cli lattice`** — 用于发现、连接、配置以及从 LATTICE 相机和同步数组中捕获数据。`.deb` 捆绑了它们所需的 Arena SDK 运行时，并将其注册到动态加载器中。
* **`chloros-cli daq pool-*`** — 通过后端池连接 DAQ-U/M/E 光传感器，流式传输校准光谱，记录 `.daq` 文件并交换捕获配置文件；对于 DAQ-E，还可更改其网络设置、重启设备并更新其固件。 `chloros-cli daq discover` 会扫描可连接的传感器，而 `chloros-cli daq --help` 会列出 `pool-*` 的子命令。
* **`chloros-cli project`** — 无界面模式下运行已保存的项目（包括其相机、传感器和处理设置）。
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

`--sensor-id` 是 `pool-latest` 的必备条件， `pool-stream`、`pool-record` 以及 `pool-set-cap` 所必需；`pool-list` 显示当前池中的 ID。

{% hint style="info" %}
**`--eth-host` 跳过发现过程。** `pool-connect --eth` 通过 mDNS 发现 DAQ-E，其 ARP 备用机制可在 ARP 缓存清空的状态下正常工作；通过 `--eth-host` 传入传感器的 IP 地址或主机名可直接与其建立连接。
{% endhint %}

**DAQ-U 串行权限**由已安装的 udev 规则处理（`uaccess` + 组 `dialout`）。如果已插入的传感器仍无法访问，请重新加载规则或重新插入传感器：

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

完整的命令集请参阅 [CLI 参考](../CLI.md)。

### 无显示器主机的常驻 PTP

该软件包为后端安装了两个 systemd 服务，但均未启用。 在需要为 DAQ-E 传感器和 LATTICE 摄像头持续运行 PTP 时间同步的无显示器 Jetson 或服务器上，或者在 SDK 脚本连接已保存项目（`connect_all()` 不会启动后端）的情况下，请以 **您的用户身份** 启用该服务：

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` 以该用户身份运行后端：其写入的捕获数据归该用户所有，详细日志记录在该用户的 `~/.cache/chloros/logs` 中，并且它使用该用户的 `chloros-cli login` （`~/.chloros`），因此请先以该用户身份登录一次。安装该软件包时，系统会打印此命令并填入您的用户名。

如果没有服务，后端（以及与其关联的 PTP 主时钟）仅在被启动时运行——例如通过 `chloros-cli` 命令或 SDK 脚本。

该单元将后端绑定到 `127.0.0.1:5000` （单元内的 `CHLOROS_HOST` / `CHLOROS_PORT` 环境设置；可通过 `sudo systemctl edit chloros-backend@.service` 进行覆盖），并在发生故障后 5 秒内重启它。

**root 服务。**`chloros-backend.service` 以**root** 身份运行后端。该服务仍被安装，以确保启用了它的机器能够持续运行， 但 root 后端写入的所有内容均归 root 所有，因此用户无法删除自己的捕获数据；其日志位于 `/var/log/chloros`，可被 `adm` 组读取。若要切换至您的用户：

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**服务的后端是共享的，绝不会被客户端停止。** `chloros-cli` 和 SDK 脚本会按其发现的状态使用该后端：`process` 在其上运行，并加载自身项目以替换上一个项目。 在通常情况下，CLI会重启后端（`--restart`，或导入未找到任何内容的 `process`）， 它会显示该单元名称，输出 `sudo systemctl restart <unit>` 并退出 `1`，因为以这种方式停止的后端将保持关闭状态。请使用 `sudo systemctl restart chloros-backend@$USER.service` 手动重启它。 服务后端绝不会在空闲时自行停止，而 `chloros-cli backend stop` 会拒绝该请求，并提示 `systemctl stop` 命令。**PTP 如何获取端口。** PTP 使用 UDP 319/320 端口，这两个端口均低于常规 1024 的特权端口下限。 该软件包的 `postinst` 会使用 `/etc/sysctl.d/60-chloros-ptp.conf` 写入 `net.ipv4.ip_unprivileged_port_start = 319`，这使得后端在以您的用户身份运行时能够绑定这些端口。 从 1.3.0 版本开始，该软件包不再为后端设置文件权限（`setcap`），且 `libcap2-bin` 也不再是依赖项：在未打包的后端中，文件权限会导致其无法启动， 因此请勿自行添加。***

## Bash 脚本示例

{% hint style="info" %}
**适合脚本使用的退出代码。**`chloros-cli process` 在成功时退出 `0`，**失败时返回非零值——包括请求了图像产品但未生成任何产品的运行情况** （此时会输出 `Processing finished but wrote no image products.`，并列出项目文件夹名称及常见原因）。成功运行时会报告写入的图像产品数量（`Image products written: N`，若写入了掩膜则额外输出 `(plus M clamp masks)`）。退出代码：`0` 表示成功，`1` 表示失败，`2` 表示参数错误，`130` 表示运行中断。
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
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

有效的 `--format` 值共有四个，且包含空格——请务必用引号括起来：

| `--format` 值 | 输出文件夹 |
| --- | --- |
| `TIFF (16-bit)` *(默认)* | `tiff16` |
| `TIFF (8-bit)` | `tiff8` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` 支持 `standard`（默认）或 `texture-aware`（Chloros+）。

### 使用 Cron 进行自动化处理

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK示例

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
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
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### 添加文件权限后后端无法启动

请勿在后端运行 `setcap`。 从 1.3.0 版本开始，该软件包不再设置任何文件权限，而 PTP 的 319/320 端口源自 [PTP 如何获取端口](#always-on-ptp-for-headless-hosts) 中所述的 `sysctl` 设置。 文件权限会将解压后的后端置于安全执行模式，在此模式下它无法找到自身的库文件，因此无法启动。如果您添加了此类权限，请将其移除：

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
```

### PTP 无法启动 / 无法绑定端口 319

确认非特权端口下限是否已下调；若未下调，请针对当前启动重新应用该设置：

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

然后检查主控节点：

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### `lattice` 命令因“未找到 LATTICE 相机驱动程序”而失败

后端无法加载其随系统安装的 LATTICE 相机运行时。所有 `lattice` 命令均在后端运行；CLI 本身不包含任何相机驱动程序。请重新安装 Chloros（`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`）：该安装程序会将运行时环境安装在后端旁（位于 `/usr/lib/chloros/arena_runtime` 目录下），并将其注册到动态加载器中。要检查该注册情况：

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
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

启动失败的后端日志位于 `~/.cache/chloros/logs/` 中。如果端口 5000 属于某个后端服务（`systemctl status chloros-backend@$USER.service`），请重启该服务，而非终止其进程。

### 未检测到 CUDA

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

`chloros-cli selftest` 在一行中报告了相同的问题：`GPU: <name>, CUDA: <bool>, PyTorch: <version>`。

### 缺少共享库

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### SD卡系统启动缓慢

从1.3.0版本开始，后端和CLI以解压后的形式安装，因此不再在每次启动时自动解压到临时目录中。 如果该文件夹存在，临时文件会自动保存到 `/mnt/ssd/tmp`；否则，您可以将其指向一个高速文件系统：

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## 在 Linux

`update` 命令仅适用于 Linux/Jetson 环境。该命令会检查在 `/etc/chloros/update.conf` 中配置的更新通道所发布的版本，并提供下载和安装匹配的 `.deb`：

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

在 Linux/Jetson 系统上，CLI 还会在每次启动时执行非阻塞式更新检查 （结果在 `~/.chloros/update_cache.json` 中缓存一小时），并在存在更新版本时输出 `Update available: vX.Y.Z`。您的设置、项目和登录状态在更新后仍会保留。

升级软件包会在安装过程中暂停所有后端服务，因此正在运行的脚本将失去其摄像头和传感器连接，随后会重新启动每个已启用的服务（手动启动但从未启用的服务将保持停止状态，如同重启后一样）。

## 卸载

```bash
sudo apt remove chloros
```

卸载操作将停止并禁用所有后端服务，恢复默认的非特权端口下限（1024），删除接收缓冲区配置文件（运行中的值将保留至重启），移除捆绑的 exiftool 符号链接、 Arena 加载器配置、该软件包添加的防火墙规则以及其安装的 Python SDK，并清除缓存的凭据。您的项目和 `~/.chloros/` 数据文件将保持不变。 `sudo apt purge chloros` 还会删除 `/var/log/chloros`。

***

## 后续步骤

* [NVIDIA Jetson 指南](nvidia-jetson-guide.md) — 针对 Jetson 的优化与部署
* [CLI：命令行](../CLI.md) — CLI 指南
* [API：Python SDK](../api-python-sdk.md) — SDK 指南
* [CLI 参考](../reference/cli-reference.md) 和 [SDK 参考](../reference/sdk-reference.md) — 1.3.0 版本的完整命令/API 列表
* [动态计算适配](../processing-architecture/dynamic-compute-adaptation.md) —— Chloros如何适配您的硬件
