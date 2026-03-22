# 动态计算适配

Chloros 1.1.0 引入了智能硬件检测和自动处理策略选择功能。处理引擎可自动适配您的硬件——从 Jetson Nano 到多 GPU 工作站——无需任何手动配置。

***

## 工作原理

当 Chloros 启动时，它会自动对您的系统进行分析：

1. **检测操作系统** — Windows 或 Linux
2. **识别 CPU 核心数和总内存**

3.**检测 GPU 存在情况** — NVIDIA CUDA 能力、显存、型号
4. **识别 Jetson 型号**（如适用）— 通过 `/proc/device-tree/model`
5. **检查温度传感器**（Jetson）— 用于温度感知处理
6. **选择最佳计算策略** —— 基于检测到的所有硬件
7. 自动**配置工作线程数、管道类型和内存分配**结果会被缓存，以便后续运行更快启动。如果硬件发生变化（例如添加了 GPU），Chloros 将在下次启动时重新进行性能分析。***

## 计算策略

Chloros 会根据您的硬件选择三种计算策略中的一种：

| 策略 | 是否需要 GPU | 工作进程数 | 管道 | 最适合 |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`** | 是（12GB+ 显存或 16GB+ 共享显存） | 3-4 | `fused_gpu` | 12GB+ 桌面 GPU、Jetson Orin NX 16GB、AGX Orin |
| **`GPU_SINGLE`** | 是（&lt; 12GB 显存） | 1-3 | `tiled_gpu` | 入门级 GPU、Jetson Nano、Orin Nano |
| **`CPU_PARALLEL`** | 否 | 核心 - 1 | `cpu_fallback` | 不带 NVIDIA GPU 的系统 |

### 管道类型

* **`fused_gpu`** — 完整的 GPU 处理路径。 所有去拜耳化、校正和索引操作均在 GPU 上通过单次融合处理完成。吞吐量最高，但需要更多 VRAM。
* **`tiled_gpu`** — 内存高效型 GPU 路径。将图像按瓦片处理以适应有限的 GPU 内存。吞吐量较低，但适用于内存受限的设备。
* **`cpu_fallback`** — 仅使用 CPU 并行处理。当没有 NVIDIA GPU 可用时使用。***

## 平台特定行为

| 平台 | 策略 | 工作线程 | 管道 | 备注 |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (串行化) | 内存高效模式，每次处理一张图像 |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 3 | `fused_gpu` (并发) | 推荐的边缘设备 — 真正的并行 GPU 处理 |
| **Jetson AGX Orin 64GB** | `GPU_PARALLEL` | 4 | `fused_gpu` (并发) | 极致边缘性能 |
| **配备 8GB GPU 的台式机** | `GPU_SINGLE` | 3 | `tiled_gpu` | 采用内存高效分块的出色台式机性能 |
| **配备 12GB+ GPU 的桌面** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | 最佳桌面性能 |
| **仅CPU系统** | `CPU_PARALLEL` | 核心数 - 1 | `cpu_fallback` | 无需GPU，使用ThreadPool |

{% hint style="info" %}
**Jetson统一内存**：Jetson设备共享GPU和CPU内存。一台16GB的Jetson Orin NX报告约15.3GB的VRAM，但这实际上是操作系统和CPU进程共用的物理内存。Chloros在设置内存分配阈值时已考虑此因素。
{% endhint %}

***

## 动态 GPU 内存分配

Chloros 采用 [4 线程处理管道](processing-pipeline.md)：

* **线程 1**（检测）— 图像加载、EXIF 解析、目标检测
* **线程 2**（校准）— 反射率校准计算
* **线程 3**（处理）—— GPU 去拜耳化、暗角校正、索引计算
* **线程 4**（导出）—— 文件写入、元数据嵌入

随着前期管道线程完成工作（例如，所有图像均已检测完毕），其 GPU 内存分配将被释放并**重新分配给剩余的活动线程**。 这意味着随着管道的推进，线程 3（GPU 密集型阶段）将获得越来越多的内存，从而提升计算强度最高的工作的吞吐量。

### 分配阶段

| 阶段 | 活跃线程 | GPU 内存分配 |
| --- | --- | --- |
| **早期** | 1, 2, 3, 4 | 分配给所有线程 |
| **中期-早期** | 2, 3, 4 | 线程 1 的内存重新分配 |
| **中期-后期** | 3, 4 | 线程 1 和 2 的内存分配给 3 和 4 |
| **后期** | 3 或 4 | 剩余线程获得最大内存 |***

## 纹理感知处理

由于采用了 AI/ML 降噪模型，纹理感知去拜耳化方法（仅限 Chloros+）比标准方法消耗更多的 GPU 内存：

* 配备 **&lt; 7GB VRAM** 的系统在纹理感知模式下会被强制进入同步处理循环（每次处理一张图像）
* 配备 **7GB+ VRAM** 的系统可并行处理纹理感知模式，但与标准模式相比，工作线程数量会减少***

## 热管理（Jetson）

Jetson 设备存在热限制，尤其是在封闭或机载部署环境中。Chloros 会监控 GPU 和 CPU 温度，并自动调整处理方式：

| 温度 | 响应 |
| --- | --- |
| **&lt; 70°C** | 正常运行 — 全速 |
| **70°C** (警告) | 减少批处理大小 |
| **80°C** (危急) | 激进限速 — 降低并发数和 worker 数量 |
| **90°C** (关机) | 完全停止 GPU 处理 |

在 Jetson 平台上，温度监控使用 `tegrastats`。在散热充足的桌面系统上，极少触发热节流。

***

## 内存压力处理

Chloros 在处理过程中监控系统内存压力：

* **内存阈值**：利用率达到 85% 时触发保守行为
* **OOM 削减**：若发生内存不足事件，分配量将减少 25%（乘数为 0.75）
* **管道回退**： 在严重内存压力下，流水线会自动从 `fused_gpu` 降级至 `tiled_gpu`
* **交换空间建议**：在 Jetson 上，若交换空间不足以容纳您的数据集大小，Chloros 会发出警告***

## 计算适配监控

### CLI 状态输出

处理开始时，CLI 会显示检测到的硬件配置文件：

```

Chloros CLI 1.1.0
Platform: Linux aarch64 (Jetson Orin NX 16GB)
Strategy: GPU_PARALLEL | Workers: 3 | Pipeline: fused_gpu
CUDA: Available | GPU Memory: 15.3 GB (shared)
```

### 系统诊断

运行 `chloros-cli selftest` 以查看完整的硬件配置文件并验证计算能力：

```bash
chloros-cli selftest
```

此操作将检查 CUDA 可用性、GPU 内存、去噪模型以及后端连接性。

***

## 后续步骤

* [处理管道](processing-pipeline.md) — 了解 4 线程管道架构
* [NVIDIA Jetson 指南](../linux/nvidia-jetson-guide.md) —— 针对 Jetson 的部署与优化
* [CLI : 命令行](../CLI.md) —— 完整的 CLI 参考文档
