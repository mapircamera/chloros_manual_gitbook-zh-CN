# Linux 安装

Chloros 作为 `.deb` 包分发给 Linux，用于安装 CLI 及后端。 Python 和 SDK 需通过 pip 单独安装。

***

## Linux amd64 (x86_64)

### 系统要求

| 要求 | 最低 | 推荐 |
| --- | --- | --- |
| **发行版** | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+ |
| **处理器** | x86_64 (Intel/AMD) | Intel Core i7 或更高 |
| **内存 (RAM)** | 8GB | 16GB 或更高 |
| **显卡** | 无（CPU 处理） | 配备 4GB+ 显存的 NVIDIA GPU |
| **存储空间** | 2GB 可用空间 | 配备 10GB+ 可用空间的 SSD |
| **Python** | Python 3.7+（适用于 SDK） | Python 3.10+ |

### 安装

下载 `.deb` 软件包并安装：

```bash
sudo dpkg -i chloros-amd64.deb
```

验证安装：

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### 系统要求

| 要求 | 最低 | 推荐 |
| --- | --- | --- |
| **平台** | 搭载 JetPack 6 的 NVIDIA Jetson | Jetson Orin NX 16GB 或 AGX Orin |
| **JetPack** | JetPack 6.x | 最新版 JetPack 6 |
| **内存 (RAM)** | 8GB（GPU/CPU 共享） | 16GB+ 共享 |
| **存储** | 2GB 可用空间 | 10GB+ 可用空间的 NVMe SSD |
| **Python** | Python 3.7+ (适用于 SDK) | Python 3.10+ |

### 安装

下载 JetPack 6 `.deb` 软件包并安装：

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

验证安装：

```bash
chloros-cli --version
```

有关包括热管理和现场部署在内的详细 Jetson 设置，请参阅 [NVIDIA Jetson 指南](nvidia-jetson-guide.md)。

***

## Python SDK 安装（所有 Linux）

Python SDK 需通过 pip 单独安装，并支持 amd64 和 arm64 架构：

```bash
pip install chloros-sdk
```

若需启用可选的进度流支持：

```bash
pip install chloros-sdk[progress]
```

验证 SDK：

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
`.deb` 包会安装 Chloros、CLI 以及后端。 Python SDK 是一个独立的 pip 包，它通过本地 HTTP API 与后端进行通信。
{% endhint %}

***

## 配置目录

Chloros 在 Linux 上的配置遵循 [XDG 基础目录规范](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html)：

| 用途 | Linux 路径 | Windows 对应项 |
| --- | --- | --- |
| **配置** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **数据 / 项目** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **缓存 / 凭据** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## 后端可执行文件位置

`.deb` 软件包将后端安装到标准位置。 CLI 和 SDK 会自动检测后端路径：

| 安装方法 | 后端路径 |
| --- | --- |
| `.deb` 软件包 | `/usr/lib/chloros/chloros-backend` |
| 手动/自定义 | `/opt/mapir/chloros/backend/chloros-backend` |

您可以通过 `--backend-exe` 标志或 `backend_exe` 构造函数参数来覆盖后端路径。

***

## 首次设置

### 1. 激活许可证

访问 CLI 和 SDK 功能需要 Chloros+ 许可证：

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. 检查许可证状态

```bash
chloros-cli status
```

### 3. 处理首个数据集

```bash
chloros-cli process ~/datasets/flight001
```

### 4. 运行系统诊断

验证系统配置是否正确：

```bash
chloros-cli selftest
```

此操作将执行 7 项诊断检查，包括版本、后端启动、API 连接性以及 CUDA/GPU 可用性。

***

## Bash 脚本示例

### 处理多个数据集

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### 使用自定义设置进行处理

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### 使用 Cron 进行自动化处理

将以下内容添加到您的 crontab 中（`crontab -e`），以自动处理新数据集：

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

### CLI 安装后未找到

如果在安装 `.deb` 包后未找到 `chloros-cli`：

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### 权限被拒绝

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
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

### 未检测到 CUDA

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### 缺少共享库

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## 在 Linux 上更新 Chloros

使用内置的 update 命令检查并安装更新：

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## 后续步骤

* [NVIDIA Jetson 指南](nvidia-jetson-guide.md) — Jetson 专属优化与部署
* [CLI : 命令行](../CLI.md) — 完整的 CLI 命令参考
* [API : Python SDK](../api-python-sdk.md) — 完整的 SDK 参考
* [动态计算自适应](../processing-architecture/dynamic-compute-adaptation.md) — Chloros 如何适应您的硬件
