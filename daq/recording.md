# 记录与 .daq 格式

`.daq` 文件是 Chloros 的光传感器记录格式：这是一个包含来自单个 DAQ 传感器的已校准光谱帧的 **SQLite 数据库**。 在采集会话期间记录一个此类文件，反射率处理管道随后即可将每张图像除以该精确时刻测得的下行辐照度。

## .daq 文件包含的内容

| 属性 | 值 |
| --- | --- |
| 容器 | SQLite 数据库，每个传感器每次记录对应一个文件 |
| 文件名 | 包含 **传感器 ID**和**时间戳**，例如 `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| 每帧光谱 | 135 个数据点，波长范围 340–1010 nm，步长 5 nm，外加 CIE XYZ 三刺激值 |
| 单位 | 校准后光谱辐照度，**W/m²/nm**（已应用出厂校准包 + 遮光罩校准曲线） |
| 标记元数据 | 传感器 ID（用于获取该设备的出厂校准数据）以及当前生效的校准曲线——参见 [校准曲线与校准范围](caps-and-range.md) |

该格式在 DAQ-U、DAQ-M、DAQ-E 和 DAQ-E-S 之间完全一致，因此下游处理无需关注记录数据时使用的传输方式。DAQ-E-S 的记录还会在每个光谱旁存储设备的俯仰角、横滚角和滚转角。

**记录文件在断电后仍会保留（1.3.0）。**实时记录会在每次提交时通过日志和全盘同步进行写入，因此断电或进程被终止只会丢失大约最后 3 秒的数据，而不会丢失整个文件。 写入过程中中断的记录可能会在 `.daq` 文件旁留下一个 `<name>.daq-journal` 文件：**请将这两个文件一并保留并复制。** Chloros 在首次读取文件时会回滚日志。 磁盘繁忙会延迟文件写入，但绝不会延迟传感器的读数。

**停止传输数据的 DAQ-E 将被重新连接（1.3.0）。** 当 DAQ-E 的数据流中断时（例如其交换机重启或链路中断），后端会使用其原有设置重新连接该设备，且该设备上的记录将在同一 `.daq` 文件中继续，文件中会标记出数据缺失的部分。

校准记录需要传感器的出厂校准包。对于 DAQ-U 和 DAQ-M，后端会根据传感器 ID 从 MAPIR 云端获取该校准包（若无法获取则拒绝记录）；DAQ-E 设备则不受此限制，因为它们将校准数据存储在设备本地。

## 通过 GUI 进行记录

在 GUI 中进行录制需要**打开项目**（否则“录制”按钮将处于禁用状态）：

* **全部录制 / 全部停止** — 位于“光传感器”侧边栏顶部；可同时启动或停止所有已连接传感器的 `.daq` 录制。
* **开始记录 / 停止记录** — 针对单个传感器，位于齿轮图标设置弹出窗口中。记录期间，传感器的实时信息行中会显示红色的“REC”指示符。

文件将写入 `<project>/light_sensor/`，当录制停止时——无论是通过“停止”、“全部停止”还是断开录制传感器——生成的 `.daq` 文件将**自动添加到打开的项目中**。 该文件会直接出现在项目的文件列表中，无需手动添加步骤，且已准备好进行反射率处理。

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->



## 通过 CLI

进行录制

CLI 通过后端的传感器池进行录制（后端必须正在运行——这些命令是轻量级 HTTP 客户端）：

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

从 `chloros-cli daq pool-list` 获取 `--sensor-id` 的值。有两个值得了解的默认值：

| 选项 | 默认值 |
| --- | --- |
| `--duration` | `0` — 记录至 `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` 位于 **后端** 的文件系统上，而非 CLI 的文件系统上 |

当CLI 指向另一台机器上的后端时，输出目录的区别就很重要：文件会保存到后端运行的位置。 如果后端位于同一台计算机上，则相对路径 `-o`（例如上文中的 `./out`）将根据您执行命令的文件夹进行解析。

## 来自 Python

的录制`DAQSensorSession`（由 `chloros_sdk.connect_daq_sensor()` 返回）暴露了相同的池化录制： `record_start(output_dir=None, device_name=None)` 返回文件路径，`record_stop()` 返回 `{path, rows}`。 有关完整的会话 API，请参阅 [SDK 参考](../reference/sdk-reference.md)。 SDK 的直接硬件类（仅限桌面安装版）默认将记录写入 `~/Documents/DAQ/`；对于已发布的构建版本，上述池化路径是受支持的途径。

## 在处理阶段使用 .daq 文件

要从影像中生成反射率，Chloros 需要与每次曝光匹配的向下辐射通量：

* **请将 `.daq` 与影像一同保存。**在处理时，处理管道会自动从记录的 `.daq`（任何 DAQ 型号）——或与图像一同存放的 DAQ-M 原生 `.csv` ——从与图像一同存储的数据中自动解析出**时间戳匹配的向下辐射强度**。GUI录制文件会自动满足此要求，因为它们在停止录制的那一刻就会被添加到项目中。
* **校准数据按需获取。** 如果尚未在本地缓存按相机或按DAQ划分的出厂校准包，Chloros 将在首次使用时从 MAPIR 的云端自动获取（需连接互联网一次；缓存于 `~/.chloros/` 中）。
* **实时捕获会写入其自身的辅助文件。** 对于任何实时捕获的反射率帧，实际使用的 DAQ 读数将作为 `.daq` 文件保存在捕获文件的 `daq/` 文件夹中，因此后续可无需原始记录即可重新处理该捕获数据。
* **来自 `chloros_scripts` 记录器的录制文件（1.3.0）。**由公开版 [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) 记录器生成的 `.daq` 文件会被列为生成该文件的 DAQ 型号，其上限值可在**“项目设置 → DAQ 光传感器”**中设置，与其他录制文件无异。这些记录器写入的是原始传感器计数；Chloros 通过串行接口获取传感器的出厂校准数据并加以应用。
* **读数所属对象（1.3.0）。**在每个 `.daq` 捕获项旁边，`.daq.cameras.json` 会标明该读数所描述的相机（或多台相机），处理过程会将其**仅与来自这些相机的图像** 进行匹配。 因此，没有自带光传感器且独立运行的相机将不再借用其他相机的读数——请为其绑定一个传感器，或自行记录一个。 使用 DAQ 本身制作的记录（GUI 记录，`daq pool-record`），以及任何不包含该文件的 `.daq`（包括较早的捕获），仍适用于所有摄像机。

## 导出辐照度数据

处理项目时，系统还会将其中包含的所有光传感器记录导出到
`Light Sensor/`文件夹中，该文件夹位于图像产品旁边。此操作**无需**图像数据：一个
单独记录的光传感器数据即为完整的捕获，而仅包含`.daq`
文件的文件夹也是有效的输入。 仅包含光传感器记录的项目会在应用程序和CLI 中列出这些记录并
进行处理，且在运行结束后仍会保留在列表中。运行
报告会显示写入的光传感器产品数量。

| 产品 | 说明 |
| --- | --- |
| `<name>_calibrated.daq` | 一个可重新处理的归档文件，其架构与实时记录相同，但会声明生成该文件的校准包。重新导入该文件**不会**对其进行二次校准。 |
| `<name>_calibrated.csv` | 以 W/m²/nm 为单位的谱辐照度，基于传感器自身的波长网格，每行对应一个读数，并包含以下光度学列：总功率、明视和暗视勒克斯、带蓝/绿/红分量的 PPFD 以及峰值波长。 |

如果 DAQ-U 或 DAQ-M 的校准包无法获取（您处于离线状态，或
该传感器无存档校准数据），则该传感器将被**跳过并给出原因**，绝不会被写入
为包含原始计数值的“已校准”文件。请连接互联网后重新运行。 DAQ-E
自带校准数据，因此仅在设备未连接且
本地无缓存数据时才需要此操作。

### DAQ-A：原始计数，以及为何这是正确答案

**DAQ-A** 诞生于“按串行号分配校准包”系统之前，因此没有可
获取的校准包。 这并非疏忽：DAQ-A是在现场通过
反射率标靶进行校准的，而基于标靶的校准仅需传感器的*相对*
响应——这恰恰就是其原始计数值。Chloros 公司至今仍使用这些数据进行校准。

因此，DAQ-A 的记录数据可以导出，但文件名会有所不同：

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`，而非 `_calibrated`——采用不同的文件名而非文件内部的标记，
因为该标识必须在文件作为纯文本名称通过电子邮件发送时保持有效。 `.csv`
的文件头显示为 `raw spectral sensor counts (NOT irradiance)`，并警告说这些值仅
**在**该文件内部可比，而非跨传感器可比。 仅对真实辐照度有意义的
列——总功率、勒克斯、PPFD——被留空，而非
根据计数值计算得出。

较早的 DAQ-A-SD 记录（模式 v1.01 / v1.02）仅记录文件的写入时间，而非
每个读数的具体时间戳。 Chloros 无法将图像与这些记录进行匹配——将帧与
写入时间配对虽然表面上看似正确，但实际上是错误的——不过导出文件能正常读取这些数据，
且CSV 文件中会注明所属时钟。

有关反射率的完整说明——单传感器配合相机，以及双传感器环境/物体测量——请参阅 [反射率工作流程](reflectance.md)。
