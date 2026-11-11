# Chloros CLI 参考手册

**版本：**

1.3.0**生成时间：**2026-07-29 19:19 ·**修订时间：** 2026-11-06**受众：** 针对大型语言模型（LLM）使用进行了优化；同时兼顾人类可读性。**范围：** `chloros-cli` 工具中所有面向用户的子命令，包含选项及可直接复制粘贴的示例。

本文档是随 MAPIR Chloros 附带的 `chloros-cli` 命令行工具的完整参考指南。本文档刻意做到详尽无遗，以便 LLM（或人类）无需查看源代码，即可根据下文列出的内容构建任何受支持的工作流。

如果您只需了解重点内容，请跳转至：
- [五分钟快速入门](#five-minute-quickstart)
- [LATTICE 相机首次连接工作流](#lattice-camera-first-connect-workflow)
- [DAQ 传感器首次连接工作流](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [采集模式、记录器与离线重处理](#capture-modes-recorders--offline-reprocess)

> **通过 AI 助手驱动 Chloros？** Claude Desktop、Claude Code、Cursor 及其他 MCP 客户端可通过 `chloros-mcp` 运行 Processing、LATTICE 相机和阵列，以及 DAQ 光传感器， 该MCP服务器随Python SDK一同安装（在Linux的.deb包中，详见`/usr/bin`）：`claude mcp add chloros -- chloros-mcp`。相关设置、选项及工具详见SDK参考文档中的[MCP服务器（AI助手）](sdk-reference.md#mcp-server-ai-assistants)中。带有 shell 的助手还可以直接运行本页上的所有命令。

> **

1.3.0 版本中的变更**（现有 1.2.0 脚本或工作流会察觉到这些变更）。
>
> **

1.2.0 脚本可能需要处理的变更：**
>
> - **`--preset` 适用。** `lattice capture`、`continuous`、`calibrate`、`multi-capture`、`multi-stream`、 `align-calibrate`、`align-apply` 和 `align-stream` 在 1.2 版本中已移除该功能。0，转而采用相机自身的设置。现在它会设置预设的格式、曝光、增益和帧率：`--preset high_speed` 以 5 毫秒、6 dB、 60帧/秒。`lattice cam-connect --preset`同样保留预设的格式（1.2.0版本在应用预设后会将相机设置为12位）。参见[通用选项](#common-options-shared-by-most-subcommands)。
> - **在 M3M（单色）相机上，拜耳像素格式即指其位深度。** 预设中指定的拜耳格式是单色传感器无法拍摄的：在 1.2.0 版本中，相机拒绝了该格式并保留了原有的格式， 因此该处的 `--preset high_speed` 并非 8 位。现在它采用了相同深度的单色格式：`high_speed` 和 `--pixel-format BayerRG8` 会生成 `Mono8`，`high_quality`、 `triggered` 以及 `BayerRG12` 则生成 `Mono12`。在混合的 M3C + M3M 数组中，每台相机都采用其所属家族的共享格式，而 `network-analysis` / `analyze-array --pixel-format` 会将每台相机投影到该深度。反之亦然：请求 Mono 格式的 M3C 会采用该深度的拜耳格式，因此主控为 M3M 的阵列将不再以 Mono 模式重新打开其 M3C 相机。参见 [Mono (M3M) 单波段相机](#mono-m3m-single-band-cameras)。
> - **`lattice array-capture` 导出文件默认会进行对齐，与应用程序的行为一致。** 每个成员的 `debayered`、 `display`、`radiance`、`reflectance` 以及索引文件都会被变换到主摄像头的帧内，并裁剪至成员摄像头的重叠区域 （该区域因相机偏移量而小于传感器，`XMP-mapir:AlignmentJSON` `"applied": true`），因此CLI的捕获文件与Chloros应用对同一场景的捕获文件完全一致；`raw` 保持原生格式。此前，未使用 `--aligned` 的捕获会将每个文件以原生尺寸保存，且不进行畸变校正。`--no-aligned` 保留了这一特性。相同的默认设置也适用于 `--smart`、 `--continuous` 和 `--interval`，以及 SDK 和 `chloros-mcp`。参见 [`array-capture` 模式]（#array-capture-modes）。
> - **`lattice array-capture --compression` 默认设置为 `none`。** 文件以未压缩格式写入（每帧全分辨率约 6.3 MB），以实现最快的持续速率；1.2.0 版本采用 `deflate` 格式（约 4.1 MB）。 若需更小的文件，请传入 `--compression deflate`。这两种模式均为无损压缩。
> - **PNG 和 JPEG 仅支持 `display` / `preview` 以及索引格式。** 如果捕获的 `--format png` 或 `jpg` 文件包含 8 位文件无法容纳的亮度级别，则在开始捕获前会被拒绝（HTTP 400，退出 `1`）， 默认的 `--processing all` 配合 `--format png`/`jpg` 时也会被拒绝。参见 [捕获导出级别](#capture-export-levels-the-all-default)。
> - **`lattice dls` 需要 `--mac`。**已不再有默认 MAC 地址（此前为某个单元的地址）；`daq pool-discover --only ble` 会查找传感器的 MAC 地址。**`lattice dls --camera SERIAL`** 是新增的。
> - **更多故障会导致 `1` 退出。**当运行的导出数据无法验证或丢失了映像时，`process` 会退出，尽管其他产品已写入；若捕获命令中包含任何无法写入的文件 （其他文件仍能保存）； 当磁盘空间已满时触发 `lattice continuous` 和 `multi-stream`；当记录器报告错误时触发 `array-burst` / `array-record`； 当请求的产品被跳过或未按要求构建时，出现 `array-build-video`；当捕获中途失败时，出现 `project burst` / `stream`； 当录制内容中没有读数时，代码为 `project sensor log`；当摄像机拒绝接受某个值时，代码为 `lattice exposure --set`、`gain --set` 和 `configure --set`。 `lattice network-analysis --json` 退出，因为摘要已退出 （`1` 对应 `needs_force_slip`）。反之：当传感器已不在池中时，`daq pool-disconnect` 会退出 `0`X。参见 [退出代码](#exit-codes)。
> - **当数组连接时，运行缓慢的 `lattice array-connect` 绝不会失败。** 它最多会等待 600 秒（1.2.0 版本中为 180 秒；`project connect` 为 90 秒），并在等待过程中打印后端的步骤；如果到时连接仍未完成，后端会撤销该连接 ——每台摄像头都会恢复到被发现时的状态——因此，出现 `Error:`（退出代码 `1`）意味着未连接到任何阵列。 在 1.2.0 版本中，相机通电后立即进行的连接可能会持续超过等待时间，并在命令失败后仍连接上阵列。现在，按 Ctrl+C 可取消该连接。参见 [持久连接](#persistent-connections-smart-prep-gui-equivalent-flow)。
> - **`lattice array-connect` 保留完整的传感器。** 若未启用 `--width` / `--height` / `--binning`，每台摄像机均设置为全传感器模式，帧率将降至该计算机和网络所能提供的水平；连接输出中列出了更快的选项 （像素合并或更小的感兴趣区域）及其预计帧率。1.2.0 版本通过缩小或合并帧来维持帧率。参见 [`array-connect` 选项](#array-connect-options)。
> - **网络速度慢时不再拒绝数组请求。** 每台摄像头的传输速率将根据其网络带宽份额进行调整，帧率也会随之降低。如果无法将摄像头的传输速率调得如此之低，数组将以串行输出模式运行 (`slip-emit-and-capture`) 模式下运行：每个帧都会到达，但相机不会同时曝光。`--force-tier sim-capture-sim-emit` 则坚持同时曝光。参见 [阵列帧率与连拍模型](#array-fps--burst-model)。
> - **`lattice network-analysis` / `analyze-array` 说明了 connect 的应用场景**： 若未指定 `--pixel-format`，则由 connect 设置的格式（Mono12 / BayerRG12，原为 BayerRG8）以及 [打包的 12 位](#packed-12-bit-transport---packed) 选项决定。 因此，同一条命令返回的帧率可能与 1.2.0 版本不同。
> - **相对路径以命令执行位置为基准。** `-o` 以及后端打开的其他路径 （`--input`、`--burst-dir`、`--profile`、`--save`、`--mask`），配方步骤的 `output`、 以及 `lattice index --live` 的默认 `output/`，均根据您运行 `chloros-cli` 的文件夹进行解析。 在 1.2.0 版本中，后端会在其自身的安装文件夹中解析这些路径，但在已安装的构建环境中会导致“访问被拒绝”的错误。请参阅 [约定](#conventions)。
> - **`lattice index` 以各种形式在后端运行**，与其他 `lattice` 命令类似，因此请先启动后端。离线索引 (`--input`) 由后端计算生成，后端会读取 TIFF 文件，并将处理结果写入其旁边或 `-o` 位置， 而 `--list-presets` / `--list-gradients` 则用于打印后端的表格。在 Windows 上，这些命令现已正常工作： 此前，Windows 命令行界面（CLI）会对所有形式的 `lattice index`（包括 `--live`）都返回 `Error: lattice_sdk not available`。在 Linux 上， 离线索引以前是在CLI中不带后端且无需登录的情况下运行的；现在它需要同时具备后端和登录，就像其他所有CLI命令一样。参见 [索引 / 植被数学](#index--vegetation-maths)。
> - **产品旁的新文件。** 在每个因波段解混而对像素进行钳位处理的辐射度、反射率及指数产品中，该像素所在的波段都会在产品文件夹旁的 `clamp_masks/` 文件夹中生成一个掩膜，并在输出中记录一行 `clamp mask:` 信息。 参见 [分量分离钳位掩膜](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on)。
> - **相机方向。** 单摄像头捕获会将 `raw` 和 `debayered` 以传感器读取时的原始格式保存，并将摄像头的翻转/旋转信息记录在 XMP 中；`process` 会将其应用于生成的产品中。 数组成员文件将保持原生格式，除非您传入新的 `--orient` (`array-capture`、`array-burst`、`project capture` / `burst` / `stream`）。参见 [相机方向](#what-a-captures-folder-looks-like)。
> - **若导出级别名称不为“one”，则会被拒绝。** `lattice capture --processing` 以及所有 `--levels` 列表均包含 `raw`、`debayered`、`radiance`、`reflectance` 或 `display`（或别名如 `preview`；`--processing` 以及 `all`）。在触摸相机之前，任何其他名称都会被拒绝（HTTP 400 `UNKNOWN_LEVEL`，退出 `1`）； 1.2.0 为其保存了 `raw` 并退出 `0`。
> - **`daq pool-set-cap` 需要 `--cap-id`。** 若缺少该参数，命令将退出 `2`；1.2.0 版本移除了盖板校正，保留了裸传感器。 `--cap-id none` 要求使用未校准的传感器。
> - **`lattice resolution --binning-mode` 已被移除。** 像素合并模式始终为“平均”；该标志仍被接受，但会打印警告信息且不执行任何操作。 参见 [传感器调校](#sensor-tuning)。
> - **对于 LATTICE 捕获数据，`process --debayer texture-aware` 会被拒绝。** 对于包含 LATTICE 捕获数据的文件夹，`process` 在导入后会退出 `1`，并报告数据条目数量，随后不会进行任何处理。 LATTICE 捕获数据始终采用标准去马赛克处理，因此 1.2.0 版本虽然接受该选项，但会忽略它；Chloros 应用对此类数据也不提供该选项。参见 [`chloros-cli process`](#chloros-cli-process)。
> - **相机将恢复至初始状态 ——除非使用 `--keep-settings`。** `lattice array-disconnect`，所有临时命令（无数组的 `multi-capture`、`multi-stream`、 `multi-test`、`align-calibrate`、`align-apply`、 `align-stream`、`index --live`）以及 `lattice continuous` 会将每台相机恢复至初始状态： 其快门触发和同步线接线、PTP设置，以及本次拍摄过程中更改的每一项相机设置（曝光和增益、自动曝光目标、限制和阻尼、白平衡、黑电平、像素格式、像素合并和感兴趣区域、翻转、帧率）。而 `--keep-settings` 则将每台相机的的设置保存到相机自身的内存中，因此开机时会直接采用这些设置；而 `--clear-settings` 则将该内存重置为出厂设置。无论采用哪种方式，相机都将处于自由运行状态。项目在连接相机时，会根据自身保存的设置对相机进行配置。 参见 [相机内存](#camera-memory----keep-settings----clear-settings)。
> - **所有模式均以相同方式保存项目设置。** 对项目下连接的任何摄像头、阵列或光传感器所做的更改——无论是通过应用程序、CLI、SDK还是MCP服务器进行的——都会保存到该项目的`cameras.json` / `sensors.json` 中保存，且在任何模式下打开该项目都会据此设置设备。此前，仅应用程序会保存该项目，且其下次保存操作会覆盖其他所有内容。参见 [一个项目记录，所有模式](#one-project-record-every-mode)。
> - **由多台相机生成的文件会标明每台相机的名称。** 多波段堆栈（`align-apply`、`align-stream`、`index --live --save-multiband`）、由多台相机波段生成的实时索引，以及数组捕获生成的 `composite/` 图像，均以列表形式包含每台源相机的序列号、型号和校准包（`XMP-mapir:SourceCameraSerials`、 `SourceCameraModels`、`SourceCalBundleSha256s`、……），而EXIF `SerialNumber`则以逗号分隔的形式列出了序列号。 以前这些文件要么完全不包含任何相机信息，要么仅包含第一台相机的序列号。参见 [由多台相机生成的文件](#files-made-from-several-cameras)。
> - **只有 `raw` 文件包含相机的校准标签。** 与 MicaSense 兼容的文件集（`BlackLevel`、`RadiometricCalibration`、按通道划分的 `Band{R,G,B}RadiometricCalibration`、 `MicaSenseVignetting*`数据集）以及`CalibrationJSON`有效载荷描述了传感器的原始值，因此它们仅包含在`raw`捕获文件中。 其他所有文件——捕获文件的 `debayered`、`display`、`radiance`、 `reflectance` 以及索引文件、`lattice continuous` 和 `multi-stream` 帧， 以及每个 `process` 产品——都包含相机的标识（`XMP-mapir:CameraSerial`、`CameraModel`、`FilterType`、 `LensModel`、`CalBundleSha256`、……），并在每次拍摄时包含该帧的专属记录（`XMP-mapir:FrameJSON`：曝光、增益、温度、像素格式、拍摄时间）。 在 1.2.0 版本中，它们还包含原始数据集，该数据集由应用该工具 （如Pix4D、Metashape）会将其应用于非原始值的像素。`process`可通过其序列号和`CalBundleSha256`来查找此类文件的校准信息。
>
> **其他变更：**
>
> - **数组的自动帧率将根据数组的实际输出进行调整。** 若帧率丢失的触发信号超过 1%，则会降低帧率（此判断基于多次长时间监测而非单次监测），且系统会不时重新尝试之前降低的帧率。用户手动设置的帧率将保持不变。
> - **自动曝光更稳定。** 它在光线恒定时保持恒定，光线变化后数秒内稳定下来，忽略微小的高光，并能恢复在强阳光下停止响应的相机。 参见 [自动曝光的工作原理](#how-auto-exposure-behaves)。
> - **`process` 仅统计您的图像和录像文件。** 若对包含项目 `camera_cal/` 的文件夹重新运行，将不再计入这些校准文件（例如，在包含 90 张图片的集合中显示“找到 96 张图片”），而已经导入过这些文件的项目会将其删除。
> - **`lattice multi-test` 基于相机的开机状态运行。** 它会在运行期间开启相机的 PTP 功能，并在首次触发前等待相机时钟锁定并稳定——每台相机从开机起需 120 秒（约 2 分钟）才能完成时钟锁定 （由 `--ptp-settle SECONDS` 设置），因为刚锁定的时钟在约 2 分钟内误差可达 ~200 微秒——并将其后将每台摄像机的 PTP 恢复至初始状态。对于仍处于先前测试锁定状态的摄像机，则不再等待。 此前，在没有 DAQ-E 响应的单交换机网络中，该程序会保持 PTP 关闭状态，并将每台摄像机的自由运行时钟视为无法锁定的 PTP 时钟（`PTP is not locked`，每次运行）。参见 [多摄像头（瞬态会话）](#multi-camera-transient-sessions)。
> - **`.daq` 记录会说明其电容器是被选定还是被默认的。** DAQ-U、DAQ-M 和 DAQ-E 出厂时未配备盖帽，且 Chloros 无法检测是否安装了盖帽，因此未指定盖帽名称的传感器默认会被标记为 `sunshine_cosine`，这是一种 *假设*。现在，每个Chloros记录（模式1.30）都会在`cap_id`旁边存储`cap_id_source`： 若您为盖帽命名（包括 `none`），则存储 `operator`；若采用默认值，则存储 `auto_default`。处理一个盖帽被默认设定的记录时，若项目中的 DAQ 盖帽设置保留为“自动”，会输出一个名为 `[daq-dls] WARNING` 的文件：在阳光校正下，一个未配置的 DAQ-U 读数约高出 30 倍。连接时**请为所安装的电容命名**，或设置项目的 DAQ 电容，警告便会消失。
> - **Windows 系统上不再出现 pip 控制台窗口。** 在安装了 Python 的 PC 上，后端及其每个图像处理进程在启动时会各运行一次 `pip`， 且每个进程都会打开一个控制台窗口。现在不再如此。
> - **实时 `.daq` 记录可在断电后保留。**记录在传感器后台进行，每次提交时都会写入日志并执行完整的磁盘同步， 因此即使磁盘速度较慢，也会延迟文件写入，但绝不会延迟读数。若录制在提交过程中中断（断电、强制终止），其旁可能会留下 `<name>.daq-journal` 文件：**将这两个文件合并即可。**Chloros在首次读取文件时会回滚日志，因此记录文件打开时包含停止前已提交的所有内容（约最后3秒的数据会丢失）；在1.2.0版本中，若在错误时刻发生断电，可能会导致整个文件无法读取（“数据库磁盘映像格式错误”）。
> - **在繁忙磁盘上进行记录不再导致传感器卡死。** `daq pool-record` 会将读取操作排入队列，并通过独立线程进行写入；无法写入的读取操作会被计数 （`daq pool-list`显示了包含最后一个错误的`N write error(s)`），而非导致传感器冻结（在四台相机同时保存TIFF文件的情况下，单次写入耗时8.4秒）。
> - **`lattice multi-capture`若未使用数组，在触发器超时后将不再丢失已保存的帧。** 相机触发超时（`SC_ERR_TIMEOUT -1011`，在单个 1 GbE 交换机上连接四台相机时观察到）会终止该帧组的运行：已保存的帧组将通过 `Saved:` 列出，随后显示错误信息， 退出 `1`。在 1.2.0 版本中，系统会返回 HTTP 500 错误且不显示列表，尽管文件已存在于磁盘上。
> - **软件触发摄像头不再丢弃每隔一次的触发信号。** 处于软件触发模式的摄像头现在会将读出期间收到的触发信号放入队列（或由 Chloros 等待其准备就绪）， 这与有线同步相机此前已实现的行为一致。
> - **`daq pool-discover` 指代 DAQ-E-S `DAQ-E-S`** （以及 `--json` 的 `model`）。此前，每个以太网传感器均显示为 `DAQ-E`； 即使某个传感器在约 2 秒内未能通过身份验证，仍会被列为 `DAQ-E`。
> - **改进的 LATTICE 指数产品**（NDVI 及其他指数）来自 `process`，适用于所有处理模式。
> - **OCN 相机：在阳光下改进了辐射度、反射率及指数产品。** 波段解混后会应用机队校正；RGN相机保持不变，而OCN的数值与1.2.0版本不同。当使用较早版本拍摄的OCN数据在安装了该数据拍摄时所用校准数据的计算机上进行处理时，此校正同样适用 （Chloros在相机连接后会保留其校准数据）。
> - **`lattice continuous`记录原始帧。** `tiff`运行后会生成去马赛克的3平面帧，而 `process` 无法以此为基础进行处理。其帧数据现为 `raw` 文件（由数组捕获写入）：一个马赛克平面采用传感器原始数值，未压缩，并包含校准数据包。 一个从这些文件中读取三个平面的脚本，实际上只读取了一个平面。
>
> - **每次捕获都会保存其使用的DAQ读数，这与GUI的行为一致。** `lattice capture`、 `lattice continuous`（每帧一个 `.daq`）、`array-capture` 和 `multi-capture` 在连接数组（及 SDK）时，会写入 `.daq` 及其 `.match.json`；在 1.2 版本中，0 版本中，保存原始或去拜耳化捕获时，仅会生成 `--force-daq` 对应的条目。`--no-daq` 用于将其关闭，`--daq-tolerance-ms MS` 用于设置该捕获的匹配容差，而新的 `daq pool-bind` 可通过 CLI 将相机或阵列绑定到 DAQ。`.daq` 保存的捕获在 `process` 中仅与它所命名的相机进行匹配。 现在，捕获会等待 DAQ 的下一次读数，以便将曝光前后两侧的读数进行融合。 请参阅 [捕获保存的 DAQ 读数](#the-daq-reading-a-capture-saves) 和 [远离曝光的 DAQ 读数](#a-daq-reading-far-from-the-exposure)。
> - **`process --indices NAME`** 为每个摄像头的波段在项目中添加索引，并包含一个新项目。在 1.2.0 版本中，若项目中尚未列出的名称，则会被 `Unknown index name(s) ignored` 忽略。
> - **`process` 位于实时数组旁：**它不再重启包含相机或数据采集设备的后端，并且会等待最多 15 秒，直至运行即将结束。**Ctrl+C** 会停止后端中的运行，而不仅仅是CLI 中的运行。 参见 [`process` 说明](#notes)。
> - **若项目文件夹路径过深，超出 Windows 的 260 个字符路径限制**，系统会直接拒绝该路径，并采用最长且符合限制的文件夹路径，而非执行一个不写入任何数据的运行。 Windows安装程序现提供“启用Windows长路径”选项，并为后端添加了Microsoft Defender排除项，两者默认均已勾选。参见[Windows (.exe)](#windows-exe)。
> - **`lattice array-capture --continuous`** 会运行后端自带的连续捕获功能：每 2 秒显示一条进度条，显示摘要而非逐个文件的列表，并且当存在未写入的文件或磁盘空间已满时，会返回非零退出代码。 参见 [`array-capture` 模式](#array-capture-modes)。
> - **在 `array-connect` 之后出现的第一个 `array-capture`** 会等待最多 20 秒以获取数组的首个帧组，而非因返回 HTTP 503 错误“数组仍在预热中”而失败。
> - **ROI 内的阵列曝光：** 较小的 ROI 不再将阵列成员的曝光时间限制在传感器的自由运行周期内（768 × 304 分辨率下为 4.9 毫秒）。参见 [阵列成员的曝光](#exposure-on-an-array-member)。
> - **自动曝光不再降低阵列的自动帧率。** 通过电缆同步的阵列以网络和相机读出允许的速率运行；自动曝光会先填满该速率所对应的周期，然后提高增益。 如果手动曝光时间超过该周期（`lattice exposure --set`、`configure --set`），系统会减慢触发速度以适应该曝光时间，而非拒绝该操作；当成员恢复为自动模式时，帧率将恢复正常。参见 [数组成员的曝光](#exposure-on-an-array-member)。
> - **数组产生的不完整帧组更少**，且捕获 503 错误（“无新的同步帧组”）的情况更少：每个从属相机的触发输入现已经过去毛刺处理。
> - **打包的 12 位传输**（`--packed`，默认 `auto`）可将全传感器数组的帧率提高约三分之一。 现已应用 `array-connect --packet-size`（1.2.0 版本虽已接受该配置但未生效）。 参见 [打包的 12 位传输](#packed-12-bit-transport---packed)。
> - **相机重启后可自动恢复的阵列。** 交换机重启后，`array-connect` 的阵列会-当相机响应时重新打开它们，并在 PTP 恢复后继续流式传输；在 1.2.0 版本中，该功能会永久停止，需要新的 `array-connect`。`array-status` 报告 `ptp_enabled: False`（以及 `ptp_off:`）的状态，而此前它会报告数组连接时的设置。参见 [持久连接](#persistent-connections-smart-prep-gui-equivalent-flow)。
> - **`lattice array-record` 在未打开 GUI 的情况下进行录制。** 当 GUI 的组合视图关闭时，后端会自行渲染视图：阵列的NDVI，或您通过新 `--expr` 传递的索引。现在，未写入任何帧的录制将失败（`no-frames: …`，退出 `1`）。 此前，它仅在 GUI 的组合视图打开时才进行录制， 而即使录制内容为空，程序仍会退出（`0`）。参见 [`array-record`](#array-record--combined-index-videogif-monitoring-grade)。
> - **Chloros已持有的摄像头不会被再次打开。** 试图自行打开该摄像头的命令将返回HTTP 409 `CAMERA_HELD`响应，其中会注明持有该摄像头的对象以及释放它的命令。 这包括当前正被其他命令使用的摄像头（例如正在打开摄像头的 `multi-test` 或 `multi-capture`，以及 `lattice capture`）：在第一个命令完成之前，第二个命令将被拒绝。 再次打开该摄像头可能会中断持有者的流，或导致后端崩溃。参见 [持久连接](#persistent-connections-smart-prep-gui-equivalent-flow)。
> - **临时多摄像头命令会成功打开所有摄像头，否则失败。** `lattice multi-capture`（未连接数组）、`multi-stream`、`multi-test`、`align-calibrate`、 `align-apply`、`align-stream` 和 `index --live` 会退出 `1`，该命令指定的摄像头未发送任何帧。 数据流会跳过缺少摄像机的帧组，并在连续三次缺失后停止；`multi-test` 会为每次触发打印一行，`(no frame)` 则在摄像机缺失时打印；`align-stream` 也会因任务错误退出 `1` 因任务错误退出。随后每次运行都会将每台摄像机恢复至初始状态：包括其触发器和同步线接线、PTP (`PtpEnable`) 以及运行过程中更改的各项相机设置——包括曝光和增益、AE 目标、限值和阻尼、白平衡、黑电平、像素格式、像素合并和 ROI、翻转、帧率——除非 `--keep-settings` / `--clear-settings`（参见 [相机内存](#camera-memory----keep-settings----clear-settings)）。 此前，这些命令会保留所有接收到的帧并退出 `0`（`align-stream` 会输出作业错误，如 `Note:` 所示）， 并将所有摄像机保留在 `TriggerMode` `On` 中。此外，在保存帧之前（最多几秒钟），它们会根据前一场景设置每台摄像机的曝光参数，目标是使连接的阵列将每台摄像机初始化为：40 多光谱， 60RGB（由`--brightness N`设定），并在整个运行过程中保持该设置； Chloros未持有的单个`lattice capture`相机也是如此。此前，每台相机的第一帧都是在相机自身的曝光极限下拍摄的，约为100毫秒，无论光线如何，且经常出现剪切。 参见 [多摄像头（瞬态会话）](#multi-camera-transient-sessions)。
> - **瞬态多摄像头命令以与 `array-connect` 相同的方式共享网络，且仅打开 `--serials`。** 每台摄像机的传输速率均按其在主机链路中的分配份额进行调节，因此四台摄像机可在1 Gbps链路上运行；此前，这四台摄像机总共请求125 MB/s的带宽（即1 GbE的全部带宽），导致每帧都会丢失数据包。 对于网络无法承载的阵列，会在任何摄像机开始流式传输前被拒绝。现在由 `--serials` 选择摄像机，第一个触发器被触发；而此前，无论 `--serials` 如何指定，每个命令都会打开网络上的所有摄像机。 当发生 GVSP 数据包丢失故障时，系统会直接提示该问题，而非归咎于 M8 同步线缆。参见 [多摄像头（瞬态会话）](#multi-camera-transient-sessions)。
> - **`lattice array-disconnect` 会将摄像头保持为自由运行状态，与检测结果一致。** `TriggerMode` `Off`，以及连接检测到的所有其他内容——触发器和同步线设置（`TriggerSource`、 `TriggerActivation`、`TriggerOverlap`、`TriggerDelay`、`LineMode` / `LineSource`），PTP，以及会话中更改的相机设置——因此，对前任成员执行一次针对前成员的-camera命令可一次性获取所有帧，且主控端不再停留在 `TriggerSource` `Software` 状态。`--keep-settings` 将会话设置保存至每台摄像机的内存中，`--clear-settings` 则会重置该内存（参见 [摄像机内存](#camera-memory----keep-settings----clear-settings)）。 `lattice continuous` 还会将检测到处于“武装”状态的相机设置为本次运行的“自由运行”模式（并在运行结束后恢复原状），而没有帧的运行将退出 `1`。
> - **`lattice configure --get` / `lattice exposure`** 会输出一个包含摄像机实际曝光值或增益值。被拒绝的 `lattice exposure --set` / `configure --set` 会显示相机可接受的范围，并将自动模式恢复，而非关闭自动曝光。参见 [传感器调校](#sensor-tuning)。
> - **新相机控制命令：**`lattice highlights`（在RGB显示渲染中降低亮白色表面的亮度）和 `lattice exposure --brightness-auto`。**`lattice multi-detect`**可在一次探测循环中查找有线相机，并将主相机默认设为RGB相机。
> - **新增DAQ命令：** `daq pool-bind`； `pool-enable-ptp`；以及用于 DAQ-E 自身设置的 `pool-info`、`pool-set-network`、`pool-set-auto-stream`、 `pool-reboot` 以及 `pool-update-firmware`。请参阅 [DAQ-E 设备设置](#daq-e-device-settings)。
> - **当 DAQ-E 停止传输数据时，后端会重新连接它。** 当池化 DAQ-E 的数据流中断（例如，为其供电的交换机重启， 摄像机连接中断），后端将使用其原有设置重新连接该设备，并在设备不可用期间以逐渐延长的时间间隔进行重试。该设备上的录制将在同一 `.daq` 中继续进行，文件中会标记间隙， 而 `pool-record --stop` 会列出每个间隙。在 1.2.0 版本中，传感器会保持静默状态，直到 `pool-disconnect` + `pool-connect` 出现，此时录制也会随之结束。 参见 [停止传输数据的 DAQ-E](#a-daq-e-that-stops-delivering)。
> - **`time-sync status`** 报告 PTP 主时钟的数据包是否到达网络（`health`），若未到达，则在标准错误输出（stderr）中输出 `WARNING:`。 参见 [`time-sync`](#chloros-cli-time-sync)。
> - **Linux .deb：** 它会增加摄像头流所需的核心内核套接字接收缓冲区（在内核默认设置下，会导致丢帧和帧率下降），添加一个以您的用户身份运行的后端服务，并由 `lattice network` 检查缓冲区。 CLI 绝不会停止作为服务运行的后端，且升级后会重新启动已启用的后端服务。参见 [将后端作为服务运行](#running-the-backend-as-a-service-headless) 和 [Linux 主机 — 套接字接收缓冲区](#linux-hosts--socket-receive-buffers)。
> - **`lattice array-capture` 会报告无法对齐的情况：**每次捕获数据因未对齐而被保存时（无对齐配置文件、成员分辨率不同、数组未与电缆同步、 帧无法映射到配置文件上），都会作为一条 `Alignment:` 行输出。**`chloros-cli daq help`** 即 `daq --help`（其退出代码为 `2`），在 Linux 系统上，帮助文档中列出了该构建版本所包含的直接硬件命令 。
> - **Jetson：** 该软件包为 `chloros_1.3.0_arm64_jp6.deb`，适用于 JetPack 6，其 GPU 处理在搭载 JetPack CUDA PyTorch 的 Orin 上运行。参见 [Linux arm64 — Jetson](#linux-arm64--jetson-jetpack-6)。

---

## 命名约定

- 所有命令均以 `chloros-cli` 为前缀。在 Windows 系统上，二进制文件名为 `chloros-cli.exe`；在 Linux/Jetson 系统上则为 `chloros-cli`。
- 可选参数表示为 `--flag`。必需的位置参数不带方括号。
- 若提供了默认值，省略该标志即采用该值。
- CLI 是一个基于 Chloros 后端的轻量级 HTTP 客户端（该服务器位于 `127.0.0.1:5000` 上，桌面应用也使用该服务器）。**哪些命令可以启动它：**
  - 核心命令——`process`、`login`、`logout`、`status`、 `export-status`、`time-sync` 和 `selftest` —— 这些命令会使用已运行的后端，若无后端运行则启动一个，并在任务完成后将其停止。
  - **`lattice`、`project` 和 `daq` 命令绝不会启动后端。** 它们需要一个已经正在运行的后端：Windows桌面应用的后端、Linux [后端服务](#running-the-backend-as-a-service-headless)，或者您自己启动的后端（参见 [启动后端](#start-the-backend))。如果没有后端，它们会输出 `Error: Chloros backend not reachable at http://127.0.0.1:5000` 以及启动后端的命令，然后退出 `1`。
- `CHLOROS_BACKEND_URL=<url>` 将 **`lattice`**、**`project`**和**`daq pool-*`** 命令族指向另一个后端（`lattice network-analysis` 则指向 `--backend-url`）。核心命令会刻意锁定 `http://127.0.0.1:<port>` 并忽略它 （IPv4 字面量可避免 Windows 中 `localhost`→`::1` 导致的每次请求约 2 秒的性能开销）。参见 [环境变量](#environment-variables)。
- **相对路径将以运行 `chloros-cli` 的文件夹为基准进行解析。** 后端进程负责打开输入和输出文件夹，因此在发送路径之前，CLI 会将其转换为绝对路径： `-o` / `--output`（包括其默认值 `output`）、`--input`、 `--burst-dir`、`--profile`、`--save` 和 `--mask`，以及 [配方](#recipe-dsl) 步骤中的 `output`（默认值为 `output`），以及 `lattice index --live` 的默认 `output` 文件夹。因此，`-o captures/` 会写入 `<current folder>/captures/`，且打印出的路径均为绝对路径。 `~` 会被展开。对于远程后端（`CHLOROS_BACKEND_URL` 指代另一台机器），路径将按输入形式发送，并在该机器上解析。
- 所有 SDK/CLI 调用均需使用 Chloros+ 账户登录 调用均需使用 Chloros+ 账户登录（每台机器运行一次 `chloros-cli login`；结果缓存于 `~/.chloros/` 中）。
- 示例使用Linux路径；在Windows上，请将 `/home/user/...` 替换为 `C:/Users/.../...`。DAQ-U&#x27;的串口在 Windows 上采用 `COM3` 格式，在 Linux 上采用 `/dev/ttyUSB0` / `/dev/ttyACM0` 格式。
- 语法概述中，互斥标志写为 `{--a | --b}` 表示互斥标志，可选标志则用 `[...]` 表示。

---

## 顶级概述

```
chloros-cli [global options] COMMAND [command options]
```

### 全局选项

| 标志 | 描述 |
| --- | --- |
| `--backend-exe PATH` | 覆盖自动检测到的后端可执行文件。 |
| `--port N` | 后端 HTTP 端口（默认： `5000`）。仅由核心命令使用（`process`、`login`、`logout`、`status`、`export-status`、`time-sync`、 `selftest`）；`lattice`、`project` 和 `daq` 使用 `CHLOROS_BACKEND_URL`（默认的 `http://127.0.0.1:5000`）。|
| `-v, --verbose` | 启用详细输出。 |
| `--restart` | 强制重启后端（终止任何正在运行的 `backend_server.py`）。由 systemd 服务 （Linux .deb）运行的后端绝不会被停止：该命令会改为输出 `sudo systemctl restart <unit>` 并退出 `1`。 |
| `--version` | 显示版本信息（`Chloros CLI 1.3.0`）。 |
| `--help` | 显示顶级帮助。 |

### 命令索引

| 命令 | 用途 |
| --- | --- |
| [`process`](#chloros-cli-process) | 对Survey3或LATTICE捕获数据所在的文件夹进行端到端处理。 |
| [`login`](#chloros-cli-login) | 使用 Chloros+ 账户对本机进行身份验证。 |
| [`logout`](#chloros-cli-logout) | 清除缓存的凭据。 |
| [`status`](#chloros-cli-status) | 显示当前许可证/身份验证状态。 |
| [`export-status`](#chloros-cli-export-status) | 在 `process` 运行期间显示 Thread-4 的实时导出进度。 |
| [`language`](#chloros-cli-language) | 设置或列出CLI显示语言（支持38种）。 |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | 默认项目文件夹（与 GUI 共享）。 |
| [`update`](#chloros-cli-update) | 检查并安装 Chloros 更新（Linux/Jetson）。 |
| [`selftest`](#chloros-cli-selftest) | 系统诊断 + 基本功能测试。 |
| [`time-sync`](#chloros-cli-time-sync) | PTP 主时钟状态/控制。 |
| [`lattice`](#chloros-cli-lattice) | LATTICE 相机控制与采集（45 多个子命令）。 |
| [`daq`](#chloros-cli-daq) | DAQ 光谱传感器控制（DAQ-U / DAQ-M / DAQ-E）。 |
| [`project`](#chloros-cli-project) | 打开并运行已保存的 Chloros 项目（相机 + DAQ）。 |

---

## 安装

`chloros-cli` 包含在所有受支持平台上的 Chloros 桌面安装程序中 ——无需单独下载CLI版本。安装平台包后，`chloros-cli`将与桌面应用程序及其驱动的后端二进制文件一同添加到您的`PATH`中。

最新下载： [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> 安装程序还附带了便捷的启动脚本（`Chloros_CLI.bat` / `Chloros_CLI.ps1`、`Launch_CLI.*`、 `chloros-cli.sh`），这些脚本可直接打开一个可立即使用的CLI命令行界面。

### Windows（.exe）

1. 从下载页面下载 Windows 安装程序。
2. 运行 `Chloros Installer 1.3.0.exe` 并按照向导操作。 默认安装路径为 `C:\Program Files\Chloros\`（CLI 位于 `C:\Program Files\Chloros\cli\`，安装程序会将其添加到 PATH 环境变量中）。
3. 打开一个新的终端（`cmd.exe`、PowerShell 或 Windows Terminal），以便识别更新后的 `PATH`。

```powershell
chloros-cli --version
```

安装程序会自动将 `chloros-cli.exe` 添加到您的系统 `PATH` 中，并捆绑了 LATTICE 摄像头驱动程序。后端位于 `C:\Program Files\Chloros\backend\chloros-backend.exe`（安装文件夹中的 `backend` 文件夹），该组件未包含在 `PATH` 中。

**系统设置。**安装程序的“选择其他任务”页面中，**系统设置**下有两个选项，均**默认勾选**：

| 选项（安装程序中的表述） | 功能说明 | 取消静默安装 |
| --- | --- | --- |
| **启用 Windows 长路径**（允许项目文件夹位于超过 260 个字符的深度；推荐） | 将 Windows 的全局设置 `LongPathsEnabled` 设为 1。Windows 默认禁用此功能， 此时任何路径长度均不得超过 260 个字符；Chloros 产品通常位于项目文件夹下方约 130 个字符处，因此更深层的项目文件夹将被拒绝（参见 [故障排除](#故障排除提示))。此设置仅对安装后启动的程序生效。卸载时保持启用状态，因为其他程序可能依赖此设置。 | `/MERGETASKS="!longpaths"` |
| **将Chloros后端排除在Microsoft Defender扫描之外**（相机捕获速度大幅提升；推荐） | 在安装文件夹中为`chloros-backend.exe`添加一个Defender**进程**排除项，因此实时保护不会扫描后端写入的每个文件，无论其写入位置在哪里。 不会排除其他程序；卸载时将移除此排除项。若 Defender 不存在或被策略锁定，该步骤将静默失败，安装将继续进行。 | `/MERGETASKS="!defenderexclusion"` |**为何 Defender 选项至关重要：**在高捕获速率下，对捕获文件进行实时扫描会消耗约 30% 的捕获速率（基于 9 台摄像头连续捕获的测量结果： 未受Defender影响时为77.7帧/秒，受其影响时为52.7帧/秒）。当`lattice array-capture --continuous`检测到Defender在捕获过程中占用一个或多个CPU核心时，会显示提示信息；请勾选该选项后重新运行安装程序。

静默安装（`/SILENT` 或 `/VERYSILENT`）会同时应用这两项设置，除非选择退出，例如 `"Chloros Installer 1.3.0.exe" /VERYSILENT /MERGETASKS="!defenderexclusion"`。

**Windows 防火墙。** 安装程序会在每个网络配置文件中始终为 `chloros-backend.exe` 添加两条入站规则，而卸载程序会将其移除：`Chloros Backend (UDP-In)`（用于摄像头流、PTP、 mDNS 以及 DAQ-E 数据流；另一条 `Chloros Backend (TCP-In)` 仅限于本地子网，用于 DAQ-E 固件更新，在此过程中传感器会连接回该计算机以采集图像（参见 [DAQ-E 设备设置](#daq-e-device-settings)）。后端的 HTTP API 仍绑定在 `127.0.0.1` 上。

### Linux amd64 (.deb)

适用于 Ubuntu 22.04 LTS 或更高版本 / 基于 Debian 的 x86_64 工作站。

> **不支持 Ubuntu 20.04。** 该软件包的依赖项列表源自
> 后端实际链接的库，其中包括 `libc6 (>= 2.35)`；
> Focal 系统自带 glibc 2.31。 `apt` 会拒绝安装，而非在
> 运行时导致安装失败。 该列表还列出了 `libgl1`，捆绑的 OpenCV 会加载该组件，而精简版
> 服务器镜像可能不包含；`apt install` 提供了该组件。

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
```

该 .deb 包将安装：
- `chloros-cli` 至 `/usr/bin/chloros-cli`，以及 `chloros-backend`（将后端）更新至 `/usr/bin/chloros-backend`
- 已编译的后端和 CLI，解压后分别位于 `/usr/lib/chloros/backend/` 和 `/usr/lib/chloros/cli/`
- LATTICE 相机驱动程序
- 降噪模型、校准包和更新通道配置
- 系统专用 Python SDK：`python3`（`import chloros_sdk`）。`apt install` 还包含其依赖项 `python3-numpy` 和 `python3-requests`（推荐但非必需：上述程序均为自包含程序； `dpkg -i`会跳过这些，安装输出会列出缺失的组件）。在虚拟环境中：`pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`
- `/etc/sysctl.d/60-chloros-gige.conf`，该程序将内核用于摄像头 流的套接字接收缓冲区提升至 32 MB，并立即生效（若该值已设置得更高，则保持原值）。卸载该软件包将删除该文件。参见 [Linux 主机 — 套接字接收缓冲区](#linux-hosts--socket-receive-buffers)。
- 两个用于后端的可选 systemd 服务，均未启用（详见下一节）

#### 将后端作为服务运行（无头模式）

若未以服务形式运行，后端（及其所属的 PTP 主时钟）仅在被启动时才会运行： 在终端中运行 ``chloros-backend``、执行 SDK 脚本，或运行核心 ``chloros-cli`` 命令，其运行时间仅限于该进程存活期间。若要在无头机器上实现后端始终在线，请以 **您的用户身份** 启用该服务：

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` 将以后台该用户身份运行。它写入的捕获数据归该用户所有，其详细日志位于该用户的 `~/.cache/chloros/logs` 中， 并且它使用该用户的 `chloros-cli login`（`~/.chloros`），因此请先以该用户身份登录一次。PTP 无需 root 权限。与所有已安装的后端一样，它仅监听 `127.0.0.1:5000` 端口。

`chloros-backend.service` 以 **root** 身份运行后端。 它之所以仍被安装，是为了确保启用了该功能的机器能持续运行，但 root 后端写入的所有内容都归 root 所有，因此用户无法删除自己的捕获数据。要切换到您的用户：

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

root 服务的详细日志位于 `/var/log/chloros`，`adm` 组的成员可以读取（该服务会设置 `CHLOROS_LOG_DIR`； 参见 [环境变量](#environment-variables)）。安装该软件包时，系统会显示已填入您用户名的 `enable` 命令，并在 root 服务启用时同时显示切换选项。

**启用一个后端单元。** 每次仅有一个后端运行。若启用了第二个单元（例如除 `chloros-backend@<user>.service` 之外的 `chloros-backend.service`，或两个用户的 `@` 单元） 将无法启动，并保持为 `failed` 状态，退出状态为 98；systemd 不会重启它。 其 `systemctl status` 说明了原因：`Another instance is already running. Exiting.`，随后指出占用后端资源的对象（端口 5000，或锁定文件及其所属用户）， 接着是 `systemd will not restart <unit> (exit status 98)`。重启后，这两个单元中的任意一个都可能被选中运行。若需切换，请按上述方法禁用不使用的那个单元。**CLI 绝不会停止服务的后端进程。** `chloros-cli` 会直接使用该后端，这与 GUI 和 SDK 的做法一致：`process` 在其上运行，并加载自己的项目以替换上一个项目。 在CLI通常会重启后端的情况下（例如`--restart`，或导入未找到任何内容的`process`）， 它会指定单元名称，输出 `sudo systemctl restart <unit>` 并退出 `1`， 因为以这种方式停止的后端将保持关闭状态。`GET /api/status` 将单元命名为 `service`（若后端未运行任何服务，则命名为 `null`）。**升级与移除。** 升级软件包会停止所有后端服务，然后依次重新启动每个已启用的服务，首先是 `@<user>` 单元（手动启动但从未启用的单元将保持停止状态，如同重启后一样）。 当启用的服务超过一个时，仅有一个运行，其余服务如上所述保持失败状态；升级过程会输出 `Chloros: N backend units are enabled: …` 和 `Keep one enabled: sudo systemctl disable --now <unit>`。卸载该软件包会停止并禁用这些服务。 彻底清除该组件还会删除 `/var/log/chloros`。

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

与 amd64 .deb 包布局相同，包含接收缓冲区和 [后端服务](#running-the-backend-as-a-service-headless)，适用于运行 **JetPack 6**（L4T R36）的 Jetson Orin / Orin NX / Orin Nano。 其 PyTorch 是 JetPack 6 为 Orin 构建的 CUDA 版本，因此 GPU 处理（包括支持纹理的去拜耳算法）将在 Jetson 的 GPU 上运行。 该软件包本身不包含 CUDA：它使用 JetPack 6 安装的 CUDA 和 cuDNN 库（在缺少这些库的镜像上，请使用 `sudo apt install nvidia-jetpack`）。 若无法使用 CUDA GPU，处理将在 CPU 上运行，且纹理感知模式将使用标准去拜耳算法（参见 [`process`](#chloros-cli-process)）。

### 每台机器仅需认证一次

每个平台都需要先进行一次 Chloros+ 登录，SDK/CLI 调用才能生效：

```bash
chloros-cli login user@example.com 'YourPassword'
```

凭据缓存于 `~/.chloros/user_session.json` 中。

### 启动后端

`lattice`、`project` 和 `daq` 命令用于与已运行的后端进行通信；它们绝不会启动后端（参见 [约定](#conventions)）。使用这些命令时，请确保有一个后端正在运行：

| 平台 | 方法 |
| --- | --- |
| Windows | 打开 Chloros 桌面应用（其后端同时也为 CLI 提供服务），或在独立终端中运行 `"C:\Program Files\Chloros\backend\chloros-backend.exe"`（位于安装目录下的 `backend` 文件夹中），并保持该终端打开。 |
| Linux / Jetson | 在单独的终端中运行 `chloros-backend`，或通过 [后端服务](#running-the-backend-as-a-service-headless) 使其始终保持运行。 |

每次仅能运行一个后端；所有客户端（GUI、CLI、SDK脚本）均共享该后端。 核心命令（`process`、`login`、`status`、…）也会使用它，并在没有后端运行时启动一个。

**后端会在命令之间持续运行**，因此后续命令无需再次支付启动费用。 当您注销时（`chloros-cli logout`，除非Chloros应用处于打开状态，或正在进行运行或离线构建），或执行`chloros-cli backend stop`时，该进程将停止；若30分钟内无任何组件使用它，它也会自动停止： 无命令、脚本或应用，无运行任务或离线构建，无连接的摄像头或光线传感器。当运行任务或离线构建正在进行时，`backend stop` 会拒绝停止，并指出具体任务名称；`--force` 会立即停止该任务，但该运行任务或构建将丢失 （会保留一个数据包，并由 `array-build-video` 重新构建）。 在后端启动前设置 `CHLOROS_BACKEND_IDLE_MIN` 以更改 30 分钟的时限（`0`：永不）；[后端服务](#running-the-backend-as-a-service-headless) 绝不会自行停止。`chloros-cli backend status` 会显示是否有后端正在运行、哪些进程正在使用它，以及是否有任何离线构建正在运行。`process` 会在运行中的后端中打开其项目，除非 Chloros 应用已打开并包含一个 ：此时会拒绝操作，您需先通过应用处理或关闭该项目。由先前运行的 `process` 或 SDK 脚本留下的未关闭项目将被直接替换。

### 验证安装

```bash
chloros-cli --version           # prints "Chloros CLI 1.3.0"
chloros-cli selftest            # 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

`selftest` 仅在通过全部七项检查后才会退出 `0`。最后一项检查需要 CUDA GPU（NVIDIA GPU、 或 Jetson Orin 的 GPU）以及去噪模型，因此若计算机未配备 GPU，则会显示 `Failed: CUDA + Denoiser` 并退出 `1`；除仅限 GPU 处理的纹理感知去拜耳算法外，其余功能在此环境下仍可正常运行。

> **需订阅 Chloros+。**命令行界面（CLI）要求拥有有效的 Chloros+ 套餐。**Copper**是 Chloros+ 的入门级套餐——所有付费的 Chloros+ 套餐均提供 CLI/SDK 访问权限；仅免费的**Iron** 套餐不提供。（套餐 ID 映射：`0`=Iron/免费， `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) 请在 [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing) 处升级。
>
> 此下限由后端强制执行，而不仅仅是CLI：任何未订阅付费套餐且通过SDK/CLI标志发起的请求，无论来自`chloros-cli`、Python SDK还是自定义的HTTP客户端，都将因`403 PLAN_UPGRADE_REQUIRED`被拒绝。 已注销的调用者则会收到 `401 AUTH_REQUIRED` 错误。在套餐的宽限期内（月付套餐为 30 天，年付套餐至到期日），离线访问仍可正常进行 ，过期后即停止；而 `chloros-cli status` 仍可正常工作，因此原因可见（这是唯一一个不受分级限制的 SDK/CLI 路径——`GET /api/license-status`）。

---

## 五分钟快速入门

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI GNDVI OSAVI

# 3. The lattice / daq commands need a running backend: on Windows open the
#    Chloros desktop app; on Linux start one (or see "Start the Backend")
chloros-backend &

# 4. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 5. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 6. Read a spectrum from a connected DAQ-U (Windows: --port COM3)
chloros-cli daq pool-connect --port /dev/ttyUSB0
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

`process`（步骤 2）会在没有后端运行时为其自身运行启动一个后端，并在运行完成后停止该后端，因此请从步骤 4 开始启动一个后端。 请参阅 [启动后端](#start-the-backend)。

---

## `chloros-cli process`

将图像文件夹通过完整的 Chloros 管道进行处理（目标检测 → 校准 → 晕影处理 → 反射率分析 → 指数导出）。

### 概述

```
chloros-cli process INPUT [OPTIONS]
```

### 位置参数

| 参数 | 描述 |
| --- | --- |
| `INPUT` | 包含 `.raw + .jpg`（Survey3）、`.tif/.tiff`（LATTICE）或 `.dng` 文件的输入文件夹路径。 |

### 常用选项

| 标志 | 默认值 | 说明 |
| --- | --- | --- |
| `-o, --output PATH` | 在默认项目文件夹中创建一个名为 `-n` 的新文件夹 | 项目文件夹。 项目将以此命名；若某个文件夹中已存在名为 `project.json` 的文件，则会生成名为 `_1` / `_2` 作为并列文件夹，而非被覆盖。默认项目文件夹为 `set-project-folder`（或 GUI）所选的文件夹，否则为 `Documents/MAPIR/Chloros Projects`。 |
| `-n, --project-name NAME` | 自动（时间戳） | 项目名称，仅在**不包含** `-o` 时使用（若包含 `-o`，则项目名称采用文件夹名称）. |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` 针对 Survey3 图像使用 Chloros+ 神经网络去马赛克算法；速度较慢但质量更高。该功能仅适用于 Survey3： LATTICE 捕获数据始终使用标准去马赛克算法，因此对于包含 LATTICE 捕获数据的文件夹，`process` 会在导入后拒绝 `texture-aware`，并显示一个指明文件数量的错误信息，随后退出 `1`，且未进行任何处理（Chloros应用对此类图像也不支持“纹理感知”功能）。该功能需要CUDA GPU（Windows/Linux x64系统上的NVIDIA GPU，或Jetson上的Orin GPU）： 若未配备，运行时将使用 `standard`，且后端日志中会显示一行说明原因的 `[DEBAYER] … needs a CUDA GPU …` 信息（如图所示为 `--verbose`）。 |
| `--vignette / --no-vignette` | `--vignette` | 暗角校正。 |
| `--reflectance / --no-reflectance` | `--reflectance` | 反射率校准（若找到面板目标则使用，LATTICE 采用 NIST 按序列校准）。对于 LATTICE 多光谱数据，此步骤同时作为反射率**乘积** 切换选项 — 参见 [按产品导出切换选项](#per-product-export-toggles-lattice-multispectral)。 |
| `--ppk` | 关闭 | 应用来自 sidecar 文件的 PPK GNSS 校正。 |
| `--exposure-pin-1 CAMERA_MODEL` | 无 | 连接至曝光引脚 1 的相机型号，用于 PPK 时间同步。 |
| `--exposure-pin-2 CAMERA_MODEL` | 无 | 连接至曝光引脚 2 的相机型号，用于 PPK 时间同步。 |
| `--recal-interval SECONDS` | `0` | 最小重新校准间隔，单位为秒（即项目的 *最小重新校准间隔*）。 |
| `--timezone-offset HOURS` | `0` | 光传感器相对于 UTC 的时区偏移量，单位为小时（即项目的 *光传感器时区偏移量*）。 |
| `--format FORMAT` | `TIFF (16-bit)` | 以下选项之一：`TIFF (16-bit)`、 `TIFF (32-bit, Percent)`、`TIFF (8-bit)`、`PNG (8-bit)`、`JPG (8-bit)` 中的一个。Chloros 绝不会提高图像的位深度： 对于所有 8 位图像，更高位深的格式将写为 `TIFF (8-bit)`，且运行说明中已明确指出这一点。 在同时包含 8 位和 12/16 位采集数据的项目中，每份采集数据均按其自身位深度或更低位深度导出。Radiance 始终为 32 位浮点数。 |
| `--indices NAME [NAME ...]` | 无 | 按预设名称生成的植被指数 (`NDVI`、`GNDVI`、`OSAVI`、`SAVI`、 `EVI`, `GCI`, …)，不区分大小写。每个指数将根据其摄像机的波段添加到项目中，并包含一个新项目。若名称不是预设，或摄像机的波段无法生成 （例如`NDRE`需要RE和NIR波段，而单台相机无法同时提供这两种波段），该名称会在警告中列出，且不会生成索引图像。 |
| `--input-level {auto,raw,debayered,processed}` | `auto` | 强制 LATTICE TIFF 文件的处理管道入口点（Survey3.raw 不受影响）。 此外，还提供了一个“逃生通道”，允许对**无原始数据**的捕获进行处理——参见 [捕获文件夹的结构](#what-a-captures-folder-looks-like)。 |
| `--debayered / --no-debayered` | 启用 | 输出线性去拜耳化结果（`Debayered_Images`）。参见 [按产品导出开关](#per-product-export-toggles-lattice-multispectral)。 |
| `--preview / --no-preview` | 启用 | 输出显示预览（`Preview_Images`）：RGB = 白平衡（如有可用，则采用DAQ光源；否则采用灰度世界）+ 伽马校正；多光谱 = 伪彩色拉伸。 |
| `--radiance / --no-radiance` | 启用 | 输出 float32 辐射度（`Radiance_Images`，W/m²/sr/nm）. |
| `--reflectance-source {daq,target,auto}` | `auto` | LATTICE反射率产品的参考标准： `auto` = 通过质量保证（QA）的帧内目标值是绝对参考（带有 QA 标志的目标值会与 DAQ 的测量分数竞争），若 DAQ 不可用则采用下行辐射（ρ = π·L/E）作为备选方案； `target` = 严格模式（不进行DAQ替换）；`daq` = DAQ权威模式。参见 [按产品导出切换](#per-product-export-toggles-lattice-multispectral)。 |
| `--target-reflectance-dir DIR` | 无 | 按单元划分的**实测**目标反射率扫描目录（`<serial>.csv`）； 若未检测到，则回退到标称的 T3/T4P 光谱。|
| `--array-alignment / --no-array-alignment` | 启用 | LATTICE 阵列： 将每个捕获数据中 `Chloros:Alignment*` XMP 标签所标记的模块间对齐信息应用于所有处理后的产品（去拜耳化/预览/辐射度/反射率/指数）。对于没有这些标签的图像，此操作不生效。 |
| `--array-alignment-crop / --no-array-alignment-crop` | 裁剪 | 将对齐后的导出图像裁剪至阵列的公共重叠区域，以便所有模块共享同一覆盖范围；`--no-…` 则保留完整的传感器画布（源区域外填充黑色）。 |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | 针对对齐变形进行重采样。`nearest` 保留源图像的精确动态范围（DN）值（不进行辐射值像素间混合）。 |

### 目标检测选项

| 标志 | 描述 |
| --- | --- |
| `--min-target-size PIXELS` | 探测器校准目标的最小像素尺寸。 |
| `--target-clustering 0-100` | 目标聚类百分比（默认值为 `60`）。 |
| `--target / --targets` | 仅在输入文件的 `target` 或 `targets` 子文件夹中搜索校准目标（速度更快）。 |

### 示例

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a project folder of your choice (the project is named after it)
chloros-cli process "C:/input" -o "C:/projects/Field_A_2026-05-26"

# Name the project and let it go to the default project folder
chloros-cli process "C:/input" -n "Field_A_2026-05-26"
```

### 按产品设置的导出开关（LATTICE 多光谱）

LATTICE 处理会在 **单次运行中生成所有适用的产品**。四种按类型设置的开关——`--debayered`、 `--preview`、`--radiance`、`--reflectance`——均**默认开启**；若需禁用其中一项，请使用 `--no-<type>` 格式。 RGB主摄像头仅输出去拜耳化后的预览图像（不包含各波段的辐射度/反射率）， 因此对于它们而言，`--radiance`/`--reflectance` 均为无操作。对于 Survey3 的 `.raw`（其遵循标准的反射率/目标路径），这些开关将被忽略。*（旧的 `--radiometric-output {reflectance,radiance,sensor-response}`标志已被**移除**并由这些开关取代；现已不再存在`sensor-response`级别。）*

| 产品 | 输出 | 是否需要DAQ下行数据？ |
| --- | --- | --- |
| `--debayered` | 线性去马赛克（`Debayered_Images`）：与捕获在 `debayered` 级别时保存的文件相同 ——即对原始数据应用校准包的暗偏移（DSNU）校正，随后采用 OpenCV 的边缘感知去马赛克处理；不包含线性查找表（LUT）、平场校正或解混色。 | 不需要 |
| `--preview` | 显示预览（`Preview_Images`）：RGB = 白平衡 + 伽马校正；多光谱 = 伪彩色拉伸。 | 否。 |
| `--radiance` | 来自完整辐射测量链（`Radiance_Images`）的 float32 单位（W/m²/sr/nm）。 | 编号 |
| `--reflectance` | uint16 反射率 ρ（`32768` = 1.0）， 支持Pix4D。 | **是**，除非有通过质量检查的帧内目标将其锚定（见下文）。 |

`--reflectance-source` 选择反射率参考：**`auto`**（默认）将通过质量保证（QA）检测的帧内目标设为**绝对参考**——以目标锚定的经验线链会在保留的面板上进行交叉评分，并按波段逐一应用测得的优胜结果：对于DAQ无法校准的波段，或其DAQ斜率拟合失败的波段，将采用目标自身的线 （由`ReflectanceElBandMethods`指定）——若目标虽通过质量保证但带有标记（如某面板被剔除、某波段顺序错误），其测量得分将与DAQ分界值进行比对，若该帧得分低于目标，则该帧被标记为`ReflectanceChain=daq` — 当无目标存在或质量保证失败时，回退至 DAQ 下行阈值（ρ = π·L/E）；**`target`**采用严格模式（不进行 DAQ 替换）；**`daq`**选择采用 DAQ 权威行为。目标几何参数（ArUco / 固定 ROI / 条带）来自项目目标配置；`--target-reflectance-dir DIR` 保存每个单元的**测量**扫描（`<serial>.csv`），通过目标单元的序列号/QR进行查找，并以标称T3/T4P光谱作为备用方案。

DAQ反射率路径会从记录的**`.daq`**（DAQ-U/M/E）**或与影像一同发现的 DAQ-M 原生 `.csv`**中自动解析出**时间戳匹配的向下辐射**。如果未在本地缓存按相机或 DAQ 分类的校准包， 管道将在首次使用时**从 AWS 自动获取**（需连接互联网一次；缓存于 `~/.chloros/` 中）。**该读数属于哪台相机。**捕获文件保存的 `.daq` 数据即为其所描述的相机（或多台相机）的读数——相机名称在旁边的 `<file>.daq.cameras.json` 辅助文件中指定 （`lattice capture`、`multi-capture`、 `array-capture` 及 GUI 均是如此；原始连拍会在其清单中列出每个成员的读数）——而 `process` 仅将其与**来自这些摄像头的图像**。因此，没有自身读数的独立摄像头将不再借用其他摄像头的数据采集（DAQ）；请为其绑定一个数据采集（DAQ），或进行一次录制。数据采集（DAQ）自身的录制 （`daq pool-record`，或在图形用户界面中启动的录制），或任何不包含该辅助文件的 `.daq`（包括较早的捕获文件），仍可服务于所有相机。

#### 读取反射率像素（Pix4D / Metashape / 您自己的脚本）

反射率以整数 DN 形式存储，且 **表示 ρ = 1.0 的 DN 值取决于源相机**：

| 源相机 | ρ = 1.0 对应的值 | 如何判断 |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768`（ρ 值上限为 2.0） | 文件中带有 XMP `Chloros:PixelScale=32768` 标记。 |
| Survey3 | `65535`（在 ρ 1.0 处被截断） | 没有 `Chloros:*` XMP 标签——这种缺失*本身*就是信号。 |

**读取 `Chloros:PixelScale` 并以此为基准进行除法**，而不是假设一个常数。 该标签在 uint16 域中定义，因此即使在进行比例缩放的输出格式中，其值仍保持不变——`32768`、`TIFF (16-bit)`、`TIFF (8-bit)`、 `PNG (8-bit)`、`JPG (8-bit)` 和 `TIFF (32-bit, Percent)` 都是自描述的 （首先将存储的数据类型归一化回 uint16：8 位数据乘以 257，浮点数乘以 65535）。

> **旧版本构建生成的文件可能不包含比例信息。** 一个 8 位反射率 TIFF 文件会被重新缩放，并像其他文件一样标记为 `Chloros:PixelScale`。1.3.0 之前的构建版本则将其*裁剪*为 0..255，因此所有大于 ρ≈0.008 的值都被压平为 255， 并且它们同时省略了 `Chloros:PixelScale` 和 `MicaSense:RadiometricCalibration` 元组。**如果 LATTICE 反射率文件中缺少该标签，请勿假设存在比例尺——应重新处理该文件**，而不是对原本无法被整除的像素进行除法运算。

#### 读取索引像素（NDVI及其他植被指数）

**整数索引栅格是对每帧图像进行的对比度拉伸，而非−1…1的映射。** `TIFF (16-bit)`、`TIFF (8-bit)`、 `PNG (8-bit)` 和 `JPG (8-bit)` 指数产品——包括 8 位TIFF（即 8 位捕获数据所写入的格式）——存储

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

其中 `IndexMin` / `IndexMax` 分别是 **该帧自身的最低和最高有限索引值**。 DN 0 并非 −1，满量程也并非 +1。

Chloros（Chloros）会在两个端点上都打上标记，以确保映射保持可逆：

| 标签 | 含义 |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — 如下所示的约定 |
| `Chloros:IndexMin` | 作为 DN 0 存储的索引值 |
| `Chloros:IndexMax` | 作为满量程存储的索引值 |

读取这些数据并进行反转，首先根据存储数据类型的最大值将存储的 DN 归一化为 0…1，以便一个表达式能覆盖所有整数格式：

```python
import json, subprocess
import numpy as np, tifffile

tags = json.loads(subprocess.run(
    ["exiftool", "-j", "-XMP:all", path],
    capture_output=True, text=True).stdout)[0]

# float(): exiftool hands back a negative real as a quoted string, so
# IndexMin arrives as str on most real NDVI data while IndexMax is a number.
lo, hi = float(tags["IndexMin"]), float(tags["IndexMax"])

raw = tifffile.imread(path)                    # uint16 for TIFF (16-bit)
ndvi = lo + raw.astype(np.float64) / np.iinfo(raw.dtype).max * (hi - lo)
```

读取时无需 `-config pix4d.config` —— exiftool 可以独立读取这些 XMP 属性。该配置仅在*写入*时才需要。

> **`-config pix4d.config`** 表示该帧的每个像素都具有相同的索引值。栅格填充在中间量程，无需除以任何数值——直接取端点值即可。

> **没有 `Chloros:Index*` 标签意味着未声明映射关系。** LUT 导出是一种着色渲染，其像素为根据您的 LUT 阈值窗口生成的渐变色；而没有有限索引像素的帧则无需声明范围。 无论哪种情况，都不要假设存在映射关系——这与缺失 `PixelScale` 的情况遵循相同规则。**由于拉伸是按图像进行的，因此同一飞行任务的两帧图像在数字数值（DN）上不可比**——每帧都有其自身的端点，因此相同的NDVI值在不同图像中对应的DN值会不同。若需对整个飞行任务进行定量分析（如分区统计、固定阈值、时间序列），请导出**`TIFF (32-bit, Percent)`**：索引栅格将以 float32 格式写入其中，即**索引值本身**，无需重新缩放且无需标签。尽管格式名称如此，索引产品并非 ×100 —— 该缩放仅适用于整数数据类型的反射率。

#### 解混剪切掩膜（记录了哪个波段在哪些像素上被剪切）

3×3 光谱解混可能会导致某个波段变为负值——例如当光线中几乎不包含该波段，或者场景超出了相机解混矩阵描述的范围时。 这并非真实值，因此导出时将其固定为 0 —— 实时预览中也显示为 0。一个波段可能仅在少数像素上被钳位，也可能在几乎所有像素上被钳位；钳位掩膜会记录这两种情况。 `Chloros:Band{R,G,B}UnmixClampFraction` 表示该现象在帧中占了多大比例；钳位掩码则表示**哪些像素**：

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— 位于产品文件夹**旁边的** `clamp_masks/` 文件夹中，绝不会位于其内部（`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`； `process`导出文件的`tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`）——对于每个辐射度、 反射率及折射率数据集，其中至少有一个像素被裁剪：包括捕获和导出的 `process` 文件（以及 `array-build-video --save-tiffs` 的每帧 TIFF 文件）。单波段 `uint8`， 基于该产品自身的像素网格（包含方向、ROI、像素合并和数组对齐）。每个掩码都会标明其对应产品的相机（`Make` / `Model` / 序列号）、拍摄时间及镜头标签，并列出其位所代表的波段 （`XMP-Camera:BandName`，从第 0 位开始）；其 `ImageDescription` 和 `Software` 标记表明其为钳位掩码，且不包含校准信息。
- **位**：第 0 位 = 产品 R 通道中的波段（`Chloros:BandRName`），第 1 位 = G（`BandGName`），第 2 位 = B （`BandBName`）。若位被设置，则表示该波段中该像素的值已被钳制为 0 —— 请将其以及由此构建的任何索引视为低可信度。
- **`Chloros:UnmixClampMask`** 标记在每个运行过解混操作的产品上：蒙版相对于产品自身文件夹的路径（`../clamp_masks/<product folder>/<product name>_clampmask.tif`， 始终以 `/` 分隔——将其与图像文件夹拼接即可打开蒙版），若无像素被限制，则为 `none`。 无标记表示该产品从未运行过解混操作（RGB或单色相机，raw/去拜耳化级别）。
- `lattice capture` / `array-capture` 会在保存文件下方打印蒙版路径 （`clamp mask: …`），且捕获结果列表中将其列为帧的 `clamp_mask`。
- `lattice index --live --save-multiband` 为多波段堆叠的每个源摄像头写入一个蒙版 — 参见 [`lattice index` — 多波段堆叠蒙版](#lattice-index--multiband-stack-masks)。视频不包含遮罩：`array-build-video` 会报告每帧的钳位比例（参见 [`array-build-video`](#array-build-video--offline-reprocess-a-saved-burst)）。

由于任何层级的产物文件夹中均不包含掩膜，因此任何产物文件夹（`radiance/`、`reflectance/`、 `process` 导出文件中的 `*_Images/` 文件夹）均可直接作为 ——包括子文件夹在内——而这些软件仅接受图像文件。请直接提供该产品文件夹，而非其上级文件夹（该上级文件夹还包含 `clamp_masks/` 以及其他所有导出类型）。Chloros 自身的导入功能会跳过蒙版。读取蒙版： `(mask & 1) != 0` 表示“此处的 R 通道被限制”，`& 2` 表示 G 通道，`& 4` 表示 B 通道。

#### 随导出保留的 EXIF 信息

`process` 会将源文件的 **GPS 块及其 ExifIFD** 复制到每个生成文件中，因此
导出文件会包含 `FocalLength`、`FNumber`、`ExposureTime`、`ISO`、 `DateTimeOriginal` 和
`CameraSerialNumber` 以及地理参考信息。

**`FocalLength` 对摄影测量而言并非可选。** Pix4D 根据
焦距和高度来计算地面采样距离；若缺少该标签，系统将退而采用一个严重错误的比例尺。在一次
拍摄了 49 张照片的橘园飞行任务中，由于缺少该标签，原本 411 米 × 160 米的场地被重建为
47.8公里×13公里——这张455 MP的正射影像中大部分区域显示为“无数据”，在有人检查GSD之前，这被误判为
拼接问题和BigTIFF问题。如果您的正射影像生成的比例尺
不合常理， 请先对导出的产品运行 `exiftool -FocalLength` 命令。

此副本特意**未**使用 `-all:all`：IFD0&#x27;的结构标签在
复制时会破坏 LATTICE 的输出结果；而 `ExifImageWidth` / `ExifImageHeight` 被排除在外，因为它们描述的是
*源*图像——否则，任何曾被调整过尺寸的导出文件都会携带与
与自身栅格相矛盾。XMP是直接写入而非复制，因为当XMP块被复制时，ExifTool
会丢弃同一调用中的XMP标签（这将导致MAPIR
校准标签丢失）。

### 输出文件的存储位置

生成文件将写入**项目文件夹下，按相机分组，再按文件格式分组**：

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

LATTICE 的相机文件夹为 `LATT-<sensor>-<lens>-F<filter>`（与拍摄文件的 EXIF
`Model`），Survey3的相机文件夹为`<model>_<filter>`——两台共享同一传感器和滤镜但
镜头不同的相机将保持独立的文件树，因为其暗角、视场和畸变存在差异。格式
文件夹结构如下：`--format`：`tiff16`、`tiff8`、`png8`、`jpg8`，或 `tiff32` 对应
`TIFF (32-bit, Percent)`。

> **每个导出文件均保留源文件的名称。** `capture_…_raw.tif`的辐射度导出文件
仍名为 `capture_…_raw.tif` —— 它只是位于
> `tiff32/Radiance_Images/` 目录下。**文件夹而非文件名用于标识该产品**， 因此使用通配符
> 搜索 `*radiance*.tif` 不会找到任何结果；请改用目录进行匹配。

### 光传感器记录 — 已校准的 `.daq` + `.csv`

`process` 还会处理输入文件夹中的 `.daq` 记录，且**无需**
需要任何图像数据：单独飞行的 DAQ-U / DAQ-M / DAQ-E 即可构成完整的
采集数据，而仅包含 `.daq` 文件的文件夹也是有效的输入。

DAQ 设备**无需**校准即可进行记录——这正是公开版
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) 记录器
(`record_daq.py`) 的默认行为：它们写入原始传感器计数并为文件添加时间戳，以便
Chloros **通过串行**（优先从本地缓存，
然后从MAPIR云端）获取该传感器的出厂校准值并应用。`process`将结果写回：

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv` 每条读数包含一行数据：UTC 时间戳、积分时间、总功率、
明视/暗视勒克斯、PPFD（及其蓝/绿/红分量）、峰值波长，然后是
基于传感器自身波长网格的全光谱数据。`.csv`文件在无需
二次校准的情况下重新导入。

`<name>`即为该记录本身的文件名。 位于不同文件夹中的两个记录可以共享
同一组——由两台相机在
同一毫秒内保存的每次捕获读数（`<capture>/daq/daq_data_<sensor id>_<ts>.daq`）——且每个记录仍会生成各自的输出结果：第一个保留
`<name>`，下一条则在文件名**之后**添加区分其所属文件夹的标识
(`<name>__<capture>_daq_calibrated.daq`)，因此录制文件的日期和时间仍位于最前。

运行成功时，报告结果为 `Light-sensor products written: N (calibrated .daq + .csv)`。
括号中的内容描述了实际写入的内容，因此读作
对于无文件包的传感器，显示为
`(RAW COUNTS — this sensor has no calibration bundle)`；对于同时包含两者的文件夹，则显示为
`(N calibrated, M raw counts)`。 后端自身的
`[DAQ-EXPORT]` 和 `[RUN-SUMMARY]` 标题也是以同样的方式得出的——这
三者均不能将原始导出数据称为“已校准”。

如果 DAQ-U / DAQ-M / DAQ-E 记录的校准包无法获取——例如您
处于离线状态，或者该传感器没有存档的校准数据——则该记录将 **会因具体原因**被跳过，并在
`[DAQ-EXPORT]` 行中记录原因，绝不会作为包含原始计数的“已校准”文件写出。
请连接互联网并重新运行。跳过原因即为读取器实际
针对该文件确定的原因（模式不可读、无校准包、写入错误），且运行
摘要会列出**不同的**原因——因同一原因被跳过的二十个文件会被视为一个
原因，而非该原因的二十次重复。 CLI会在运行结束时列出每个被跳过的记录，
并附上相应原因：

```
⚠ Light-sensor recording not exported: <file>.daq: no calibration bundle available for …
```

一个未导出**任何**记录的文件夹会导致运行失败：参见
[未写入任何图像的运行会失败](#a-run-that-writes-no-images-fails)。

名称以 `_calibrated` 或 `_raw`（`field_raw.daq`）结尾的录制文件**不**：**这些是 `process` 为其自身生成结果所赋予的名称，若将生成结果
作为录制文件读回，将会被重复计数。 CLI会明确提示这一点，并显示文件名——
`Not imported as a light-sensor recording: field_raw.daq … Rename the file to import it.`
——而非报告文件夹为空。请在处理此类记录前将其重命名。

#### DAQ-A 记录以原始计数形式导出

**DAQ-A** 系列早于按序列号分捆的系统，且没有可获取的校准包
——它是在现场通过反射率标靶进行校准的，因此
从未需要校准包。若拒绝这些记录，将导致其
数据完全无法导出，因此它们会以**不同名称**导出：

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

采用不同的文件名而非文件内部标记，是因为该信息必须能够
在仅以纯文本名称通过电子邮件转发时保持完整。 `.csv` 文件头声明
`raw spectral sensor counts (NOT irradiance)`，并警告说这些值仅在
**文件内部**可比——这正是基于目标校准的使用目的——
而非跨传感器可比。 与功率相关的光度学列（总功率、明视和
暗视勒克斯、PPFD）被写为 **NULL**，而非通过计数积分得出，且运行
摘要显示为 `RAW COUNTS`，因此日志中“导出”字样不能被解读为辐照度。

旧版 **v1.01 / v1.02** 记录（由 DAQ-A-SD 生成）不包含每读数的时戳，
仅包含文件的写入时间。 图像↔下行光匹配器仍然拒绝这些数据——将
帧与写入时间进行匹配会导致隐性错误——但导出器可以读取它们，且
CSV文件会输出 `clock=daq_created_on`，因此该产品会标明其所属的时钟。

### 注意事项

- `process` 会自动检测您的文件夹是 Survey3、LATTICE 还是混合类型。
- 进度通过服务器发送事件（Server-Sent Events）流式传输；CLI 会显示各线程的实时进度（检测中、分析中、处理中、导出中）。
- 对于 Linux/Jetson，CLI 会检查交换空间，并在处理大型文件夹前可能发出警告。支持纹理识别的去拜耳算法还会在低功耗 Jetson 设备（Nano、Orin Nano）上自动应用 GPU 频率限制。
- 运行成功时，会报告写入的图像产品数量（`Image products written: N`；若去混色过程中对任何像素进行了裁剪，则显示为 `(plus M clamp masks)`）。
- **`process` 可安全地共享正在运行的后端。** 它仅在后端处于空闲状态（即加载了先前项目且未持有任何摄像头、数组或数据采集设备）时，才会自动重启该后端。 若后端持有上述任何资源（例如 GUI 正在流式传输数组、SDK 脚本或 DAQ 正在录制），则直接使用该后端，绝不重启，因此 `process` 可在实时数组运行时并行运行。如果后端报告有运行任务正在进行，`process` 会等待最多 15 秒直至其完成（刚结束的运行大约需要一秒钟才能清理完毕），随后会因 `Backend is mid-run (processing). Wait for it to finish, or pass --restart to force a fresh backend (this kills the in-flight run).` 报错而拒绝。全局标志 `--restart` 仍会强制使用新的后端，并终止其上的任何相机、数组或 DAQ 会话。由 Linux systemd 服务运行的后端绝不会被重启： `process` 正在使用它，而 `--restart` 则会输出 `sudo systemctl restart <unit>`（参见 [将后端作为服务运行](#running-the-backend-as-a-service-headless)）。Chloros应用自身的后端也是如此：当应用处于打开状态时，`process`会使用该后端，并加载其项目以替代应用的项目（若在此位置启动后端，将导致已打开的应用无法访问该后端）。
- **正在运行的处理任务会保留其项目。** 当处理运行正在进行时——无论是由应用程序、`process`、SDK脚本还是`chloros-mcp`启动的——后端都会拒绝创建、打开、导入、添加或删除文件，以及移动、 删除或更改项目的处理设置，并返回HTTP 409 `PROCESSING_IN_PROGRESS`和`A processing run is in progress on project '<name>', so ... now is refused`；若收到`process`请求，则退出`1`。显示设置（缩略图大小）仍可保存。处理模式和项目的目标反射率设置也会以同样的方式被拒绝（运行将保持其开始时的模式）。 运行过程中保存到项目中的捕获数据（无论来自应用程序、CLI、SDK还是`chloros-mcp`），都会按常规方式保存，但仅在运行结束时才会加入项目的文件列表； 其响应信息中已明确说明这一点（`project_add`），在项目中停止的光传感器记录也是如此。此前，第二个客户端可以在运行过程中切换已打开的项目，且正在运行的任务会被导出到另一个项目中。
- **Ctrl+C 会停止后端中的运行，而不仅仅是CLI。** CLI会请求后端停止处理，输出 `Waiting for the backend to stop (Ctrl+C again to stop waiting)...`，并等待最长 120 秒以获取 `Processing stopped.`，因此 `chloros-cli process … && next` 不会在仍处于运行状态的后端上启动 `next`。再次按下 Ctrl+C 将终止等待。随后 `process` 将退出 `130`，如同任何被中断的命令一样。

#### 未写入任何图像的运行会失败

如果您请求了产物，而运行结果为**无**——仅输出 `project.json` 和
`calibration_data.json` ——则 `process` 会将其视为失败：它会输出
`Processing finished but wrote no image products.` 并**以非零状态退出**，因此脚本可以
检测到此情况。 该消息会列出项目文件夹名称及常见原因：

- 输入文件夹未被识别为采集文件夹（请检查布局和 `--input-level`），或者
- 所有请求的产品均因不适用于这些相机而被跳过（例如，请求
  从仅支持RGB的相机获取辐射度/反射率）。

请使用 `--verbose` 重新运行，并检查后端日志中是否包含 `[LATTICE-EXPORT]` / `[EXPORT-CHECK]` 相关行，
这些日志会说明按相机跳过的具体情况，而这些信息通常不会显示在CLI的输出中。

一次刻意仅处理元数据的运行——所有生成项均关闭且不使用 `--indices`——仍然是
**成功的**，因为在此情况下输出空图像才是正确结果。

#### 无法验证导出文件的运行将失败

运行结束后，`process` 会使用 ExifTool 检查导出文件的标签。如果 ExifTool 无法运行，它
将输出 `导出验证失败：ExifTool 无法运行，因此 M 个导出文件中有 N 个的标签
未被检查（请参见后端日志中的 [EXIFTOOL] 行）` and `处理结果已写入并保留
在原位置；请重新运行以验证结果。`, does not print `处理完成！`，并**以非零状态退出**。
处理结果将被保留。

#### 捕获运行所需的时间

在运行 `process` 之前完成的捕获会被包含在内。运行期间保存的捕获则不会被包含；`process`
会提示有多少捕获被遗漏，再次运行即可将其包含进来。

#### 丢失图像的运行将失败

如果运行过程中某个图像出现故障，`process` 将输出 `N image(s) failed and were not exported` 并注明
最常见的原因，不会输出 `Processing complete!`，并且 **以非零状态退出**。 其他生成文件
会被写入并保留在原处，后端日志中会记录每次失败。

**仅光传感器运行**也是如此： 根据定义，包含 `.daq` 记录的文件夹中没有可导出的图像，
因此该运行将根据其生成的已校准文件 `.daq` / `.csv` 进行评估。
如果它未写入**任何**数据——即所有记录均被跳过——则会输出
`Processing finished but wrote no light-sensor products.`，并为每个记录命名，其中包含
被跳过的具体原因，同时**以非零状态退出**。

---

## `chloros-cli login`

使用 Chloros+ 云账户对本机进行身份验证。凭据将安全地缓存于 `~/.chloros/user_session.json` 中。

```
chloros-cli login EMAIL PASSWORD
```

### 示例

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$`（从密码中删除或复制部分内容）。遇到 401 错误时，CLI 会自动重试，先使用 `$$` 重新，随后使用去重后的密码一半进行重试；若重试成功，系统将登录并显示下次应使用的正确单引号语法。

> **无界面/脚本使用：无缓存会话意味着会出现登录提示。** 核心命令（`process`、`status`、 `export-status`、`time-sync`、…）在无缓存会话的情况下运行时，会在继续执行前通过标准输入（stdin）请求 `Email:` / `Password:`。 若标准输入已关闭（如 cron 或 systemd 单元），程序会立即因 `login cancelled` 报错；若标准输入为打开的终端或管道，则会等待输入。在调度无头任务前，请在每台机器上运行一次 `chloros-cli login EMAIL PASSWORD`。

---

## `chloros-cli logout`

清除缓存的会话，并在下次调用时强制重新登录。

在处理运行进行中注销不会停止该运行：该运行将以启动时的处理模式完成，而下一次运行将采用标准处理模式。`logout` 对此有说明（`The processing run in progress finishes in <mode> mode; the next run uses standard`），当您注销时，应用程序也会显示相关提示 注销时，应用程序也会显示相关提示。

```bash
chloros-cli logout
```

---

## `chloros-cli status`

显示当前许可证等级（铁/铜/青铜/银/金）、已认证用户以及设备绑定数量。

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

轮询 Thread-4 实时导出进度。在另一个 shell 中运行 `process` 时，**期间**调用此命令是安全的。

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

设置CLI的显示语言（支持38种语言，包括中日韩、从右到左（RTL）及印度语系）。在无法渲染脚本的旧版控制台上，会优雅地回退为英语。

```
chloros-cli language [LANG_CODE] [--list]
```

### 示例

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## 项目文件夹命令

这些命令用于管理默认项目文件夹位置（与 GUI 共享）。

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

`get-project-folder` 会输出新项目存放的文件夹路径。如果配置的文件夹已不存在（被删除、重命名或位于未连接的磁盘上）， CLI将与图形界面一样回退到默认文件夹：默认文件夹将输出到标准输出（stdout），包含缺失文件夹名称的警告将输出到标准错误（stderr），且退出代码为 `0`。

---

## `chloros-cli update`

仅限 Linux / Jetson。检查 `version_url` 与 `/etc/chloros/update.conf` 之间的差异，若存在更新版本， 将下载对应的 `.deb` 并立即使用 `sudo dpkg -i` 进行安装（无确认提示；`sudo` 可能会要求输入密码）。 在 Jetson 上，仅向 JetPack 6 系统提供此更新。在 Windows 上，该命令不会执行任何操作，并报告 `up to date`：请使用新安装程序进行更新。

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

在 Linux/Jetson 系统上，CLI 还会在**每次启动时自动检查更新** （非阻塞模式，绝不会延迟命令执行）：它会读取 `/etc/chloros/update.conf`，将结果缓存 1 小时（`~/.chloros/update_cache.json`），并在存在更新版本时输出 `Update available: vX.Y.Z / Run: chloros-cli update`。 若出现任何错误或在Windows系统上，该操作将静默跳过；通过设置`CHLOROS_DISABLE_UPDATE_CHECK=1`可将其关闭。由于存在1小时的缓存机制，在此期间发布的版本仍可能被识别为最新版本。

---

## `chloros-cli selftest`

执行 7 步烟雾测试：版本、端口可用性、后端启动、`/api/test`、`/api/system-info`（GPU/CUDA/PyTorch）、去噪模型存在性、 CUDA+去噪器就绪状态。该测试会使用正在运行的后端，或为测试启动一个后端并在测试后停止它。它会输出 `N/7 checks passed` 以及未通过的检查项，并且仅当全部七项均通过时才退出 `0`； 在未安装 CUDA GPU 的计算机上，CUDA+去噪器检查将失败（参见 [验证安装](#verify-the-install)）。

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

PTP主时钟状态与控制。Chloros主机运行PTP主时钟；LATTICE相机和DAQ-E单元作为从设备与其同步，以实现跨设备时间戳 （跨设备延迟约 1 毫秒）。PTP 负责对齐 **时间戳**：阵列中的相机之所以能同步曝光，是因为 M8 硬件触发器控制其触发（模块间延迟小于 100 微秒），而非依赖 PTP。 未带子命令的 `chloros-cli time-sync` 将执行 `status`。

| 子命令 | 描述 |
| --- | --- |
| `status` | 显示主时钟状态、BMCA 优先级、时钟标识，以及其数据包是否已到达网络：当主时钟未运行时，`health` 即为 `OK` （当主控节点未运行时为 `STOPPED`），或 `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` 且存在 `health_detail`， 随后会向 stderr 输出 `WARNING:` 这一行（stdout 仍保持 JSON 格式）。`sync_sent` / `announce_sent` 仅统计已发送的消息；`*_send_errors` 和 `last_send_error` 则统计未发送的消息。 当相机连接断开后又恢复时，主节点会自动重建其套接字（`rebinds`、`last_rebind`）。 |
| `peers` | 列出通过 Delay_Req 检测到的从设备（摄像头 + DAQ-E 传感器）。 |
| `cameras` | 每台摄像机的 PTP 健康状态（`PtpStatus`、`PtpOffsetFromMaster`、`PtpMeanPathDelay`）。 |
| `restart` | 重启主控进程。 |
| `set-priority --priority1 N --priority2 N` | 覆盖 BMCA 优先级。 |

主控进程通过摄像头的网卡进行广播 （包含 `169.254.x.x` 地址的任何网卡）。若无可用网卡，它仅在默认路由对应的网卡为有线连接时才回退至该网卡，绝不会使用 Wi-Fi 或虚拟网卡（如 Tailscale、WSL），因为没有摄像头或 DAQ-E 位于其后。若无内容可广播，则不使用套接字运行（`health` `NO_SOCKETS`），而下一台相机或 DAQ 连接时，系统会将其重建在当时已上线的网卡上；`time-sync restart`会立即完成此操作。

主控程序在后端之外运行于独立进程中（`own_process` / `process_pid` 在 `time-sync status` 中），在 Windows 系统上具有高优先级（`process_priority`），因此即使后端处于繁忙状态（例如正在进行处理、导出或 同时执行多个命令）也无法延迟其时间戳的生成；此前，一次此类延迟曾导致所有摄像头的时钟同时偏移。该进程随后端结束；若无法启动，主时钟将如以往一样在后端内部运行（`own_process` false，且后端日志中会记录此情况）。

### 示例

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

LATTICE 摄像头控制。每个子命令都通过 Chloros 后端路由；后端拥有摄像头池，因此后续的 CLI 调用将复用同一个已打开的句柄。

### 常用选项（大多数子命令共用）

| 标志 | 描述 |
| --- | --- |
| `-d, --device N` | 摄像头索引（默认：0）。 |
| `-s, --serial SN` | 特定序列号；覆盖 `--device`。 |
| `--serials SN1,SN2,…` | 用于多摄像头操作的以逗号分隔的序列号。 [临时多摄像头命令](#multi-camera-transient-sessions) 仅打开这些摄像头，其中第一个摄像头将触发快门。 |
| `--all` | 对所有已发现的摄像头进行操作。 |
| `--exposure US` | 曝光时间（单位：微秒）。 |
| `--gain DB` | 增益（单位：分贝）。 |
| `--pixel-format FMT` | 例如：`BayerRG8`、`BayerRG12`；`Mono8`、 `Mono12`。该名称在相机所属系列中设定其位深度： 在 M3M 上，拜耳名称决定其深度的单色格式（`BayerRG8` 设定 `Mono8`），在 M3C 上，单色名称则决定拜耳格式 （`Mono12` 设置 `BayerRG12`）。 |
| `--width N` / `--height N` | 图像尺寸。 |
| `--preset {default,high_quality,high_speed,triggered}` | 应用预设：其像素格式、曝光、增益和帧率（`high_speed`: `BayerRG8`, 5 毫秒，6 dB，60 帧/秒；`high_quality`：`BayerRG12`，20 毫秒，0 dB；`lattice presets` 列出了这些参数）。 在 M3M 上，格式为相同深度的单声道格式：`high_speed` 捕获 `Mono8`、 `high_quality`、`triggered`以及`Mono12`。随其提供的标志 （`--exposure`、`--gain`、`--pixel-format`）将优先于预设，而预设中未列出的设置则保留摄像机自身的设置。所有自由运行，但 `triggered` 除外，该参数会使相机对第 2 号线的硬件沿进行触发——如果该线没有信号驱动，相机将无限期等待而非进行捕获。 |
| `-o, --output DIR` | 输出目录 （默认：`output`）。 |
| `--packet-size {auto,jumbo,standard,N}` | GVSP 数据包大小。`auto` 运行 ICMP+GVSP 探测；`jumbo` = 9000； `standard` = 1500。 |

### LATTICE 摄像头首次连接工作流

```bash
# 0. A backend must be running: the Chloros desktop app (Windows),
#    `chloros-backend`, or the Linux service (see "Start the Backend")

# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Full sensor (no binning, no ROI); the frame rate falls to
#        what the host and wire deliver, and the faster options are
#        printed
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### 子命令参考

#### 发现与信息

| 子命令 | 用途 |
| --- | --- |
| `lattice info` | 列出已连接的摄像头（厂商、型号、序列号、IP、MAC）。 |
| `lattice info` | 分析主机系统以获取最佳摄像头配置。`--no-discover` 跳过摄像头发现步骤（速度更快，仅分析网卡）。 |
| `lattice network [--fix] [--estimate] [--cameras N] [--pixel-format FMT]` | 检查/修复网卡设置（在 Linux 上，还包括内核的套接字接收缓冲区 — 参见 [Linux 主机](#linux-hosts--socket-receive-buffers))；估算带宽/帧率。 |
| `lattice network-analysis --master SN [--slaves SN1,SN2,…] [--width N] [--height N] [--pixel-format FMT] [--binning {1,2,4}] [--force-tier TIER] [--packed {auto,on,off}] [--backend-url URL] [--json]` | 稳定架构后端网络-能力 + 数组推荐（返回 `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`)。`--width` / `--height` 默认采用完整传感器， `2048` × `1536`。它为 `ok`、`auto_capped_fps` 以及 `auto_shrunk`，并从 `1` 切换至 `needs_force_slip` 和 `error`， 无论是否包含 `--json`。它会在 `--backend-url`（默认值为 `http://127.0.0.1:5000`）处到达后端，而非 `CHLOROS_BACKEND_URL`。 `auto_capped_fps` 保留请求的分辨率，但会限制目标帧率——读取 `recommended.recommended_target_fps` 并将其作为连接目标传递；将其视为成功， 而非错误。`auto_shrunk` 仅在无法以任何帧率传输所请求帧时出现；连接操作绝不会自行缩减帧。 |
| `lattice analyze-array [--models M1,M2,…] [--binning {1,2,4}] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--packed {auto,on,off}] [--json]` | 不打开摄像头的假设分析（`--width` / `--height` 默认 `2048` × `1536`）。此项与 `network-analysis` 均会预测 connect 将传输的内容：若未设置 `--pixel-format`， 连接将应用的每台摄像机格式（Mono12 / BayerRG12）；而当使用 `--packed auto`（默认）时，则显示连接在此计算机上将做出的 [12 位紧凑传输](#packed-12-bit-transport---packed）——因此所列出的帧率即为该数组运行时的速率。两者均会打印打包方案。**`--n-active` 表示网络上摄像头的总数，而不仅仅是该数组的**— 当独立摄像机并发传输时，或带宽预算的计算所依据的需求低估了摄像机数量时，请提高该值（默认值：`len(--models)`）。 始终打印聚合的 `Wire budget:`（所需 MB/s 与防冲突上限）和 `Max cameras:` 行，并在数组对总线超额订阅时标记 `** OVER-SUBSCRIBED**` ——参见 [数组 fps 与突发模型](#array-fps--burst-model)。 |
| `lattice gpu` | 显示 GPU 状态。 |
| `lattice firmware [--update] [--force] [-y\|--yes]` | 检查或更新相机固件。 本地 `.fwa` 选项被锁定：当存在与该构建的 `MIN_FIRMWARE_VERSION` 匹配的 `firmware/<MODEL_PREFIX>/` 文件时，该文件会被刷入 （仅将最高版本作为备用方案），因此存储在磁盘上的更新供应商镜像在该引脚被触发前保持闲置状态——较新的版本会通过签名的 AWS 清单有意地推送至设备，当有更新版本时，建议优先采用此方式。 |
| `lattice presets [--apply NAME]` | 列出或应用相机预设。`--apply` 设置预设的名称（其像素格式、曝光、增益、帧率、触发方式），并在相机读取以下参数后打印这些设置： 在 M3M 上，预设深度的 Mono 格式；其运行帧率，该帧率可能低于预设要求的值（`high_speed` 要求 60 fps；但通过 1 Gbps 链路连接的全画幅相机实际运行帧率约为 35）。 其 open 命令写入的其他所有设置均按实际读取值恢复。`--apply default` 将相机重置为出厂默认值：即 Line2 上的自由运行自动曝光、链路吞吐量限制和缺陷校正关闭，同时保留 RGB 相机的校准白平衡（其他模式下均为默认值）。 没有任何数据会被保存到摄像机的内存中，因此下次开机时，它将像之前一样从自身内存开始运行。作为捕获或连接命令的 `--preset`，`default` 并非预设。 |
| `lattice status` | 实时摄像头状态。 |

#### 捕获

| 子命令 | 用途 |
| --- | --- |
| `lattice capture [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--no-daq] [--daq-tolerance-ms MS]` | 单帧。 **默认保存所有导出类型**（`--processing all`）；参见 [捕获导出级别](#capture-export-levels-the-all-default)。 `--processing` 接受 `all`、`raw`、`debayered`、 `radiance`、`reflectance` 或 `display`（对于 `display`，`preview` 被接受X)；在触摸相机之前，任何其他名称均会被拒绝（退出 `1`）。`--levels` 保存一个显式子集（覆盖 `--processing`）。 为捕获操作打开一台Chloros未持有的摄像机，并首先根据场景设置其曝光参数（最多几秒钟），这与[临时命令](#multi-camera-transient-sessions)的操作方式一致； 无论捕获以何种方式结束，该摄像机都会被恢复至初始状态（格式、曝光、色彩处理管道，以及捕获过程中更改的每一项设置——详见[多摄像机（瞬态会话）](#multi-camera-transient-sessions)中的列表）。 用于读取该捕获的 DAQ 数据将作为 `.daq` 旁路文件保存，其中包含仅含原始数据的抓取内容；可通过 `--no-daq` / `--daq-tolerance-ms` 对其进行调整 ([DAQ读取捕获时保存的内容](#the-daq-reading-a-capture-saves))。`--force-daq` 仍被接受。`--jpeg-quality` = JPEG质量 1–100 （默认 95）。**保存到打开的项目中的文件将加入该项目。** 与应用程序中一样，如果捕获生成的文件位于 Chloros 中打开的项目文件夹内，则会将其添加到该项目的文件列表中；保存在其他位置的文件则保持不变。该命令会打印一行说明 (`Project: ...`)，或者，如果在处理运行过程中，则会提示这些文件已被暂存，待运行结束时再加入。`multi-capture`、`array-capture` (`--smart` 亦然) 以及 `project capture` 均遵循此规则。 |
| `lattice continuous [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--queue-depth N] [--no-daq] [--daq-tolerance-ms MS]` | 将数据流持续写入磁盘直至按下 Ctrl+C。一次 `tiff` 运行的帧即为 `raw` 文件，该文件包含该相机捕获的数组数据：传感器的马赛克图像，每个平面均采用其自身数值，未压缩，并附带校准数据包 （无论场景如何，一个 2048 x 1536 的 12 位帧大小为 6.0 MiB 加上数据包），因此 `process` 以此为基础开始处理。 `png` 和 `jpg` 各包含 8 位，因此这些数据流是用于查看的去马赛克帧，而 `process` 并非由此开始。若绑定到 DAQ （`daq pool-bind`）或采用池化处理时，每帧都会在该运行的 `daq/` 文件夹中拥有独立的 `.daq` 和 `.match.json`， 因为 `lattice capture` 会保存一个；`--no-daq` 会关闭该功能，而停止时会打印已捕获的帧数。每帧数据仅写入一次，并包含完整的元数据； 按下 Ctrl+C 时，会在释放相机之前写入已捕获的帧。当存储设备已满时，程序会自动停止，并打印 `Failed:` / `Disk full:`，并退出 `1`；若存在无法写入的帧，也会退出 `1`。若相机处于硬件触发待机状态 （`TriggerMode` `On`）的相机，在本次运行期间将设置为自由运行模式。运行结束时，相机将恢复至初始状态——其触发设置以及运行期间更改的各项参数 （曝光、增益、格式等）——除非遇到 `--keep-settings`（将运行设置保存到相机自身内存中，即自由运行模式），或 `--clear-settings`（将该内存重置为出厂设置）。 未记录任何帧的运行视为失败：若连续10秒无帧，系统将自动停止，打印包含原因的`No frames:`，并退出`1`。 写入器无法跟上的帧将被计为丢帧，停止时会打印包含原因的 `Dropped:`（包括相机的帧率、写入器的帧率以及队列帧率）；若采用较低的帧率或 8 位像素格式，则可保留所有帧。 该命令每 2 秒向后端报告一次；如果它在未停止录制的情况下终止（`kill -9`，内存不足），后端将在最后一次报告后 30 秒停止录制，并将摄像机恢复 （`stop_reason` `client_gone`，并在后端日志中记录一行）。`multi-stream` 和 `align-stream` 也会执行同样的操作。 如果摄像头开启所需时间超过命令的等待时限（60 秒；对于 `multi-stream` / `align-stream` 则为 180 秒）——即摄像头仍在启动中——后端将撤销该启动操作： 摄像头将被恢复并关闭，且不会保留任何正在运行的录制任务（HTTP 504 `CLIENT_DEADLINE_PASSED`）。|
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | 基于浏览器的 MJPEG 实时预览：连接摄像头，显示流媒体URL并在浏览器中打开。`--ae-damping` 设置自动曝光衰减（0.4–100）。 |

##### 实时流（MJPEG；不支持 RTSP）

Chloros 通过 **HTTP 传输 MJPEG**（`multipart/x-mixed-replace`，边界条件 `frame`）从后端提供实时视频流。**不支持 RTSP。** 在浏览器中打开该URL，或使用任意MJPEG客户端（OpenCV的`cv2.VideoCapture(url)`、ffmpeg、VLC）进行读取。必须先在后端连接摄像头（`lattice cam-connect`、`lattice array-connect`、`lattice viewer` 或通过图形界面）。

| URL（`http://127.0.0.1:5000` + …） | 流 |
| --- | --- |
| `/api/camera/stream/<serial>` | 显示预览，帧率最高约 30 fps。对于数组成员， `?max_h=`（默认值为 `720`）和 `?jpeg_q=`（默认值为 `80`）分别用于设置高度和 JPEG 质量。 这是URL `lattice viewer`所打开的页面。 |
| `/api/camera/<serial>/stream-annotated` | 相同的预览图，其中绘制了GUI的叠加层： `zebra=1`、`crosshair=1`、`grid=3x3`（或 `9x9`）， `histogram=1`、`peaking=1`、`spot=1`；若未进行查询，则采用该项目为该相机保存的叠加层设置。 |
| `/api/camera/<serial>/stream-reflectance` | 每波段 8 位反射率：ρ = 像素值 ÷ 该部分’`X-Chloros-Pixel-Scale` 头（除非触发了 `refl_max`，否则为 255）。`fps`（默认值为 `10`）， `jpeg_q`（默认值为 `10`），`max_h`（默认值为 `0` = 原生）， `band=<name>`、`align=1`。JPEG 格式，因此有损。当相机无反射率时（HTTP 409 状态码），将返回 HTTP 409 状态码（RGB 相机，无校准包，无 DAQ 读数）。|

- **跨相机帧匹配。**每个**数组成员**在三个流中发送的每个数据块，都携带 `X-Chloros-Group-Id`（该数组的同步组计数器）、`X-Chloros-Frame-Id` 和 `X-Chloros-Timestamp-Ns` 头部。来自不同相机但具有相同 `X-Chloros-Group-Id` 的数据包源自同一触发事件。独立相机没有组，也不发送组 ID。 浏览器会忽略各部分的头部信息；由客户端自行解析多部分流时，会读取这些头部信息。
- **数组视图打开期间处于暂停状态。** 当数组的合并实时流处于打开状态时（例如 GUI 的数组视图），数组成员的 `/api/camera/stream/<serial>` 会发送静态的“暂停”帧，并在流关闭时恢复。

#### 传感器调谐

| 子命令 | 用途 |
| --- | --- |
| `lattice configure [{--get N1 N2… \| --set N=V N=V… \| --dump}] [--json]` | 读取/写入任何 GenICam 节点。 `--set ExposureTime=…`（`Gain=…`）在未先通过其 `ExposureAuto`（`GainAuto`）开关将自动模式设为“关闭”的情况下， 如`lattice exposure --set`所做的那样；若相机拒绝该值，则在stderr上打印`Failed to set <node>: …`并退出`1`。`--get`以及 默认分组视图会在该自动模式运行期间在 `ExposureTime` / `Gain` 下打印 `note:` —— 参见 [自动模式下的曝光和增益读取](#reading-exposure-and-gain-under-auto-mode)。|
| `lattice exposure [{--auto \| --auto-once \| --off \| --set US}] [--brightness 0-255] [--brightness-auto] [--damping 0.4-100] [--upper-limit US]` | 曝光与自动曝光（AE）。系统将保持您设定的曝光值（`--set`）或目标值（`--brightness`） （Chloros的自动曝光控制不会对其进行干预）；增益保持原值，因此在自动模式下，系统会持续向目标值调整，直到你设置`lattice gain`。 `--auto`将曝光控制权交还，`--brightness-auto`将目标交还给Chloros的自动目标系统。若未设置标志，它将读取曝光节点， 其行为与 `note:` 相同。若相机拒绝某个 `--set` 值，则会在标准错误输出（stderr）上打印 `Failed to set ExposureTime: ExposureTime=<value> refused (<camera error>); the camera accepts <min>..<max> now (ExposureAuto put back to <mode>)` ——被写入操作关闭的自动模式会被恢复，而非保持关闭状态——不会输出 `Exposure: … us (auto OFF)` 行，并退出 `1`。 |
| `lattice gain [{--auto \| --off \| --set DB}]` | 增益与自动增益。 当 `--set` 值超出相机允许范围时，相机会在标准错误输出（stderr）上打印 `Failed to set Gain: …`，不输出 `Gain: … dB (auto OFF)` 行，并退出 `1`。 |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N]` | 传感器感兴趣区域（ROI）与像素合并。像素合并方式始终为**平均**——详见下文。 |
| `lattice format [--set FMT] [--list]` | 像素格式。 |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | 硬件/软件触发。 |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]`（无标志 = 单次白平衡） | 白平衡操作。 仅限 RGB/拜耳传感器相机；在单色 M3M 上为无操作（跳过）。单次白平衡会保留其计算出的白平衡值（`BalanceWhiteAuto` 及增益）；其他所有设置的写入操作均按原始值恢复，当无法执行单次白平衡时则全部恢复。 |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct 2000-10000] [--get]` | RGB显示色彩处理管道。`natural`（默认）是低成本的实时处理方案； `enhanced` 增加了去色散 + 鲜活度 + CLAHE 局部对比度，以实现完整的 Hub-Parity 效果，每帧处理成本约为 2 倍，因此会降低 **实时** 帧率——无论哪种情况，保存的截图始终会获得完整的处理效果。仅限 RGB/拜耳传感器相机； 在单色 M3M 上被跳过。 |
| `lattice color [--saturation 0-200] [--contrast 0-200] [--reset] [--get]` | 实时预览饱和度/对比度（RGB滤镜相机）；`100` 为中性设置，`--reset` 将两者均设为 100。 已保存的截图不受影响。在单色 M3M 上跳过。 |
| `lattice highlights [{--set -100..0 \| --reset \| --get}]`（无标志 = 读取） | 显示 **高光**（RGB滤光片相机）：将最亮、最白的像素调暗——例如，原本明亮的白色表面会被渲染成一片平淡的白色斑块。`0` = 关闭， `-100` = 最强。实时预览和显示级别的截图会遵循此设置；辐射度/反射率始终不遵循，且无法恢复已剪切的传感器数据（请降低该情况下的曝光值）。在单色 M3M 上跳过此设置。 |
| `lattice filter [--list]` | 列出 Chloros 已知的滤镜型号（`RGN-IMX265`、`OCN`、`NGB`、…）。相机的滤镜信息从其型号字符串中读取，无法手动设置：`--set NAME` 仅查询一个型号，随后通过 `1` 退出并返回 `setting the filter model is not supported`。 |
| `lattice power [--sleep]` | 检测电源/热节点；切换低功耗空闲模式。 `--sleep` 使相机进入休眠状态（其链路吞吐量限制已启用（`DeviceLinkThroughputLimitMode` `On`）并设为最低阈值，若相机具备节能模式则启用该模式） 其余所有设置均恢复为最初检测到的状态； 睡眠失败的情况则将所有设置原样恢复。唤醒操作会将链路恢复至睡眠状态时的原始状态（包括限值、模式以及数据包延迟 `GevSCPD`——当限值开启时，摄像头会将其保持为 0；相关记录保存在这台计算机上， 因此后端重启后的唤醒操作也能找到该记录），或者，对于这台计算机未将其置于睡眠状态但看起来处于睡眠状态的摄像头（其限值在底限处开启），在关闭限值并将 `GevSCPD` 保持为 0 且限值为 80 （出厂设置）；若限制值设定在阈值以上，则保持原状。探针和唤醒操作会将它们在打开时写入的数据恢复原状。 |

> **像素合并模式始终为“平均”。** `lattice resolution --binning N` 设置“平均”合并模式，且
> 不提供模式切换功能。 “求和”分桶会将每个动态数值（DN）乘以 N²，每步分桶大约消耗两档
> 高光余量；Chloros会在
> 辐射测量链之前将该系数除去，但“求和”模式将推至传感器上限的像素会被剪切且无法
> 恢复。 它并没有带来“平均”模式所不具备的任何优势——指数是尺度不变的，且辐射度
> 标量是在“平均”尺度上校准的。如果相机出厂时已设置为“求和”模式（由
> 其他工具设置，或因工厂重置所致）， Chloros会通过节点读回检测到该状态并
> 进行补偿；运行`lattice resolution --binning N`可将其恢复为“平均”模式。1.2.0
> `--binning-mode` 标志仍被接受，因此旧脚本仍可运行，但仅会打印一条警告：
> 像素合并模式仍保持为“平均”。

> **更改相机的分档、像素格式和触发方式后，设置均不会生效。** 在相机上，
> 后端保持打开并进行流式传输（`cam-connect`，即 GUI），`lattice resolution --binning`
> 均无法生效（会提示当前像素合并模式仍有效，`Binning: 1x1`），且
> `lattice format --set` 和 `lattice trigger --mode` 被相机拒绝
>（`SC_ERR_ERROR -1001`）。 请先释放该设置（`lattice cam-disconnect -s <SN>`），或在
> 连接时进行设置（`cam-connect --pixel-format …`、`array-connect --binning …`）。

##### 在自动模式下读取曝光和增益

当摄像机的 `ExposureAuto` (`GainAuto`) 正在运行时，其 `ExposureTime` (`Gain`) 节点会保留**上次写入的值**，而非摄像机当前的运行状态。 对于后端保持打开状态的摄像头（`cam-connect`、`array-connect`、GUI），`lattice configure --get`、默认视图 `lattice configure` 以及 `lattice exposure`（无标志）因此会显示该节点的值，并在其下方附注摄像头的最新帧信息：

```

ExposureTime = 2000.448
  note: ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448); the camera's newest frame was at 4900 us.
```

从注释中读取帧值；节点值仅为最后一次写入的内容。通过HTTP，相同的`POST /api/camera/<serial>/read-nodes`响应会保持`nodes`不变，并添加`frame_values` （即最新帧的 `{"ExposureTime": <us>, "Gain": <dB>}`）和 `notes`（即上文所述内容）。这两项仅在自动模式非 `Off` 且已知帧的情况下存在； 仅用于读取的相机没有帧，因此不会获取上述任何内容。`lattice gain`（无标志）仅打印节点——若需显示注释，请使用 `lattice configure --get Gain`。

##### 数组成员的曝光

当数组进行流式传输时，每个成员以 **`AcquisitionFrameRateEnable`**，帧率略高于数组的触发率：其周期等于触发周期减去 15%（至少 3 毫秒），或者当触发率已接近相机自身最大帧率时，则采用该最大帧率。 因此，在成员上启用 `AcquisitionFrameRateEnable` 时，该设置下会将 `AcquisitionFrameRateEnable` 读取为 `True` 和 `AcquisitionFrameRate`；当数组 停止时，使能信号将再次关闭。**请勿在数组成员上写入这两个节点**——数组会使其与触发信号保持同步并重新写入。

该设置也是成员的曝光上限：在 10 fps 触发频率（100 ms 周期）下，无论采用**任何 ROI 或像素合并模式**，曝光上限约为 85 ms下均适用。在 1.3.0 版本之前，即使阵列触发速度再慢，一个较小的 ROI 也会将成员的曝光时间限制在传感器的自由运行周期内（768 × 304 分辨率下为 4.9 毫秒）。**当使用 `ExposureTime` 写入（或通过 GUI 设置）时，若手动设置的 `ExposureTime` 值超过该上限，系统会减慢触发速率以适应该上限**。数组首先为该成员的上限值打开通道，并让较慢的速率运行几个触发周期，因此写入操作可能需要多花几秒钟；随后，触发速度不会超过曝光允许的上限。在自动速率数组中，一旦所有因手动曝光而导致触发变慢的成员恢复为自动曝光，数组将恢复之前的运行速率。 相机本身无法拍摄的数值仍会被拒绝，错误信息中会显示其可接受的范围，且如果该成员的自动曝光仅因此次写入而被关闭，则会恢复自动曝光。配方 `apply` 的曝光不会减慢触发速度：它会被限制在上限值内。 Chloros 绝不会为适应更快的触发频率而缩短手动曝光时间；若固定速率（`array-connect --target-fps F`）快于曝光允许的速率，则该成员的上限将保持足够长的时间以适应该速率，且该成员的触发频率不会超过其曝光允许的上限。

#### 校准与传感器

| 子命令 | 用途 |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | 通过反射率目标进行校准。系统将搜索最多 `--attempts` 个帧（默认 `30`）； 不完整的帧（GVSP数据包丢失）将消耗一次尝试机会，而非整个运行。若帧未完整到达，则在达到 `no target detected in N attempts` 时失败， 若无完整帧到达，则返回 `no complete frame in N attempts: <last error>`。|
| `lattice dls {--connect \| --spectrum \| --irradiance} --mac MAC [--filter NAME] [--camera SERIAL] [--json]` | DAQ-M 的频谱和频带幅度， 通过后端经由蓝牙（BLE）读取。`--mac` 是传感器自身的 BLE MAC 地址，执行以下三项操作中的任何一项时均需提供：该参数无默认值，因为每台设备都有其专属地址（`daq pool-discover --only ble` 列出了这些地址）； 若未提供该参数，命令将退出（`1`）。若未指定任何操作，则显示帮助信息。`--camera SERIAL` 通过该相机自身的测量光谱响应（而非滤光片模型的校准曲线）对打印出的波段幅度进行积分 ——RGB相机，前提是相机的校准包已加载（已连接）或缓存于本机。标题会标明使用了哪条曲线：`Band magnitudes (camera <SERIAL>):`，或者当相机没有校准曲线时（非RGB相机、 或未加载/缓存校准包），此时将改用标准曲线。通过HTTP，`POST /api/dls/cli-read` 接收 `camera_serial` 并返回 `band_magnitudes_source`。 |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | 对现有图像应用暗角校正，每张图像均使用其镜头自身的平场 （来自其 EXIF 型号字段）。`--lens-model` 适用于型号字段中未指定相机的图像；若图像的型号字段与其不符，则该图像将被跳过。 |

#### 多相机（临时会话）

临时命令会在单次运行期间打开相机，并在运行结束时关闭它们：未连接持久化数组的 `multi-capture`，以及此处的 `multi-stream` 和 `multi-test`， 以及下方的 `align-calibrate`、`align-apply`、`align-stream` 和 `index --live`。以下规则适用于所有这些命令：

- **`--serials SN1,SN2,…` 负责选择摄像头。** 仅这些摄像头会被打开，其中第一个打开的会触发动作 （若指定了 `align-calibrate`、`--reference`，则以此为准）。若序列号不在网络中，则在任何摄像头打开之前运行即告失败。若未指定 `--serials`，则网络中的所有摄像头都会被打开。 Chloros已持有的摄像头（GUI、`cam-connect`、`array-connect`）无论`--serials`是否指定其名称，都会拒绝运行： 因为它共享了该运行所针对的网络。（若未启用 `--serials`，`index --live` 将转而打开其配置文件中启用的摄像头：参见 [索引 / 植被数学](#index--vegetation-maths)。）
- **这些摄像头共享网络的方式与 `array-connect` 相同。** 每台摄像头的传输速率都根据主机链路承载量的份额进行调节（即包间延迟，而非摄像头的吞吐量上限——在 1 GbE 网络上，四台摄像头的 31.25 MB/s 下限早已被突破）， 因此四台摄像机可在 1 Gbps 链路上运行。当摄像机无法以各自达到的最慢速率同时发送数据时，它们将依次被触发，每台摄像机独占链路， 此时该命令会输出 `Serial emit: the wire cannot carry these cameras at once, so each is triggered <ms> ms after the one before; their frames are not exposed together`。一旦有哪怕一台摄像头无法适应该速率，在任何摄像头开始传输之前，运行就会被拒绝： `array over-subscribes the wire: <n> cameras demand <x> MB/s but the collision-safe ceiling is <y> MB/s. …`，HTTP 409，退出 `1`。
- **每台摄像头都必须成功传输，否则运行失败。** 未传输任何帧的摄像头即视为错误： 该命令会输出 `Error: … no frame from <n> of <total> cameras: SN:<serial> (<why>), …`，其中 `<why>` 表示该摄像头的抓取错误，随后显示导致该错误的原因，并退出 `1`。 若帧在传输过程中出现 GVSP 数据包丢失（`IncompleteBufferError: GVSP packet loss …`），则表明网络无法承载所有摄像头的总带宽（`Every missing frame arrived with GVSP packet loss: …`）：检查主机和交换机上行链路的链路速率（`lattice network`），启用巨帧， 或减少同时运行的摄像头数量（`--serials`）。若某台摄像头未传输任何数据，通常是因为未接收到主摄像头的触发信号：请检查其 M8 同步线缆，然后运行 `lattice multi-detect`。针对缺失摄像头的帧组，各命令的具体处理方式详见对应行。
- **每台摄像机将恢复至初始状态。** 运行结束时，每台摄像机将恢复至运行开始时的状态：`TriggerMode`、`TriggerSource`、 `TriggerActivation`、`TriggerOverlap`、`TriggerDelay`、 以及位于 `Line2` 和 `Line3` 上的 `LineMode` / `LineSource` （以及数组对从机线路的触发输入去毛刺处理，`LineFilterSelector` / `LineFilterWidth`）； 其 `PtpEnable`（以及与其配对的 `PtpSlaveOnly` / `PtpDomainNumber`）； `ExposureAuto` / `GainAuto` （以及 `ExposureTime` / `Gain`，其中该参数设置为手动值）；`TargetBrightness`、`ExposureAutoLimitAuto` / `ExposureAutoUpperLimit` / `ExposureAutoLowerLimit`， `GainAutoUpperLimit`、`ExposureAutoDamping`；`BalanceWhiteAuto`（以及所有白平衡为手动模式的 `BalanceRatio`）；`BlackLevelAuto` / `BlackLevel`；`PixelFormat`（采用像素合并），`Width` / `Height` / `OffsetX` / `OffsetY`；`GammaEnable`（以及启用了伽马校正的`Gamma`）， `LUTEnable`、`ColorTransformationEnable`；`ReverseX` / `ReverseY`；`UserSetSelector`； 数据包大小（`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`）；速率控制，`DeviceLinkThroughputLimitMode`（以及当模式为`On`时的`DeviceLinkThroughputLimit`），数据包间数据包间隔延迟 `GevSCPD` 以及起始错开时间 `GevSCFTD`，一个数组为每台摄像机提供；每个流启用的数据块 (`ChunkModeActive`，每个 `ChunkEnable` 位于其 `ChunkSelector` 之下)；`AcquisitionFrameRateEnable` （以及 `AcquisitionFrameRate`，其中启用状态为开启——若关闭，该速率为只读，且读取值不超过步进允许的最大值，因此这不是一个可配置项）。 仅写入不同之处，相机先停止，在需要帧周期的曝光之前，步进和帧率（相机将 `ExposureTime` 限制在一个帧周期内）。 在 `--keep-settings` 模式下，触发和同步线连接恢复，相机处于自由运行状态，其余参数按运行设置保存到相机自身内存中；`--clear-settings` 则将出厂设置保存至该内存。
- **曝光参数基于保留第一帧之前的场景进行设置。** 拍摄过程会采集同步帧组并将其丢弃，同时根据这些帧设置每台相机的曝光和增益。目标是使相机目标区域的平均亮度不超过帧亮度的1%；若场景中存在高光区域，则会提前停止采集。 此过程通常需要 2-6 组帧，最多耗时 5 秒。
  - 每台相机通过其滤波器瞄准连接数组初始设定的目标：多光谱相机（RGN、OCN、NGB）的目标亮度为 **40**，RGB 相机为**60**。 这相当于目标曝光值80的一半（RGB相机为四分之三），因此高亮反射板发生剪切的可能性较小。`--brightness N` (0-255) 则用于设置每台摄像头的目标值（`multi-test` 没有 `--brightness`：其自动曝光功能已关闭）。 后端日志会标注每台相机的目标值：`[transient-ae] AE target per camera (filter default): SN:<serial> <target>, …`。
  - 曝光值优先，上限为相机自身的自动曝光限制。增益仅用于补足剩余部分。
  - 确定的曝光值和增益值在整个运行过程中保持不变，因此 `multi-stream` 的每一帧都以相同的曝光值拍摄。需要跟踪光线变化的捕获应放在一个连接的数组（`array-connect`）中， 该序列的自动曝光会持续调整。
  - `--exposure` 和 `--gain` 将完全按给定参数保留。 若两者均已指定，则不会丢弃任何帧。
  - 运行结束时，每台相机的自动曝光功能将重新启用。
  - 后端日志会报告结果：`[array-ae] exposure solved in <s> s (<n> warm-up groups): SN:<serial> <ms> ms <dB> dB mean <m>/<target>, …`，或附带原因的 `exposure NOT solved after …`，以及未成功摄像头的 `(not settled)`。在这种情况下，帧仍会被拍摄。
  - `CHLOROS_TRANSIENT_AE_SETTLE_S` 用于更改 5 秒的限制；`0` 用于关闭此功能（参见 [环境变量](#environment-variables)）。

**当Chloros持有任何摄像头**（包括GUI、`cam-connect`或已连接的阵列）时，`multi-stream`、`multi-test`、 `multi-detect`、`align-apply`、`align-stream` 和 `index --live` 将拒绝运行，因为它们会打开网络上的所有摄像头，而第二次打开 已锁定的摄像头，可能会导致后端崩溃。（当`index --live`试图打开的摄像头已被锁定时，它会拒绝运行：无论是其自身的`--serials`，还是其配置文件中的摄像头，以及任何未指定摄像头的配置文件中的摄像头。） 后端返回 HTTP 409 状态码 `CAMERA_HELD`，列出每台被占用的摄像头及其释放命令 （`chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`），随后该命令退出 `1`。 当连接了一个数组时，`array-capture --continuous` 会将其记录在 `multi-stream` 的位置，而 `array-capture` （默认对齐）将一个对齐的集合保存到 `align-apply` 的位置；`multi-capture` 和 `align-calibrate` 使用一个连接的数组来存放其相机，而不是打开它们。

| 子命令 | 用途 |
| --- | --- |
| `lattice multi-info [--serials SN1,SN2,…]` | 列出所有具有同步角色的摄像头；配合 `--serials` 使用时，仅列出这些摄像头，而 `MASTER` 则仅列出第一个。 |
| `lattice multi-capture [--serials SN1,SN2,…] [--format FMT] [--jpeg-quality N] [--processing LEVEL] [--brightness N] [--no-daq] [--daq-tolerance-ms MS]` | 从每台摄像头获取一帧同步帧。当连接了持久数组时，默认保存**所有导出类型**，并保存其使用的 DAQ 读数（[捕获所保存的 DAQ 读数](#the-daq-reading-a-capture-saves)）； 临时无数组备用方案仅进行**去拜耳处理**（如需处理其余内容，请先运行 `array-connect`）。 `--brightness` 仅适用于备用模式：已连接的阵列将保持其自身的自动曝光设置。备用模式不保存 `.daq` （`--no-daq` / `--daq-tolerance-ms` 在此处无效），且 `multi-capture` 不采用 `--force-daq`。在回退模式下， 如果摄像机未输出帧，则捕获失败（`Error: multi-capture failed: no frame from …`，退出 `1`），且该帧组的文件不会被保存。 |
| `lattice multi-stream [--serials SN1,SN2,…] [--fps F] [--count N] [--format FMT] [--jpeg-quality N] [--brightness N]` | 流同步帧（临时）， `--fps` 默认 `3`，直至按下 Ctrl+C 或 `--count` 分组。对于某台摄像头未提供任何帧的帧组，该帧组不会被写入， 且连续三个帧组将终止数据流。若跳过了任何帧组，程序将通过输出 `Incomplete: <n> frame group(s) not written, a camera delivered nothing (SN:<serial> x<count>, …)` 结束运行，并附加 `Error: the stream stopped: …` 标明终止数据流时的摄像头名称，随后退出 `1`X。当驱动器空间已满时，程序会自动停止，输出 `Failed:` / `Disk full:` 并退出 `1`； 任何无法写入的文件也会导致程序退出 `1`。 |
| `lattice multi-test [--serials SN1,SN2,…] [--count N] [--ptp-settle SECONDS]` | GPIO 同步定时测试（瞬态），`--count` 默认 `10` 触发条件。打印网络共享方式 （`Wire: each camera paced to <x> MB/s (<n> on a <y> MB/s ceiling, <link> Mbps link)`，以及当相机依次触发时显示的`Serial emit:`行，此时时间戳将包含其时间间隔），每个同步组对应一行 `Sync group: master SN:<serial> -> SN:<serial>, …`（`(no slaves: not synced)` 表示未连接同步线的相机，该相机独立触发）， 随后按每次触发生成一行：`[  0] SN:<serial>=<timestamp> \| …`，若某台相机在该次触发中未输出数据，则对应行号为`SN:<serial>=(no frame)`。 接着是 `Offsets from master SN:<serial> (<PTP / device (PTP off)> clock, <n> trigger(s)):` 和每台摄像机各一行 `SN:<serial>: mean <±x> us, jitter <y> us`，表示每台摄像机在一个时钟周期内相对于主时钟的状态：其中包含时间戳的触发器会混合摄像机的设备时钟和 PTP （PTP 关闭或仍在锁定中）的触发器将被排除，并由 `Note:` 进行计数。 任何遗漏的帧都会导致测试失败（`Error: multi-test failed: <k> of <n> frame groups incomplete (SN:<serial> missed <k>); …`，退出 `1`）；连续三个不完整的触发信号会提前终止测试。 此外，当少于两个触发器使所有摄像机保持在同一时钟上（`… the clocks cannot be compared`），或者时钟始终无法同步时（ PTP时钟与主时钟的偏差超过1毫秒（加上任何串行层时序偏移）（`… PTP is not locked`），或者设备时钟在两次触发之间偏移量超过该阈值。 如果 PTP 总主时钟尚未运行，则启动它；开启每台摄像机的 PTP 功能（如同 `array-connect` 所做的那样，无论网络状况如何）；并在首次触发前，等待每台摄像机的 PTP 时钟锁定到总主时钟并稳定下来： `PtpStatus` 从属时钟连接到一个主时钟，与主时钟的偏差在 100 微秒以内且稳定 3 秒，且每台摄像机均以这种方式锁定，且不间断，持续 120 秒——从通电起约 2 分钟。刚锁定的时钟在读取“从属”状态后的约2分钟内，误差可达~200微秒，而相机自身的`PtpOffsetFromMaster`并不会显示这一情况；120秒后，相机时钟将稳定在与主时钟相差~10微秒的范围内。一台相机 若自上次测试后一直保持PTP开启状态（按此后端所见）， 将从该锁定时刻开始计时，因此连续测试时无需再次等待；而测试中刚开启的相机，或被发现处于PTP开启状态但该后端尚未观察到其锁定（可能在其他地方开启，或在后端重启前开启）的相机，则需等待120秒；在包含这两类相机的数组中，刚刚锁定的相机仍需等待其预定时间。 在等待期间，脱离从属状态、计时超过100微秒，或连续3次（1.5秒）跟随其他主时钟进行读取的摄像头将重新开始计时；此类读取会记录一到两次 (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`)，计数继续进行；而无法读取的主时钟 ID 将被视为未知，而非变更。 所有摄像头的计数在同一时间（一秒内）同时重启，表明是主节点的时钟在移动，而非摄像头自身：后端日志显示 `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`， `ptp_lock` 在 `grandmaster_steps` 中列出了该计数，随后超时的等待以 `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …` 开始报错。 该等待的时限不会因此延长。`--ptp-settle SECONDS` 设置了本次运行的稳定时间（0-600；若不进行稳定等待，则由 `0` 设置）。该等待的上限为稳定时间加 90 秒（默认 210 秒，至少 90 秒），且该命令会等待后端 240 秒加上稳定时间 （默认 360 秒）。它会输出 `PTP: locked after <s> s (each camera locked <settle> s, already locked: SN:<serial>, …; from the grandmaster: SN:<serial> <±x> us, …)`，后端日志中每 5 秒会记录一行，包含每台摄像机的状态和偏移量、已锁定时长以及剩余时间： `[array-sync] PTP at <s> s: SN:<serial> Slave <±x> us for <s> s (<s> s left), … -- <s> s left, the wait gives up at <s> s`。每台相机开机时均处于 PTP 关闭状态； 随后时钟约在 10 秒内完成锁定。此后，每台相机将恢复至测试发现时的状态——其 PTP、自动曝光和增益（测试期间被设为关闭），其余设置（除非出现 `--keep-settings`） ——因此，PTP保持开启的摄像机（`lattice configure -s <serial> --set PtpEnable=True`，或包含`--keep-settings`的运行）将保持锁定状态；一旦该后端检测到其已锁定并达到稳定时间，后续测试将不再等待该摄像机。 如果时钟在等待期间未能稳定，则在触发前失败（`PTP: NOT locked …`，退出 `1`） 并说明了处理方式：从未锁定的相机（`PtpStatus Listening`）意味着主控无法与其建立连接（`time-sync status`）；频繁断开连接的相机将按其锁定时长命名 （`… had not been locked for the 120 s settle time … (SN:<serial> locked <s> s, <s> s to go)`）；时钟仍在收敛中：请检查 `time-sync status`， 并确保在测试前完成锁定，请开启 PTP 并保持开启状态（每台摄像机均需执行 `lattice configure -s <serial> --set PtpEnable=True`，或使用 `--keep-settings` 运行测试；该测试会恢复检测到的设置，因此 PTP 将保持开启），然后运行测试。 |
| `lattice multi-detect [--line LINE] [--json]` | 查找 M8 同步线上的摄像头，并确定默认主摄像头：若已连接线缆，则为 RGB 摄像头（型号以 `-FRGB` 结尾）；否则，按序列号顺序为第一台已连接线缆的摄像头。 由于 M8 线缆汇聚于一个接点，因此 **任何已接线的摄像头均可作为主摄像头** ——打印出的 `MASTER` 仅为默认值，并非实际布线情况。每台摄像机依次驱动该线路，并检测其他摄像机是否接收到其脉冲；若某台摄像机已在布线组中被识别，则不再对其进行探测， 因此，一个完全有线连接的阵列只需一轮扫描。若未检测到 `--line`，则扫描 `Line2`， 随后扫描 `Line3`。打印 `MASTER: SN:…`，每台其他已连接的摄像头对应一个 `SLAVE: SN:…`，以及 `(probed on <line>)`；`--json` 则将完整结果添加进去（`master`、`slaves`、 `no_cable` — 脉冲未到达的摄像头 — `line`，以及每台摄像头的 `results`）。处理完成后，每台摄像机将恢复其原有的触发和同步线设置（与上文相同的节点）。 |

#### 对齐

| 子命令 | 用途 |
| --- | --- |
| `lattice align-calibrate [--serials SN1,SN2,…] [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N] [--brightness N]` — 加上检测器/匹配器控件 `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`， RANSAC参数 `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`、多帧组合 `[--averaging mean\|median\|inlier_weighted]`、几何约束 `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`、空间限制 `[--roi X0,Y0,X1,Y1] [--mask PATH]`，以及按从属节点覆盖设置 `[--per-cam-override SN:KEY=VALUE]`（可重复）| 根据实时摄像头数据计算对齐轮廓。默认值：`--method orb`、`--model rigid`、 `--frames 3`、`--save alignment_profile.json`（位于运行该命令的文件夹中）， `--max-features 2000`、`--ratio-threshold 0.75`、`--matcher bf`、`--knn-k 2`、`--ransac-threshold-px 3.0`、 `--ransac-iters 2000`、`--ransac-confidence 0.99`、`--averaging mean`。（`project align calibrate` 有其自身的默认设置；请参阅 [其表格](#project-align-calibrate-options)。) `--prefilter` 的默认设置为 `gradient` （边缘映射；与GUI/数组对齐器一致——边缘在各光谱波段间得以保留）。当特征数超过约5000个时，`--matcher flann`的效果显著； `--averaging median` 对单次捕获失败具有鲁棒性，`inlier_weighted` 按匹配次数加权；`--lock-scale` 投影至最近的旋转（无缩放）， `--lock-axis` 将一个平移分量设为零； `--mask` 适用于所有相机（如需按相机设置，请使用 `--per-cam-override`，例如 `--per-cam-override 214701292:method=phase`）。 `--rms-threshold-px` 拒绝保存重投影均方根误差（RMS）超出阈值的校准。`--method manual` 从 `--points PATH` 获取其点： 一个 JSON 对象 `{"<serial>": [[x, y], ...], ...}`，其中包含每台相机上相同特征的像素坐标（包括参考相机的坐标，每台相机至少 2 个）；每个 `--frames` 都是根据这些点求解得出的。 当相机存储在连接数组中时，校准会使用该数组的实时帧，且该配置文件也会成为该数组捕获帧的对齐基准。否则，它将打开一个 [临时数组](#multi-camera-transient-sessions)： 若任何一台相机在 `--frames` 中均未提供帧，则校准失败（退出 `1` 并命名），且不会保存任何配置文件。 |
| `lattice align-apply --profile PATH [--serials SN1,SN2,…] [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N] [--brightness N]` | 捕获一帧对齐的多波段帧。`--bit-depth` 默认与相机匹配； `--no-crop` 保留完整帧（用黑色填充）；`--interpolation`（默认 `linear`）和 `--border-mode`/`--border-value`（默认 `constant`/0）控制 CPU 变换——无论如何，GPU 路径均为双线性。打开一个 [临时数组](#multi-camera-transient-sessions)：若某台摄像机未输出帧，则捕获失败（退出 `1` 并命名该摄像机），且不会写入任何数据。 |
| `lattice align-stream --profile PATH [--serials SN1,SN2,…] [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N] [--brightness N]` | 流对齐的多波段帧（与 `align-apply` 具有相同的波段控制参数）、`--fps` 默认值 `3`。打开一个 [瞬态数组](#multi-camera-transient-sessions)：对于某台摄像头未发送任何帧的帧组，该帧组不会被保存，且连续三次未发送帧将终止流。 如果跳过了任何一组帧，运行将通过打印 `Incomplete: <n> frame group(s) not saved, a camera delivered nothing (SN:<serial> x<count>, …)` 和 `Error: …` 结束， 并退出 `1`。因其他任何错误而终止的流也会打印 `Error: …` 并退出 `1`。 |
| `lattice align-info --profile PATH [--json]` | 显示配置文件详细信息。 |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | 更改图层顺序。 |

#### 索引 / 植被数学

```bash
# Offline: compute NDVI from an aligned multi-band TIFF (an RGN camera's bands)
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: open the cameras, capture one aligned set with a profile made by
# `lattice align-calibrate`, and compute the index from it
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --save-multiband -o output/
```

**`--channel` 将预设的符号映射到波段** （NDVI：`red`、`nir`），各一个 `--channel SYMBOL=BAND`。

- **`--live`：仅适用于堆栈无法响应的符号。** 与 `--input` 一样，您未指定的预设符号将采用堆栈中（即该配置文件中已启用相机的波段）名称以此开头的那个波段。 在任何摄像头打开之前都会进行此检查：如果某个符号没有波段或多个波段能响应，则会拒绝该符号，并列出其名称，随后退出 `1`。 自定义的 `--formula` 需要为每个符号配对一个 `--channel`。
- **`--live` 产品会标明所属对象。** `index.tif` 和 `index_LUT.png` 包含该堆栈中相机的信息（品牌、型号、序列号）、其拍摄时间（参考相机的曝光时间，UTC） 以及镜头信息，两者均将索引命名为其唯一波段（`XMP-Camera:BandName` `NDVI`；彩色PNG文件以索引命名，而非其颜色通道），这与`--input`产品一致。 由多台相机波段组成的索引则会分别命名每个波段，其文件名为 `Model` —— 详见 [由多台相机生成的文件](#files-made-from-several-cameras)。
- **`--input`：仅适用于文件无法响应的符号。** 若省略某个符号，则采用文件中名称以此符号开头的那个波段： `red` 会采用 `Red_660` 或 `Red_660_RGN`，`nir` 会采用 `NIR_850`。 这些名称来自文件本身：栈自身的波段列表，或者， 在Chloros辐射度/反射率捕获数据或`process`产品中，其`Chloros:Band{R,G,B}Name`/`XMP-Camera:BandName`标签。因此，`lattice index --input <reflectance capture> --preset NDVI`无需 `--channel`。
- **两个对应同一符号的波段不会被推测出来**（9相机堆栈中有两个NIR波段）： 请为该波段命名。若某个符号没有对应的波段，则使用 `channel_map missing entries` 拒绝该符号，并退出 `1`。**堆栈中的波段名称**包含相机的滤光片信息：M3C相机的波段为 `<band>_<filter>` — `NIR_850_RGN`、`Green_550_RGN`、 RGN上的`Red_660_RGN`；`NIR_808_OCN`、`Cyan_490_OCN`、 `Orange_615_OCN`位于OCN上；`Blue_475_RGB`、`Green_550_RGB`、`Red_625_RGB`位于RGB上；`Blue_475_NGB`、`Green_550_NGB`、`NIR_850_NGB`位于NGB上——而M3M相机的的单波段为 `Mono_<filter>`（`Mono_660`、`Mono_850`）。`lattice align-info --profile PATH` 列出了配置文件的波段。

完整标志集：`--input PATH | --live --profile PATH`、`--preset NAME`（NDVI/NDRE/EVI/SAVI/GNDVI/…）， `--formula EXPR`、`--channel SYM=BAND`（可重复）、`--capture-level raw|debayered|radiance|reflectance|unknown`（覆盖源TIFF文件中记录的捕获级别；默认： 从TIFF元数据中读取，若未找到则回退至Chloros文件的`Chloros:ProcessingLevel`，因此反射率采集报告的值为`reflectance`），`--output PATH` / `-o`（离线模式下：一个路径；若带图像扩展名，文件名将基于其根目录 ——`-o ndvi_FRGN.png`会写入`ndvi_FRGN.tif`和`ndvi_FRGN_LUT.png`——若无扩展名，则作为前缀，如`-o ndvi` -&gt; `ndvi.tif`、`ndvi_LUT.png`；若未指定，则默认位于输入文件旁， 如 `<input name>_<preset>`；若包含 `--live` 且不包含 `-o`，则会在执行命令的文件夹下生成一个名为 `output/` 的文件夹）， `--output-format all|raw|tif|colorized|lut|png`、`--gradient NAME|JSON`、`--vmin/--vmax/--percentile LO,HI`、 `--bg-mode clip|transparent|indexColor|backgroundColor`、`--colorize`、`--list-presets`、 `--list-gradients`。对于 `--live`，对齐变形旋钮同样适用：`--save-multiband`、`--gpu/--no-gpu`、`--no-crop`、 `--bit-depth 8|12|16`、`--vignette`、`--interpolation nearest|linear|cubic|lanczos`、`--border-mode constant|replicate|reflect|wrap`、`--border-value N`、 `--brightness N`（摄像机的自动曝光目标，参见瞬态规则），`--serials SN1,SN2,…`。`--live` 打开一个 [瞬态数组]（#多摄像头瞬态会话）：若某摄像头未传输任何帧，则运行失败（退出 `1`，并命名该摄像头），且不会写入栈或索引。它会打开 `--serials`（第一个触发器），否则则依次处理配置文件中 `layer_order` 条目已启用的相机，优先处理其参考相机——即使该参考相机的波段被禁用，因为其他相机已与 该相机，且由其触发）。禁用波段的相机永远不会被校准，因此配置文件中禁用的RGB相机将不再导致反射率索引（`no-absolute-radiance-scale`）失败。未指定任何相机的配置文件将像以前一样打开网络上的所有相机。

所有表单都在后端运行，因此后端必须处于运行状态： 请通过 Chloros GUI 启动它，或使用 CLI 在无法连接时输出的命令。在脱机状态下，后端会读取 `--input`，并自行写入浮点 TIFF 和着色 PNG 文件，因此相对路径 `--input` 或 `-o` 文件名将根据您执行命令的文件夹路径进行解析（参见 [约定](#conventions)）。 这两种离线生成文件均包含输入文件的相机记录，与 `process` 索引导出格式一致：品牌/型号/序列号、GPS 信息、镜头标签、拍摄时间（即输入文件自身的拍摄时间；当其 EXIF 信息中未包含该字段时，则采用其记录的曝光瞬间或文件名中的时间戳， 以UTC为单位）以及MAPIR校准标识——绝非raw-DN校准，因为后者不适用于索引——此外还包含`Chloros:ProcessingLevel=index`和索引表达式；浮点数TIFF标识一个波段，即预设。 `--list-presets` 和 `--list-gradients` 会输出后端的预设表和渐变表。 如果后端拒绝某个公式，或者TIFF文件中不存在某个 `--channel` 波段，则会以后端给出的原因退出 `1`。

> **`--channel` 符号区分大小写。** 符号一侧必须与预设的通道名称完全匹配（预设使用小写，例如 NDVI = `red`,`nir` — 请检查 `--list-presets`），且波段一侧必须与对齐堆栈中的波段名称匹配（或在离线模式下为以 0 为起点的波段索引）。`--channel red=Red_660_RGN --channel nir=NIR_850_RGN` 有效；`--channel RED=660` 会因 `channel_map missing entries` 错误。

#### `lattice index` — 多波段堆栈掩码

在 `--capture-level radiance` 或 `reflectance` 处，每台相机的 3×3 解混运算可将某个波段的值限制为 0 （参见 [解混剪切掩码](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on))。多波段堆栈包含多台相机的波段数据，每台相机都会对自身波段进行钳位，因此 `--live --save-multiband` 会在堆栈旁为**每台源相机写入一个掩膜**：

```
output/
├── index_multiband.tif
└── clamp_masks/
    ├── index_multiband_SN<serial A>-<filter A>_clampmask.tif
    └── index_multiband_SN<serial B>-<filter B>_clampmask.tif
```

- 每个掩膜都是基于**堆栈**像素网格（经过相同的对齐变形和裁剪）的单波段 `uint8`，包含该相机自身的位：位 0 = 其 R 通道的波段，位 1 = G， 位 2 = B。从某台摄像机的第 *c* 通道提取的堆栈波段，会在该摄像机的位 *c*（R = 0, G = 1, B = 2）被设为 1 时进行裁剪。
- 每个波段的嵌入式来源信息会指定其摄像机的蒙版 （`unmix_clamp_mask`，相对于堆栈文件夹，或 `none`）以及待读取的位（`unmix_clamp_mask_bit`）。
- 若相机的解混操作未对任何通道进行限幅，则不会生成文件；单色或RGB相机不执行解混操作，同样不会生成文件。CLI会在图集下方打印每个掩码（`clamp mask SN…: …`）；响应中将其列为`clamp_masks`。
- 同一运行的索引 TIFF / PNG 文件是基于同一栈、在同一网格上计算得出的，因此这些掩码也描述了该文件。`--capture-level raw` 不执行解混操作，也不写入掩码。
- 堆栈和索引直接位于 `--output` 中，因此掩膜文件应存放在 `--output` 自身的 `clamp_masks/` 中——绝不会位于您指定的文件夹之上。 该文件夹混杂了多种产品类型（堆栈、索引、彩色渲染）：请从中挑选所需文件，不要将整个文件夹直接交给摄影测量软件。

#### 持久连接（Smart-Prep，等同于 GUI 的流程）

这些命令可在多次调用 CLI 时，让相机在后端池中保持打开状态。

| 子命令 | 用途 |
| --- | --- |
| `lattice cam-connect [--serial SN]` | 向池中添加一台相机（单相机，非数组）。[通用选项](#common-options-shared-by-most-subcommands) `--exposure`、`--gain`、`--preset`、 `--pixel-format`、`--width` / `--height` 和 `--packet-size`（`standard` = 1500，`jumbo` = 9000，或字节；`auto` 探测）在连接时 时应用（`--exposure` 或 `--gain` 开关会自动关闭该模式）。 当摄像头的流开始传输后，该命令会返回（等待约15秒，然后等待10秒以获取其第一个完整的缓冲区），并在流未传输任何数据时打印`stream: streaming`或`stream: no_frames -- connected, no frames yet: …` ——对于使用巨型数据包的摄像头，此时会建议使用 `--packet-size standard` 重新连接。当一台摄像头正在连接时（首次连接会下载其校准数据，约需 70 秒），另一个需要该摄像头的命令最多会等待 5 秒， 随后因HTTP 503错误（错误代码`CAMERA_POOL_BUSY`，其中包含正在连接的摄像头名称）而失败；请稍后重试。`lattice status`及摄像头列表从不等待：处于连接过程中的摄像头会显示为“正在连接”，并标注其连接阶段。 |
| `lattice cam-disconnect [--serial SN] [--all] [--keep-settings \| --clear-settings]` | 释放。 相机将恢复至连接时被发现时的状态（触发器接线、PTP、设置）；`--keep-settings` 将其设置保存到自身内存中，`--clear-settings` 将该内存重置为出厂设置 （参见 [相机内存](#camera-memory----keep-settings----clear-settings)）。作为数组成员的相机将释放整个数组。 |
| `lattice cam-list` | 列出池中的相机。 |
| **`lattice array-connect`**|**连接一个持久化同步数组（推荐的入口点）。** 运行完整的 GUI 智能预备流程。处理 9 台摄像头约需一分钟； 摄像头刚通电时可能需要数分钟。等待期间，它会打印后端执行的每个步骤，如 `    <s> s: <step>`（例如 `    41 s: open SN:214000535 (4 of 9)`）， 后端日志中每一步会有一行时间记录（如 `[array-connect <id>] <step>: <s> s …`），最后一行则列出耗时最长的步骤。 等待时间最长为 600 秒。若届时仍未建立连接，后端将撤销操作：所有已打开的摄像头均将恢复至初始状态并关闭， 该命令会输出包含当前所达步骤的 `Error:` 并退出（`1`），此时没有任何数组连接成功（HTTP 504 `CLIENT_DEADLINE_PASSED`）。 Ctrl+C 也会以同样的方式取消连接（`Interrupted: …`）；如果数组已建立连接，该行会注明此情况并列出释放连接的 `array-disconnect`。在等待期间被终止的命令 (`kill -9`)，当后端在 30 秒内未收到该命令的响应时，连接将被撤销。已启动但尚未同步帧组的数组仍会被报告为已连接，其流标识为 `no_frames`。 |
| `lattice array-disconnect [--array-id ID] [--all] [--keep-settings \| --clear-settings]` | 释放一个数组。 每台摄像机均保持自由运行状态（`TriggerMode` `Off`），其余所有设置均保持连接建立时的状态——包括其触发器和同步线设置（主机的的 `TriggerSource` 恢复为原状，其同步线恢复为输入模式），其 PTP 设置，以及会话中修改过的相机设置——因此对成员设备执行单相机命令可立即生效。`--keep-settings` 将每台相机的的设置保存到相机自身的内存中；`--clear-settings` 将该内存重置为出厂设置。参见 [相机内存](#camera-memory----keep-settings----clear-settings)。 |
| `lattice array-list` | 列出已连接的阵列。 |
| `lattice array-status [--array-id ID]` | 实时帧率、PTP、 最近错误。`ptp_enabled` 仅在所有成员的 PTP 读取状态均为“开启”时等同于 `True`；`ptp_off:` 则指代一个 PTP 读取状态未开启的成员。当 `state:` 行在设置重启期间读取 `restarting (applying settings)`，在断电成员重新打开时读取 `recovering (…)`（参见下文注释）， `stalled (…)` 表示抓取循环正在运行但未发布帧组，而 `stopped (…)` 表示抓取循环已放弃（请重新连接阵列）。 |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--orient] [--index\|--no-index] [--no-daq] [--daq-tolerance-ms MS] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | 来自实时阵列的一次同步捕获 — 单次 / 连续 / 间隔 / 最快。**默认值为 `all`**（每台摄像头按适用导出类型生成一个文件），与应用程序的捕获方式**对齐**（原生文件为 `--no-aligned`）。被跳过的摄像头（例如 RGB）将通过 `Skipped: SN:<serial> (<reason>)` 报告；用于读取该捕获数据的 DAQ 读数将一并保存，其中包含仅原始数据的捕获，并通过 `DAQ: <path>` 报告（[DAQ 读取捕获数据时的保存](#the-daq-reading-a-capture-saves))。参见 [捕获模式、记录器与离线重处理](#capture-modes-recorders--offline-reprocess)。|
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only] [--expr EXPR]` | 将实时组合索引视图录制为视频/GIF（监控级；当未打开任何GUI视图时，后端会渲染数组的NDVI，或`--expr`）。 |
| `lattice array-burst [--duration S] [--max-frames N] [--orient] [--build] [--products …]` | 高帧率原始拜耳连拍 （分析级；需离线重新处理）。与 `array-record` 和 `array-capture --continuous` 类似，它会在每次轮询时向后端报告状态：若被终止则不停止运行（`kill -9`），后端会在最后一次报到后 30 秒终止该进程（`stop_reason` `client_gone`；此时连拍的清单已最终确定，并开始构建合并索引，这与终止操作的效果相同），而数组保持连接状态。 |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | 将保存的原始连拍数据重新处理为校准后的视频。 |

> **若相机因断电或链路中断而失去响应，它会自动恢复。** 当某个成员停止响应时（交换机重启会重启其供电的每台相机），阵列会等待该成员，并在其再次响应时重新建立连接， 并将 `array-connect` 设定的参数（PTP、触发和同步信号、网络步调、像素格式和帧大小以及自动曝光）恢复给该摄像头。随后，该摄像头将无需新的 `array-connect` 设置即可继续流式传输。 与此同时，`array-status` 显示 `state: recovering (...)`，而 `last_error` 则指示正在等待的摄像机；随后它会输出 `re-opened:`。 重启后的相机恢复时 PTP 处于关闭状态，因此直到其被重新打开之前，`ptp_enabled` 显示为 `False`，而 `ptp_off:` 则标明其名称。

> **被占用的摄像头绝不会被第二次打开。**第二次打开会停止持有者的流并重写其设置，还可能导致后端崩溃。 因此，对于由池（GUI 或 `cam-connect`）持有的摄像头、数组成员，或由后台任务持有的摄像头（正在运行的 `lattice continuous`， `multi-stream` 或 `align-stream`，无论是自身还是其他客户端的）、`lattice continuous`、 `lattice capture`、`cam-connect`、`power --sleep`、`calibrate`、一个次请求的`white-balance`和`presets --apply`会以HTTP 409状态码拒绝`CAMERA_HELD`，并注明持有者及释放命令 （`chloros-cli lattice cam-disconnect -s <SN>`、`chloros-cli lattice array-disconnect --array-id <ID>`，或对于任务而言，其ID和发起者），然后退出 `1`。 另一条命令当前正在使用的摄像头也是如此（一条 `multi-test` 或 `multi-capture` 命令正在打开其摄像头，另一条是一次性命令），对于此处的每条命令：错误信息提示等待该命令完成。 没有 `--sleep` 的 `lattice power` 会读取持有者自己的句柄（如果存在的话，例如池、数组或作业的已打开摄像头），并因该摄像头正被其他命令使用而被拒绝。 每个 `CAMERA_HELD` 响应都包含 `code` 和 `error_code` 中的代码。有关打开所有摄像头的命令，请参阅 [多摄像头（临时会话）](#multi-camera-transient-sessions)。

##### `array-connect` 选项

| 标志 | 默认值 | 说明 |
| --- | --- | --- |
| `--serials SN1,SN2,…` | 自动发现所有 LATTICE 摄像头（需 ≥2 个） | 首个串行设备即为主设备。若省略该选项，则仅发现 LATTICE（M3C / M3M）型号并连接所有设备。 |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO 同步线。 |
| `--target-fps F` | 自动 | 主控器触发频率。 |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | 自动 | 覆盖层级选择器。 |
| `--wire-ceiling-mbps MB_PER_S` | 自动检测 | **主机的持续带宽预算（单位：MB/s）——整个阵列的资源分配以此数值为依据。** 当数组报告 GVSP 损坏帧时，请降低此值：自动生成的数值基于网卡报告的链路速率，但该数值会高估 USB 适配器、带宽有限的 PCIe 通道以及繁忙的共享互连网络的实际性能。该值会保存在项目的数组捕获块中，因此重新打开连接、通过 CLI 或 SDK 重新连接均可恢复该值。 参见 [数组健康状况](#array-health--which-subsystem-is-losing-frames)。 |
| `--binning {1,2,4}` | 无（全传感器） | 硬件像素合并（始终为平均值）。 |
| `--width N` / `--height N` / `--pixel-format FMT` | 全传感器；按摄像头格式 | 为每台摄像头指定显式的帧大小或像素格式，而非使用全传感器和按摄像头格式（Mono12 / BayerRG12）的默认选择。 |
| `--no-recommend` | 关闭 | 跳过网络分析步骤 （此时每台摄像机将保留其当前尺寸设置，而非被设为全传感器尺寸）。 |
| `--no-ptp` | 关闭 | 禁用 PTP（此时跨摄像机时间戳将**无法**进行比较）。 |
| `--packed {auto,on,off}` | `auto` | 在本计算机上以打包形式流式传输 12 位帧并进行解包——在全画幅模式下每秒帧数更高，数据内容完全相同。`auto` 仅在 且本计算机性能允许时才进行打包；`off` 绝不进行打包。参见 [打包的 12 位传输](#packed-12-bit-transport---packed)。 |
| `--packet-size {auto,jumbo,standard,N}` | `auto` | 阵列中每台摄像头的 GVSP 数据包大小。 `auto` 允许每台摄像机探测网络；若 `jumbo` = 9000、`standard` = 1500 或指定了以字节为单位的大小，则会锁定该值。 在 1.3.0 版本之前，`array-connect` 会接受此标志但将其忽略。 |

> **默认使用全传感器模式。** 若未设置 `--width` / `--height` / `--binning` /
> `--pixel-format`、`array-connect` 时，会将每台摄像头设置为全传感器模式——不
> 进行像素合并、 不进行ROI裁剪——并允许帧率降至该计算机和
> 网络所能提供的水平，而非通过缩小帧尺寸来维持帧率。
> 网络分析行显示预计帧率，下一行列出了
> 更快的选项（像素合并或更小的ROI）及其预计帧率；跳过
> `--binning` 或 `--width`/`--height` 即可选择其中一项。Connect 绝不会自行进行像素合并或裁剪
> 操作。

##### 压缩的 12 位传输（`--packed`）

在全传感器模式下，数组的帧率取决于每台相机发送
帧的速度。 12 位格式（`BayerRG12`、`Mono12`）通常以 16 位
字的形式传输；**紧凑**传输方式将相同的 12 位数据以每像素 1.5 字节的形式发送，并
在该计算机上进行解包，因此每帧的传输时间减少25%——
在**全传感器模式下每秒帧数增加约33%**（实测：10.7 → 14.0 fps
，基于9台摄像机组成的阵列）。 **节省下来的数据完全相同**：数值相同、
16位文件格式相同、元数据也相同。

- `auto`（默认模式）仅在能提升效率**且**该计算机有足够
  处理能力进行解包时才进行压缩。 连接时，后端会测量自身的解包速度、CPU 核心数、
  当前 CPU 负载和网络带宽，因此核心资源充裕的台式机会在
  全传感器模式下进行压缩，而繁忙或低功耗的计算机则以未压缩形式进行流式传输。较小的感兴趣区域（ROI）和合并帧能带来
  的收益微乎其微。
- `off` 绝不进行压缩——可在诊断时使用它来排除压缩，或用于
  必须将全部 CPU 资源留作其他工作的计算机上。
- `on` 始终对 12 位格式进行打包。

connect 输出将显示决策结果及原因（`打包 12 位：开启 (自动：
预计 11.0 -&gt; 14.4 fps，…))`); `/api/camera/array/status` 报告
`packed_transport` 和 `packed_decision`。若要设置全机默认值
而非每次都传递该标志，请在后端环境中设置 `CHLOROS_ARRAY_PACKED12=off` （或
`on`）——但显式设置的 `--packed on|off` 仍具有优先级。
8 位和 16 位格式绝不会被压缩。

### 一个项目记录，支持所有模式

一个项目文件夹包含所有设备的设置：`cameras.json` —— 每台摄像头的设置（`cameras[].settings`， 针对单个摄像头及数组中每个成员）以及每个数组的全局设置（`arrays[].capture`：格式、ROI、像素合并、触发率、带宽预算）——还有 `sensors.json`， 每个光传感器的参数（积分时间、帧平均值、上限）。应用程序、CLI、SDK 和 MCP 服务器均以相同方式读写相同的文件：

- **连接到某个项目的设备会被保存到该项目中，并将每次更改记录到该项目中。** “在项目下”是指后端（应用程序）中已打开该项目，或者——通过SDK——经由一个名为`ChlorosProject`的文件，该文件指定了自己的文件夹。当项目尚未包含的相机、阵列或光传感器连接时，其相关信息会被添加到该项目的文件中 （`lattice cam-connect`、`array-connect`、`daq pool-connect`、应用程序、SDK、`chloros-mcp`），并包含其设置。 从任何模式（`lattice exposure`、`gain`、`configure --set`、`resolution`、一个数组的的设置和触发率、光传感器的上限、应用程序的控制项）的任何更改，都会在片刻后保存到该项目的文件中。 在相机上设置的曝光、增益或自动曝光（AE）目标会被保存为“保持”状态，因此重新打开项目时会再次保持该设置。
- **打开项目会根据该项目设置其设备**，在所有模式下均如此：未设置自身参数的相机、阵列或光传感器将采用项目中保存的设置 （`lattice cam-connect` / `array-connect` 在项目打开时，`chloros-cli project connect`、`open_project().connect_all()`，以及应用重新打开时）。 使用新 ID 连接的阵列会更新项目中已保存的包含相同摄像头的阵列；包含其他摄像头的阵列则会添加在其旁边。
- **应用程序自身的保存操作绝不会用其较旧的副本**覆盖设备的实时设置，并会保留通过 CLI 或 SDK 调用者保存的每个成员的覆盖设置。
- **您设置的像素格式即为项目所保留的格式。** 在“数组设置”中、通过 `--pixel-format` 或 `connect_array(pixel_format=)` 设置的格式（包括 8 位格式）都会被精确保存并原样恢复。 若未设置，每台相机将以 12 位默认格式连接（单色传感器上为 `BayerRG12`、`Mono12`）。格式名称代表其位深度；“家族”指传感器类型，因此彩色相机绝不会被设置为单色格式，单色相机也不会被设置为拜耳格式，且当格式被映射时，行 `[PixelFormat <serial>] ... written` 会明确标注这一点。8位捕获的数据导出均为8位，包括反射率数据：经过重新缩放并标注其`PixelScale`标识。
- 未关联任何项目的设备不会进行任何记录。
- **录制操作将遵循其后创建的项目；关闭时则保留原项目。**当创建项目时——无论是通过SDK的`create_project()`、`chloros-mcp`的`create_project`，还是应用程序的**新建项目**——或复制（应用中的**复制**，该操作会打开副本）时，如果此时录制仍在向之前打开的项目中写入数据，则录制会继续进行，其写入的内容最终会进入新项目，位于该项目内的相同位置（下方）。打开现有项目——`chloros-cli process`、`project devices --reconnect`、 `ChlorosProject.process()`，即应用的**“打开项目”**功能——会使正在录制的视频继续在原项目中进行：录制完成后，其文件将加入该项目的文件列表（应用在打开项目前会停止所有正在进行的录制）。
  - `array-record` 和 `array-burst`：录制停止时，视频文件或包含所有帧的连拍文件夹会整体移动到该位置（其 `--duration`， Ctrl+C）。它输出的 `Burst dir` 是新文件，而 `--build` 和 `array-build-video` 则以此为基础进行处理。若移动操作被磁盘拒绝，录制文件将保留在原写入位置；末尾的日志会说明原因，停止时的HTTP响应（`relocation_error`）也会说明原因。已停止的突发录制中的原始帧和`.daq`会加入其最终所属项目的文件列表； 视频（即 `array-record`，即突发数据生成的视频）不会被添加到项目的文件列表中。
  - `array-capture --continuous`：变更后的每次捕获都会保存到新项目中；变更前保存的则保持原位。
  - `daq pool-record` 和 `project sensor log`（以及应用自带的光线传感器录制）：当录制停止时，`.daq` 会移入新项目的的 `light_sensor` 文件夹中，并加入该项目的文件列表。

  若录制保存到打开项目之外的文件夹（例如位于其他位置的 `-o`） 将保留在原位置，而 `lattice continuous` 和 `multi-stream` 则保留其被分配的文件夹。
- **在应用中，新建项目后的捕获操作会进入该项目。** 创建或复制项目时，所有正在进行的捕获运行和录制都会继续，此后进行的每次捕获（“捕获全部”、“开始捕获”）都会保存在新项目中并加入其文件列表。在应用中打开现有项目会首先停止所有这些操作。
- **项目在录制过程中不会被移动。**将已打开的项目移动到另一个工作目录（应用程序的**设置 → 工作目录**； 没有CLI命令可以移动项目，但HTTP客户端向`/api/move-project`发送请求可以实现）将因HTTP 409 `PROJECT_RECORDING`错误而被拒绝，前提是此时数组录制或连拍、数组连续捕获、 `lattice continuous` / `multi-stream` 任务或光传感器录制正在向其中写入数据时：`Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.` 应用程序也会以同样的方式拒绝， 在请求选择文件夹之前，当其自身的“全景捕获”、“开始捕获”或光传感器录制正在运行时。

这与断开连接时摄像机的状态无关（参见 [摄像机内存](#camera-memory----keep-settings----clear-settings)）：项目在打开时会自行设置其摄像机。

### 摄像头内存 — `--keep-settings` / `--clear-settings`

当连接一个阵列时，其摄像头会采用该阵列的设置，随后您可以对其进行修改。当Chloros释放摄像头时——`array-disconnect`，即瞬态命令结束时 （`multi-capture` 且无数组、`multi-stream`、`multi-test`、`align-calibrate`、 `align-apply`、`align-stream`、`index --live`）， `lattice continuous`的停止，以及其未持有的某台摄像机的`lattice capture`的结束——它会将每台摄像机恢复到连接前的原始状态： 其触发器和同步线接线、PTP 以及相机设置（完整列表见 [多相机（临时会话）](#multi-camera-transient-sessions)）。

若要在项目外使用相机，上述任意命令中的两个标志会将设置写入相机的**自身内存**——即无论在何种软件环境下，相机开机时的初始状态：

- **`--keep-settings`** 将每台摄像机的设置按会话结束时的状态保存至其内存中（即 UserSet2，作为启动配置）。 相机仍保持自由运行状态（`TriggerMode` `Off`），同步线连接保持原状。没有 UserSet2 的相机将设置保存到 UserSet1；本计算机会记住该设置，因此当 Chloros 的阵列进行自愈 ——该机制会重置已偏离Chloros基准值的UserSet1——但不会对该相机进行干预。
- **`--clear-settings`** 将每台摄像机的内存重置为出厂设置：即Chloros的基准值，该基准值是其阵列自愈机制所确定的 ——通过`TriggerMode`、`Off`、`TriggerSource`和`Line2`实现的出厂默认值， 自动曝光，RGB摄像头的校准白平衡将保留——在UserSet1中，作为启动集。
- 这两者无法结合使用。 每台相机对应一行：`SN:…: settings saved to the camera's memory (UserSet2); it powers up with them.` 或 `SN:…: camera memory reset to the factory settings.`；响应中的 `camera_memory` 针对每台相机也相同。

该应用程序不提供这些标志。

**每次退出都会将相机恢复原状。**与初始状态相同的恢复操作会在每个退出路径上运行，而不仅仅是上述命令：`cam-disconnect`（以及应用程序中的相机**X**）； 应用程序中的**X** 位于数组上或任何数组成员的行上（数组无法在没有成员的情况下继续运行，因此整个数组会被释放）；关闭应用程序（它会给后端最多 20 秒的时间来将每台摄像头、 数组以及正在运行的 `lattice continuous` 恢复正常，然后才终止它；此时仍有命令正在运行——例如，正在打开摄像头或等待其 PTP 时钟的临时命令，单次执行的 `lattice capture`、 `power`、`white-balance` 或 `presets --apply`——将在下一个安全点停止，首先将摄像头恢复原状，此过程最长持续 8 秒， 并返回HTTP 503 `SHUTTING_DOWN`；仍在运行的连接数组——`lattice array-connect`（即该应用的连接）——会在到达下一台摄像头之前停止，并关闭已打开的摄像头，持续时间最长为10秒， 并返回HTTP 503 `SHUTTING_DOWN`；而在关闭过程中到达的请求则不打开任何摄像头：HTTP 503 `SHUTTING_DOWN`）；以及被终止或终端关闭的CLI命令（SIGTERM / SIGHUP会停止其任务 ——并将摄像头恢复原状——然后才退出（`143` / `129`，类似于Ctrl+C）。 对于被直接终止（SIGKILL）的 `lattice continuous`、`multi-stream` 或 `align-stream`（此类情况无法停止任何操作），后端会在其最后一次签入后 30 秒 。发生崩溃或被强制终止的后端无法恢复任何内容；重启会将每台摄像机恢复到其内存状态，而下一次Chloros连接会重新配置它发现已启用的摄像机。**项目会自行设置其摄像机。** 当一个阵列在项目已开启的状态下连接时，每个成员将获取该项目为其保存的设置——包括曝光值、增益或自动曝光目标（作为您的设置保留，因此自动曝光功能不会对其进行调整）及其白平衡模式（由 Chloros 进行校正；相机自身的白平衡功能保持关闭）——无论相机内存中存储了什么。

### 智能自动曝光 / 智能拍摄

LATTICE 阵列一经连接，便会在后台运行连续自动曝光，但新对准的场景需要片刻时间才能收敛。 `array-capture --smart` 提供了一种**便捷的解决方案**：它会等待阵列中所有摄像头的自动曝光稳定后，再触发拍摄。当您在拍摄过程中更换场景时，请使用此功能。

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

默认的稳定策略较为保守：5 秒超时，1.5 秒稳定性窗口，±5% 的曝光波动容差。若需与自动化功能不同的行为，请通过 SDK（`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`）进行调整。

#### 自动曝光的工作原理

- **先曝光，后增益。** 当增益设为“自动”时，增益将保持在 0 dB，直到曝光时间达到允许的最长值。
- **手动设置曝光后，增益仍保持在“自动”模式**，直到您更改增益或将其设为手动。切回“自动”模式后，增益限制恢复为之前的值。
- **微小的高光不会使场景变暗。** 占画面面积不超过 1% 的镜面高光将被忽略；在已知校准目标的面板中，单个面板的镜面高光占比不超过 0.5% 时亦被忽略。
- **DAQ 曝光上限仅在校准目标处于视野内时生效。** 它可防止白色面板出现剪切；当视野中无面板时， 相机将根据场景进行曝光。
- **恒定光照下参数保持稳定，光照变化后数秒内稳定。** 在恒定光照下，曝光值及其上限保持不变；光照发生大幅变化后，画面将在几秒内稳定下来。剪切现象不再永久限制曝光，且当感兴趣区域（ROI）发生变化时，系统会重新调整曝光值。
- **亮度目标始终在可控范围内。**当相机处于曝光极限且增益无法提升时，亮度不会攀升，因此光线恢复时也不会导致画面过曝。
- **当相机的自动曝光停止响应时，其曝光值将保持在当前光照条件下的水平。** 在强阳光下，相机可能会降至最短曝光时间并保持该状态，画面变暗， 而所有设置读数均显示正常。Chloros会持续监测画面，将相机保持在该光照条件下的曝光水平，并在光线变化时自动调整。光线传感器被遮挡或处于黑暗中也不会影响这一过程。

### 捕获导出级别（`all` 默认设置）

自本版本（`lattice capture`）起， `lattice multi-capture` 以及 `lattice array-capture` **默认设置为 `--processing all`** — 每种导出类型对应一个保存文件，适用于每台相机，与图形用户界面（GUI）的“捕获全部”行为一致。级别如下：

| 级别 | 输出 | 适用对象 |
| --- | --- | --- |
| `raw` | 直接来自传感器的单通道拜耳（单色摄像头：单波段）图像。 | 所有摄像头。 |
| `debayered` | 3通道彩色，按**RGB**通道顺序保存（TIFF文件采用RGB光度解释；PNG/JPEG同样以RGB格式打开）：从原始数据中扣除校准包的暗偏移（DSNU），然后进行OpenCV的边缘感知去马赛克——未应用线性化LUT、平场校正或解混色（单色相机：1通道灰度）。Python SDK 返回的内存中去马赛克帧（`grab()`、`frame_stream()`）为**BGR** 格式，即 OpenCV 的顺序； 只有保存的文件才是RGB格式。 | 所有相机。 |
| `radiance` | 通过完整的辐射测量链获得的 float32 W/m²/sr/nm 值。 | 仅限多光谱（M3C/M3M）——**对于 RGB滤镜相机**。 |
| `reflectance` | uint16 ρ（`32768` = 1.0），支持Pix4D。 | 仅限多光谱，且**仅当已绑定数据采集卡（DAQ）且相机已校准时**有效；否则跳过。 |
| `display` | 完整的GUI预览链 （根据相机配置文件进行CCM + 白平衡 + 伽马校正）。保存在 `preview/` 文件夹中。`lattice capture` 也接受名称 `preview` 作为其名称； `array-capture` / `multi-capture` 仅接受 `display`。 | 所有摄像头。 |

传递单个级别以仅保存该级别（`--processing debayered`）。当请求 `all` 时，不适用于给定凸轮的级别会被跳过 （并予以报告），不会报错——未连接或未校准的摄像头仍会返回 `raw` / `debayered` / `preview`。

#### 捕获所使用的 DAQ 读数

每当 DAQ 与相机或阵列绑定，或被纳入池中时，捕获所使用的下行读数会被写入 **`.daq`** 文件中，该文件位于 captures 文件夹的 `daq/` 子文件夹内，其中包含其 `.match.json` （读数与曝光时间的间隔）和 `.cameras.json`（该读数所属的相机），并在 `DAQ:` 行中报告。 这一点适用于所有级别，包括原始或去拜耳化的捕获，因此任何捕获数据都可以在以后重新处理为辐射度、反射率或某个指数。 无论捕获来自GUI、CLI还是SDK，文件都是同一个，名称和sidecars也相同。如果完全没有DAQ读数，则没有`.daq`；而一个未连接数组的`multi-capture` （瞬态备用方案）则不会保存任何数据。

| 标志 | 效果 |
| --- | --- |
| `--no-daq` | 对于未在电平中使用该标志的捕获，不保存读数（`raw`、`debayered`）。但 `radiance`、`reflectance` 或 `display` 级别仍会写入该读数，因为重新导入需要该产品生成时所用的读数。 |
| `--daq-tolerance-ms MS` | 读数与曝光值的偏差范围，在此范围内仍视为在公差内（默认 500）。仅判定结果会发生变化 （`.match.json` 的 `in_tolerance` 和 `tolerance_ms`，以及反射率的 `DlsInTolerance`），绝不会改变所使用的读数。 参见 [远离曝光值的 DAQ 读数](#a-daq-reading-far-from-the-exposure)。 |
| `--force-daq` | 在仅原始数据捕获中请求 `.daq` 的旧方法。目前这是默认方式，而 `lattice capture` 和 `array-capture` 仍接受该标志（`multi-capture` 不接受）。`--no-daq` 优先于它。 |

使用哪个数据采集器（DAQ）：与相机或阵列绑定的那个（GUI 中的光传感器选择、项目中保存的绑定，或 `chloros-cli daq pool-bind`），否则使用后端中最新的那个。原始连拍 （`array-burst`）仍会像以前一样，在连拍文件夹下为每次独立读取保留各自的 `.daq`。

**PNG和JPEG每通道为8位，因此它们仅采用`display` / `preview`及索引结果。** 若 `--format png` 或 `jpg` 包含其无法容纳的电平，则在捕获任何数据之前会被拒绝，并标明该电平以及能够容纳该电平的格式（TIFF）：

- `radiance`（float32）和 `reflectance`（uint16，`32768` = 1.0） — 始终如此，因此 `--processing all`（默认值）与 `--format png`/`jpg` 也会被拒绝： 请使用 `--format tiff` 或 `--levels display`。
- 来自位深度超过 8 位的摄像头的 `raw` 和 `debayered` （即 12 位 M3C / Mono12 M3M 帧）——CLI 无法识别摄像头的位深度，因此后端会拒绝这些请求（HTTP 400）。而单帧帧的 `lattice capture --processing debayered` 抓取了 8 位帧用于 PNG/JPEG 格式，未被拒绝；同样，没有持久数组的 `multi-capture`（8 位去拜耳化）也不会被拒绝。

每条捕获路径的响应方式都相同 （`error_code` `FORMAT_CANNOT_HOLD_LEVEL`）：`lattice capture`、`multi-capture`、`array-capture`、 `project capture` / `burst`，以及其后面的HTTP路由。

### 捕获文件夹的结构

每种导出类型都会被存放在 `-o` 下的**独立子文件夹**中，因此多级捕获中绝不会混淆不同类型。 下图展示了**由 `lattice capture` 对单台摄像头进行的默认 `all` 捕获**——每台摄像头包含多个级别，因此每个文件都带有 `_<level>` 后缀：

```
output/
├── raw/           capture_<ts>_SN<serial>-<filter>_raw.tiff
├── debayered/     capture_<ts>_SN<serial>-<filter>_debayered.tiff
├── radiance/      capture_<ts>_SN<serial>-<filter>_radiance.tiff
├── reflectance/   capture_<ts>_SN<serial>-<filter>_reflectance.tiff
├── preview/       capture_<ts>_SN<serial>-<filter>_display.tiff
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_radiance_clampmask.tif, reflectance/…, index/… — the unmix clamp
│                  masks, one subfolder per product folder, only when a band clamped
└── daq/           daq_data_<sensor id>_<ts>.daq: the downwelling reading matched to the capture
                   (+ .daq.cameras.json: whose)
```

文件名的变化规则：

- **摄像机字段 — 序列号，随后是滤镜：**每个捕获文件的命名都包含拍摄该文件的摄像机名称，格式为 `SN<serial>-<filter>`，其中 `<filter>` 是该摄像机型号中的滤镜部分： `SN261902142-F450` 对应 `M3M-L87-F450`， `SN213800234-FRGN` 对应 `M3C-L87-FRGN`，`SN214701292-FRGB` 对应 RGB 相机。滤镜部分将序列号与 `-` 连接， 而非 `_`，因此名称仍采用与之前相同的 `_` 分隔字段（前缀、日期、时间、毫秒、摄像头、级别）。 无法读取型号的摄像头（DeviceUserID 已被清除）仅以 `SN<serial>` 命名。早期版本的捕获文件名称中不包含过滤器；Chloros 会读取这两种文件。
- **前缀：** `lattice capture` 将文件命名为 `capture_…`；`array-capture`、`multi-capture` 带有连接的数组，而 `project capture` / `burst` / `stream` 则命名为 `sync_<ts>_SN<serial>-<filter>…`。
- **仅当存在多个级别时才使用级别后缀。**若摄像机在一次捕获中保存了**多个**级别（如 `all` 或 `--levels a,b`），则会追加 `_<level>`。若摄像机仅保存**一个** 层级的相机则保留原名，不添加后缀——`--processing reflectance` 会写入 `reflectance/sync_<ts>_SN<serial>-<filter>.tiff`——因此针对单层级捕获编写的脚本仍可正常运行。 级别仍由文件夹名称表示。
- **钳位掩码紧跟在产品名称之后：** `<product name>_clampmask.tif`。单级反射率捕获会写入 `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`；而 `all` 捕获则写入 `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`。
- **扩展名** 紧跟在 `--format` 之后：默认的 `tiff` 生成 `.tiff`，`--format tif` 生成 `.tif`（钳位掩码始终为 `.tif`）。
- **唯一例外：** `lattice capture --processing debayered`（或 `--levels debayered`）采用快速单帧路径，并将 `capture_<ts>_SN<serial>-<filter>.tiff` 直接写入 `-o` —— 不经子文件夹。
- **流和突发** 携带相同的字段： `lattice continuous` 写入 `frame_<seq>_SN<serial>-<filter>.tiff`、`lattice multi-stream`、`stream_<seq>_SN<serial>-<filter>_<frame id>.tiff` 以及一个原始连拍文件 `<burst>_f<frame>_SN<serial>-<filter>_raw.tiff`。

`<ts>` 是捕获时间戳，`<serial>` 是相机序列号，因此一个同步组在各相机间共享
同一时间戳。**请注意一个不对称之处：** `display` 级别存储在名为
`preview/` 的文件夹中，而多级文件仅在该级别中保留 `_display` 作为 ——仅该级别中文件夹和后缀
有所不同。未知级别将回退到以其自身名称命名的文件夹中，如果子文件夹
无法创建，文件将写入输出根目录，而非丢失。

**重新处理捕获文件夹：**将 `chloros-cli process` 指向**捕获根目录**
(`output/`)。`process` 通常仅导入您指定的文件夹，但当该文件夹中没有
图像且包含子文件夹时，它会自动向下遍历——因此该级别的子文件夹以及
`daq/`子文件夹都会一次性被全部导入。捕获的每一层都会作为单张图像导入，
其余层级则作为模式提供，而非每层生成一张图像。

直接命名**层级子文件夹**（例如 `output/raw/`）同样有效。这样做会保留 `daq/`
文件，因此当您从 `raw/` 生成辐射测量
产品时，请将 DAQ 读数复制或指向该文件——否则时间戳匹配将无法找到对应的文件。

**处理始终从 `raw/` 开始——否则时间戳匹配将无法找到对应的文件。**从 `raw/` 衍生辐射测量
产品时，请将 DAQ 读取数据一并复制或指向该文件——否则时间戳匹配将无法找到对应的文件。

**处理始终从 `raw` 开始。** 在每次捕获中，原始帧是管道的源文件；
`debayered`、`radiance`、`reflectance` 和 `preview` 作为可视化模式存在，但绝不会
被送回处理管道。重新处理衍生产品将重新应用已经烘焙到其像素中的暗角、CCM和
辐射度计算，因此Chloros会拒绝处理，而非
进行重复处理。有两个值得注意的后果：

- `index/` 和 `composite/` 的渲染结果 **绝不会** 处理。它们是输出文件，而非捕获文件——
  NDVI LUT 渲染结果不具备有意义的辐射度解释。
- **未**包含 `raw`（例如 `array-capture --processing reflectance`）的捕获文件夹
  没有有效的管道源。这些捕获文件虽可正常导入和显示，但 `process` 会跳过
  它们并提示如下：

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  若您确实需要将衍生产品推送通过——例如启用了
  `demosaic` 启用的集线器会话，或旧版文件夹——`--input-level {raw,debayered,processed}` 将强制指定
  入口点并覆盖跳过设置。该标志是刻意设计的应急通道；`auto` （默认值）
  绝不会处理没有原始数据的捕获数据。

**校准标签：仅在 `raw` 上有效。** 捕获数据的 `raw` 文件包含相机的校准数据包：`CalibrationJSON` 有效载荷以及与 MicaSense 兼容的标签（`BlackLevel`、 `RadiometricCalibration`、`Band{R,G,B}RadiometricCalibration`、`MicaSenseVignetting*`），这些标签描述了传感器的原始值。 其他各级的文件（`debayered`、`display`、`radiance`、 `reflectance`、`index/`渲染文件）以及每个`lattice continuous` / `multi-stream`帧都携带摄像机的标识（`XMP-mapir:CameraSerial`、`CameraModel`、`FilterType`、 `LensModel`、`CalBundleSha256`、……），以及帧本身的记录文件 `XMP-mapir:FrameJSON`（曝光、增益、温度、像素格式、拍摄时间、 DAQ匹配），因为每个 `process` 产品都带有该标识。`process` 会根据其序列号在 Chloros 的校准缓存中查找该文件的校准数据，若 计算机中未存储该文件包时，仅需下载一次该文件包。

#### 由多台相机生成的文件

由多台相机的波段组合而成的文件——多波段堆叠（`align-apply`、`align-stream`、`index --live --save-multiband`）， 由多台相机生成的实时索引（`index.tif`、`index_LUT.png`）、阵列捕获生成的 `composite/` 图像——列出所有参与生成的相机名称：

- **按波段顺序排列的每台摄像机列表**（合成图像：背景层，然后是前景层；索引层，以及其波段来源的摄像机）：`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`、`SourceSensorProfiles`、`SourceProductLines`、`SourceFilterTypes`、`SourceLensModels`、 `SourceCalDateUTCs`、`SourceCalBundleSha256s`（每台相机波段所经过的校准包，否则 — 一个`align-apply` / `align-stream`堆栈，其波段未经过任何校准 — 相机缓存的Chloros校准包，即其捕获文件所命名的那个）， `SourceLightboxCertIds`、`SourceHalogenLampCertIds`、`SourcePackTiers` ——并列列表，每个列表的第 *n* 个条目描述同一台相机；`-` 表示某台相机无相关值（无校准包，或镜头型号未知）。 每台相机对应**一个**条目：如果其数据包中某个字段有多个值（例如数据包中的卤素灯证书），则用 `;`（`F-1479-1;F-1480-1;F-1481-1`）连接，绝不使用逗号， 因此无论列表如何显示，每台相机都对应一个条目。
- **EXIF `SerialNumber`** 包含相同的序列号，以逗号分隔（`214000535,214000551`）。
- 它不包含**任何**单台相机的标识（`XMP-mapir:CameraSerial`、`CalBundleSha256`、……）： 这些标识符分别代表一台相机及其数据包，而 `process` 将根据它们解析文件的校准信息。由多台相机的波段计算得出的索引（`index.tif`、 `index_LUT.png`）所计算的索引，其EXIF信息中包含`Model` `LATTICE` （即拍摄装置），而非单台相机的 `LATT-…` 型号。

由**单台**相机各波段数据组成的文件——无论是按波段划分的 TIFF文件（`--per-band`）、一个`align-apply --per-camera`帧、单台相机的堆栈或实时索引——都带有该相机自身的标识标签，正如其他所有捕获文件一样。 `align-apply`的堆栈、各波段文件和各相机文件，以及每个`align-stream`堆栈及其各波段文件，都包含其所属组的采集时间（参考相机的曝光时间，UTC）， 而单色摄像机的 `--per-camera` 帧仅包含一个平面，与 `--bit-depth` 堆栈类似。**摄像机方向（翻转/旋转）。** 摄像机方向设置（GUI 中的“镜像 H /
垂直镜像/旋转）决定了捕获数据的保存方式：

- **`raw` 和 `debayered` 完全按照传感器读取的数据保存**，无论设置如何。一个
  如果拜耳马赛克被旋转或翻转，其色彩滤光片图案将发生变化，且校准
  映射（暗偏移、平场）基于传感器自身的网格。
- **`preview`/`display`、`radiance`、 `reflectance` 和 `index/` 的渲染图像已进行定向处理**——在完成所有校准计算后
  旋转了一圈，因此打开时呈现的画面与相机取景时的方向一致。这些
  `clamp_masks/…_clampmask.tif` 文件位于同一网格上。
- **每个拍摄文件都会在 XMP 中记录该设置**： `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`)，`ChlorosRotation`（0 / 90 / 180 / 270，顺时针，在
  翻转之后），以及 `ChlorosOrientationApplied` — `false` 针对原始和去拜耳化（原生像素）的图像，
  `true` 用于定向处理后的结果。`ChlorosProcessing` 列出了对该文件
  像素所施加的操作（原生像素时为 `none`）。这包括 `lattice continuous` 流中的每一帧（
  相机自身的设置，未应用）以及 `lattice multi-stream` 的每一帧（恒等映射：其
  成员帧为原生像素，如同数组捕获一样）。 `lattice align-stream` / `align-apply`
  堆栈记录主相机的设置（未应用）：其波段被变形到主相机的
  帧中，但未进行旋转。
- **`process` 在处理原始数据时应用记录的朝向**：首先在传感器坐标系中进行暗偏移、平场校正、
  去马赛克和解混，接着进行阵列对齐变形，最后应用
  最后应用方向——因此预览、辐射度、反射率及指数结果与
  实时捕获的数据完全一致，并记录为 `ChlorosOrientationApplied=true`。未校准的备用
  生成结果（`Sensor_Response_Images`、`Vignette_Corrected_Images`）在
  平场校正后。去拜耳化产品保持原始状态。
- **Chloros图像查看器会根据记录信息旋转原始或去拜耳化文件**（仅限显示——
  文件本身不会被修改），因此其显示效果与产品一致； 缩略图、全尺寸视图、
  光标读数以及索引/LUT沙盒均采用相同的旋转角度。未包含记录的文件，以及
  （已定向的）产品，则按存储时的原始状态显示。 沙盒导出会记录其
  像素携带的方向（`ChlorosOrientationApplied=true`；这是无记录图层的标识符）。
- **未包含记录的拍摄数据将按原生格式处理**（行为未变）。在 1.3.0 版本之前，
  带有翻转或旋转设置的单相机拍摄会自行旋转其原始文件和去拜耳化文件；
  Chloros 无法识别此类已-旋转过的原始文件与原生文件区分开来，也不会进行推测，因此
  将其按存储状态进行处理。校准图基于传感器自身的网格，因此对于此类
  拍摄，当校准数据至关重要时，请使用 1.3.0 版本拍摄的校准图。
- **数组默认不旋转任何成员文件**（对齐变形和 `Alignment*` XMP 文件均位于
  相机自身的、未旋转的帧中），并记录其身份信息。 `array-capture --orient` 会将每个
  子帧的处理结果（在对齐变形之后）旋转：若子帧曾被
  变形到主帧坐标系中，则按**主帧**的姿态旋转（以确保子帧保持共注册）；否则按其自身相机的姿态旋转。 每个
  文件记录其自身像素所在坐标系的定向——一个原始数据（即其自身相机的定向）——
  而包含对齐变换的原始数据还会记录 `ChlorosAlignedFlipHorizontal` /
  `ChlorosAlignedFlipVertical` / `ChlorosAlignedRotation`：主帧的变换，用于将该产品
  到主帧中的产品。`process` 会根据其像素所属的变换值旋转每个产品：
  当对原始数据进行畸变时采用对齐后的变换（`--array-alignment`，默认值——实时对齐
  捕获的产物），若未对齐则使用相机自身的（`--no-array-alignment` ——实时
  未对齐捕获的产物）。`array-burst`、`project capture` / `burst` /
  `stream`、预设以及SDK中均提供该选项。

### 混合滤光片阵列中的跳过采集

当您在一个阵列中混合使用RGB和多光谱相机时，`array-capture --processing radiance`（或`reflectance`）会保存多光谱帧并**跳过** RGB摄像头——对于宽带传感器而言，拜耳阵列的每像素辐射度没有实际意义。 CLI会明确打印每个已保存的文件（及其导出级别）、每个写入的`.daq`文件，以及每次跳过的情况，因此文件数量并不令人意外：

```
  Saved: …/output/reflectance/sync_…_SN213800234-FRGN.tiff [reflectance] (SN:213800234, fid:1)
  Saved: …/output/reflectance/sync_…_SN214000533-FOCN.tiff [reflectance] (SN:214000533, fid:1)
  Saved: …/output/reflectance/sync_…_SN214701288-FNGB.tiff [reflectance] (SN:214701288, fid:1)
  DAQ:   …/output/daq/daq_data_daq-e-54b5e0_….daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

跳过原因标记遵循 `<level>-not-applicable-to-rgb-cam` 的模式。若未加载校准包，相机将跳过 `radiance` / `reflectance`（同时使用 `<level>-needs-calibration (connect the camera so its cal pack loads)` 跳过；若无可用的 DAQ 反射率读数，则跳过 `reflectance-skipped-no-dls`。反射率还可能跳过 `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`， 当波段主要位于DAQ光传感器辐射校准范围（~374–974 nm）之外时，`dls-uncalibrated-band-<nm>`也会出现反射率读数跳过的情况——在已发货的SKU中，仅F988支持该路径，其工作流程为反射率面板模式。

使用 `--processing debayered`（或 `display`）可包含所有相机（无论滤光片类型），或使用默认的 `all` 一次性获取每台相机所有适用的级别。

### 与曝光值相距甚远的DAQ读数

每次捕获保存的每个`.daq`值都是与该曝光值匹配的DAQ读数，而旁边的`.match.json`则记录了两者之间的偏差 (`gap_ms`)，以及该偏差是否在容差范围内（`in_tolerance`，默认500毫秒，除非捕获集另行设置`--daq-tolerance-ms`）。 超出容差范围的读数仍会被采用，因此不会丢失任何帧，并且会进行标记：反射率文件的XMP中包含`DlsInTolerance=False`和`DlsMatchGapS`。 `lattice capture`、`multi-capture` 和 `array-capture` 也会在相关的 `.daq` 下打印该信息，并附带 DAQ&#x27;流故障：

**读数如何与曝光时间匹配。** 捕获操作会等待曝光结束后DAQ的首次读数，以便融合该读数前后两侧的读数（`method` `interpolated`）。DAQ-E在日光下每0.4秒发送一次读数，在室内每1-2秒发送一次，因此等待时间会使命令的总时长增加约一个半这样的间隔，而非曝光时间。如果DAQ停止发送数据，则不会等待。DAQ-E 的读数是自上次读数以来该时间间隔内光照的平均值，因此被放置在该时间间隔的中点：`gap_ms`是从曝光开始到该中点的测量值，而`bracket_stamps_ns`则给出读数本身的时间戳。 `array-capture --continuous` 在不影响捕获速度的情况下进行等待：其原始文件会立即写入，而 `.daq` 则在读数到达后生成。 对于 `--interval`，每次循环的等待方式与单次采集相同。

```
  DAQ:   …/output/daq/daq_data_daq-e-54b774_….daq
         DAQ reading 114.9 s from the exposure (tolerance 500 ms): reflectance uses it, labelled DlsInTolerance=False
         DAQ stream fault: no frame accepted from daq-e-54b774.local in 25 s, after 1932 good frame(s). …
```

若该读数未写入反射率文件，则第一行以 `saved anyway; its .match.json says in_tolerance: false` 结尾。

捕获仍以 `0` 结束：所有文件均已写入，并标明了其内容。数据流已停滞的 DAQ 仍会持续提供其最后一次读数，因此在依赖该捕获的反射率之前，请先通过 `chloros-cli daq pool-list` 进行核查。 后端会重新连接一个已停止传输数据的池化 DAQ-E （参见 [停止传输数据的 DAQ-E](#a-daq-e-that-stops-delivering)）。超出容差范围的读数仅打印其 `DAQ:` 行，除非该读数被保留（详见下文）。

**DAQ 失效后仍被保留的读数。** 当没有 DAQ 能为某次曝光提供读数时（绑定的 DAQ 不可用，或没有池化 DAQ 提供读数，例如在 `daq pool-disconnect` 之后），捕获操作将复用Chloros拥有的最后一个读数，其`.match.json`将该读数与本次曝光关联：`method`、`retained`、`gap_ms`，将该曝光与产生该读数的样本相关联；`in_tolerance`来自该间隙；以及`retained_from_epoch_ns`，即该读数最初匹配的曝光。因此，在数据采集（DAQ）停止后很久才进行的捕获会打印出上述行，并在 下方还会多打印一行，即使该读数仍处于公差范围内：

```
         This is the DAQ's last reading: the DAQ is no longer reporting, so each capture reuses it until a new one arrives
```

其 `.daq` 仍标明了采集该读数的 DAQ，以及该读数所处的校准和封顶条件，因此重新导入时可知该读数由哪个 DAQ、校准和封顶条件生成。

### 写入失败的文件

如果某台摄像头的文件无法写入（例如硬盘已满或权限错误），也不会阻止其他摄像头的文件保存。每个捕获命令（`lattice capture`、`multi-capture`、 `array-capture`；`project capture`、`burst`、`stream`； 以及配方中的 `capture`、`burst` 和 `stream`）都会将每个保存失败的文件输出到 stderr， 随后**退出 `1`**，尽管其余文件已保存：

```
  Failed: SN:214701292 [reflectance] OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

捕获所涉及的每个文件均已涵盖，其中包含单帧单帧去拜耳化文件 `lattice capture --levels debayered` 也不例外。一个 `.daq` 读取操作，其 `.match.json` 来源 sidecar 文件无法写入，被列为 `[match]`（`.daq` 文件本身已保存）。

在磁盘已满的情况下，重复的捕获（`array-capture --continuous` / `--interval`、`project burst` / `stream`、一个配方 `burst` / `stream`) **在该迭代中停止**，而不是在后续运行中对同一驱动器不再写入任何数据。任何其他错误都会在发生时于每轮迭代中报告，且运行会继续进行至结束，随后退出 `1`。`array-capture --continuous` （后端的连续捕获）中不会列出任何文件（包括失败的文件）：它会在进度和摘要行中统计这些文件 （`N failed`、`N file(s) failed to write.`）中进行统计，并退出 `1` —— 参见 [连续模式](#continuous-mode---continuous)。

记录器 — `array-burst` 和 `array-record` — 会报告自身的错误。 当驱动器空间已满或即将已满时，`array-burst` 会自动停止。这两者都会在遇到其他写入失败时继续运行。当其中任何一个报告错误时，该错误会显示在计数器之后，且命令 **退出 `1`**：

```
  Recorder error: disk-full: OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

`Disk low:` 代替 `Disk full:` 表示突发写入在填满 。停止前录制的帧将被保留。当空间不足时，`array-burst --build`会跳过构建过程，因为视频将写入同一驱动器。请先释放空间，然后运行 `array-build-video --burst-dir …`。在发生任何其他错误后，构建仍将继续运行。

`array-record` 若未写入任何帧，也属于录制失败 （`Recorder error: no-frames: the combined view published no frame to record …`，退出 `1`）——参见 [`array-record`](#array-record--combined-index-videogif-monitoring-grade)。

---

## 捕获模式、录像机与离线重处理

这些操作均基于 **持久数组** 进行（请先运行 `array-connect`）。它们与 GUI 捕获面板的功能一致。

### `array-capture` 模式

`array-capture` 是一个包含四个快门模式及一组导出开关的单一命令：

| 模式 | 标志 | 行为 |
| --- | --- | --- |
| **单次** *(默认)* | (无) | 捕获一个同步捕获组，然后退出。 |
| **连续** | `--continuous` | 连续捕获，直至捕获到 `Ctrl+C`、`--count N`， `--duration S`，或磁盘已满。运行后端自身的连续捕获（即 GUI 中的“连续”模式）——详见下文 [连续模式](#continuous-mode---continuous)。 |
| **间隔** | `--interval S` | 每 `S` 秒执行一次（从每次循环开始计时），边界范围不变。 |
| **最快** | `--fastest` | 仅原始模式 + 分配的 DAQ 读数 + 组合指数复合值；跳过辐射度/反射率/显示计算，从而快速输出帧。隐含 `--processing raw --force-daq`。稍后将保存的 `.daq` 重新处理为校准产品。 |

导出开关（可与任何模式组合使用；均共享同一 GUI/SDK 端点）：

| 标志 | 效果 |
| --- | --- |
| `--array-id ID` | 要捕获数据的数组（仅连接一个时省略）。 |
| `-o, --output DIR` | 输出文件夹（默认 `output`，位于运行该命令的文件夹下）。 |
| `--format {tiff,tif,png,jpg,jpeg}` | 文件格式（默认 `tiff`）。 PNG/JPEG仅支持`display`及索引渲染；参见[捕获导出级别](#capture-export-levels-the-all-default)。 |
| `--processing LEVEL` | 单个导出级别 (`raw`、`debayered`、`radiance`、`reflectance`、`display`)，或 `all` （默认）。 |
| `--levels L1,L2,…` | 导出类型的显式子集（例如 `raw,radiance,reflectance`）；**覆盖 `--processing`**。 |
| `--aligned` / `--no-aligned` | **默认：对齐，与Chloros应用捕获同一数组时的设置一致**（其针对每个数组的**对齐**捕获设置，默认开启，除非您将其关闭）。每个成员的非原始导出（`debayered`、`display`、`radiance`、`reflectance`，即索引渲染） 都会被变换到主摄像头的帧内，并裁剪至各成员的共同重叠区域，因此生成的文件在像素上已对齐，其尺寸、裁剪范围和 `Alignment*` 标签与应用程序生成的文件一致——比传感器 （或ROI）的尺寸小；在ROI模式下，变换会重新基于窗口进行校准。`XMP-mapir:AlignmentJSON`表示`"applied": true`，并生成`crop_roi`。 原始文件保持原生尺寸且未进行变形，但会在元数据中携带变换信息（`"applied": false`）。`--no-aligned`将每个文件以原生尺寸保存，且未进行变形。 `--aligned` 是默认的明确设置。无论是否应用了畸变校正，一旦数组拥有对齐配置文件，每个成员文件都会记录其摄像机的 `Alignment*` 变换 ——但以下情况除外：无法与配置文件对齐的相机（被排除在外，或因求解失败而被迫设为单位矩阵），以及在无其他相机对齐时的参考相机：这些文件不包含`Alignment*`标签。若数组上无配置文件，则以未对齐状态保存捕获数据，且CLI会在`Alignment:` 行中打印原因，这与其他无法应用畸变的情况（成员分辨率不同、数组未通过电缆同步、轮廓无法映射到帧上）的处理方式一致。数组的自动对齐功能会在曝光稳定后生成轮廓； `align-calibrate` 则会按需生成一个。 |
| `--orient` | 将每个成员的**产物**——`display`、`radiance`、 `reflectance`、`index/` 渲染结果以及 `composite/` —— 根据相机方向，在对齐变形后：将产品通过**主帧**的姿态进行畸变，从而保持各成员文件的配准一致；未对齐的渲染结果（如 `--no-aligned`，或无法应用畸变变换的捕获数据）则采用其自身摄像机的姿态。**默认：关闭 — 所有成员文件均保持原生格式**，这与数组捕获数据一贯的做法一致。 无论哪种情况，`raw` 和 `debayered` 均保持原生坐标系。每个文件都会记录其所包含的产物所携带的朝向（参见 [相机朝向](#what-a-captures-folder-looks-like))；`Alignment*` 的 XMP 文件保留原生帧变换（在方向处理之前应用），因此 `process` （先进行畸变校正，再应用方向）可还原原始渲染结果。 |
| `--index` / `--no-index` | 保存 / 跳过已配置的每台相机植被指数叠加层。默认：进行渲染。 |
| `--no-daq` | 当所选层级均未使用 DAQ（仅抓取原始数据）时，跳过 `.daq` 辅助文件。 默认情况下会保存读数，以便离线将帧重新处理为反射率/植被指数。仍接受 `--force-daq`（默认值）。 |
| `--daq-tolerance-ms MS` | 这些捕获的 DAQ 匹配容差 （默认 500 毫秒）。 |
| `--smart` | 在触发前等待所有相机上的 AE 稳定（参见 [智能 AE / 智能捕获](#smart-ae--smart-capture)）。 |
| `--compression {deflate,none}` | TIFF像素压缩。`none`（自1.3.0起为默认值；1.2.0版本写入的是`deflate`）= 无压缩 ——完全按读取时的原始状态保存传感器像素，每帧全分辨率图像约 6.3 MB，且具有最快的持续写入速率；`deflate` = 无损 zlib L1 压缩 + 水平预测器，每帧约 4.1 MB，速度较慢。这两种格式均为无损，导入时显示效果完全一致。 |

> **单次写入TIFF + 持续速率模型。**每个文件仅写入**一次**：像素、 XMP数据包、IFD0中的制造商/型号/相机序列号以及EXIF中的`ExposureTime` / `SerialNumber`标签，均在单次写入中完成，此后不会有任何内容覆盖该文件。`--continuous`X 会捕获连续的同步数据组——每次捕获都会获取前一个数据组之后的数据组，包括在当前捕获仍在保存时发布的那个数据组——因此它以数组的速率运行。DEFLATE 会占用 Python 的 GIL，因此压缩写入在各个相机写入线程之间**不会**并行化； 以传感器速率持续进行全分辨率捕获需要未压缩写入（默认设置）**以及** NVMe级别的磁盘（9摄像头阵列在约10 fps下可持续传输速率约为600 MB/s）。该参数在 `POST /api/camera/array/capture` 上以 `compression` 的形式提供。

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (aligned is the default; drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --no-index -o out/

# Each member at its native size, unwarped
chloros-cli lattice array-capture --processing reflectance --no-aligned -o native/

# Continuous raw capture for 60 s, as fast as the array publishes
chloros-cli lattice array-capture --continuous --duration 60 --fastest -o flightline/
```

#### 连续模式（`--continuous`）

`array-capture --continuous`（不包含 `--interval` 或 `--smart`）将任务交由**后端自身的连续捕获**处理——即 GUI 的“连续模式”运行的会话。 后端会自行保存每个同步组，同时保持两个捕获任务在处理中，且组与组之间不接收来自CLI的请求，因此它能跟上高速数组的节奏——而在每捕获一次才发送一次请求的情况下则会落后（在9台摄像头的数组上，以512像素ROI为基准，测得73–77次捕获/秒）。 上述所有导出开关均适用于每次捕获。

- **边界条件：** `--count N` 在完成 N 次捕获后停止（一次捕获 = 一组同步数据：该组中所有成员的文件）；`--duration S` 则在 S 秒后停止。若未设置上述任一条件，程序将持续运行直至触发 `Ctrl+C`、磁盘空间已满、发生错误或阵列断开连接。`Ctrl+C` 将终止后端的会话， 随后打印摘要。
- **进度**每 2 秒更新一次，当值不为零时会附加 `, N failed` 和 `, N groups skipped`（被跳过的组是指阵列已发布但未被任何捕获处理的组）：

  ```
     12.0s  880 captures (73.3/s), 7920 files
  ```

- **不显示按文件列出的信息。** 运行结束时会显示一行摘要——捕获数量、速率、文件数及输出文件夹——若非零值，则随后附带 `N file(s) failed to write.` / `N group(s) skipped.`：

  ```
  Done — 880 captures (73.3/s), 7920 files in /home/user/flightline.
  ```

- **停止**：当磁盘空间已满时显示 `Continuous capture stopped: the drive is full.`；发生错误或存储阵列消失时显示 `Continuous capture stopped: <reason>.`（两者均输出至 stderr）。
- **退出状态：**当已保存捕获数据且所有文件均已写入（包括 `Ctrl+C`）时，返回 `0`；当任何文件写入失败、 磁盘已满、发生错误、数组消失或完全没有捕获数据时。
- **Microsoft Defender：**当 Defender 占用一个或多个 CPU 核心来扫描捕获写入的文件时，运行时会在 stderr 上打印一次：`Note: Microsoft Defender is using N.N CPU cores scanning the capture files, CPU the capture needs. Re-run the Chloros installer with "Exclude the Chloros backend from Microsoft Defender scanning" checked.`。请参阅安装程序的 [系统设置](#windows-exe)。
- 每个数组每次仅允许一个连续捕获；若尝试启动第二个，将返回错误代码 `a continuous capture is already running for this array`。 通过 HTTP 建立的会话为 `POST /api/camera/array/continuous/start`（包含 `/api/camera/array/capture` 主体以及 `max_captures` / `duration_s`）， `GET /api/camera/array/continuous/status?array_id=ID` 和 `POST /api/camera/array/continuous/stop`。

**间隔**（`--interval S`，无论是否包含 `--continuous`）和**`--continuous --smart`** 保持按请求循环：每轮一次捕获请求——对于“间隔”模式，在每个预定时刻执行；对于 `--smart`，在 AE 稳定后执行 ——每轮的文件列表与“单模式”相同，末尾有一行 `Done — N capture pass(es).`，并使用 `--count N` 计数轮次。

### `array-record` — 视频/GIF 组合索引（监控级）

将数组的 **实时组合索引视图** 记录到 `.avi` 文件中（并可选地记录到 `.gif` 文件中）。当 GUI 的组合视图打开时，它会记录该视图的显示内容。 若未打开任何视图，后端将自行渲染视图：即您通过 `--expr` 传递的索引，或默认情况下，在数组对齐后渲染该数组的 NDVI。 默认的NDVI值采用最接近660 nm和850 nm的红光和NIR波段，与实时视图一致。录制过程中打开的GUI视图将在其保持打开状态期间被记录。该功能每2秒轮询一次进度，并在收到`--duration`、 `Ctrl+C`，或当记录器自动结束时。

若录制过程中未写入任何帧，则录制失败：记录器将报告 `no-frames: …`（数组可能仍在对齐或稳定曝光），且命令将 **退出 `1`**。

```bash
# 30-second combined-index clip at 10 fps, plus a GIF (no GUI needed)
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# Record a chosen index instead of the default NDVI
chloros-cli lattice array-record --duration 30 \
  --expr "(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)" -o monitoring/
```

| 标志 | 默认值 | 描述 |
| --- | --- | --- |
| `--array-id ID` | 仅数组 | 目标数组（若仅连接一个则省略）。 |
| `-o, --output DIR` | `output` | 输出目录。相对路径位于您 运行命令的文件夹下。 |
| `--fps F` | `10` | 录制帧率。 |
| `--duration S` | 直到按下 Ctrl+C | `S` 秒后自动停止。 |
| `--gif` | 关闭 | 同时生成动画 GIF 文件。 |
| `--gif-only` | 关闭 | 仅生成 GIF 文件（不生成 `.avi`）。 |
| `--expr EXPR` | 数组的NDVI | 当未打开任何GUI视图时，用于记录数组波段名称的索引表达式（例如 `'(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)'`； 参见 [波段名称](#index--vegetation-maths))。简短的颜色名称（`NIR`、`Red`）仅在数组中有一个波段具有该名称时才有效。若存在打开的 GUI 视图，则记录其显示内容。 |

### `array-burst` — 原始拜耳高帧率连拍（分析级）

直接读取抓取循环的同步组缓冲区 — **无需校准链、无需 exiftool、无需实时预览** —— 因此可达到相机的全帧抓拍速率。写入原始帧 + 每帧清单 + 每个独立 DLS 读数对应一个 `.daq` 文件（位于 `<output>/bursts/<base>/` 目录下）。 离线重新处理（下一条命令），或传递 `--build` 以在停止时立即执行。

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| 标志 | 默认值 | 描述 |
| --- | --- | --- |
| `--array-id ID` | 仅数组 | 目标数组。 |
| `--array-id ID` | 仅数组 | 目标数组。 || `-o, --output DIR` | `output` | 输出目录（突发数据存入 `<DIR>/bursts/<base>/`）. |
| `--duration S` | 直到按 Ctrl+C | 在 `S` 秒后自动停止。 |
| `--max-frames N` | 无限制 | 在处理完 `N` 个原始帧后自动-停止 |
| `--orient` | 关闭 | 构建从该数据包生成的结果将根据相机方向进行旋转——单相机生成的结果由其自身相机方向决定， 合并索引则由主相机决定——如 `array-capture --orient` 所示。原始帧保持原生格式； 该连拍的 `manifest.json` 会记录每个成员的朝向及该选项。 |
| `--build` | 关闭 | 停止后，立即重新处理该连拍（与 `array-build-video` 相同）。 |
| `--products …` | `combined:index` | 配合 `--build` 使用：指定要生成的视频（见下文）。 |
| `--fps F` | `10` | 配合 `--build` 使用：输出视频的帧率。 |
| `--save-tiffs` | 关闭 | 配合 `--build` 使用：同时保存每帧校准后的 TIFF 文件（当像素被钳位时，每个文件包含其 [解混钳位掩码](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on）；构建结果中将其列为 `clamp_masks`）。 |
| `--gif` | 关闭 | 启用 `--build` 时：还会写入动画 GIF 文件。 |

### `array-build-video` — 离线重新处理已保存的连拍

将每个原始帧与最近的已保存 `.daq` 读数进行时间匹配，并将其通过**与导入管道相同的辐射度/反射率/指数处理链**进行处理，从而渲染一个或多个视频。

`--products` 是由 `kind:level` 项组成的逗号分隔列表，其中 `kind` ∈ `per_cam` | `combined`，且 `level` ∈ `radiance` | `reflectance` | `index`。 仅有的 `level`（无 `kind:`）默认值为 `per_cam`。默认值为 `combined:index`。

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| 标志 | 默认值 | 说明 |
| --- | --- | --- |
| `--burst-dir DIR` | （必填） | 突发文件夹的路径 (`…/bursts/<base>/`)。 |
| `--products …` | `combined:index` | `kind:level` 列表，同上。 |
| `--fps F` | `10` | 输出视频帧率。 |
| `--save-tiffs` | 关闭 | 同时保存每帧校准后的 TIFF 文件与视频。 |
| `--gif` | 关闭 | 同时生成动画 GIF 文件。 |

**视频不包含钳位掩码**（单图像流，8 位）。取而代之的是，构建结果包含 `clamp_fractions`：针对每个视频产品， 该视频产品中，每个帧在R/G/B通道所含色带内被解混的像素份额——即该帧的辐射度/反射率乘积将标记的`Chloros:Band{R,G,B}UnmixClampFraction`值（针对组合索引，每台参与摄像机的数据）。 该运行会在 `Clamp:` 行中打印每个视频产品的最差帧。`--save-tiffs` 为 TIFF 文件添加了逐像素掩码， 将其保存在 TIFF 文件所在的 `<level>/` 文件夹旁边的 `clamp_masks/<level>/` 中。**构建过程若提前终止，也不会产生残缺的视频文件。** 每个视频以 `<name>.partial.avi` 的形式写入，并在完成后才获得最终名称，因此同名的旧视频会保留至其替换文件完整生成为止。请重新运行构建以替换遗留的 `.partial.avi` 文件。

运行过程会打印出写入的每个视频，然后是任何 `Skipped:` 结果（指请求的相机/层级——或组合索引——未生成视频的情况，并附带原因——例如：无校准包、 无反射率DLS读数，或组合索引缺少红光/NIR波段对）以及任何`Note:`。若RGB相机被要求生成辐射度或反射率数据，则会输出其显示视频，并在相机校准包可用时进行白平衡； 未提供校准包而渲染的摄像头将在 `Note:` 中列出。当请求的产品被跳过或未按要求构建时（配置的索引表达式不会在离线状态下进行评估），该程序将**返回非零值**；已构建的视频仍会被写入并列出。

> **选择合适的记录器。** `array-record` 属于 *监控级* —— 它记录渲染后的实时合成图像：即 GUI 的组合视图，或者在未打开 GUI 时，记录后端自身的 NDVI（或 `--expr`）视图。`array-burst` → `array-build-video` 属于 *分析级* ——它以全速率保存原始传感器数据，并在事后重建经校准的辐射度/反射率/指数视频，无需实时预览。

### 单色（M3M）单波段相机

**M3M**系列是拜耳**M3C**的单色版本：每台相机配备一个窄带干涉滤光片（例如 `M3M-<lens>-F<wavelength>`、`M3M-L87-F685`）， 因此传感器输出的是**单一灰度波段**，不包含拜耳马赛克。无需进行去马赛克处理，无需分离通道间的串扰，也无需设置白平衡——整个RGB显示色彩处理流程在此完全不适用。

这在CLI中的表现为：

- **`lattice white-balance`、`lattice color-profile`、`lattice color`**检测到单色摄像头时，会**跳过并显示一行提示信息**，而非推送无意义的设置。 在同一会话中，它们对RGB/拜耳 M3C 摄像头仍能正常运行。
- **拜耳像素格式即代表其色深。** M3M 仅支持单色格式，因此来自 `--pixel-format` 或 `--preset` 的拜耳格式会设置为相同深度的单色格式：`BayerRG8` 和 `--preset high_speed` 将生成 `Mono8`；`BayerRG12`、`high_quality` 和 `triggered` 将生成 `Mono12`； 紧凑型名称保持紧凑（`BayerRG12p` 生成 `Mono12p`）。文件和 `lattice format` 采用 Mono 格式。混合了 M3C 和 M3M 相机的数组采用相同的格式 ：每个 M3C 采用拜耳模式，每个 M3M 采用其单色深度。名称即其位深度：反之亦然， 因此 M3C 相机绝不会被置于 Mono 格式中（无论数组中哪台相机为主相机，`--pixel-format Mono12` 在 M3C 上都对应 `BayerRG12`）， 而 `lattice format --set` 也会按相同方式映射，并输出其设定的格式。
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** 仍然有效——辐射度和反射度是*按波段*的辐射度图，对于单个波段而言定义完全明确。单色帧携带一个**单位**传感器响应矩阵（无 3×3 解混），因此该平面在经过校准运算时保持不变。
- **单个单波段相机无法生成植被指数。**NDVI/NDRE 等指数至少需要两个波段（例如 Red + NIR）。若要通过单波段硬件获取指数，需将**多台** 不同波长的 M3M 相机对准目标，将其对齐组合成一个多波段堆栈，然后对 *该* 堆栈进行指数计算：

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Mono_660 --channel nir=Mono_850 \
  --save-multiband -o output/
```

`index --live` 会自行启动相机（一个 [临时会话](#multi-camera-transient-sessions)），因此请先不要连接任何设备：`--serials`，否则配置文件中启用的相机 （参见 [索引 / 植被数学](#index--vegetation-maths)）。当 Chloros 持有任何摄像头（GUI、`cam-connect`、`array-connect`）时 ，系统将返回HTTP 409错误（`CAMERA_HELD`）并退出（`1`）；请使用`chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`释放这些摄像头。

`--channel` 符号必须与预设的通道名称**完全**匹配（区分大小写；NDVI 为小写 `red`、`nir` ——参见 `--list-presets`），且波段侧名称必须与对齐堆栈中的某个波段名称一致（离线模式也接受以 0 为起点的波段索引，例如 `--channel red=0 --channel nir=1`）。

整个堆栈中的区分符是模型字符串中的 `M3M` 标记（该标记绝不会出现在 `M3C` 字符串中），并在 GUI/SDK 中以 `is_mono` 的形式呈现。

---

## 主机网卡设置与调优（LATTICE 阵列）

LATTICE 摄像机通过主机的以太网适配器传输 GVSP 流，因此对于多摄像机阵列，适配器的 **驱动程序**和**接收环大小**与链路速率同样重要。错误的设置会导致“`FRAMES WILL DROP`”或 `Reduce ROI to enable` 提示（在“阵列设置”面板中，以及 `lattice network-analysis` / SDK 的 `analyze_array_network()` 中），即使摄像头本身运行正常。

### USB 10GbE 适配器 — Realtek RTL8157（“Realtek USB 10GbE 系列控制器”）

以下设置为 Windows 驱动程序属性。在 Linux 系统中，USB 适配器同样存在带宽限制（x86_64 架构下为 400 MB/s， NVIDIA Jetson 上为 200 MB/s）(参见 [数组 fps 与突发模型](#array-fps--burst-model))，且内核的套接字接收缓冲区对每个适配器都至关重要——参见 [Linux 主机](#linux-hosts--socket-receive-buffers)。

| 项目 | 所需值 | 为何重要 |
| --- | --- | --- |
| **驱动程序版本**|**≥ v10.67（2026年1月）**，INF `rtump64x64sta.inf` | 旧版**2016**驱动程序（v10.65， `rtump64x64.inf`）在关机/重启/休眠时处理电源关闭和错误检查存在缺陷，会导致**`DRIVER_POWER_STATE_FAILURE`（蓝屏 `0x9F`）**。 系统转换过程中会卡死（约 5 分钟超时），用户被迫强制关机，而反复发生的不正常关机将**导致 WMI 存储库损坏**（PowerShell/工具开始因 `Invalid class` 报错而失效），并**导致 USB 堆栈卡死** （网卡无法启用；USB 设备停止枚举）。在依赖干净重启之前，请先从 realtek.com（或 USB 适配器厂商）获取更新。 |
| **接收缓冲区**— 关键词 `ReceiveBufferLen` |**256**（驱动程序最大值） | 网卡接收环。驱动程序默认值**32**仅留下约 0.26 MB 的可用环空间——对于多摄像头突发传输而言远不足够——因此阵列面板会报告 `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` 并阻止连接。当设置为**256**时，环缓冲区容量较大（**在实验室 10GbE 主机上测得约 13.5 MB**），为接收管道处理多机位 GVSP 突发数据提供了充足的余量。（特定配置能否实际 *建立连接* 取决于两项检查——**考虑数据流耗尽的**通量检查和**聚合超订阅**检查——而非单纯的突发数据与环带宽的比较；参见[阵列帧率与突发模型](#array-fps--burst-model)。) |
| **接收 URB**— 关键字 `PendingReceives` |**64**（最大） | 正在传输中的 USB 请求块；需与接收缓冲区配合使用以吸收突发流量。 |
| **巨型帧** — 关键字 `*JumboPacket` | **9014** | 用于 9000 字节的 GVSP 数据包（每帧数据包数量比 1500 字节帧少 6 倍）。 |

> ⚠️ **网卡驱动程序更新会将这些高级属性重置为默认值。**更新或更换适配器驱动程序后，请**重新应用** `ReceiveBufferLen=256` 和 `PendingReceives=64`，否则即使“硬件未发生任何变化”，阵列面板仍会再次进入门控状态。这是原本正常运行的设备突然无法连接的首要原因。

请在**提升权限的** PowerShell 中应用（替换为您的适配器名称，例如 `"Ethernet 5"`）：

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` 适用于 USB 10GbE 适配器。**现在它会检测适配器类型并调整正确的接收环关键字：对于 PCIe 网卡（Intel I219、 等），或 `ReceiveBufferLen`→256 + `PendingReceives`→64 用于 Realtek**USB** 10GbE 控制器（该控制器不暴露 `*ReceiveBuffers`）。目标值会被限制在各驱动程序报告的最大值（`NumericParameterMaxValue`）内，因此绝不会写入超出范围的值。 请在**提升权限**的终端中运行该命令；与任何基于注册表的调整一样，该更改需在网卡重启或系统重启后生效。上述手动 `Set-NetAdapterAdvancedProperty` 命令仍是一个不错的替代方案——它们无需重启即可实时生效（重新绑定网卡）。

### Linux
主机 — 套接字接收缓冲区

在 Linux
系统（包括 NVIDIA Jetson）上，内核的套接字接收缓冲区与网卡的接收环同样重要。 在内核默认设置下（`net.core.rmem_max` / `net.core.rmem_default` = 212,992 字节），摄像头发出的 GVSP 数据包突发会使流套接字溢出，而此时 Chloros
尚未读取这些数据包。 这些数据包在内核中被丢弃，而网卡本身却报告没有丢包，导致相机阵列丢失帧组并降低帧率。 在连接 USB 10 GbE 适配器的 4 台摄像头阵列上进行测试，将两项参数均提高至 32 MB 后，阵列在连接后的帧率从 4.96 fps 提升至 7.52 fps，连续原始捕获速率从 4.7 帧/秒提升至 7.6 帧/秒，且内核中未丢包。

- **.deb 包会设置这些参数。** 安装该软件包会写入 `/etc/sysctl.d/60-chloros-gige.conf`（两者均位于 `33554432`）并立即应用。 若该值已被设置为更高，则保持原值不变。
- **`lattice network` 负责检查这些设置。** 当任一值较低时，系统会在 *发现的问题* 下列出 `socket receive buffers too small for GigE Vision (…)`，且后端日志会在第一台摄像头或阵列连接时记录相同的警告。
- **若要手动提高这些值**（效果持续至下次重启；该软件包的文件会在每次启动时应用这些设置）：

  ```bash
  sudo sysctl -w net.core.rmem_max=33554432 net.core.rmem_default=33554432
  ```

  `lattice network --fix` 也能提高并保持这些值，但仅当后端以 root 身份运行时有效。以普通用户身份运行的后端（即以 `chloros-backend` 或 [`chloros-backend@<user>`X](#running-the-backend-as-a-service-headless)) 无法更改内核设置，而是会输出 `sudo sysctl` 这一行。

**自动网卡调优仅针对已连接的网卡。** 当第一台摄像头连接时，以 root 身份运行的后端会调优那些接收环较小的有线网卡。 现在，它仅对已建立连接的适配器进行调整，因此未连接的板载端口将保留其 MTU，且不会更改其接收缓冲区。`lattice network` 不再将未连接适配器的的 MTU 列为问题。`lattice network`、`--fix`、`lattice probe` 以及适配器调整功能仅考虑有线以太网端口， 不包括 Jetson 的 CAN 总线（`can0`）或 USB 设备模式链接 （`usb0`、`usb1`）——这些链接是Jetson提供给主机PC的；而用于摄像头的USB以太网适配器仍被计入其中。

### 网络基础知识（所有LATTICE链接）

- **地址分配：** 链路本地地址 `169.254.0.0/16`（GigE Vision LLA）。主机采用静态地址 `169.254.x.x/16`；相机和 DAQ-E 在同一地址范围内自动分配地址。 无需 DHCP/网关。
- **数据包大小：**优先使用巨型数据包（9000），但由自动探测功能确定——该功能通过 GVSP 探测绕过相机 1500 字节的 ICMP 限制，因此无论线缆实际支持何种大小，最终都会采用巨型数据包。**每台摄像机在每次后端会话中测量一次，当其链路发生变化时（如更换网卡、链路速率改变或链路断开，例如交换机切换时）会再次测量；重新连接时将复用先前结果。若交换机保持在线状态但其后端发生变化，请重启后端以重新测量。**首次连接时，此项检查偶尔可能需要几分钟，日志中会记录这一情况。仅在您比探测器更了解情况时才使用 `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` 进行固定，且应优先采用按命令设置而非永久设置：固定会跳过探测器， 因此，如果路径实际上无法承载 9000，则使用 `SC_ERR_TIMEOUT -1011` 时，**每次**捕获都会超时（参见 [环境变量](#environment-variables)）。
- **RX 环大小随 `ReceiveBufferLen` 调整：**在默认值 `32` 下，可用环带宽约为 0.26 MB（对于任何多摄像头突发传输而言都太小）；在最大值 `256` 下，带宽很大 （在实验室 10GbE 主机上测得约 13.5 MB），提供了实际的冗余空间。配置能否建立连接，取决于“考虑流量消耗的接入检查”**以及**下文所述的聚合超订阅检查——而非单纯的突发流量与环带宽的比较。

### 阵列帧率与突发模型

如何解读“阵列设置”面板（以及 `lattice analyze-array` / SDK
中的 `analyze_array_network`）：

- **突发速率按每台摄像机的实际像素格式进行累加。**单色**M3M**摄像机以**Mono12（2 B/px）**格式流式传输，或通过 `--pixel-format` 采用其深度信息的单色格式 （`BayerRG8` 即 `Mono8`， 1 B/px）；**M3C**拜耳相机输出 8 位或 12 位数据（即使请求的是 BayerRG8，M3C 固件也会默认输出 BayerRG12）。因此，4 台摄像头的全分辨率帧大小为**若均为 8 位则约为 12.6 MB，但若其中三台为 12 位单色摄像头则约为 25 MB**。 该投影功能会根据相机型号（身份缓存）解析每台相机的格式，因此数据流与实际传输内容相符——而非默认采用“一刀切”的 BayerRG8 格式。
- **USB 以太网适配器的传输速率上限为 200 MB/s，无论其标称参数如何。** 将链路速率转换为持续传输速率的效率表源自 PCIe；USB 网卡虽宣传其 *以太网* 链路速率，但实际受限于 USB 总线及其驱动程序。一款 USB 10GbE 适配器曾测得约 1063 MB/s “持续”速率——该数值从未经过实际测试——而由此产生的速率调节导致 6–18% 的帧数据受损，同时仍报告正常的帧率目标值。目前，USB 连接的网卡已设定绝对上限：在 Windows
和 NVIDIA Jetson 平台上为 **200 MB/s**， 在 Linux
x86_64 系统上为 **400 MB/s**（在 Linux
系统中，连接至 USB 2.0 端口的适配器配额为 **30 MB/s**，且能力记录提示应将其插入 USB 3.0 端口） （该限制由主机的 USB 总线和驱动程序决定，因此不会随标称速率线性提升；USB 1 GbE 适配器可达到约 80 MB/s，且不受此限制）。在Linux
系统中，该适配器可通过sysfs识别（若无法识别则回退至`ethtool -i`的`bus-info`）。各项数据均经实测：在Windows
系统上，同一套4摄像头阵列在传输速率超过200 MB/s时，6–10 %的帧会出现数据损坏；在Linux
x86_64系统上，同一网卡在高达480 MB/s的任何速率下均运行稳定（实际传输速率为273 MB/s，即摄像头自身的上限）。在 NVIDIA Jetson 上，同一适配器仅在 200 MB/s 时无帧丢失，超过该速率后会间歇性丢失帧。能力记录中的 `wire_ceiling_source` 以文字形式说明了这一点，而 `nic_is_usb` 则通过标志进行了标注。可通过 `--wire-ceiling-mbps` 即可。
- **通量控制机制基于引脚负载，而非“整批数据”与“环形缓冲区”的对比。** 同时发生的突发传输只需满足 *瞬态积压* = `max(0, Σ per-cam arrival − host drain) × emit_window` 的要求，而非整个突发传输。在高速主机/低速摄像头架构（例如 **PCIe**10G 主机 + 4× 1 GbE CAM：到达速率 ≈ 320 MB/s，排出速率 ≈ 1063 MB/s）的架构中，主机的排空速度快于摄像头的填充速度，积压量 ≈ 0，因此全分辨率模拟发射会**被允许**，尽管 25 MB 的突发数据量超过了 13.5 MB 的环带容量。 若将这四台摄像头连接到Windows
系统上的**USB**10GbE适配器后，数据排出速率为200 MB/s，而非1063 —— 数据到达速率超过了该速率，且数据丢失表现为帧损坏，而非帧率降低。 在 1 GbE 主机上，摄像头 31.25 MB/s 的 DLThr 下限导致数据到达速度超过数据吞吐速度 → 系统会正确地**阻塞**（对于 *此类* 阻塞情况，应缩小 ROI 或使用 ≥ 2 的像素合并）。 允许通过是**两个**连接门之一——另一个是下文提到的聚合超订阅检查。
- **预计帧率取决于最慢摄像头的周期。**在同时发射模式下，每台摄像头都会在同一时间发送其帧，因此数组的周期取决于其最慢成员的传感器读出时间与发射时间中的较长者（帧字节数 ÷ 其发射速率：即摄像头的**访问链路**，1 GbE ≈ 80 MB/s，或当该值更小时，其占主机带宽上限的份额）。每次曝光都会与前一帧的读出时间重叠，因此曝光过程不计入周期： 自动曝光会填满由传输速率决定的周期，随后提高增益，且昏暗场景不再降低传输速率（在 1.2.0 版本中会降低）。只有手动曝光时间超过该周期时，才会减慢触发速度（参见 [数组成员上的曝光](#exposure-on-an-array-member))。增加相机数量不会降低数组的运行速度，除非其总线共享带宽缩小。在串行发射模式（`slip-emit-and-capture`）中，各相机的发射时间会累加，且每个成员的曝光时间都会计入时序错开中。 在 9 台摄像机阵列上，使用 PCIe 10 GbE 主机，以全传感器、12 位模式测得：10.7 fps，若采用 [打包传输](#packed-12-bit-transport---packed)，则为 14.0 fps。
- **连续捕获结束后，投影数据会显示其最终稳定帧率。** 30 秒或更长时间的自动帧率捕获会记录该阵列规模下本计算机最终稳定的帧率；帧率控制器对更高帧率进行的短暂测试则不被计入。 此后，当实际速率低于模型预测值时，`array-connect` 的行以及“数组设置”面板会显示该速率，并注明限制因素：`clock-bound`（数组以最终稳定的速率运行， 抓取循环正在等待其触发时钟），`loop-bound`（抓取循环已无剩余时间），`capture-bound`（捕获操作滞后并跳过了若干组）或 `measured` （未明确指出具体原因）。该模型的数据见 `wire_fps_max`。数组仍会攀升至线路所能承载的上限。
- **数组可在任何网络上运行——帧率才是决定因素。**每台相机都能平分**防冲突带宽上限**（`sustained × sim_emit_factor`），其速率由 GevSCPD 控制，且所有相机仍能响应同一触发信号。 在低速网络上，分配份额较小，帧率也会随之下降：8台全分辨率12位相机在未启用巨型帧的1 GbE网络上，每台可获得52 / 8 = 6.5 MB/s，约为**1 fps**（压缩后约1.3帧/秒），而10G主机运行时帧率为11-14帧/秒。如果无法以如此低的速率同步相机——可通过读取每台相机的GevSCPD范围来确认——则数组将切换为**串行**输出 (`slip-emit-and-capture`)：相机将以全链路速率轮流使用总线，依次触发，因此每个帧仍会到达，但成员数据**不会同时暴露**。 `analyze-array` 以及 connect 命令的输出结果会标明该模式；将 `--force-tier sim-capture-sim-emit` 引脚置为低电平可拒绝此降级机制。帧大小不影响此共享机制，因此**像素合并和缩小 ROI 虽能提高帧率，但不会改变数组是否适配**。 只有当一条线缆甚至无法以串行方式传输数组时，才会出现**超额订阅**——例如，摄像头数量远超一条线缆的处理能力，或者该线缆上同时传输的摄像头过多——此时的症状是 GVSP 数据包丢失，而非速率降低： `analyze-array` 将可实现的帧率值清零并输出 `**OVER-SUBSCRIBED**`，连接仍会继续进行但伴随强烈警告；而带有 `pin_resolution: true`（严格模式）的 HTTP
连接则会 **被拒绝**。解决方法包括减少摄像头数量、端到端启用巨型帧，或使用更快的网卡。`CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1`会将拒绝降级为“强烈警告”，以便进行基准测试——参见[环境变量](#environment-variables)。

### 阵列健康状况 — 哪个子系统正在丢失帧

已连接阵列的 `GET /api/camera/array/<array_id>/capability` 携带一个实时
`health` 块，该块在滚动 **10 秒** 窗口内重新评估。 它将帧丢失
细分为两种需要采取相反修复措施的原因，而不是报告一个未指明具体原因的“不完整”
率：

| 字段 | 含义 | 涉及子系统 |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct`（按串口） | 帧**已到达但结构错误**——GVSP数据包丢失。 |**网络**：带宽预算、速率控制、网卡接收环、MTU |
| `never_arrived_rate_pct`（按串口） | 帧**从未到达**——相机未触发，或未发送任何数据。 |**触发/同步**：M8 电缆、`--line`、`TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | 每种情况下的最差摄像机速率。 | — |
| `per_cam_rate_pct` | 每台摄像机的综合不完整速率（两种原因合计）。 | — |
| `stable_for_seconds` | 每台摄像头保持在 0.01% 以下的时间长度。 | — |

当比例超过 5% 时，后端会记录一条 `[array-health <id>] WARN` 日志行，其中注明了分界点——在
首次触发时、严重性等级区间发生变化时、持续期间每分钟记录一次，以及
恢复正常时记录一次。 损坏的半部分会在
每个摄像头和原因的首次触发时将 `[gvsp-corrupt <SN>]` 写入后端日志文件，随后每 60 秒汇总一次；而应用程序的日志面板则显示
`[array-health <id>]` 这一行。每次评估结果仍会写入后端日志文件；
无论打印出什么内容，计数器都会随着每次缓冲区操作而递增。

同一条记录报告了整个分配所依赖的数字：

| 字段 | 含义 |
| --- | --- |
| `wire_ceiling_mbps` | 主机当前生效的持续带宽配额，MB/s。 |
| `wire_ceiling_source` | 该数值来源的文字说明——例如 `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` （在 Linux
x86_64 系统上，为 `USB-capped 400 MB/s (…; sysfs: bus=usb (usb2/2-1/2-1:1.0))`）或 `user override 120 MB/s (auto said 200)`。 |
| `wire_ceiling_is_user_set` | 当 `--wire-ceiling-mbps`（或 GUI 中的 **带宽预算** 字段）将其设置为 `true` 时。 |
| `nic_is_usb` | 对于 USB 以太网适配器，该值为 `true` —— 参见上文的 USB 限制。 |

**解读：**当 `gvsp_corrupt_rate_pct` 不为零且 `never_arrived_rate_pct` 为 0 时
表示触发和电缆同步完美，100% 的丢包发生在网络
路径上——降低 `--wire-ceiling-mbps` 并重新连接。反向模式则指向
同步电缆或触发线。

> **`--target-fps` 并非解决帧损坏问题的关键。** GevSCPD 节律在
> 连接时仅设置一次，因此降低触发率会改变占空比，而不会改变
> 同时发射的突发速率。经测量，将需求削减 5 倍并未带来改善；
> 将线速上限从 240 MB/s 降至 200 MB/s 后，同一套设备的
> 数据包损坏率从 10.4%
> 降至 0.00%。

> **在 LATTICE（M3C / M3M）固件环境下，运行中的阵列无法自行重新规划。** 它
> 无法自行修复此问题；请断开并重新连接，以便连接时间选择器能够
> 根据新的上限重新规划。

### 症状 → 解决方法

| 症状（阵列设置 / 连接 / `analyze_array_network`） | 原因 | 解决方法 |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` 重置为 32（通常发生在驱动程序更新后） | 将 `ReceiveBufferLen`→256，`PendingReceives`→64；重新打开控制面板（如果后端缓存了旧的环大小，则重启后端） |
| 重启/关机卡死；随后出现 `Invalid class` WMI 错误， 网卡无法启用，USB 设备缺失 | 旧版 2016 Realtek USB 10GbE 驱动程序 → 蓝屏 `0x9F` → 强制关机 | 将适配器驱动程序更新至 ≥ v10.67 (2026)，然后重新应用上述接收环设置 |
| 阵列以满传感器速率连接但帧率较低 | “自动”模式可保持满传感器速率，并让帧率降至线速 | 选择“连接输出”列表中较快的选项之一（`--binning 2` / `4`，或较小的 `--width`/`--height`），启用巨型帧，或升级链路 |
| Array 报告的目标帧率正常，但实际传输仅为其一部分；`health.gvsp_corrupt_rate_pct` 不为零， `never_arrived_rate_pct` 0 | 主机推断的线速预算高于其实际可持续传输速率（常见于 USB 以太网适配器、带宽较窄的 PCIe 通道或共享互连结构） | 请使用较低的 `--wire-ceiling-mbps` 值重新连接，并再次检查健康状态块。 **非** `--target-fps` — GevSCPD 速率控制在连接时已固定 |
| Linux
：数组丢失帧组并降低帧率，而网卡报告无丢包； `lattice network` 列出 `socket receive buffers too small` | 内核的套接字接收缓冲区处于默认大小，因此相机数据包在内核中被丢弃 | 将 `net.core.rmem_max` / `rmem_default` 至 32 MB — 参见 [Linux
主机](#linux-hosts--socket-receive-buffers)。.deb 包会在安装时自动完成此操作 |
| 已发布组中缺少摄像头；`health.never_arrived_rate_pct` 不为零， `gvsp_corrupt_rate_pct` 为 0 | 触发/同步路径问题——摄像头未启动，非网络问题 | 检查 M8 同步线缆和 `--line`；确认每个成员均已启用 (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` 在 `analyze-array` 中超限，或在严格模式下连接被拒绝 (`pin_resolution`; `array over-subscribes the wire`) | 无论摄像头达到何种传输速率，该线路均无法同时或依次承载该数组——摄像头数量远超线路可共享的上限，或该线路上有其他摄像头正在传输 | 减少摄像头数量、端到端使用巨型帧，或使用更快的网卡。 **ROI/像素合并无济于事**（共享与帧大小无关）。在测试台上，`CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1`会覆盖该错误（接受数据包丢失） |
| 阵列通过 `slip-emit-and-capture`（串行）连接到低速网络 | 相机无法以足够慢的速率同步共享该线路，因此只能轮流工作；帧数据无法在同一瞬间传输 | 使用巨型帧或更快的网卡可恢复同时捕获； `--force-tier sim-capture-sim-emit` 坚持此模式（结果反而导致数据包丢失） |

---

## `chloros-cli daq`

光谱传感器命令。 `pool-*` 命令是轻量级的 HTTP
客户端，它们通过后端的持久化连接池驱动传感器，因此需要一个正在运行的后端（参见 [启动后端](#start-the-backend))。后端拥有传输通道，因此CLI
和SDK
脚本共享一个活动句柄，而非争夺串行端口。`pool-discover`是获取DAQ-M的BLE MAC地址的唯一方法。

> **`chloros-cli daq --help`**（或 `-h`、`daq help`）会列出 `pool-*` 的子命令（在 Linux
上，还会列出构建包中包含的直接硬件命令）， 而 `chloros-cli daq <subcommand> --help`（或 `daq help <subcommand>`）则列出了每个子命令的选项。这些选项涵盖连接、 流、记录校准后的 `.daq` 文件、交换电容曲线以及将 DAQ 绑定到相机；对于 DAQ-E，还包括其网络设置、PTP、重启和固件更新。 还可以通过 `chloros_sdk.connect_daq_sensor()` 命令使用 Python
控制 DAQ，该命令使用相同的池化路径。
>
> **请在所有平台上使用 `pool-*` 命令。** 这些命令在 Windows
、Linux
和 Jetson 上的行为一致。在 Windows
上， `daq discover` 和 `daq list` 是 `pool-discover` 的别名，而任何其他非  的 `daq` 命令2567的任何其他`daq`命令，都会以`1`和`DAQ direct-hardware commands require the full daq package`退出。在Linux
和Jetson上， `daq discover`、`daq list` 以及其他非 `pool-*` 的 `daq` 命令是 DAQ 工具的直接非硬件命令：它们在CLI
的独立进程中打开传感器， 位于后端进程池之外（因此并非由后端、GUI 或脚本持有的传感器），且该处接受 `daq discover`、`--only` 和 `--timeout`，但不接受 `--json`。本文档仅记录 `pool-*` 命令。

### DAQ 传感器首次连接工作流

```bash
# 1. Scan every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U serial port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq pool-discover
chloros-cli daq pool-discover --only ble           # BLE only
chloros-cli daq pool-discover --json               # machine-readable

# 2. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect                                   # smart-detect
chloros-cli daq pool-connect --port /dev/ttyUSB0               # DAQ-U on a serial port (Windows: COM3)
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 3. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 4. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

#    Bind it to a camera, so that camera's captures use and save its readings
chloros-cli daq pool-bind --camera 213800234 CB-7C-A8-2E-5F

# 5. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 6. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### `pool-*` 参考文档

| 子命令 | 用途 |
| --- | --- |
| `daq pool-connect` (smart-detect) | 在后端池中打开一个传感器。 |
| `daq pool-connect --port PORT` (`-p`) | 在特定串行端口上启用 DAQ-U。 |
| `daq pool-connect --ble` | 通过 BLE 进行 DAQ-M，自动扫描 MAC 地址。 |
| `daq pool-connect --mac MAC` (`-m`) | 通过蓝牙低功耗（BLE）连接至已知 MAC 地址的 DAQ-M（隐含 `--ble`）。 |
| `daq pool-connect --eth-host HOST` | 通过以太网连接的 DAQ-E，使用已知主机名或 IP 地址（隐含 `--eth`）。 |
| `daq pool-connect --eth` | 通过以太网进行 DAQ-E 连接，主机自动发现（mDNS + ARP 备用方案；在 Windows
和 Linux
上即使 ARP 缓存为空也能正常工作）。 |
| `daq pool-connect --integration-time MS (-t) --frame-avg N (-f) --no-ae` | 调整积分窗口/AE状态。 |
| `daq pool-connect --no-stream` | 建立连接但暂不开始流式传输 （随后接 `pool-stream --start`）。 |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | 峰值校正配置文件。后端默认值为 `sunshine_cosine`； `none` 是一个裸传感器。[DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor) 会忽略它（固定扩散器）。**传感器出厂时为裸传感器，因此请为安装的盖子命名：**默认记录为假定的 (`cap_id_source=auto_default`)，处理过程会对此发出警告，因为未经阳光校正的裸机 DAQ-U 读数会高出约 30 倍。|
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | 扫描每个传输通道中可连接的传感器，但不进行连接。 **这是查找 DAQ-M 的 BLE MAC 地址的方法。** `--timeout` 按传输通道设置（默认 5 秒，范围 1-20）。传感器池中已打开的传感器不会被列出——已连接的 DAQ-M 会停止广播——因此请使用 `pool-list` 查询这些传感器。 `usb` 仅列出 USB 串行设备：计算机自身的串行端口（`COM1`，Linux
的 `/dev/ttyS*`）不能作为传感器，因此被排除在外。 |
| `daq pool-list` | 显示后端池中的所有传感器。 |
| `daq pool-bind (--camera SERIAL \| --array ARRAY_ID) {SENSOR \| --clear}` | 将相机或数组绑定到 DAQ，类似于 GUI 中光传感器选择的功能： 其捕获操作将使用、匹配并保存该数据采集卡的读数。`SENSOR` 是来自 `pool-list` 的传感器 ID，或是访问该数据采集卡的地址。`--clear` （不包括 `SENSOR`）将解除绑定， 捕获操作将回退到最新的数据采集设备。绑定状态将持续至后端重启，或直至该摄像头、数组或数据采集设备断开连接：断开连接（此处指从SDK
或应用程序中断开）会解除其所断开硬件的所有绑定，无论由谁设置，数组成员的绑定将随数组一同解除。 相机必须已连接、位于当前打开的项目中，或此前已在该计算机上打开过；数组也必须已连接或位于当前打开的项目中；任何其他序列号或 ID 都会因名称不符而被拒绝，且不会建立任何绑定。 |
| `daq pool-disconnect {--sensor-id ID \| --all}` | 释放 — 无论由谁打开该传感器，因此它也会释放因脚本崩溃而留在池中的传感器（先完成该传感器上的任何录制操作）。不在池中的传感器无内容可释放： 它将输出 `ID: already released (not in the pool)` 并退出 `0`；若后端拒绝释放或未确认释放，则退出 `1`。 SDK
：`DAQSensorSession.force_disconnect()`。 |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | 最近的 N 个频谱帧（默认 1）。 |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | 恢复/暂停流式传输。 |
| `daq pool-record --sensor-id ID [--duration SEC] [-o DIR] [--device-name NAME] [--confirm-after S] [--heartbeat S]` / `--stop` | 开始/停止 .daq 记录。 `-o` 在后端计算机上的默认值为 `~/Documents/DAQ Live View`（相对路径位于您运行该命令的文件夹之下）；`--duration`（`-d`，默认值为`0`，直至`--stop`）。 `--confirm-after S` 会在开始后 S 秒内未收到任何读数时发出警告（默认值为 20；设为 0 则关闭该功能）；使用 `--duration` 时， `--heartbeat S` 每 S 秒输出读数计数，并在计数停止增长时发出警告（默认 30；设为 0 则关闭该功能）。当 DAQ-E 上的记录一旦静默，在后端重新连接后会继续写入同一文件，而 `--stop` 会列出数据缺失段（[停止传输数据的 DAQ-E](#a-daq-e-that-stops-delivering))。Ctrl+C 可提前终止 `--duration` 记录并报告该情况； 若未启用 `--duration`，Ctrl+C 后录制仍会继续运行（退出 `130`），直至 `--stop`。 |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | 在运行时切换电容校正配置文件。必须设置 `--cap-id`；`--cap-id none` 会移除电容校正 （即裸传感器）。DAQ-E-S 设备会将 `as_recorded` 保持为当前设置值。 |
| `daq pool-enable-ptp --sensor-id ID` | 为 PTP 处于关闭状态的 DAQ-E 设备重新启用 PTP。该设置将保存在设备上， 因此，已关闭的设备将保持关闭状态——其时间戳不会与PTP同步——直到执行此命令。 `pool-list` 会标记此类设备（`!! PTP is switched OFF`）并为其打印此命令。Chloros
绝不会自行将设备重新启用，因为关闭可能是故意的。 |
| `daq pool-info --sensor-id ID [--json]` | DAQ-E 自身的设置：其固件（以及该Chloros
版本是否包含更新版本）、其地址及获取地址的方式 （DHCP / 链路本地 / 静态），传感器运行该命令前保存的网络设置（包含通过 `--no-reboot` 保存的设置）、开机时的数据流以及 PTP。对于数据流无法到达本计算机的传感器，该命令会在建立新连接时读取传感器状态并报告此情况。 |
| `daq pool-set-network --sensor-id ID {--auto \| --static IP [--mask M] [--gateway G]} [--no-reboot] [--force] [--timeout S]` | 将 DAQ-E 设置为 DHCP (`--auto`) 或静态地址 (`--mask`，默认值为 `255.255.255.0`)。传感器将重启以应用该设置，并使用其原有设置重新连接，等待时间最长为 `--timeout` 秒（默认 45 秒）； `--no-reboot` 仅保存该设置。参见 [DAQ-E 设备设置](#daq-e-device-settings)。 |
| `daq pool-set-auto-stream --sensor-id ID {--on \| --off}` | 指定 DAQ-E 在开机时若未连接任何软件，是否开始数据流传输（该设置保存在传感器中）。无论如何，Chloros
都会自行启动数据流。 |
| `daq pool-reboot --sensor-id ID [--timeout S]` | 重启 DAQ-E 并使用其先前设置重新连接 （等待时间最长为 `--timeout`，默认 45 秒）。 |
| `daq pool-update-firmware --sensor-id ID [--timeout S]` | 将此 Chloros
版本随附的 DAQ-E 固件刷写到运行旧版本的传感器上，然后重新连接 （约一分钟；请勿关闭电源；`--timeout` 默认等待 45 秒直至其恢复）。已处于最新版本的传感器将保持原状。 |

### 停止传输数据的 DAQ-E

若泳池中的 DAQ-E 在 20 秒内未传输光谱数据，后端将重新连接该设备。这通过重启为传感器供电的交换机来实现：传感器随交换机重启，并在收到指令前不会再次进行数据流传输 （除非设置了 `pool-set-auto-stream --on`）。

- **首先会验证是否为同一台传感器。** 系统会通过传感器原有的地址（不做任何更改）查询传感器的名称、MAC 地址和序列号。 它还会使用传感器的 `daq-e-<id>.local` 名称进行查询，但仅当响应来自传感器自身的网络适配器（MAC）时才信任该答案。其他任何响应都不会被重新连接或写入数据。
- **它将使用会话的原始设置重新建立连接：** 曝光时间、帧均值、自动曝光和快门曲线均保持原样，而非传感器开机时的默认值。如果传感器之前启用了 PTP 功能，但恢复后该功能已关闭，则会将其重新启用。随后，它将重启数据流，并重启计算机的接收程序。
- **它会以逐渐延长的间隔重试：** 首先立即重试，随后分别在 5、10、20 和 40 秒后重试，之后在传感器离线期间每 60 秒重试一次。 每次尝试仅向传感器查询其身份。若传感器虽有响应，但在三次重新连接后仍未发送任何数据，则不再继续尝试并报告该情况（`auto-reconnect GAVE UP`）。`pool-stream --start`将再次尝试。
- **`pool-list` 显示当前状态。** 当传感器处于静默状态时，其 `STREAM FAULT` 行会显示重连操作的进展。 该行在第一个光谱出现时消失，随后传感器显示为 `reconnected automatically N time(s)`。
- **记录在同一 `.daq` 中继续进行。** 该文件用两行标记每个间隙：`event_type` 间隙前最后一个光谱处为 5，间隙后第一个光谱处为 6，每行仅包含时间戳。`pool-record --duration` 在传感器缺席时输出 `!! STREAM LOST …`，传感器返回时输出 `stream resumed after …`。`pool-record --stop` 会列出每个间隙并发出警告，如同对待 空文件的情况（不包含任何读数的记录将退出为 `1`）。
- **每条记录仅保留一次校准值。** 若在记录运行期间传感器恢复连接时校准值发生变化，该记录的剩余部分仍将保留初始校准值，且停止警告中会明确说明这一点。 `pool-disconnect` + `pool-connect` 才会采用新的校准值。
- **您的命令优先。** 通过 `pool-disconnect` 断开连接或使用 `pool-stream --stop` 暂停的传感器，绝不会被重新连接，即使重新连接操作已正在进行中。
- **对静默传感器执行 `pool-info`** 会在新连接时读取传感器，并添加 `!! stream NOT reaching this computer`。 无法响应的传感器会立即被报告。对于已停止响应的传感器，`pool-list` 不再显示其 PTP 状态。
- 本文未涵盖 DAQ-U 和 DAQ-M。DAQ-M 会通过蓝牙自动重新连接，而从 USB 断开的 DAQ-U 需要再次执行 `pool-connect`。

### DAQ-E-S — 固定式漫射器与倾斜传感器

**DAQ-E-S** 是一款 DAQ-E，但有两处对软件而言至关重要的差异。其余所有功能——包括设备发现、连接、PTP、记录以及下文所述的设备设置——均与 DAQ-E 完全一致。**其漫射器为固定式，因此无需进行顶盖校正。** 该 Sunshine 风格的漫射器为内置式，且在工厂校准测量时已安装在设备上，因此该校准值已准确反映向下辐射通量。Chloros
无论被要求什么，都会将其顶盖设置为 `as_recorded`（无配置文件）： 若指定以该设备命名的上限（如 `sunshine_cosine`、`fov_*`、`none`），系统会忽略并显示提示信息，且其记录数据将存储为 `cap_id_source=model`，因此针对该传感器的“假定上限”警告永远不会触发。也不要手动为其数据设置上限：若在顶部放置名为 `sunshine_cosine` 的上限，读数将高出约 11 倍。

**该传感器配备了一个三轴加速度计。** 阳光漫射器的余弦响应仅在设备水平放置时有效，因此倾斜的传感器会报告错误的辐照度——且反射率会继承该误差——且无其他异常症状。因此，每个 DAQ-E-S 读数都包含其姿态信息：

| 字段 | 含义 |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | 角度（单位：度）。当固件将角度标记为未定义时（自由落体或死轴）时，`None` |
| `x_mg`、`y_mg`、`z_mg`、 `mag_mg` | 加速度单位为毫g。若 `mag_mg` 远离 ~1000，则表示设备处于运动状态，该样本中的角度数据无实际意义。 |
| `cal_applied` | 启用：坐标轴和角度以扩散器坐标系为基准（已应用设备安装校准）。禁用：原始电路板坐标系，可能显示高达约 2.3° 的倾斜角，但这并非真实倾斜。 |
| `sample_age_ms` | 发送频谱时加速度计采样数据的采集时间。 |
| `temp_raw` | 未校准的芯片温度**趋势**——仅显示温差，绝非温度计读数。 |

`daq pool-latest --json` 将其返回至每个帧的 `imu` 键下（在 DAQ-U、DAQ-M 或普通 DAQ-E 上则为 `null`）。从 DAQ-E-S 记录的 `.daq` 会将其相同值存储在 `imu_*` 列中（`imu_tilt_deg`、`imu_roll_deg`、`imu_pitch_deg`、`imu_mag_mg`、 `imu_cal_applied`、……）中存储相同的数值，因此读者可以区分云层与三脚架被撞倒的情况。在其他所有型号中，这些列均为空。

姿态数据流单独报告：当`daq pool-list`发生故障时，会输出`!! ATTITUDE STREAM FAULT (spectra unaffected)`，并记录其随后恢复了多少次故障。该处的故障绝不会导致光谱记录出现缺口。 `daq pool-discover` 将该设备列为 `DAQ-E-S`（若其在约 2 秒内未响应身份验证，则列为 `DAQ-E`）。

### DAQ-E 设备设置

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330

# A fixed address on the network this computer is on (192.168.1.x)
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1

# Back to the factory setting: DHCP, falling back to a 169.254.x.x
# link-local address when no DHCP server answers
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto

chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

- **若未配置 `--gateway`，则 `--static` 不会保存网关**（之前设置的任何网关都会被丢弃），这对于没有路由器的摄像头网络来说是正确的。
- **静态地址必须位于本计算机的网络中。** 若传感器网络中该计算机的子网不包含此IP地址，Chloros
将拒绝该设置：重启后将无法连接传感器以恢复原设置。仅当该计算机将被重新分配地址或路由至新子网时，才应启用`--force`。
- **传感器正在录制时，系统将拒绝重启。**请先停止录制（`pool-record --stop`）。在重启前启动的录制任务将因重启而终止， 且 `pool-set-network`、`pool-reboot` 和 `pool-update-firmware` 将以一行 `Warning:` 结束，该行会标明录制文件的名称 （即使传感器随后未恢复连接也是如此）。
- **重启后，Chloros
仅重新连接传感器本身。** 它会在传感器原先所在的位置——以及在 `--auto` 之后，还会在其 `daq-e-<id>.local` 名称下——等待包含传感器名称、 主机名和型号的响应，并根据其原有的网络适配器（MAC）地址，在响应的地址上重新连接该传感器。 只有来自该 MAC 地址的响应才会被接受，因此，对于通过路由器连接传感器的计算机，或处于默认 NAT 网络环境下的 WSL2，如果传感器因 DHCP 分配而被移至新地址，则无法重新连接： 该命令会提示此情况，并打印`pool-connect --eth-host <its address> …`命令，该命令将传感器恢复至其原有的快门速度和曝光设置。
- **`pool-update-firmware`会拒绝无法读取其固件版本的传感器。** 闪光后，如果传感器恢复连接的时间超过了等待时限，并不表示其已故障：一旦其响应，系统将使用新固件的设置与其重新建立连接，且该命令会提示连接速度较慢。
- **固件更新需要一个入站 TCP 连接。** 传感器会重新连接到这台计算机以收集图像。Chloros
会在路由至传感器的网卡上监听（若无法确定具体网卡，则在所有网卡上监听），并仅服务于传感器自身的地址； 任何其他连接都会在未处理的情况下被关闭。在 Windows
系统中，安装程序的 `Chloros Backend (TCP-In)` 防火墙规则允许其通过（参见 [Windows
(.exe)](#windows-exe))；若防火墙阻挡该连接，更新将因 `… device never connected back on TCP <port> …` 错误而失败。
- **在 WSL2 环境下，请使用镜像网络**（`networkingMode=mirrored` 中的 `%UserProfile%\.wslconfig`）。 在默认的NAT模式下，该计算机在传感器网络中的地址属于WSL自身，因此所有静态地址都会被拒绝（除非出现`--force`），实时流无法到达WSL，固件更新也无法完成，因为传感器需要回连至计算机以接收镜像。
- **传感器 ID 和主机名不会改变。**无论其地址为何，DAQ-E 仍然是 `daq-e-<id>` 和 `daq-e-<id>.local`，因此 `--eth-host daq-e-<id>.local` 仍能找到它。
- **图形界面（GUI）的设置保持不变**：在“光传感器”选项卡中，进入传感器设置（齿轮图标），选择“网络”。图形界面会保持与传感器的独立连接，因此若通过图形界面修改其已建立连接的传感器： 通过CLI
重启将终止GUI与该传感器的会话。
- 对应的SDK
设备为`DAQSensorSession.device_info()`、`set_network()`、`set_auto_stream()`、 `reboot()` 和 `update_firmware()`。

---

## `chloros-cli project`

打开、连接并运行一个已保存的 Chloros
项目 （包含 `cameras.json`、`sensors.json` 和 `project.json` 的文件夹）。所有操作均通过后端处理，因此图形界面（GUI）和命令行界面（CLI
）生成的硬件状态完全一致。

### 子命令参考

| 子命令 | 用途 |
| --- | --- |
| `project open PATH [--relink FOLDER]` | 打印项目的设备清单（摄像头、阵列、传感器），从其文件中读取数据，并检查其源文件（参见 [已移动的项目](#a-moved-project---relink)）。 该命令不会在后端打开该项目：后端已打开的项目及其设备和录像将保持原状。若无法找到该项目的任何源文件，则退出 `1`。 |
| `project devices PATH [--reconnect]` | 列出或重新运行设备发现。 |
| `project connect PATH [--cameras-only] [--sensors-only] [--packed {auto,on,off}]` | 连接所有已保存的摄像头/阵列/传感器。每个阵列的连接方式与 `lattice array-connect` 相同： 最长 600 秒，其步骤将打印到 stderr（`[array <id>]    <s> s: <step>`；标准输出为 JSON
报告）， 若阵列在此时间内未能连接成功，则操作将被撤销——其摄像头将恢复为初始发现状态——并在 `errors` 下报告，绝不会保持连接状态。`--packed` 为每个阵列设置压缩的 12 位传输模式 （默认值为 `auto`；项目不保存每个数组的设置；参见 [12 位压缩传输](#packed-12-bit-transport---packed))。每个光传感器都会在后端的DAQ池中，以其保存的地址、积分时间、帧均值、自动曝光和盖帽参数被打开： 未选择上限的传感器将采用 `sunshine_cosine`；保存为“裸”状态的传感器（`none`）保持“裸”状态；而 DAQ-E-S 传感器永远不设置上限。 GUI 的 DAQ 面板不属于该池：请先在该面板中关闭传感器。 |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | 来自命名相机或阵列的单次捕获（`--prefix` 为默认值，`capture`）。`--orient` （阵列）：阵列保存原始数据，该数据保持原生格式——这表示由 `process` 生成的产品应按 `array-capture --orient` 所示进行定向处理。默认：原生。 `project burst` / `stream` 以及配方中的 `capture` / `burst` / `stream`（`orient: true`) 均采用相同的选项。 |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | 来自指定相机或阵列的 N 帧连拍（`-n/--count` 默认值为 5；`-i/--interval` 帧间隔（秒），默认值为 0）。 数组连拍会去除重复的同步组（过期监视器），因此部分循环的数组无法返回同一帧的 N 个副本； 并打印每次迭代的结果。若捕获过程中发生失败，将停止该突发捕获并退出 `1`，同时报告此前已保存的帧数。 |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--orient]` | 循环将数据捕获到磁盘：`--fps F` 捕获第二 （默认3），每次从摄像头捕获一帧，从数组捕获一组同步帧（采用与`project burst`相同的过期看门狗机制），直到`-n N`捕获 （默认 `0` = 直到按下 Ctrl+C，此时会打印已保存的帧数并退出 `0`；若未保存任何帧或文件写入失败，则退出 `1`； 若在 `-n N` 捕获前按下 Ctrl+C，则退出 `130`)。若捕获过程中失败，将停止流并退出 `1`，同时显示此前已保存的帧数。`--poll-interval` 已被弃用且将被忽略： 但系统仍会接受该指令，以便旧版脚本能够运行，同时会在标准错误输出中添加提示。 |
| `project sensor read PATH NAME [--json]` | 最新频谱帧。 |
| `project sensor log PATH NAME [--seconds SEC] [-o DIR] [--device-name NAME]` | 为 `--seconds` 录制 .daq 文件 （默认 10）。若记录中未包含任何读数，系统将在标准错误输出中报告该情况并退出 `1`。 |
| `project run PATH RECIPE.yaml` | 执行 YAML/JSON
捕获配方。 `--dry-run` 进行验证而不执行。 |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | 计算数组的对齐参数 — 参见 [下方的标志表](#project-align-calibrate-options)。 |
| `project align status PATH NAME [--json]` | 打印当前对齐配置文件。 |
| `project align clear PATH NAME` | 清除缓存的配置文件。 |
| `project align tweak PATH NAME --serial SN [--dx N] [--dy N] [--rotation-deg N] [--scale N]` | 微调一个从属节点的变换 （默认值为 `0`、`0`、`0`、`1.0`：不作更改）。 |
| `project align export PATH NAME --to FILE` | 将配置文件保存为JSON
。 |
| `project align import PATH NAME --from FILE [--no-validate]` | 加载已保存的配置文件。 |

#### `project align calibrate` 选项

| 标志 | 默认值 | 说明 |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | 对齐方法。**这些拼写与 `lattice align-calibrate`** 不同，后者接受简写形式 `orb` / `akaze` / `phase`；在此标志下，这两个命令不可互换。 |
| `--model {translation, rigid, affine, homography}` | `affine` | 变换模型以适配。 |
| `--frames N` | `1` | 将同步帧快照与 平均值。 |
| `--reference SN` | 主帧 | 参考摄像机序列号；其余所有成员均被变形到该帧上。 |
| `--max-features N` | `5000` | ORB特征点数量上限。 |
| `--ratio-threshold F` | `0.75` | Lowe’比率测试。 |
| `--ransac-threshold-px F` | `3.0` | RANSAC 内点阈值。 |
| `--min-matches N` | `15` | **质量阈值** — 若符合的内点数量低于此值，则拒绝该求解结果。 |
| `--max-reproj-err-px F` | `4.0` | **质量阈值** — 若重投影均方根误差超过此值，则拒绝该求解结果。 |
| `--checkerboard RxC` | — | `--method checkerboard` 的板级几何信息，例如 `9x6`。 |
| `--name PROFILE` | 空 | 嵌入到保存的 JSON
中的轮廓名称。 **非数组名称** — 该名称指位置参数 `NAME`。 |

这两个质量门槛是校准虽然成功解算却仍
拒绝保存的原因：若配置文件未能通过其中任何一个门槛，将导致后续每次
捕获都出现无提示的配准错误，因此系统会拒绝保存而非保留该配置。

### 已移动的项目 (`--relink`)

项目的 `project.json` 会使用完整路径为每个源文件（包括每张图像和光传感器
记录）命名。若将项目文件夹或整个项目文件夹
移动到另一驱动器，这些路径仍会指向旧位置。

打开项目时，系统会自行修复可修复的部分：原本位于项目
文件夹内的文件会在项目当前所在的文件夹中被查找，并在该处找到的每个路径
都会被重新指向并保存 （`project.json` 会先作为
`project.json.before-relink-<date>-<time>` 进行备份）。每次打开项目时——无论是应用程序本身，
`chloros-cli project open`， `chloros-cli process` 导入现有项目、SDK

以及 `chloros-mcp` 时，都会发生这一操作。路径仅会重定向到该位置实际存在的文件。

位于项目文件夹之外的文件——例如通过 Hub 会话或“添加文件夹”导入的文件——
无法通过这种方式找到。请指定它们当前所在的文件夹名称：

```bash
chloros-cli project open "N:/Chloros Projects/Field_A" --relink "E:/backup/Hub Projects"
```

`--relink` 会采用文件被移动到的文件夹，或其上级的任何文件夹：系统会根据每个缺失
文件的文件夹后缀名称
（`Hub Projects/<session>/raw/<serial>`，然后是 `<session>/raw/<serial>`，……），并且每个
文件将从能找到最多文件的那个位置中提取。`project open` 会输出
重新指向的内容、找到的源文件数量，以及其余文件所在的
文件夹。 仍有部分缺失：显示警告，退出 `0` —— 项目将随其余部分打开。未
找到任何文件：显示错误并列出这些文件夹名称以及 `--relink`，退出 `1`；后端同样
拒绝为 `process` 或 `project devices --reconnect` 打开此类项目（HTTP
409 `PROJECT_SOURCES_MISSING`）。

### 示例

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Its hub captures moved to another drive: point it at them
chloros-cli project open "/home/user/Chloros Projects/Field_A" --relink "/media/backup/Hub Projects"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### 配方 DSL

`project run RECIPE.yaml` 接受一个 YAML 或 JSON 文件，该文件描述了一系列操作：

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        auto_exposure: "Off"
        gain_db: 6.0
        auto_gain: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

支持的操作：`apply`、`wait`、`capture`、`stream`、 `burst`、`sensor`。 `apply` 包含 `cameras.json`、`settings` 键——`exposure_us`、 `auto_exposure`、`gain_db`、`auto_gain`、`target_brightness`、…… （旧名称 `exposure_time`、`exposure_auto`、`gain`、`gain_auto` 作为别名被接受）。 以这种方式设置的曝光、增益或目标值将被视为您的设置：Chloros 的自动曝光控制（包括 DAQ 测光曝光）不会对其进行调整。 手动曝光会保持增益不变：在自动模式下，相机会持续调整增益（在其整个范围内）以趋近目标亮度，直到您设置 `gain_db` 或 `auto_gain: "Off"`。 后续的`auto_exposure: "Continuous"`将曝光交还给自动模式，而`target_brightness_auto: true`将锁定的目标交还给Chloros的自动目标。 `burst` 操作需要 `name`（必填）、`count`（默认值为 5）、 `interval`（秒，默认 0）、`output`、`format`、 以及 `settings`（与 `apply` 具有相同的每台相机设置结构）；数组连拍使用与 `project burst` 相同的“新同步组”看门狗。

在 `capture`、 `burst` 或 `stream` 步骤中的 `output` 默认值为 `output`。 相对路径 `output` （默认已包含）将根据您运行 `chloros-cli project run` 的文件夹进行解析，而非配方文件所在的文件夹或项目文件夹（在 1.2.0 版本中，它位于后端的安装文件夹内）。

运行它：

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

**每次运行都会首先与项目进行核对。**每个操作的键必须恰好是上述动词之一——例如 `- capture: {name: FrontLeft}`，而非 `- action: capture` 或 `- type: capture` ——例如，`name` 对应项目中的 `cameras.json` / `sensors.json`，而 `apply` / `capture` / `burst` / `stream` 以及用于 `sensor` 的光传感器； `stream` 需要 `count` &gt; 0，而 `wait` 需要持续时间。包含任何此类错误的配方将**不**执行任何操作 （`[recipe] error: …`、`[recipe] not run: …`、退出 `1`），即使存在 `stop_on_error: false` 也不例外。 步骤未使用的键会被标记为 `[recipe] warning:`，该配方驱动的、当前未连接的每个数组或光传感器也是如此。 配方绝不会连接设备：请先运行 `chloros-cli project connect PATH`。`--dry-run` 会打印配方内容，随后显示相同的错误和警告，当配方出现错误时，`[recipe] valid: …` / `NOT valid: …`，不执行任何操作，并在配方出现错误时退出 `1`。

---

## 环境变量

| 变量 | 效果 |
| --- | --- |
| `CHLOROS_BACKEND_URL` | 覆盖后端URL（默认 `http://127.0.0.1:5000`） — **仅由 `lattice`、`project` 和 `daq pool-*` 命令族支持** （`lattice network-analysis` 则采用 `--backend-url`）。 核心命令（`process`、`login`、`logout`、`status`、 `export-status`、`time-sync`、`selftest`) 会将 `http://127.0.0.1:<port>` 固定，并忽略此变量 （IPv4字面量可规避Windows中`localhost`→`::1`约每请求2秒的性能开销），因此它们始终将目标指向本地机器。 |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | 在后端环境中设置。 `1` 会降低超额订阅拒绝的严重程度——将包含 `pin_resolution` 的数组连接，或 [临时多摄像头命令](#multi-camera-transient-sessions) (`error_code` `ARRAY_OVERSUBSCRIBED`) — 降级为强烈警告，并接受 GVSP 数据包丢失。 仅用于测试——参见 [数组 fps 与突发模型](#array-fps--burst-model)。 |
| `CHLOROS_ARRAY_PACKED12` | 在后端环境中设置： `on` 或 `off` 将每个数组的默认值设置为 [打包的 12 位传输](#packed-12-bit-transport---packed) （默认值为 `auto`）。显式设置的 `--packed` 仍具有优先级。 |
| `CHLOROS_DISABLE_UPDATE_CHECK` | `1` （或 `true` / `yes`）可在启动时禁用 Linux CLI 的 [自动更新检查](#chloros-cli-update)。 |
| `CHLOROS_CLI_MODE` | 由CLI自身设置；指示后端启用并行处理。 |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` 跳过 GVSP 备用探测 （仅限 ICMP 结果）。**这会禁用巨包，而不仅仅是抑制日志输出**——摄像头在每条路径上仅对不超过 1500 的 DF ping 进行响应，因此该探测是检测巨包的唯一途径。 每次连接可为每台摄像头节省约 1 秒；如果网络*能够*传输巨包，则成本约为线速上限的 1.45 倍。设置此选项时，SDK 会发出警告。 |
| `CHLOROS_GVSP_SESSION_REUSE` | 在后端环境中设置：`0` 会在每次连接时测量数据包大小，而非每台摄像机每个后端会话仅测量一次。 |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | 将 GVSP 数据包大小固定为 N 字节；完全跳过探测过程。建议使用按命令设置（`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`）而非永久设置： 固定大小将无法适应前端网络的变化，且在无法传输巨型数据包的路径上将大小固定为 9000 会导致使用 `SC_ERR_TIMEOUT -1011` 时 **每次** 捕获都超时。 |
| `CHLOROS_TRANSIENT_AE_SETTLE_S` | 在后端环境中设置：[临时多摄像头命令]（#multi-camera-transient-sessions）或某台相机（无其他设备占用）的 `lattice capture`，用于在相机保存帧之前，根据前一场景设置每台相机的曝光时间（默认 `5`）。 `0` 将其关闭：此时每个摄像头的自动曝光将从第一帧开始运行，起始值为其曝光限制。 |
| `CHLOROS_LOG_DIR` | 在后端环境中设置：详细日志的存储文件夹。 Windows 系统默认值为 `%LOCALAPPDATA%\Chloros\logs`，Linux 系统默认值为 `~/.cache/chloros/logs`（若已设置，则为 `$XDG_CACHE_HOME/chloros/logs`）。 .deb 包的根服务会设置 `/var/log/chloros`，该文件可被 `adm` 组读取 — 参见 [将后端作为服务运行](#running-the-backend-as-a-service-headless)。 |
| `TMPDIR`（Linux） | 临时文件的存储位置。若该路径存在，CLI将自动使用 `/mnt/ssd/tmp`。 |

---

## 退出代码

| 代码 | 含义 |
| --- | --- |
| `0` | 成功。 |
| `1` | 通用错误（大多数子命令错误）。 当捕获命令被要求写入的任何文件写入失败时，即使其他文件已保存，该命令也会退出并返回 `1`，而当记录器报告错误时，会退出并返回 `array-burst` / `array-record` 会退出 `1`，当记录器报告错误时——参见 [写入失败的文件](#files-that-fail-to-write)。 当无法验证运行的导出结果，或丢失了图像时，`process` 也会退出 `1`，尽管其他产品已成功写入。 |
| `2` | 参数错误（未知或缺失的标志；当两者均不满足时，也会出现 `time-sync set-priority`）。 |
| `130` | 被 Ctrl+C 中断：任何被其中断的命令——`lattice`、`project`、 `daq` 和 `process` 均会中断（当后端已停止运行后，`process` 也会中断，参见 [`process` 注释](#notes))。正常停止方式为 Ctrl+C 的命令——`lattice array-capture --continuous` / `--interval`、`lattice viewer`、`lattice continuous`、`multi-stream`、`align-stream`、 `array-record`、`array-burst` 和 `project stream` 均使用 `-n 0` — 根据其实际执行情况退出： 当所有数据写入完毕时返回 `0`，当出现任何错误时返回 `1`。 |
| `143` / `129` | 被 SIGTERM / SIGHUP（128 加上信号编号）终止，此时命令的清理工作已执行完毕：正在运行的任务已停止且其摄像头已归位，后端处理进程已停止。 `lattice continuous` 或其他通常通过 Ctrl+C 停止的录制任务也会退出 `143`，而非 `0`。 |

对于 `needs_force_slip` 和 `error` 的判定结果，`lattice network-analysis` 会退出 `1`， 而 `selftest` 会退出 `1`，除非所有七项检查均通过。

---

## 故障排除提示

- **“需要登录”** → 在该计算机上运行一次 `chloros-cli login EMAIL PASSWORD`。
- **“该项目文件夹在该计算机上层级过深”** → 该计算机已禁用 Windows 长路径（Windows 默认设置），因此任何路径长度均不得超过 260 个字符，且产品路径位于项目文件夹下方最多约 130 个字符处。 未处理任何内容。该消息给出了符合要求的最长项目文件夹路径：请使用更短的文件夹（`-o` 或 `chloros-cli set-project-folder`），或启用 Windows 长路径并重启 Chloros。 Windows安装程序中提供“启用Windows长路径”选项，默认已勾选；否则请使用组策略：*计算机配置 &gt; 管理模板 &gt; 系统 &gt; 文件系统 &gt; 启用 Win32 长路径*。
- **“后端无法访问”** → 启动 Chloros 桌面应用（Windows），或手动启动后端：在 Windows 上运行 `"C:\Program Files\Chloros\backend\chloros-backend.exe"`（位于安装文件夹中的 `backend` 文件夹内； 该文件不在 `PATH` 中），Linux 系统上为 `chloros-backend`。若为远程连接，请检查 `CHLOROS_BACKEND_URL`。
- **`lattice` 命令执行失败，提示“未找到 LATTICE 相机驱动程序”** → 后端无法加载随其安装的 LATTICE 相机运行时。 所有 `lattice` 命令均在后端运行；CLI 本身不包含任何相机驱动程序。请重新安装 Chloros：Windows 安装程序和 .deb 包会将运行时文件安装在后端旁边。
- **“Array connect”/“Array Settings”显示“FRAMES WILL DROP”或“Reduce ROI to enable”** → 主机网卡的接收环大小过小（通常在网卡驱动程序更新后会重置为 32）。 请参阅 [主机网卡设置与调优](#host-nic-setup--tuning-lattice-arrays) — 设置 `ReceiveBufferLen=256`、`PendingReceives=64`。
- **机器在重启/关机时卡死，随后出现 WMI `Invalid class` / 网卡无法启用 / USB 驱动器缺失** → 过时的 USB 10GbE 适配器驱动程序导致 `DRIVER_POWER_STATE_FAILURE` （蓝屏错误 `0x9F`）。请更新适配器驱动程序——参见 [主机网卡设置与调优](#host-nic-setup--tuning-lattice-arrays)。
- **Linux：捕获数据归 root 所有，因此无法删除，且后端日志不在您的 `~/.cache`** → 后端以 root 身份运行，通过 `chloros-backend.service`。 请切换到 `chloros-backend@<user>.service`，该设备将以后端用户身份运行后端——请参阅 [以后端服务模式运行后端](#running-the-backend-as-a-service-headless)。 在此之前，root 服务的日志位于 `/var/log/chloros`（`adm` 组可读取）。
- **在 Linux 中：若后端单元 `failed` 显示“已有另一个实例正在运行”（退出状态 98）** → 说明该端口已被另一个后端占用：同一时间仅允许一个后端运行。 如果启用了两个后端单元，请保留其中一个：将未使用的那个设置为 `sudo systemctl disable --now <unit>` —— 参见 [将后端作为服务运行](#running-the-backend-as-a-service-headless)。 如果仅启用了该后端，则说明该端口被 systemd 外部启动的后端（通过 `chloros-backend`、`chloros-cli` 或 SDK 脚本）占用：先停止该后端，然后执行 `sudo systemctl start <unit>`。
- **Jetson 交换分区警告** → 添加基于文件的交换分区；CLI 将显示精确的 `fallocate` / `swapon` 命令。
- **来自 `lattice`、`project` 或 `daq` 命令的“无法连接后端”提示** → 没有后端正在运行：这些命令不会启动后端。 请运行 `chloros-cli backend status`，并参阅 [启动后端](#start-the-backend)。
- **“Chloros 中不支持 `daq net ...`”**（Linux 和 Jetson）→ DAQ-E 设备自身的部分设置被保留用于 MAPIR 服务。 网络设置、重启和固件更新分别由 `pool-set-network`、`pool-reboot` 和 `pool-update-firmware` 处理；其余情况请联系 MAPIR 支持团队。
- **“DAQ 直接硬件命令需要完整的 daq 软件包”**（Windows）→ 请使用 `pool-*`（该命令通过后端驱动同一传感器），或使用 Python 中的 `chloros_sdk.connect_daq_sensor()`。

---

## 参见

- [Python SDK 参考](sdk-reference.md) — 每个 CLI 命令的编程等效实现。
- [DAQ 光传感器](https://mapir.gitbook.io/chloros/daq-light-sensors)（见 Chloros 手册）—— 传感器、其规格与校准范围、数据记录、DAQ-E 网络连接。
- [LATTICE 相机](https://mapir.gitbook.io/chloros/lattice-cameras/lattice)（见 Chloros 手册）—— 连接、相机设置、捕获模式、阵列。
- 在线文档：`https://mapir.gitbook.io/chloros/cli`
