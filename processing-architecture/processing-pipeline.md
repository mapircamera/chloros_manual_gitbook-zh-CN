# 处理管道

Chloros 1.1.0 采用了一个 4 线程的处理管道，其运作方式类似于分阶段的流水线。每个线程负责处理工作流中的一个特定阶段，从而允许多张图像在不同阶段并行处理。

***

## 管道架构

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

每张图像会依次流经所有四个线程。借助 Chloros+ 的多线程处理功能，多张图像可同时位于不同的线程中——当第 3 线程处理一张图像时，第 1 线程可以检测下一张，第 2 线程可以校准另一张，而第 4 线程可以将之前处理好的图像写入磁盘。

***

## 线程详情

### 线程 1：检测

**目的**：加载图像并检测校准目标。

* 从磁盘读取图像文件（RAW、JPG）
* 提取 EXIF 元数据（GPS、相机型号、时间戳、曝光参数）
* 在标记的目标图像中检测 ArUco 校准目标
* 输出：图像数据 + 元数据 + 目标检测结果

这是一个主要受 I/O 和 CPU 性能限制的线程。

### 线程 2：校准

**目的**：根据检测到的目标计算校准参数。

* 根据目标图像计算反射率校准系数
* 计算暗角校正参数
* 确定各波段的校准曲线
* 输出：每张图像的校准参数

这是一个受CPU性能限制的计算线程。

### 线程 3：处理（GPU）

**目的**：应用校正并计算植被指数。**这是计算强度最高的线程。*** **去拜耳化**：将原始拜耳模式数据转换为多通道图像
  * 标准（快速，中等质量）——默认
  * 纹理感知（慢速，最高质量）——仅限 Chloros+，使用 AI/ML 降噪
* **暗角校正**：对整个图像应用镜头暗角校正
* **反射率校准**：应用校准系数以转换为反射率值
* **指数计算**：计算植被指数（NDVI、NDRE、GNDVI 等）
* 输出：已处理且可导出的图像数据

此线程最能受益于 GPU 加速。[动态计算自适应](dynamic-compute-adaptation.md) 系统主要优化了此线程的行为。

### 线程 4：导出

**目的**：将处理后的图像写入磁盘。

* 以选定格式写入输出文件（TIFF 16 位，TIFF 32 位，PNG，JPG）
* 在输出文件中嵌入 EXIF 元数据（GPS、时间戳、处理参数）
* 将输出文件整理到相机型号子文件夹中
* 输出结果：磁盘上的最终文件

这是一个主要受 I/O 性能限制的线程。SSD 存储可显著提升线程 4 的性能。

***

## 顺序处理与流水线处理

### 免费模式（顺序处理）

在 Chloros 的免费版本中，图像将**逐张**依次处理，依次经过全部四个阶段：

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

GUI 进度条显示两个阶段：目标检测和处理。

### Chloros+ 模式（流水线处理）

拥有 Chloros+ 许可证时，所有四个线程将对不同的图像进行 **并行** 处理：

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

GUI 进度条显示 4 个阶段：检测、分析、校准、导出。将鼠标悬停在进度条上可查看各线程的进度。

{% hint style="success" %}
**使用 Chloros+ 进行流水线处理**的速度可能比顺序处理快 3-5 倍，具体取决于您的硬件和数据集大小。在配备高速 GPU 和 SSD 的系统上，提速效果最为显著。
{% endhint %}

***

## 线程 4 导出进度

在 Chloros 1.1.0 版本中，导出线程（线程 4）拥有独立的进度跟踪功能。您可以单独监控导出进度：

**CLI:**
```bash
chloros-cli export-status
```

**SDK:**
```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

当线程 4 达到 100% 时，处理即告完成。

***

## 与动态计算适配的关系

[动态计算自适应](dynamic-compute-adaptation.md) 系统主要影响 **线程 3（处理）**：

* **`GPU_PARALLEL`** 策略：线程 3 使用 `fused_gpu` 管道同时在 GPU 上处理多个图像
* **`GPU_SINGLE`** 策略：线程 3 使用内存高效的 `tiled_gpu` 管道逐张处理图像
* **`CPU_PARALLEL`** 策略：线程 3 使用基于 CPU 的多线程并行处理

随着线程 1 和 2 的完成，线程 3 的 GPU 内存分配也会动态变化 — 参见 [动态 GPU 内存分配](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation)。

***

## 后续步骤

* [动态计算适应](dynamic-compute-adaptation.md) — Chloros 如何为您的硬件选择最佳策略
* [NVIDIA Jetson 指南](../linux/nvidia-jetson-guide.md) — Jetson 平台上的特定管道行为
* [监控处理过程](../processing-images-gui/monitoring-the-processing.md) — GUI 进度监控
