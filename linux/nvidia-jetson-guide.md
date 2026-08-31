# NVIDIA Jetson 指南

NVIDIA Jetson 上的 Chloros 可在边缘端实现多光谱图像处理——无论是在野外、无人机上， 以及远程安装环境中。Chloros 1.2.0 能在启动时自动检测您的 Jetson 型号，并根据检测到的硬件优化其处理策略。 **无需手动调整。**

***

## 支持的 Jetson 机型

| 机型                | 内存            | 处理策略                                     | 推荐用途                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64GB 共享 | `GPU_PARALLEL`（2 个工作线程）                              | 最高性能，适用于大型数据集                      |
| **Jetson Orin NX**   | 8-16GB 共享 | `GPU_PARALLEL`（2 个工作线程，16GB） / `GPU_SINGLE` (8GB)   | 空中和野外部署的首选方案 |
| **Jetson Orin Nano** | 8GB 共享     | `GPU_SINGLE`（1 个工作进程，顺序处理）                     | 入门级边缘计算                                 |

{% hint style="info" %}
Linux arm64 软件包需要 **JetPack 6**，该版本可在 Jetson Orin 系列上使用。旧型号（Nano、TX2、Xavier NX）无法运行 JetPack 6，且不受当前软件包支持。
{% endhint %}

***

## 系统要求

* **JetPack 6.x**（建议使用最新版本）
* **NVIDIA CUDA**（随 JetPack 附带）
* **付费 Chloros+ 套餐** — 铜级或更高（访问所有 CLI/SDK 功能均需此条件；服务器端强制执行）

## 安装

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.2.0"

# Install Python SDK (optional) — the bundled wheel always matches this build
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl

# Run system diagnostics
chloros-cli selftest
```

有关 Linux 的通用安装详情、文件位置及故障排除，请参阅 [Linux 安装指南](linux-installation.md)。

{% hint style="info" %}
**请将解压目录放置在高速存储设备上。** 编译后的二进制文件会在每次启动时自动解压到临时目录中——若从 SD 卡读取，速度会极其缓慢。 Chloros 会在 `/mnt/ssd/tmp` 存在时自动使用它；否则，请将 `TMPDIR` 设置为 NVMe 上的路径（`export TMPDIR=/mnt/nvme/tmp`）。
{% endhint %}

***

## Jetson 上的动态计算适配

### 工作原理

启动时，Chloros 会对您的系统进行分析：

1. 通过 `/proc/device-tree/model` **检测 Jetson 型号**

2.**读取可用的共享 GPU/CPU 内存**（Jetson 使用统一内存）
3. **选择处理策略**（`GPU_PARALLEL`、`GPU_SINGLE` 或 `CPU_PARALLEL`）
4. 自动**设置工作线程数、流水线类型和内存分配**该决策由**总共享 RAM**决定，而非模型名称：

* **总内存低于 12GB**（所有 8GB Jetson 设备）：采用 `GPU_SINGLE` 并配置**1 个工作线程 —— 刻意采用顺序处理**。由于内存空间不足以支持并发工作线程，因此图像将逐张处理。 在**8GB 或以下** 的 Jetson 设备上，线程 3 完全跳过工作线程池，直接在进程内处理每张图像的工作。
* **12GB或以上**（Orin NX 16GB、AGX Orin）：统一内存符合`GPU_PARALLEL`的条件， 但在 Jetson 上**工作进程数量上限为 2 个** —— GPU、工作进程的 RAM 以及每个工作进程的 CUDA 上下文均共享同一内存池，因此增加工作进程数量可能会导致内存不足的故障。

您可以通过环境变量 `CHLOROS_STRATEGY` 覆盖自动选择——请参阅 [动态计算自适应](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override)。

### 按模型的行为

| Jetson 模型                | 策略       | 工作进程数 | 执行                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8GB**    | `GPU_SINGLE`   | 1       | 顺序进程内循环（内存压力下为 `tiled_gpu`） |
| **Jetson Orin NX 8GB**      | `GPU_SINGLE`   | 1       | 进程内顺序循环                     |
| **Jetson Orin NX 16GB**     | `GPU_PARALLEL` | 2       | 并发工作进程，`fused_gpu` 路径  |
| **Jetson AGX Orin 32-64GB** | `GPU_PARALLEL` | 2       | 并发工作进程，`fused_gpu` 路径  |

各平台之间的关键区别在于**内存**。当负载较高时，8GB 版 Jetson 必须采用内存高效的瓦片式处理方法逐张处理图像，而 16GB 及以上版本的 Orin 则可利用高吞吐量的融合管道，通过 GPU 同时处理 2 张图像。

### 单模型 GPU 配额

每个 Jetson 模型都附带一个硬件配置文件，该文件限定了共享池处理可占用的资源上限，并调整批处理大小：

| 模型 | GPU 配额上限 | 批处理大小倍数 | 系统/显示预留 |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70% | ×0.8 | 2.0 GB |
| **Jetson Orin NX** | 75% | ×1.0 | 3.0 GB |
| **Jetson AGX Orin** | 80% | ×1.5 | 4.0 GB |

系统会根据检测到的 RAM 容量调整配置：若 Jetson 报告的 RAM 为 **16GB 或更多**，其批次倍数将提高至 ×1.2。应用倍数前的基础批次大小为 8 张图像。

有关完整的计算适配参考，请参阅 [动态计算适配](../processing-architecture/dynamic-compute-adaptation.md)。

***

## Nano 和 Orin Nano 上纹理感知功能的 GPU 频率限制

“纹理感知”去拜耳算法会运行 GPU 神经网络推理，这可能会在低功耗 Jetson 型号（10-15W 级）以全速运行 GPU 时触发 **过流警告**。 在**Jetson Nano 或 Orin Nano**上进行 Texture Aware 处理之前，Chloros 会检查 GPU 的最高频率，如果当前频率高于**510 MHz**（510000000），则将其限制在该值：

* 如果 CLI 能够向 GPU 频率 sysfs 节点写入数据，则限制将**自动应用**，并打印确认信息。
* 若无法写入（需要 root 权限），CLI 将显示用于手动设置限制的确切 `sudo` 命令，稍作等待以便您阅读，随后继续执行——处理过程仍会继续，但可能会显示过流警告。

若要在处理前自行应用限制：

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

高功耗型号（Orin NX 25W、AGX Orin 60W）以全速运行 GPU；不应用任何限制。标准去拜耳算法在任何型号上均不会触发限制。

{% hint style="info" %}
**Jetson 上的“纹理感知”模式始终一次仅处理一张图像。** 每个工作线程都需要独立的 CUDA 上下文（约 1GB）以及去噪模型的独立副本，而统一内存无法满足这一需求——因此在 Jetson 上，“纹理感知”路径被固定在单个工作线程上，且 GPU 访问被串行化。 预计在任何 Jetson 设备上，Texture Aware 的运行速度都将明显慢于标准模式。
{% endhint %}

***

## 热管理

Jetson 设备的散热余量有限，尤其是在密闭或机载部署环境中。Chloros 会监控 SoC 温度并自动调节批处理大小：

| 温度         | 操作                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70°C**          | 正常运行 — 全速处理                           |
| **70°C** (警告)  | 批处理大小逐步缩减（70°C 至 80°C 区间内从 100% 降至 50%） |
| **80°C** (危急) | 激进限速（80°C 至 90°C 区间内从 50% 降至 0%） |
| **90°C** (关机) | 完全停止 GPU 处理 — 需冷却 |

{% hint style="warning" %}
为确保持续处理，**请确保通风良好并配备散热措施**，尤其是在封闭的现场机柜或机载系统中。为保护硬件，热节流会降低处理吞吐量。
{% endhint %}

***

## 内存管理

Jetson 设备采用 **统一内存** —— GPU 和 CPU 共享同一块物理 RAM。 报告的 VRAM（例如 Orin NX 16GB 上的 ~15.3GB）并非专用的 GPU 内存；它与操作系统及其他所有进程所使用的 RAM 是同一块内存。

### 交换空间警告与建议

在 Jetson 上进行处理之前，CLI 会统计输入文件夹中的 RAW 图像数量（`.tif`、`.tiff`、`.raw`， `.dng` — JPG 预览图不计入其中），估算运行所需的峰值内存，如果 RAM + 交换空间可能不足，则会在**开始前发出警告**。 该警告以 `LOW MEMORY WARNING - Jetson Detected` 为标题，会显示您的图像数量、RAM、当前交换空间以及预估峰值，随后提供针对您项目规模调整后的精确 `fallocate` / `chmod` / `mkswap` / `swapon` 命令，其大小根据您的项目量身定制（绝不小于 8GB）。程序会暂停几秒钟，以免消息在滚动记录中丢失，随后继续处理。**警告所用的内存估算值：**

| 去拜耳模式 | 基准值 | 每张图像 |
| --- | --- | --- |
| 标准 | ~1.5 GB | ~10 MB |
| 纹理感知 | ~2.5 GB （模型 + Python 运行时） | ~15 MB |

当估计峰值超过 RAM + 交换空间减去 1 GB 安全裕量时，该警告会被触发，且仅计算**基于文件的**交换空间——仅使用 zram 的配置仍会被标记。

要手动添加交换空间（示例：8 GB）：



<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->

```bash
# Check current memory and swap
free -h

# Create a swap file
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```### OOM（内存不足）处理

在处理过程中，Chloros 会监控 GPU 内存，并采取平滑降级措施而非直接崩溃：

1. 当 GPU 内存利用率超过 **85%** 时，会预先减少批处理大小
2. 若仍发生内存不足事件，批处理大小将 **减半**，且每次连续发生 OOM 时再次减半；每次后续成功的批处理都会将该惩罚回退一步
3. 在持续高负载压力下，处理管道将从 `fused_gpu` 回退至内存效率更高的 `tiled_gpu` 路径，并最终作为最后手段转为 CPU 处理

***

## 现场部署

### 功耗考虑

| Jetson 型号     | 典型功耗 | 备注                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7-15W              | 直流桶形插头          |
| Jetson Orin NX   | 10-25W             | 直流桶形插头          |
| Jetson AGX Orin  | 15-60W             | USB-C PD 或桶形插头 |

请为持续处理预留足够的功耗预算——在GPU密集型线程3（处理）期间会出现峰值功耗。

### 存储建议

* **强烈建议在 arm64 部署中使用 NVMe SSD**
* SD 卡速度过慢，不适合处理任务——仅作为启动介质使用
* 处理后的输出数据大小应预留为原始图像数据大小的 2-3 倍

### 通过 SSH 实现无显示器运行

Chloros CLI 非常适合无显示器的 Jetson 部署：

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### LATTICE / DAQ-E 时间同步的常驻后端

如果您的 Jetson 以无头模式控制 LATTICE 相机或 DAQ-E 光传感器，请启用后端 systemd 服务，以确保 PTP 主时钟持续运行（该服务默认已安装但未启用）：

```bash
sudo systemctl enable --now chloros-backend.service
chloros-cli time-sync status
```

详情请参阅 [Linux 安装指南](linux-installation.md#always-on-ptp-for-headless-hosts)，其中包括该软件包如何在无需 root 权限的情况下使 PTP 端口 319/320 可绑定。

### 使用 systemd 进行自动化处理

创建一个 systemd 服务以实现自动化处理：

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

当请求生成产品的运行未写入任何图像时，`chloros-cli process` 会以非零状态退出，因此 systemd 的失败状态对监控具有重要意义。

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

## 工作流示例

### 基础 Jetson 处理

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI
```

### Jetson 上的 Python 和 SDK

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
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## 推荐的现场应用 Jetson 系统

对于现场和机载部署，请考虑以下 Jetson Orin NX 16GB 载板选项：

* **机载/无人机**：具备抗振动等级（MIL-STD）、重量轻（低于 300 克）且采用被动散热的系统
* **坚固型现场应用**：配备 IP67/IP69K 级防水外壳，支持 PoE GigE 摄像头连接
* **精简/经济型**：配备可选外置外壳的开发套件

如需针对您的部署场景获取具体的硬件建议，请联系 [MAPIR 技术支持](https://www.mapir.camera/community/contact)。

***

## 后续步骤

* [Linux 安装](linux-installation.md) — Linux 安装的一般说明
* [动态计算适应](../processing-architecture/dynamic-compute-adaptation.md) — 完整的计算策略参考
* [处理管道](../processing-architecture/processing-pipeline.md) — 了解 4 线程管道
* [CLI：命令行](../CLI.md) — CLI 指南
* [API：Python SDK](../api-python-sdk.md) — SDK指南
* [CLI 参考](../reference/cli-reference.md) 和 [SDK 参考](../reference/sdk-reference.md) — 1.2.0 版本的完整命令/API 列表
