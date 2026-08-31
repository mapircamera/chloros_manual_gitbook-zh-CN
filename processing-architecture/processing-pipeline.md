# 处理管道

Chloros1.2.0 采用了一种 4 线程的处理管道，其工作原理类似于分段式装配线。每个线程负责工作流中的一个特定阶段，因此可以同时有多个图像处于不同阶段的处理过程中。

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## 管道架构

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

每张图像都会按顺序流经所有四个线程。借助 Chloros+ 的多线程处理机制，多张图像可同时占据不同的线程——当第 3 个线程处理一张图像时，第 1 个线程可以检测下一张，第 2 个线程校准另一张，而第 4 个线程则将已处理完成的图像写入磁盘。

每个线程都会报告进度，并通过服务器发送事件（Server-Sent Events）进行流式传输（后端在 `/api/events` 上发布这些事件）。在 CLI 的实时进度显示中，这四个阶段分别标记为 **检测、分析、处理、导出**。***

## 线程详情

### 线程 1：检测

**目的**：加载图像并检测校准目标。

* 从磁盘读取图像文件 — Survey3 `.raw`+`.jpg` 对， LATTICE `.tif`/`.tiff` 拍摄数据，以及 `.dng`
* 提取 EXIF 元数据（GPS、相机型号、时间戳、曝光参数）
* 检测校准目标：针对 LATTICE 拍摄的 ArUco 标记目标几何形状，以及针对 Survey3 校准目标照片的经典面板检测器
* 输出：图像数据 + 元数据 + 目标检测结果

该线程主要受 I/O 和 CPU 性能限制。

### 线程 2：校准

**目的**：根据检测到的校准目标计算校准参数。

* 根据目标图像计算反射率校准系数
* 计算暗角校正参数
* 确定各波段的校准曲线
* 输出：每张图像的校准参数

这是一个受CPU性能限制的计算线程。当启用反射率校准时，线程 3 会等待该线程完成，以确保在处理任何图像之前，其校准系数已准备就绪。

### 线程 3：处理（GPU）

**目的**：应用校正并计算植被指数。**这是计算强度最高的线程。*** **去拜耳化**：将 RAW 拜耳数据转换为多通道图像
  * 标准（快速、中等质量）——默认，`--debayer standard`
  * 纹理感知（慢速、最高质量）——仅限Chloros及以上版本，`--debayer texture-aware`，使用 AI/ML 降噪模型
  * LATTICE 单色（M3M）采集为单波段：对此类图像将跳过去马赛克和白平衡步骤（并生成一行日志消息），而同一批采集中的任何 M3C/拜耳图像仍会执行这些步骤
* **暗角校正**：对整个图像应用镜头暗角校正
* **反射率校准**：应用校准系数将数据转换为反射率值
* **指数计算**：计算植被指数（NDVI、NDRE、GNDVI 等）
* 输出：已处理且可直接导出的图像数据

该线程最能受益于 GPU 加速，也是 [动态计算自适应](dynamic-compute-adaptation.md) 所调优的线程。

### 线程 4：导出

**目的**：将处理后的图像写入磁盘。

* 以选定格式写入输出文件 — `TIFF (16-bit)`、`TIFF (32-bit, Percent)`、`PNG (8-bit)`、`JPG (8-bit)`
* 在输出文件中嵌入元数据（GPS、时间戳、处理参数）
* 将输出文件组织在项目文件夹下，命名为 `<camera>/<format>/<Product>_Images/` —— 例如 `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`。 **导出文件保留源文件名称；文件夹名称标识相应产品。**
* 对于 LATTICE 捕获数据，一个源帧可拆分为多个产品（去拜耳化、预览、辐射度、反射率、折射率），每个产品位于独立的产品文件夹中
* 输出：磁盘上的最终文件

这主要是受I/O限制的线程——使用SSD存储可显著提升性能。

***

## 技术原理：执行器

在第3个线程中，每张图像的处理工作通过Python的标准`concurrent.futures`进行并行化：

* **GPU 策略**（`GPU_SINGLE`、`GPU_PARALLEL`）使用带有**spawn** 启动方法的 `ProcessPoolExecutor` —— 每个工作线程都是一个独立的进程，拥有自己的 CUDA 上下文（`fork` 会继承父进程初始化的 CUDA 状态，从而导致子进程状态损坏）
* **`CPU_PARALLEL`** 使用 `ThreadPoolExecutor` —— NumPy 和 OpenCV 会释放 GIL，因此使用线程即可
* 共享内存为 8GB 或更少的 Jetson 设备将完全跳过执行器，转而在同一进程内顺序处理
* 在 VRAM 低于 7GB 的 GPU 上启用 Texture Aware 时也会顺序运行——去噪模型无法进行多次拟合

Chloros不使用任何第三方分布式框架（如 Ray）。 有关策略和工作线程数量的选择方法，请参阅 [动态计算适应](dynamic-compute-adaptation.md)。

***

## 顺序处理与流水线处理

### 自由模式（顺序）

在 Chloros 的免费版本中，图像会**一次处理一张**，依次经过全部四个阶段：

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

在免费模式下，图形用户界面（GUI）会显示一个简化的进度条；其串行处理阶段分别显示为**目标检测**和**处理**。

### Chloros+ 模式（流水线处理）

拥有 Chloros+ 许可证时，所有四个线程将对不同的图像进行 **并行** 处理：

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

GUI 进度条显示这四个阶段；将鼠标悬停在其上可查看各线程的进度。在 CLI 中，这四个阶段以 **检测、分析、处理、导出** 的形式实时流式传输。

{% hint style="info" %}
**一个标签，两个名称。** CLI 将第 3 阶段称为 _Processing_。后端的高级模式进度流（即 GUI 进度条渲染的内容）将同一阶段标记为 _Calibrating_。它们是同一个线程在执行相同的工作（线程 3：去拜耳滤波、校正、索引）。
{% endhint %}

{% hint style="success" %}
**使用 Chloros+ 进行流水线处理** 的速度可能比顺序处理快 3 到 5 倍，具体取决于您的硬件和数据集大小。 在配备高速 GPU 和 SSD 的系统上，提速效果最为显著。
{% endhint %}

***

## 线程 4 导出进度

导出线程拥有独立的进度跟踪机制，您可以单独轮询其状态：

**CLI：**

```bash
chloros-cli export-status
```

**SDK：**

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

当线程 4 达到 100% 时，处理即告完成。

{% hint style="info" %}
**未写入任何图像的运行即视为失败。**成功时，`chloros-cli process` 会报告写入的图像产品数量（`Image products written: N`）。 如果请求了产品但**未**写入任何产品——仅包含`project.json`和`calibration_data.json`——则CLI将打印`Processing finished but wrote no image products.`并**以非零状态退出**， 同时会标明项目文件夹名称及常见原因（输入文件夹未被识别为捕获文件夹——请检查布局及 `--input-level`——或者所有请求的产品均不适用于这些摄像头）。脚本可依赖该退出代码。
{% endhint %}

***

## 与动态计算适配的关系

[动态计算适配](dynamic-compute-adaptation.md) 主要影响 **线程 3（处理）**：

* **`GPU_PARALLEL`**：线程 3 使用 `fused_gpu` 管道同时将多张图像传入 GPU 处理
* **`GPU_SINGLE`**：线程 3 使用信号量对 GPU 访问进行串行化，同时工作进程重叠执行 I/O 操作，采用 `fused_gpu` 或内存高效的 `tiled_gpu` 管道
* **`CPU_PARALLEL`**：线程 3 使用基于 CPU 的处理，并采用多线程并行方式

随着线程 1 和 2 完成，线程 3 的 GPU 内存分配也会随之增加——参见 [动态 GPU 内存分配](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation)。

***

## 后续步骤

* [动态计算适应](dynamic-compute-adaptation.md) — Chloros 如何为您的硬件选择最佳策略
* [NVIDIA Jetson 指南](../linux/nvidia-jetson-guide.md) — Jetson 平台特有的管道行为
* [处理监控](../processing-images-gui/monitoring-the-processing.md) — 通过 GUI 监控进度
* [《CLI》参考指南](../reference/cli-reference.md) — `process`、`export-status`、退出代码及输出布局
