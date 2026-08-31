# 拍摄设置与模式

“摄像头”选项卡中的拍摄功能由一个红色的 **“全部拍摄”**按钮和一个**“拍摄设置”** 窗格控制，该窗格决定了该按钮的输出结果：哪些摄像头参与拍摄、每台摄像头保存的导出类型，以及快门是单次触发、连续触发还是按间隔触发。 本页面详细记录了整个工作流程——包括配置、实际采集过程、文件的磁盘存储位置，以及后续如何将文件重新处理为校准后的产品。相机和阵列的控制选项位于 [相机设置](camera-settings.md)。

{% hint style="info" %}
**捕获操作需要打开一个项目。**在打开项目之前，“捕获全部”和“捕获设置”齿轮图标将被禁用（“创建或打开一个项目以保存捕获数据”）。每次捕获的数据都将保存在 `captures/` 中的项目文件夹下。
{% endhint %}

## “捕获设置”面板

可通过侧边栏相机列表中**“全部捕获”旁边的齿轮图标**，或任何单个相机设置面板底部的**“打开捕获设置…”**按钮打开该面板。 标题栏显示“捕获设置”，并带有 ← 返回按钮。

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->

此处的选项——包括已选摄像头、按类型勾选框以及捕获模式——均按 **项目** 保存，并在您重新打开项目时恢复。

### 捕获模式

面板顶部有三个模式按钮：

| 模式 | 功能 | 子设置（默认值） |
| --- | --- | --- |
| **单次** *(默认)* | 对所有选中的摄像头进行一次捕获。 | — |
| **单次** *(默认)* | 对所有选定摄像头进行一次捕获。 | 通过 **捕获次数**（默认 1）*或* **捕获时长**（默认 10 秒；单位：秒 / 分钟 / 小时 / 天）停止。 |
| **间隔**（延时摄影） | 按定时器进行连拍。 |**每次间隔拍摄**（默认 1） ·**每**N 单位（默认 5 秒） ·**持续** N 单位（默认 1 分钟）。 |

在连续或间隔模式下，运行时“全部拍摄”按钮将变为**“停止 (N)”**按钮，并实时统计已拍摄张数。

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->

### 选择相机和导出类型

该面板的帮助文本对此进行了总结：“‘捕获全部’将生成哪些相机和导出类型”——默认情况下所有选项均处于启用状态，且选择内容将随该项目一同保存。

* **全选 / 全不选** 按钮可一次性切换所有相机的“包含”复选框状态。
* **批量导出类型切换按钮**（两行按钮）：**全部原始格式 / 全部去拜耳化 / 全部预览 / 全部辐射度 / 全部反射率 / 全部索引**。 每个选项采用三态配色：绿色 ✓ = 所有支持该类型的相机均已启用，琥珀色 – = 部分相机已启用，灰色 = 无相机支持。当无任何已连接相机支持该类型时，该切换按钮将被禁用。启用“最快捕获”模式时，所有选项均会变灰。
* **按相机划分的行**：一个“包含”复选框，以及该相机适用导出类型的可展开（▸/▾）列表，列表中包含各自的复选框。该行会显示启用数量，例如“4/6”。

### 导出类型及支持的相机

共有六种导出类型：**Raw、去拜耳化、辐射度、反射率、预览、索引**。每台相机的行中仅显示适用的类型：

| 导出类型 | 内容 | RGB (FRGB) | 拜耳多光谱 (FRGN/FOCN/FNGB) | 单色 (M3M) |
| --- | --- | --- | --- | --- |
| **原始** | 直接来自传感器的拜耳马赛克（单色：单波段） | ✓ | ✓ | ✓ |
| **去拜耳化** | 线性去马赛克（单色：单通道灰度） | ✓ | ✓ | ✓ |
| **预览** | 完整显示链（根据相机配置文件进行白平衡和伽马校正；多光谱：假彩色拉伸） | ✓ | ✓ | ✓ |
| **辐射度** | 通过完整辐射测量链获得的 float32 W/m²/sr/nm | —（未提供）| ✓ | ✓ |
| **反射率** | uint16 ρ（32768 = 1.0）| —（未提供） | ✓ — 仅当相机具有 DAQ 光传感器时显示（自身拥有或从阵列继承） | 与多光谱相同 |
| **指数** | 植被指数（LUT）渲染 | — | ✓ — 要求相机上启用且非空的指数表达式，且不向组合阵列成员提供（阵列拥有一个共享指数） | — （指数需要 ≥2 个波段；参见 [单色相机与植被指数](mono-indices.md)) |

RGB 相机绝不提供辐射度和反射度——对于宽带光度传感器而言，按拜耳阵列计算的辐射度没有实际意义。

### 最快捕获

**⚡ 最快捕获 — 仅原始数据**切换按钮（开启时为橙色）将覆盖所有导出选项，强制设置为**仅原始数据** — 并为阵列提供一个免费的组合指数合成图像 — 从而使帧数据以最快速度保存： 在捕获时完全跳过辐射度/反射率/显示的计算过程。

{% hint style="info" %}
**仍会保存一个 `.daq` 文件。** 当分配了光传感器时，“最快捕获”仍会在原始帧旁边写入 DAQ 下行读数——因此，辐射度、反射率和折射率产品均可通过后续重新处理生成（参见 [将捕获数据重新处理为校准产品](#re-processing-captures-into-calibrated-products)）。 “最快捕获”功能也不会影响您的复选框选择：关闭该功能后，这些选项会恢复原状。
{% endhint %}

### 阵列级控制

每个连接的阵列在窗格中都有独立的组卡片：

* **“包含”复选框**（各成员三态）以及阵列名称及其显示模式：“(合并 | 分离)”。
* **对齐**复选框（默认**开启**）：将成员导出数据扭曲调整为阵列的对齐配置文件，以便不同相机之间的导出数据在像素级别上对齐。 原始数据保持未畸变状态，但其元数据中包含变换信息。（该配置文件本身是在 [阵列设置面板](camera-settings.md#alignment-co-registration-combined-only) 中计算得出的。）
* 成员摄像机的行数据嵌套在卡片内。

该数组卡片还包含两个记录器。可将其视为**监看与分析**：

| 记录器 | 级别 | 记录内容 |
| --- | --- | --- |
| **● 录制索引视频 / ■ 停止录制** *（仅限组合数组）* | **监控** | 将实时组合索引合成视频以 10 fps 速率录制 — 8 位，预览分辨率，LUT 已烘焙。需要打开项目并启用实时流媒体预览。 录制时显示帧数和已用时间。 |
| **⦿ 录制原始连拍 / ■ 停止原始连拍** *（任何阵列）* | **分析**| 以实时抓取速率录制原始拜耳帧（无处理），并附带每帧清单及 `.daq` 读数，保存为 `captures/bursts/` 格式。 连拍结束后，将出现一个**构建视频** 按钮：它会离线重新处理连拍数据，生成经过校准的视频——包含综合指数和/或每台相机的辐射度/反射率/折射率——以及可选的 TIFF 文件。当您停止连拍时，综合指数构建会自动启动。

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

|## “全量捕获”工作

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

流在侧边栏的摄像机列表中点击 **“全量捕获”**：

1. 所有包含在内、可见且未暂停的摄像机将按其选定的导出类型进行捕获。 **阵列中的相机将作为单个同步触发单元工作**（所有成员构成一个同步组——参见 [多相机阵列](arrays.md)）；独立相机则分别进行拍摄。
2. 隐藏（眼睛图标）或已暂停的摄像机将被跳过。只有当阵列中*所有*成员均被隐藏或暂停时，该阵列才会被完全阻塞。
3. 只要分配了光传感器，相应的 DAQ 下行辐射读数就会作为 `.daq` 文件与影像一同保存——即使是仅捕获原始数据的拍摄也是如此——因此后续始终可以衍生出辐射测量产品。
4. 该按钮显示实时进度——“正在捕获……完成/总计”——在连续/间隔模式下将变为 **停止 (N)**。每个捕获项的超时时间为 300 秒。
5. 当采集过程完成时，结果提示栏会显示 **“已保存 N 个文件”**或**“已保存 N 个，F 个失败”**，若存在跳过的摄像头，还会显示“（S 隐藏/暂停/跳过）”。

## 捕获文件的存储位置

捕获文件将保存在当前打开的项目下，路径为 `<project>/captures/`。每种导出类型都会存放在其**独立的子文件夹**中，因此多级捕获中不同类型文件绝不会混杂：

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── *.daq          the downwelling reading matched to the capture
```

* `<ts>` 表示捕获时间戳，`<serial>` 表示相机序列号。 独立捕获的文件名为 `capture_<ts>_SN<serial>_<level>`；来自同一同步触发器的数组捕获文件名为 `sync_<ts>_SN<serial>_<level>`，且**该组内所有相机共用一个时间戳**（当相机仅保存单一层级时，层级后缀将被省略）。
* **需注意的一处差异：**显示层级存储在名为 `preview/` 的文件夹中，而文件名中保留 `_display` —— 仅此层级存在文件夹名与后缀不一致的情况。
* 未知级别将回退到以其自身名称命名的文件夹中；如果无法创建子文件夹，文件将写入捕获根目录，而非丢失。
* 捕获的 TIFF 文件默认采用无损压缩（DEFLATE），并将完整的校准和处理元数据**存储在文件的 XMP 中**——捕获文件具有自描述性，除 `.daq` 读取文件外，不包含任何辅助文件。

这与 `chloros-cli lattice capture` / `array-capture` 写入其 `-o` 目录时的布局相同——相关说明详见 [CLI 参考指南 § 捕获文件夹的结构](../reference/cli-reference.md#what-a-captures-folder-looks-like)中已有记载。

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, a .daq file at the root, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## 将捕获数据重新处理为校准产品

捕获的原始帧加上保存的 `.daq` 文件，就是处理管道所需的一切——这就是为什么“最快捕获”模式在实际工作中是安全的。

* **图形界面**：将捕获文件夹添加到项目中（[将文件添加到项目](../processing-images-gui/adding-files-to-a-project.md)），然后按常规流程进行处理。
* **CLI**：将 `process` 指向**捕获文件根目录**：

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` 通常只导入您指定的文件夹，但如果该文件夹中没有图像且包含子文件夹，它会自动向下遍历 ——因此，该级别的子文件夹及根目录 `.daq` 中的文件会被一次性全部导入。每次捕获都会以 **单张图像** 的形式导入，其余级别作为可查看模式附加其中，而非每个级别各成一张图像。

直接命名层级子文件夹（例如 `…/captures/raw/`）同样有效， 但会导致根级 `.daq` 文件被遗漏——当您从 `raw/` 重新生成辐射测量产品时，请将这些文件一并复制过去，否则时间戳匹配将无法找到对应的文件。

{% hint style="warning" %}
**处理始终从 `raw` 开始。**在每次捕获中，原始帧是处理管道的源文件； `debayered`、`radiance`、`reflectance` 和 `preview` 作为可视化模式存在，但绝不会被重新输入到处理管道中 ——重新处理衍生产品会再次应用已烘焙到其像素中的暗角、色彩和辐射度运算，因此 Chloros 会直接拒绝处理，而非进行重复处理。 `index/` 和 `composite/` 的渲染结果根本不会被处理（它们是输出文件，而非捕获文件）。 一个**未**包含原始文件导入的“captures”文件夹可正常显示，但 `process` 会跳过它并发出提示；`--input-level {raw,debayered,processed}` 则是为强制建立入口而设计的刻意留下的后门。 有关确切的跳过提示信息，请参阅 [CLI 参考文档](../reference/cli-reference.md#what-a-captures-folder-looks-like)。
{% endhint %}

在编写重新处理脚本时，还有两点行为值得注意：

* 若某次 `chloros-cli process` 运行请求了产品但**未生成任何图像产品，则会报错并返回非零退出代码**——绝不会出现无声的空运行。成功的运行会报告其产品数量。（刻意仅生成元数据的运行仍被视为成功。）
* 重新导入的已处理导出数据绝不会占据捕获的原始数据槽——原始数据始终是管道的源。

## CLI 的等效操作

本页所述的所有操作均可通过无界面模式执行。GUI 捕获模式与 `chloros-cli lattice array-capture` 直接对应：

| GUI | CLI |
| --- | --- |
| 单次 | `chloros-cli lattice array-capture` |
| 连续 | `array-capture --continuous [--count N] [--duration S]` |
| 间隔 | `array-capture --interval S [--duration S]` |
| 最快捕获 | `array-capture --fastest` |
| 对齐复选框 | `--aligned / --no-aligned` |
| 导出类型复选框 | `--processing LEVEL` 或 `--levels L1,L2,…`（默认 `all`） |
| 录制索引视频 | `chloros-cli lattice array-record` |
| 录制原始连拍 / 生成视频 | `chloros-cli lattice array-burst` / `array-build-video` |

完整的标志表、智能自动曝光（smart-AE）稳定捕获选项（`--smart`）以及恒定速率模型详见 [CLI 参考指南 § 捕获模式、记录器与离线重处理](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess) 中。
