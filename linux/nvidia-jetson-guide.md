# NVIDIA Jetson 指南NVIDIA Jetson 上的

Chloros
支持在边缘设备上进行多光谱图像处理——无论是在野外、无人机上还是远程安装环境中。Chloros
1.3.0 会在启动时自动检测您的 Jetson 型号，并根据检测到的硬件优化其处理策略。**无需手动调整。**
{% hint style="info" %}
**

1.3.0 版 Jetson 软件包在 Jetson 的 GPU 上进行处理。** `chloros_1.3.0_arm64_jp6.deb` 专为 JetPack 6 构建，并使用 JetPack 6 的 CUDA 版本 PyTorch，因此包括“纹理感知去拜耳化”在内的所有处理均在 Orin 的 GPU 上运行。 该软件包本身不包含 CUDA：它使用 JetPack 6 安装的 CUDA 和 cuDNN 库（在缺少这些库的镜像上运行的是 `sudo apt install nvidia-jetpack`）。当无法使用 CUDA GPU 时，处理将在 CPU 上运行，且 Texture Aware 将使用标准去拜耳算法。
{% endhint %}

***

## 支持的 Jetson 型号

| 型号                | 内存            | 处理策略                                     | 推荐用途                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64GB 共享 | `GPU_PARALLEL`（2 个工作线程）                              | 最高性能，适用于大型数据集                      |
| **Jetson Orin NX**   | 8-16GB 共享内存  | `GPU_PARALLEL`（2 个工作线程，16GB）/ `GPU_SINGLE`（8GB）   | 现场部署的首选方案              |
| **Jetson Orin Nano** | 8GB 共享     | `GPU_SINGLE`（1 个工作进程，顺序执行）                     | 入门级边缘计算                                 |

{% hint style="info" %}
Linux
arm64 软件包需要 **JetPack 6**，该版本可在 Jetson Orin 系列上使用。较旧的型号（Nano、TX2、Xavier NX）无法运行 JetPack 6，且不受当前软件包支持。
{% endhint %}

***

## 系统要求

* **JetPack 6.x**（建议使用最新版本）
* **来自 JetPack 6 的 NVIDIA CUDA 和 cuDNN**（若您的镜像中缺少这些组件，请安装 `sudo apt install nvidia-jetpack`）
* **付费版 Chloros
+ 套餐** — 铜级或更高（所有 CLI
/SDK
访问均需此条件；服务器端强制执行）

## 安装

```bash
# Install the arm64 .deb package (JetPack 6) -- apt, not dpkg -i, so the
# Python SDK's requirements (python3-numpy, python3-requests) come too
sudo apt install ./chloros_1.3.0_arm64_jp6.deb

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.3.0"
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # the .deb installs the SDK

# Run system diagnostics
chloros-cli selftest
```

`.deb` 会为系统 `python3` 安装与该构建版本匹配的 Python
SDK
。 仅当在虚拟环境中使用时，才需将捆绑的 wheel 包安装到该环境中：`pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`（完全离线）。有关 Linux
的一般安装细节、文件位置及故障排除，请参阅 [Linux
安装指南](linux-installation.md)。

{% hint style="info" %}
**启动时不再进行解压。** 从 1.3.0 版本开始，后端和 CLI
已以未解压形式安装在 `/usr/lib/chloros/` 下，因此它们不再在每次启动时自动解压到临时目录中。 如果该文件夹存在，临时文件仍会自动保存到 `/mnt/ssd/tmp`；否则，您可以将 `TMPDIR` 设置为 NVMe 上的路径（`export TMPDIR=/mnt/nvme/tmp`）。
{% endhint %}

***

## Jetson 上的动态计算适配

### 工作原理

启动时，Chloros
会对您的系统进行配置：

1. 通过 `/proc/device-tree/model` **检测 Jetson 型号**

2.**读取可用的共享 GPU/CPU 内存**（Jetson 使用统一内存）
3. **询问 PyTorch 是否可以使用 CUDA** —— 若已安装 JetPack 6 的 CUDA 和 cuDNN，则可以使用
4. **自动选择处理策略**（`GPU_PARALLEL`、`GPU_SINGLE` 或 `CPU_PARALLEL`）并确定工作线程数

该决策取决于**总共享内存**，而非模型名称：

* **总内存低于 12GB**（所有 8GB Jetson 设备）：**1 个工作线程 — 刻意采用顺序处理**。由于内存空间不足以支持并发工作线程，因此图像将逐张处理。 在**8GB 或以下** 的 Jetson 设备上，线程 3 会完全跳过工作线程池，直接在进程内处理每张图像的工作。后端日志将此策略命名为 `GPU_SINGLE`。
* **12GB及以上**（Orin NX 16GB、AGX Orin）：统一内存符合 `GPU_PARALLEL` 的条件，但**Jetson 上的工作进程数量上限为 2 个** ——GPU、工作进程的 RAM 以及每个工作进程的 CUDA 上下文均共享同一内存池，因此增加工作进程数量可能会导致内存不足的故障。
* **无法使用 CUDA**（缺少 JetPack 的 CUDA 库）：12GB 或以上时为 `CPU_PARALLEL`，低于此容量时每次仅限一个镜像。

您可以通过环境变量 `CHLOROS_STRATEGY` 覆盖自动选择——请参阅 [动态计算自适应](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override)。

### 各模型的行为

| Jetson 模型                | 策略       | 工作进程 | 执行                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8GB**    | `GPU_SINGLE`   | 1       | 顺序进程内循环（内存压力下采用 `tiled_gpu`） |
| **Jetson Orin NX 8GB**      | `GPU_SINGLE`   | 1       | 进程内顺序循环                     |
| **Jetson Orin NX 16GB**     | `GPU_PARALLEL` | 2       | 并发工作进程，`fused_gpu` 路径  |
| **Jetson AGX Orin 32-64GB** | `GPU_PARALLEL` | 2       | 并发工作进程，`fused_gpu` 路径  |

各平台之间的关键区别在于**内存**。当负载较高时，8GB 的 Jetson 必须采用内存高效的平铺处理方式逐张处理图像，而 16GB 及以上的 Orin 则可利用更高吞吐量的融合管道，同时通过 GPU 处理 2 张图像。

### 单模型 GPU 配额

每个 Jetson 模型还附带一个硬件配置文件，该文件限定了共享池处理可占用的资源上限，并调整批处理大小：

| 模型 | GPU 配额上限 | 批处理大小倍数 | 系统/显示预留 |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70% | ×0.8 | 2.0 GB |
| **Jetson Orin NX** | 75% | ×1.0 | 3.0 GB |
| **Jetson AGX Orin** | 80% | ×1.5 | 4.0 GB |

系统会根据检测到的 RAM 容量调整配置：若 Jetson 报告的 RAM 为 **16GB 或以上**，其批处理倍数将提高至 ×1.2。应用倍数前的基础批处理大小为 8 张图像。

有关完整的计算适配参考，请参阅 [动态计算适配](../processing-architecture/dynamic-compute-adaptation.md)。

***

## Nano 和 Orin Nano 上 Texture Aware 的 GPU 频率限制

“纹理感知”去拜耳算法会运行 GPU 神经网络推理，这可能会在低功耗 Jetson 型号（10-15W 级）以全速运行 GPU 时触发 **过流警告**。当您在**Jetson Nano 或 Orin Nano**上运行 `--debayer texture-aware` 时，CLI
会检查 GPU 的最大频率，若当前频率高于该值，则将其限制为 **510 MHz** (510000000)：

* 如果CLI
能够写入GPU频率的sysfs节点，则限制将**自动应用**，并显示确认信息。
* 若无法写入（需要 root 权限），CLI
将显示用于手动设置限制的确切命令 `sudo`，稍作等待以便您阅读，随后继续执行——处理过程仍会运行，但可能会显示过流警告。

若要在处理前自行应用限制：

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

高功耗型号（Orin NX 25W、AGX Orin 60W）以全速运行 GPU；不应用任何限制。标准去拜耳算法在任何型号上均不会触发该限制。

{% hint style="info" %}
**Jetson 上的“纹理感知”模式始终一次仅处理一张图像。** 每个工作线程都需要独立的 CUDA 上下文（约 1GB）以及独立的去噪模型副本，而统一内存无法满足这一需求——因此在 Jetson 上，“纹理感知”路径被固定到单个工作线程，且 GPU 访问被串行化。预计在任何 Jetson 设备上，“纹理感知”模式的运行速度都将明显慢于“标准”模式。 “纹理感知”模式适用于Survey3
图像：LATTICE捕获的图像始终采用“标准”去拜耳处理。若无可用的CUDA GPU，运行时将使用“标准”模式，且后端日志中会记录一行`[DEBAYER] … needs a CUDA GPU …`说明原因（如图中所示的`chloros-cli --verbose`）。
{% endhint %}

***

## 热管理

Jetson 设备的散热余量有限，尤其是在封闭或机载部署环境中。Chloros
会监控 SoC 温度并自动调节批处理大小：

| 温度                 | 操作                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70°C**          | 正常运行 — 全速处理          |
| **70°C** (警告)  | 批处理大小逐步缩减（70°C 至 80°C 区间内从 100% 降至 50%） |
| **80°C**（危急） | 激进限速（80°C 至 90°C 区间内从 50% 降至 0%） |
| **90°C**（关机） | 完全停止 GPU 处理 — 需进行降温 |

{% hint style="warning" %}
为确保持续处理，请**保证充足的通风和散热**，特别是在封闭的现场机箱或机载系统中。为保护硬件，热节流会降低处理吞吐量。
{% endhint %}

***

## 内存管理

Jetson 设备采用 **统一内存** —— GPU 和 CPU 共享同一块物理 RAM。报告的 VRAM（例如 Orin NX 16GB 上的 ~15.3GB）并非专用的 GPU 内存；它与操作系统及其他所有进程所使用的 RAM 是同一块。

### 交换警告与建议

在 Jetson 上进行处理之前，CLI
会统计输入文件夹中的 RAW 图像数量 (`.tif`、`.tiff`、`.raw`、`.dng` ——不计入 JPG 预览图），估算运行所需的峰值内存，并**在开始前发出警告**，如果 RAM + 交换空间可能不足。该警告以 `LOW MEMORY WARNING - Jetson Detected` 为标题，显示您的图像数量、 RAM、当前交换空间以及预估峰值，随后提供针对 项目量身定制的精确命令（大小绝不小于 8GB）。系统会暂停几秒钟，以防止消息在滚动记录中丢失，随后继续处理。**警告中使用的内存估算值：**

| 去拜耳模式 | 基础值 | 每张图像 |
| --- | --- | --- |
| 标准 | ~1.5 GB | ~10 MB |
| 纹理感知 | ~2.5 GB（模型 + Python 运行时） | ~15 MB |

当估计峰值超过“RAM + 交换空间 - 1GB安全裕量”时，该警告会被触发，且仅计算**基于文件的**交换空间——仅使用zram的配置仍会被标记。

要手动添加交换空间（示例：8 GB）：

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
```

<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->


### OOM（内存不足）处理

在处理过程中，Chloros
会监控GPU内存，并采取平滑降级措施而非直接崩溃：

1. 当 GPU 内存利用率超过 **85%** 时，会预先减少批处理大小
2. 若仍发生内存不足事件，批处理大小将 **减半**，且每次连续发生 OOM 时再次减半；此后每次批处理成功执行，该惩罚措施将回退一步
3. 在持续压力下，处理管道会从 `fused_gpu` 回退到内存效率更高的 `tiled_gpu` 路径，并最终作为最后手段转为 CPU 处理

由于 GPU 与 Jetson 的 RAM 共享，因此运行仍受 RAM 加交换空间的限制——上述交换空间警告即为需重点关注的检查项。

***

## 现场部署

### 功耗考虑因素

| Jetson 型号     | 典型功耗 | 备注                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7-15W              | 直流桶形插头          |
| Jetson Orin NX   | 10-25W             | 直流桶形插头          |
| Jetson AGX Orin  | 15-60W             | USB-C PD 或桶形插头 |

请为持续处理预留足够的功耗预算——在 GPU 密集型线程 3（处理）期间，功耗会达到峰值。

### 存储建议

* **强烈建议**在 arm64 部署中使用 NVMe SSD
* SD 卡速度过慢，不适合处理任务——仅用作启动介质
* 请预留相当于原始图像数据大小 2-3 倍的空间用于存储处理后的输出结果

### 通过 SSH
进行无头操作

Chloros
CLI
非常适合无头 Jetson 部署：

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### LATTICE / DAQ-E 时间同步的常驻后端

如果您的 Jetson 以无头模式控制 LATTICE 相机或 DAQ-E 光传感器，请以 **您的用户身份** 启用后端 systemd 服务，以便 PTP 主时钟持续运行，且 SDK
脚本能检测到正在运行的后端（该组件已安装，但默认未启用）：

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
chloros-cli time-sync status
```

此时后端将以您的身份运行：它写入的捕获数据属于您，并且使用您的 `chloros-cli login`。 详情请参阅 [Linux
安装指南](linux-installation.md#always-on-ptp-for-headless-hosts)，其中包括 root 权限的 `chloros-backend.service` 以及该软件包如何在无需 root 权限的情况下使 PTP 端口 319/320 可绑定。

### 通过 systemd 实现自动化处理

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

当请求生成产品的运行未写入任何图像时，`chloros-cli process` 会返回非零退出状态，因此 systemd 的失败状态对监控具有重要意义。

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
    --indices NDVI GNDVI OSAVI
```

### Jetson 上的 Python
SDK


```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "GNDVI", "OSAVI"],
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
        --indices NDVI OSAVI
    echo "Completed $name"
done
```

***

## 推荐的现场应用 Jetson 系统

对于现场和机载部署，请考虑以下 Jetson Orin NX 16GB 载板选项：

* **机载/无人机**：具备振动等级（MIL-STD）认证、重量轻（低于 300 克）且采用被动散热的系统
* **坚固型野外应用**：配备 IP67/IP69K 防水外壳，支持 PoE GigE 摄像头连接
* **精简/经济型**：配备扩展外壳的开发套件

如需针对您的部署场景获取具体的硬件建议，请联系 [MAPIR
支持团队](https://www.mapir.camera/community/contact)。

***

## 后续步骤

* [Linux
安装](linux-installation.md) — Linux
安装的通用说明
* [动态计算适配](../processing-architecture/dynamic-compute-adaptation.md) — 完整的计算策略参考
* [处理管道](../processing-architecture/processing-pipeline.md) — 了解 4 线程管道
* [CLI：命令行](../CLI.md) — CLI
指南
* [API
：Python
SDK
](../api-python-sdk.md) — SDK
指南
* [CLI
参考](../reference/cli-reference.md) 和 [SDK
参考](../reference/sdk-reference.md) — 1.3.0 版本的完整命令/API
列表
