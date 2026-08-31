# 记录与 .daq 格式

`.daq` 文件是 Chloros 的光传感器记录格式：这是一个包含来自单个 DAQ 传感器经校准的光谱帧的 **SQLite 数据库**。 在采集会话期间记录一份，反射率处理管道随后即可将每张图像除以该精确时刻测得的下行辐照度。

## .daq 文件包含的内容

| 属性 | 值 |
| --- | --- |
| 容器 | SQLite 数据库，每个传感器每次记录一个文件 |
| 文件名 | 包含 **传感器 ID**和**时间戳**，例如 `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| 每帧光谱 | 135 个数据点，波长范围 340–1010 nm，步长 5 nm，外加 CIE XYZ 三刺激值 |
| 单位 | 校准后的光谱辐照度，**W/m²/nm**（已应用出厂校准包 + 遮光罩校正曲线） |
| 标记元数据 | 传感器 ID（用于检索该设备的出厂校准参数的密钥）以及当前生效的校准曲线——参见 [校准曲线与校准范围](caps-and-range.md) |

该格式在 DAQ-U、DAQ-M 和 DAQ-E 之间完全一致，因此下游处理无需关注记录数据时使用的传输方式。

校准记录需要传感器的出厂校准包。 对于 DAQ-U 和 DAQ-M，后端会根据传感器 ID 从 MAPIR 的云端获取该包（若无法获取则拒绝记录）；DAQ-E 设备则不受此限制，因为它们将校准数据存储在设备本地。

## 通过 GUI 进行记录

在 GUI 中进行记录需要**打开项目**（否则“记录”按钮将处于禁用状态）：

* **全部记录 / 全部停止** — 位于“光传感器”侧边栏顶部；可同时启动或停止所有已连接传感器上的 `.daq` 记录。
* **开始记录 / 停止记录** — 针对每个传感器，位于齿轮设置弹出窗口中。记录期间，传感器实时信息行中会显示红色的“REC”指示符。

文件将写入 `<project>/light_sensor/`，当录制停止时——无论是通过“停止”、“全部停止”还是断开录制传感器——生成的 `.daq` 文件将**自动添加到打开的项目中**。 该文件会直接出现在项目的文件列表中，无需手动添加步骤，且已就绪可进行反射率处理。

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->

## 从 CLI 进行录制

CLI 通过后端的传感器池进行记录（后端必须处于运行状态——这些命令是轻量级的 HTTP 客户端）：

```bash
# Connect the sensor into the backend pool
chloros-cli daq pool-connect --eth-host daq-e-def330.local

# Record for 150 seconds, with a human-friendly device label
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
    -o ./out --device-name "rooftop-A"

# Or run open-ended and stop explicitly
chloros-cli daq pool-record --sensor-id daq-e-def330            # --duration defaults to 0 = run until --stop
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

从 `chloros-cli daq pool-list` 获取 `--sensor-id` 的值。有两项值得了解的默认值：

| 选项 | 默认值 |
| --- | --- |
| `--duration` | `0` — 记录至 `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` 位于 **后端** 的文件系统上，而非 CLI 的 |

当 CLI 指向另一台机器上的后端时，输出目录的区别就很重要：文件会保存在后端运行的位置。

## 来自 Python 的录制

`DAQSensorSession`（由 `chloros_sdk.connect_daq_sensor()` 返回）暴露了相同的池化录制： `record_start(output_dir=None, device_name=None)` 返回文件路径，`record_stop()` 返回 `{path, rows}`。 有关完整的会话 API，请参阅 [SDK 参考](../reference/sdk-reference.md)。 SDK 的直接硬件类（仅限桌面安装）默认将记录写入 `~/Documents/DAQ/`；对于已发布的构建版本，上述池化路径是受支持的途径。

## 在处理阶段使用 .daq 文件

要从影像中生成反射率，Chloros 需要与每次曝光相匹配的向下辐照度：

* **请将 `.daq` 与影像一同保存。**在处理时，处理管道会自动从记录的 `.daq`（任何 DAQ 型号）——或与图像一同存放的 DAQ-M 原生 `.csv` ——从与图像一同存储的数据中自动解析出**时间戳匹配的下行辐照度**。GUI录制文件会自动满足此要求，因为它们在停止录制的那一刻就会被添加到项目中。
* **校准数据按需获取。**如果尚未在本地缓存特定相机或特定 DAQ 的出厂校准包，Chloros 将在首次使用时从 MAPIR 的云端自动获取 （需连接一次互联网；缓存于 `~/.chloros/` 下）。
* **实时捕获会生成独立的辅助文件。** 对于任何实时捕获的反射率帧，实际使用的 DAQ 读数都会作为 `.daq` 辅助文件保存在图像旁边，因此日后无需原始记录即可重新处理该捕获数据。

## 提取辐照度数据

处理项目时，系统还会将其中包含的所有光传感器记录导出到
图像产品旁边的 `Light Sensor/` 文件夹中。 这**不需要**图像：单独飞行的一台
光传感器即构成完整的采集数据，而仅包含 `.daq`
文件的文件夹也是有效的输入。运行报告会显示写入的光传感器产品数量。

| 产品 | 说明 |
| --- | --- |
| `<name>_calibrated.daq` | 一个可重新处理的归档文件，其结构与实时记录相同，但会声明生成该文件的校准包。重新导入该文件时，**不会**再次进行校准。 |
| `<name>_calibrated.csv` | 以 W/m²/nm 为单位、基于传感器自身波长网格的光谱辐照度，每行对应一个读数，并包含以下光度学列：总功率、明视和暗视勒克斯、带蓝/绿/红分量拆分的 PPFD 以及峰值波长。 |

如果 DAQ-U 或 DAQ-M 的校准包无法获取（您处于离线状态，或者
该传感器没有存档的校准数据），则该传感器将被**跳过并给出原因**，绝不会被写入
作为包含原始计数的“已校准”文件。请连接互联网并重新运行。 DAQ-E
自带校准信息，因此仅当设备未连接且
本地无缓存数据时才需此操作。

### DAQ-A：原始计数，以及为何这是正确答案

**DAQ-A**诞生于“按串行号分配校准包”系统之前，因此没有可
获取的校准包。 这并非疏漏：DAQ-A 是在现场通过
反射率标靶进行校准的，而基于标靶的校准仅需传感器的*相对*
响应——这恰恰就是其原始计数值。Chloros 目前正是使用这些数据进行校准的。

因此，DAQ-A 的记录数据可以导出，但文件名会有所不同：

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`，而非 `_calibrated`——这是文件名不同，而非文件内部的标记，
因为该标识必须在文件作为纯文本名称通过电子邮件发送时依然有效。 `.csv`
的头部信息显示为 `raw spectral sensor counts (NOT irradiance)`，并提示这些数值仅在
**同一**文件内可比，而非跨传感器可比。 仅对实际辐照度有意义的
列——总功率、勒克斯、PPFD——被留空，而非
根据计数值计算得出。

较早的 DAQ-A-SD 记录（模式 v1.01 / v1.02）仅记录文件的写入时间，而非
每个读数的单独时间戳。 Chloros无法将图像与这些记录匹配——若不进行核对，将帧与
写入时间配对虽看似无误，实则错误——但导出文件能正确读取这些数据，
且CSV会注明所依据的时钟类型。

有关反射率的完整说明——单传感器搭配相机，以及双传感器环境/物体模式——请参阅 [反射率工作流程](reflectance.md)。
