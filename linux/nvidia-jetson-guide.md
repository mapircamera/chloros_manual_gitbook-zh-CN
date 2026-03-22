# NVIDIA Jetson 指南

Chloros 在 NVIDIA Jetson 上支持边缘端的多光谱图像处理——无论是现场、无人机还是远程安装环境。Chloros 会自动检测您的 Jetson 型号，并针对您的硬件优化其处理策略。

***

## 支持的 Jetson 型号

| 型号                | 内存            | 处理策略                                   | 推荐用途                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64GB 共享 | `GPU_PARALLEL` (4 个工作线程)                            | 极致性能，适用于大型数据集                      |
| **Jetson Orin NX**   | 8-16GB 共享内存  | `GPU_PARALLEL` (3 个工作进程, 16GB) / `GPU_SINGLE` (8GB) | 主要推荐用于机载和现场部署 |
| **Jetson Orin Nano** | 8GB 共享     | `GPU_SINGLE` (1 个工作进程)                               | 入门级边缘计算                                 |
| **Jetson Nano**      | 4-8GB 共享   | `GPU_SINGLE` (1 个工作进程)                               | 入门级，内存受限                          |

{% hint style="info" %}
**旧版 Jetson 型号**（TX2、TX1、Xavier NX）可能不受支持。性能将根据可用 GPU 内存和 CUDA 功能而有所不同。
{% endhint %}

***

## 要求

* **JetPack 6.x**（建议使用最新版本）
* **NVIDIA CUDA** （随 JetPack 附带）
* **Chloros+ 许可证**（访问 CLI/SDK 所需）

## 安装

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

有关 Linux 的常规安装详情，请参阅 [Linux 安装指南](linux-installation.md)。

***

## Jetson 上的动态计算适配

Chloros 会自动检测您的 Jetson 型号并选择最佳处理策略。**无需手动调整。**

### 工作原理

启动时，Chloros 会对您的系统进行分析：

1. 通过 `/proc/device-tree/model` **检测 Jetson 型号**

2.**读取可用 GPU/共享内存**

3.**选择处理策略**（`GPU_PARALLEL`、`GPU_SINGLE` 或 `CPU_PARALLEL`）
4. 自动**设置工作线程数、流水线类型和内存分配**

### 按模型的行为

| Jetson 模型                | 策略       | 工作线程 | 流水线                       | 并发数 |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8GB**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (内存高效) | 串行化  |
| **Jetson Orin Nano 8GB**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | 串行化  |
| **Jetson Orin NX 8GB**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | 序列化  |
| **Jetson Orin NX 16GB**     | `GPU_PARALLEL` | 3       | `fused_gpu` (完整 GPU 路径)    | 并发  |
| **Jetson AGX Orin 32-64GB** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | 并发  |

{% hint style="success" %}
**Jetson Orin NX 16GB** 是边缘部署的理想选择——它采用 `GPU_PARALLEL` 策略，支持 3 个并发工作线程，在紧凑的机身内实现了真正的并行 GPU 处理。
{% endhint %}

各平台之间的关键差异在于**内存**。 配备 8GB 共享内存的 Jetson Nano 必须采用内存高效的切片处理方式逐张处理图像，而配备 16GB 内存的 Orin NX 则可利用高吞吐量的融合管道，让 3 张图像同时通过 GPU 进行处理。

完整的计算适配参考，请参阅 [动态计算适配](../processing-architecture/dynamic-compute-adaptation.md)。

***

## 热管理

Jetson 设备的热余量有限，尤其是在封闭或机载部署环境中。Chloros 包含自动热监测和限速功能：

| 温度                        | 操作                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70°C**          | 正常运行 — 全速处理                            |
| **70°C** (警告)  | 自动减少批处理大小                            |
| **80°C** (危急) | 激进限速 — 降低并发度         |
| **90°C** (关机) | 完全停止 GPU 处理 — 需冷却 |

{% hint style="warning" %}
**确保充足的通风和散热**以维持持续处理，特别是在封闭的现场机箱或机载系统中。热限速会降低处理吞吐量以保护硬件。
{% endhint %}

***

## 内存管理

Jetson 设备采用 **统一内存** —— GPU 和 CPU 共享同一块物理 RAM。这意味着报告的 VRAM（例如 Orin NX 16GB 上的 15.3GB）并非专用的 GPU 内存；它是与操作系统及其他进程共享的。

### 交换空间建议

对于大型数据集或支持纹理感知去拜耳处理的场景，Chloros 可能会建议创建交换空间：

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**每张图像的内存估算值：**

* 标准去拜耳化：每张图像约 10 MB
* 纹理感知去拜耳化：每张图像约 15 MB

Chloros 会根据您的数据集大小自动计算所需内存，并在建议使用交换空间时发出警告。

### OOM（内存不足）降级机制

若在处理过程中检测到内存不足：

1. Chloros 会自动减少 GPU 工作线程数
2. 从 `fused_gpu` 管道降级至 `tiled_gpu` 管道（内存利用率更高）
3. 继续以降低的吞吐量进行处理，而非崩溃

***

## 现场部署

### 功耗注意事项

| Jetson 型号     | 典型功耗 | 备注                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5-10W              | USB-C 或桶形插头    |
| Jetson Orin Nano | 7-15W              | DC 桶形插头          |
| Jetson Orin NX   | 10-25W             | DC 桶形插头          |
| Jetson AGX Orin  | 15-60W             | USB-C PD 或桶形插头 |

请为持续处理预留足够的功耗预算——在 GPU 密集型线程 3（处理）期间会出现峰值功耗。

### 存储建议

* **强烈建议**在 arm64 部署中使用 NVMe SSD
* SD 卡速度过慢，不适合处理任务——仅用作启动介质
* 处理后的输出数据大小通常为原始图像数据的 2-3 倍，请预留相应空间

### 通过 SSH 实现无头运行

Chloros CLI 非常适合无头 Jetson 部署：

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

### 基于 systemd 的自动化处理

创建 systemd 服务以实现自动化处理：

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

配合 systemd 定时器进行定时处理：

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## 示例工作流

### 基础 Jetson 处理

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI
```

### Python SDK 在 Jetson 上的应用

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### 批量处理多个飞行任务

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## 推荐的现场应用 Jetson 系统

对于现场和机载部署，请考虑以下 Jetson Orin NX 16GB 载板选项：

* **机载/无人机**：具备抗振等级（MIL-STD）、轻量级（300g 以下）且采用被动散热的系统
* **坚固型野外**：配备 IP67/IP69K 防水外壳，支持 PoE GigE 摄像头连接
* **精简/经济型**：配备扩展外壳的开发套件

如需针对您的部署场景获取具体的硬件建议，请联系 [MAPIR 支持团队](https://www.mapir.camera/community/contact)。

***

## 后续步骤

* [Linux 安装](linux-installation.md) — Linux 安装的通用细节
* [动态计算适配](../processing-architecture/dynamic-compute-adaptation.md) — 完整的计算策略参考
* [处理管道](../processing-architecture/processing-pipeline.md) — 了解 4 线程管道
* [CLI : 命令行](../CLI.md) — 完整的 CLI 参考
* [API : Python SDK](../api-python-sdk.md) — SDK 完整参考
