# 动态计算自适应

Chloros 1.3.0 采用硬件检测和自动处理策略选择机制。处理引擎可自动适配您的硬件——从 Jetson Orin Nano 到多 GPU 工作站——无需任何手动配置。

***

## 工作原理

Chloros 启动时，会分析您的系统：

1. **检测操作系统** —— Windows 或 Linux
2. **识别 CPU 核心数和总内存**

3.**检测 GPU 是否存在** —— NVIDIA CUDA 能力、显存、模型
4. **识别 Jetson 型号**（如适用）—— 通过 `/proc/device-tree/model`
5. **检查温度传感器**（Jetson）—— 用于温度感知处理
6. **选择计算策略**—— 基于所有检测到的硬件
7. **自动配置工作线程数、管道类型和内存分配**

检测到的配置文件会缓存到内存和磁盘中，以便后续运行更快启动：

| 平台 | 缓存配置文件 |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json`（优先采用 `XDG_CONFIG_HOME`） |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

删除该文件可强制进行重新检测——在添加 GPU 或更多内存后此操作非常有用。当缓存由不兼容的旧版本写入时，Chloros 也会自动重新检测。

***

## 计算策略

Chloros 会根据您的硬件从三种计算策略中选择一种：

| 策略 | 何时选用 | 工作进程 | 执行器 | 管道 |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| CUDA GPU 报告**12GB+ VRAM**（在 Jetson 统一内存环境下，还需 12GB+ 总共享 RAM） | `min(4, ⌊(VRAM − 0.5GB) × 0.9 ÷ 3.5GB⌋)`，至少 2 个 —**在 Jetson 上上限为 2** | 工作进程（独立的 CUDA 上下文） | `fused_gpu` |
| **`GPU_SINGLE`**| 配备**2-12GB VRAM**的 CUDA GPU | 3（I/O 重叠；GPU 访问由信号量串行化）。**在内存低于12GB的Jetson上为1（顺序）** | 工作进程（独立的CUDA上下文）；在低内存Jetson上为顺序进程内执行 | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | 无 CUDA GPU，或显存低于 2GB | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

`GPU_PARALLEL` 工作进程计算公式的示例：12GB 显存 → 2 个工作进程，16GB → 3 个工作进程，20GB+ → 4 个工作进程。 每个工作进程分配 3.5GB 显存，这是纹理感知任务的实测需求，该分配量在预留 0.5GB 后，占剩余显存的 90% 以内。

{% hint style="info" %}
**GPU 策略需要能够使用 CUDA 的 PyTorch。** 若机器上的 PyTorch 无法使用 CUDA 设备，则无论其 GPU 型号如何，均会获得 `CPU_PARALLEL`。 1.3.0 版 Jetson 软件包使用 JetPack 6 的 CUDA 版本 PyTorch，因此只要安装了 JetPack 的 CUDA 和 cuDNN 库，Jetson 设备即符合条件。内存低于 12GB 的 Jetson 设备始终一次仅处理一张图像，并在进程内处理 （日志记录为 `GPU_SINGLE`）。
{% endhint %}

GPU 策略会运行一组工作进程，每个进程都是一个独立的操作系统进程，拥有自己的 CUDA 上下文；Chloros 通过再次启动其已安装的后端程序来启动这些进程（而非通过 Python 的 `multiprocessing`，因为该方式的工作进程启动在打包应用程序内部无法正常工作）。 CPU 策略使用线程池（Python 的 `ThreadPoolExecutor`）。Chloros 不使用任何第三方分布式框架（如 Ray）。

### 管道类型

* **`fused_gpu`** — 全GPU处理路径。去拜耳化、校正和索引操作在GPU上通过单次融合处理完成。吞吐量最高，但需要最多的VRAM。
* **`tiled_gpu`** — 内存高效型 GPU 路径。将图像按瓦片处理，以适应有限的 GPU 内存。吞吐量较低，但可在内存受限的设备上运行。
* **`cpu_fallback`** — 仅使用 CPU 并行处理。当没有 NVIDIA GPU 可用时使用，并在两个 GPU 处理路径均失败时作为最后的备用方案。

运行时的回退链始终为 `fused_gpu` → `tiled_gpu` → `cpu_fallback`。

***

## 手动策略覆盖

设置 `CHLOROS_STRATEGY` 环境变量以强制采用特定策略——这是一种专家级应急方案，适用于自动检测选定的策略不适合当前情况时（例如，为其他任务保留 GPU 资源）：

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

该变量匹配时不区分大小写；除这三个名称以外的任何内容都将被忽略，自动检测将继续正常进行。在覆盖设置下，Chloros 仍会为您选择工作进程数量：

| 覆盖值 | 实际使用的工作进程数量 |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

在没有可用 CUDA 设备的机器上，对 `GPU_SINGLE` 或 `GPU_PARALLEL` 的覆盖请求会被拒绝，日志中会显示 `[STRATEGY]` 这一行， 且该运行将使用 `CPU_PARALLEL`。

建议按命令逐次设置，而非永久设置，以便常规运行能保持自动适应。

***

## 平台特定行为

| 平台 | 策略 | 工作进程 | 管道 | 备注 |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu`（顺序） | 内存高效模式，每次处理一个图像 |
| **Jetson Orin NX 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu`（顺序） | 共享内存小于12GB，强制顺序处理 |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 2 | `fused_gpu`（并发）| 推荐边缘设备——Jetson 限制为 2 个工作线程 |
| **Jetson AGX Orin 32-64GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (并发) | 最高边缘性能（Jetson 同样限制为 2 个工作线程） |
| **配备 8GB GPU 的台式机** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 个工作线程在信号量对 GPU 访问进行串行化处理的同时重叠进行 I/O 操作 |
| **配备 12GB+ GPU 的台式机** | `GPU_PARALLEL` | 2-4 | `fused_gpu` （并发） | 桌面系统最佳性能：12GB → 2 个工作线程，16GB → 3 个，20GB+ → 4 个 |
| **纯CPU系统** | `CPU_PARALLEL` | 物理核心数 - 1（至少2个） | `cpu_fallback` | 无需GPU，使用线程池 |

{% hint style="info" %}
**Jetson 统一内存**：Jetson 设备共享 GPU 和 CPU 内存。一台 16GB 的 Jetson Orin NX 报告的 VRAM 约为 15.3GB，但这实际上是操作系统和 CPU 进程所使用的同一块物理 RAM。 这就是为什么 16GB 及以上的 Jetson 设备与 12GB 及以上的台式机 GPU 一样符合 `GPU_PARALLEL` 的条件，但工作线程数上限为 2 个——GPU、工作线程及其各自的 CUDA 上下文均从同一共享池中获取资源。
{% endhint %}

### 基于显存的 GPU 预算（独立 GPU）

在配备独立 NVIDIA GPU 的 x86_64 主机上，检测到的显存大小还决定了显卡处理任务可占用的资源量以及批处理规模的最大值：

| 检测到的 VRAM | GPU 预算上限 | 批处理大小倍数 |
| --- | --- | --- |
| **8GB+** | 90% | ×2.0 |
| **6-8GB** | 85% | ×1.75 |
| **3.5-6GB** | 80% | ×1.5 |
| **2-3.5GB** | 75% | ×1.25 |
| **低于 2GB** | 70% | ×1.0 |

独立显卡仅为系统预留 0.5GB 内存，因为它们不与系统内存共享。Jetson 配置文件预留的内存远多于此，且上限更低——请参阅 [NVIDIA Jetson 指南](../linux/nvidia-jetson-guide.md#per-model-gpu-budget)。

***

## 动态 GPU 内存分配

Chloros 采用[4线程处理管道](processing-pipeline.md)：

* **线程1**（检测）——图像加载、EXIF解析、目标检测
* **线程2**（校准） — 反射率校准计算
* **线程 3**（处理）— GPU 去拜耳化、暗角校正、指数计算
* **线程 4**（导出）— 文件写入、元数据嵌入

线程 1、2 和 4 对 GPU 的消耗较小； 而线程 3 是资源消耗最大的。随着前期管道线程的完成，其 GPU 资源将**重新分配给剩余的活跃线程**，因此随着处理进程的推进，线程 3 获得的内存会逐渐增加。

### 分配阶段

| 阶段 | 活跃线程 | GPU 内存分配 |
| --- | --- | --- |
| **早期** | 1、2、3、4 | 分配给所有线程，其中大部分分配给线程 3 |
| **中期-早期** | 2、3、4 | 重新分配线程 1 的份额 |
| **中后期** | 3, 4 | 线程 1 和 2 的份额分配给 3 和 4 |
| **后期** | 3 或 4 | 最后一个活跃线程获得其最大分配量 |

以下两条规则决定具体数值：

* 若某个线程是**唯一**处于活动状态的线程，则将其配置文件中的最大分配量授予该线程。
* 当有多个*高负载*的 GPU 任务处于活动状态时，每个高负载任务的基础分配量将在它们之间进行分配（但绝不会低于其配置的最小值）。

运行时实际使用的值是平台配置文件分配值与 GPU 内存监视器实时建议值中的**较低者**，因此繁忙的显卡总是优先于过于乐观的配置文件。***

## 纹理感知处理

纹理感知去拜耳算法（**仅限 Chloros+** — `--debayer texture-aware`）运行一个 AI/ML 去噪模型，该模型每次复制大约需要 1.75GB 的 FP16 格式显存，因此其 GPU 内存占用远高于标准方法：

* **VRAM 低于 7GB**的系统会以**同步循环方式逐帧处理** 纹理感知——无法同时容纳多个模型副本，且工作线程池只会增加资源竞争
* **VRAM 7GB+** 的系统可以并行处理“纹理感知”模式，但与“标准”模式相比，工作线程数量会减少
* 在 **Jetson** 上，“Texture Aware” 始终被固定到单个工作线程
* 如果 **没有可用的 CUDA GPU**，运行时将使用标准去拜耳算法，后端会记录一条 `[DEBAYER] … needs a CUDA GPU …` 日志行说明原因
* 在低功耗 Jetson 设备（Nano、Orin Nano）上，当您请求使用 Texture Aware 时，CLI 会限制 GPU 频率——请参阅 [NVIDIA Jetson 指南](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)

***

## 热管理（Jetson）

Jetson 设备存在热限制，尤其是在封闭或机载部署环境中。Chloros 会监控 Jetson 的板载温度传感器，并自动调整批处理大小：

| 温度 | 响应 |
| --- | --- |
| **< 70°C** | 正常运行 — 全速 |
| **70°C**（警告）| 批处理大小逐步缩减（70°C 至 80°C 之间从 100% 降至 50%）|
| **80°C**（危急）| 激进限速（80°C 至 90°C 区间内从 50% 降至 0%）|
| **90°C**（关机）| 完全停止 GPU 处理 |

在散热条件良好的台式机系统上，极少会触发热节流。

***

## 内存压力处理

Chloros 在处理过程中持续监控 GPU 内存，并根据三种情况采取相应措施。

**批处理大小调整。** 一个批处理的初始大小为 8 张图像乘以上表中的平台倍数。随后，Chloros 会检查可用 VRAM，预留其中 20% 用于 PyTorch 自身的开销，并假设每张 12MP 的图像约占用 100MB 的 GPU 内存——批处理大小取内存限制值与平台基准值中的较小者。 批次大小绝不会低于 1。

**预先缩减。**当**VRAM 利用率超过 85%** 时，会在任何任务失败之前缩减批次大小。**按线程分配的削减机制。** 随着实时利用率攀升，每个线程的 GPU 配额将相应缩减：利用率超过 80% 时缩减至 0.75 倍，超过 90% 时缩减至 0.5 倍。 监控阈值分别为 70%（保守值）、85%（正常运行限值）和 95%（内存不足风险）。**内存不足的退避与恢复。** 若仍发生内存不足事件：

* 批处理大小将**减半**，且每次连续发生内存不足时再次减半——每次后续成功的批处理都会将该惩罚措施回退一步
* 活动线程的内存分配量将削减至当前值的70%，且分配器切换至保守策略，在连续成功分配后再次放宽限制
* 在严重压力下，管道会从 `fused_gpu` 降级至 `tiled_gpu`，作为最后手段则降级至 `cpu_fallback`

**主机内存（Jetson）。** 在处理之前，CLI 会根据图像数量和去拜耳模式估算主机内存峰值，如果内存加上基于文件的交换空间可能不足，则会发出警告，并打印出添加交换空间的确切命令——请参阅[NVIDIA Jetson 指南](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations)。

***

## 监控计算适配

### 系统诊断

`chloros-cli selftest` 是确认计算层当前状态的最快方法：

```bash
chloros-cli selftest
```

其 7 项检查涵盖版本、端口可用性、后端启动、`/api/test`、系统信息、去噪器模型存在性以及 CUDA + 去噪器就绪状态。检查 5 会直接输出硬件行：

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

检查 7 会输出 `CUDA: <bool>, Denoiser: <bool>` —— 这两项都必须为真，Texture Aware 功能才能被使用。

### 后端日志

策略和工作线程数是在每次运行开始时由后端内部自动选择的——CLI 界面中没有相关提示信息。当出现异常情况（如 GPU 路径回退、内存不足（OOM）或去噪器无法加载）时，相关信息会显示在该会话的后端日志中：

| 平台 | 日志位置 |
| --- | --- |
| **Linux / Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log`（每次启动生成一个文件） |
| **Linux，通过 CLI 启动的后端** | 同样为 `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### 实时进度

运行期间，CLI 会通过 Server-Sent Events 实时流式传输各线程的进度（检测、分析、处理、导出）——这是判断线程 3 是否为瓶颈的实用依据。参见 [处理管道](processing-pipeline.md)。

***

## 后续步骤

* [处理管道](processing-pipeline.md) — 了解 4 线程管道架构
* [NVIDIA Jetson 指南](../linux/nvidia-jetson-guide.md) —— 针对 Jetson 的部署与优化
* [CLI：命令行](../CLI.md) —— CLI 指南
* [CLI 参考](../reference/cli-reference.md) —— 1.3.0 版本的完整命令列表
