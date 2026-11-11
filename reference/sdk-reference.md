# Chloros Python SDK 参考手册

**版本：**

1.3.0**生成时间：**2026-07-29 19:19 ·**修订时间：** 2026-11-06**包：** `chloros-sdk` (PyPI)**适用对象：** 针对大型语言模型（LLM）使用进行优化；易于人类阅读。**范围：** `import chloros_sdk` 公开的所有类、函数和辅助函数，包含可直接复制粘贴的示例，涵盖图像处理、单摄像头控制、同步数组、 数据采集（DAQ）传感器以及项目自动化，此外还包含用于人工智能助手的 `chloros-mcp` 服务器。

若您只需了解重点内容，请跳转至：
- [安装与快速入门](#安装)
- [适用于 LATTICE 阵列的 Smart-Connect](#smart-connect-for-lattice-cameras)
- [DAQ 传感器会话](#daq-sensor-sessions)
- [相机实用工具](#camera-utilities)
- [瞬态阵列、对准配置文件与索引](#transient-arrays-alignment-profiles--indices)
- [项目文件夹与配方](#project-folder--recipes)
- [项目自动化](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [在您自己的处理流程中使用实时帧](#live-frames-in-your-own-pipeline)
- [MCP 服务器（AI 助手）](#mcp-server-ai-assistants)

> **

1.3.0 版本中的变更**（现有 1.2.0 脚本会检测到这些变更）。
>
> **

1.2.0 脚本可能需要处理的变更：**
>
> - **后端未确认的发布现在会触发异常。** `CameraSession.disconnect()`、`ArraySession.disconnect()` / `force_disconnect()` 发生在会话打开的数组上，`DAQSensorSession.disconnect()` / `force_disconnect()` 发生在会话打开的传感器上，而 `ChlorosProject.disconnect_all()` 会在后端拒绝释放或未能及时确认时触发 `ChlorosConnectError`，因为设备可能仍被占用。 在 1.2.0 版本中，这些错误会静默返回。`with` 块的退出仍不会触发异常。
> - **已借用的设备永远不会被释放。** `with attach_array(...)` 以及针对该设备的显式 `disconnect()` 调用会使数组保持运行状态（在 1.2.0 版本中，退出代码块会释放该设备，即使 GUI 仍持有该设备）；若要终止该设备，请调用 `force_disconnect()`。同样，当 `connect_daq_sensor()` 发现传感器已存在于池中（包括零标志智能检测）时，该传感器在 `disconnect()` 时，以及在离开其 `with` 块时，仍会保持该传感器处于打开状态。
> - **PNG和JPEG仅支持`display`和索引产品。** `capture()`配合`format="png"` / `"jpg"` (`ext=".png"` / `".jpg"`) 以及 `radiance` 或 `reflectance` 级别， 或 `processing="all"`，会在任何请求之前提升至 `ValueError`； 12 位的 `raw` / `debayered` 会被后端拒绝。参见 [PNG / JPEG](#png--jpeg-take-the-display-and-index-products-only)。
> - **`connect_daq_sensor(sensor_id=...)` 会打开该传感器或触发** `ChlorosConnectError` 并列出其检测结果；它绝不会回退到其他传感器。参见 [固定传输](#pinned-transport)。
> - **`frame_stream()` 触发异常而非保持静默：** `FrameFetchError` 在被拒绝后，下一个时钟周期会重复 （执行反射率读取，且完全不包含数据采集），在 60 秒内未收到帧，以及在 60 秒内未收到新触发信号后。参见 [每次读取一个触发信号](#one-trigger-per-read--group_id)。
> - **读取超时或连接中断会触发 `ChlorosConnectError`** 异常，该异常源自每个 smart-connect 调用，以及 `analyze_array_network()` 异常（其中 1.2.0 版本会触发底层的超时或连接错误）。
> - **`connect_array()` 默认等待 600 秒（`timeout`，1.2.0 版本中为 180 秒），且其放弃的连接将被撤销。** 该调用将其 `timeout` 发送至后端，后端会将每台摄像头恢复至初始状态，并在连接未及时完成时不建立任何连接；该调用会抛出 `ChlorosConnectError` （HTTP 504 `CLIENT_DEADLINE_PASSED`）。在 1.2.0 版本中，摄像头通电后立即进行的连接可能会持续时间超过该调用，并在调用抛出异常后仍成功连接数组。`connect_all()` 以相同方式等待每个已保存的数组 （1.2.0 版本中为 120 秒）。参见 [`connect_array()` 签名](#connect_array-signature)。
> - **`connect_array()` 保留完整的传感器数据。** 如果未启用 `width` / `height` / `binning`，每台摄像机均设置为全画幅模式，帧率将降至主机和网络所能提供的水平；1.2.0 会缩小或合并帧以维持帧率。参见 [同步数组](#synchronized-array--arraysession-smart-prep)。
> - **M3M（单色）摄像机上的拜耳像素格式即为其深度格式。** M3M仅提供单色格式，而预设名称中包含“拜耳”字样： 在 1.2.0 版本中，相机拒绝了该格式并保留了自身的格式，而 `settings.pixel_format` 则命名为拜耳格式。`LatticeCamera(preset=...)`、`apply_preset()`、`set_format()`、`CameraSettings(pixel_format=...)`、`connect_camera(preset=...)`以及一个数组'共享格式现在将相同深度的 Mono 格式设置在 M3M 上：`"BayerRG8"` 和 `high_speed` 生成 `Mono8`；`"BayerRG12"`、`high_quality` 和 `triggered` 生成 `Mono12`；`cam.settings.pixel_format` 对其进行命名。 反之亦然：M3C 绝不会被转换为 Mono 格式，因此 `"Mono12"`——无论是源自上述任何一种， `connect_array(pixel_format=...)`、`set_settings(pixel_format=...)` 或数组的保存格式——在 M3C 上都会生成 `BayerRG12`，无论该数组的主摄像机是哪一台。 您传递的 `CameraSettings` 会保留其名称，因此同一对象仍会向其所属家族中的每台摄像机请求该深度。`connect_camera(preset=...)` 保留预设的格式 （1.2.0 版本在应用预设后将相机设置为 12 位），且通过预设打开的相机不再将采集的数据写入 `PRESETS`。对于相机拒绝的任何其他格式 （其 `PixelFormat` 列表中未包含的格式，例如 M3M 上的 `RGB8` 或未包含该格式的固件上的 `BayerRG16`），也不会被命名为独立的名称：在 `LatticeCamera` 连接后，`apply_settings()` 或 `set_format()`， `cam.settings.pixel_format` 和 `packed_transport` 分别标明了摄像机传输的格式，而其中一行打印内容则标明了被拒绝的格式。
> - **`analyze_array_network()` 显示连接适用的设置：** `pixel_format=None`（原为 `"BayerRG8"`）表示连接设定的格式 (Mono12 / BayerRG12)，而 `packed="auto"` 表示压缩传输的设置，因此其给出的帧率可能与 1.2.0'版本有所不同。您传递的 `pixel_format` 表示以该深度处理 M3M 文件。参见 [网络分析助手](#network-analysis-helper)。
> - **`CameraSession.capture()` 返回一个 `CaptureResult`** （仍为帧字典列表），其中包含 `.skipped`、`.failed`、`.daq_files` 和 `.daq_matches`。一个 `ArraySession.capture()` 的结果还包含 `.alignment`（说明是否应用了对齐变形，以及未应用的原因）和 `.oriented`。
> - **数组捕获默认会进行对齐，与应用程序的行为一致。** `ArraySession.capture()`、`continuous()`、 `capture_repeated()` 和 `ArrayHandle.capture()` / `capture_smart()` 会将每个成员的 `debayered`、`display`、`display`、`reflectance` 以及索引文件映射到主摄像机的帧中，并将其裁剪至成员的重叠 （比传感器小，大小为相机偏移量，`XMP-mapir:AlignmentJSON` `"applied": true`），因此它们与Chloros应用捕获的同一场景的文件相同； `raw` 保持原生格式。 此前，未启用 `aligned=True` 的捕获会将每个文件以原始尺寸保存，且不进行畸变校正。`aligned=False` 保留了这一行为。`chloros-mcp`'的 `capture_array` 和 `start_capture_run` 采用相同的默认设置。参见 [`ArraySession` 方法](#arraysession-methods)。
> - **`connect_all()` 指代拒绝 SDK 的后端。** 已注销或使用低于 Copper 套餐的运行中后端，将通过 `errors` 报告，并附带其特定原因 （HTTP 401 / 403），而非“没有正在运行的 Chloros 后端”；且每次 `connect_all()` 调用都会重新检查后端，因此登录后调用该方法对同一项目对象仍有效。
> - **相对路径的 `output_dir` 调用针对本地后端有效。** 相对路径在发送前会转换为以脚本工作目录为基准的绝对路径。在 1.2.0 版本中，后端会在其自身的安装文件夹内解析这些路径，这会导致“访问被拒绝”的错误。
> - **`numpy` 现已成为 SDK 的必备条件**（live-frame 调用会返回 numpy 数组）。
> - **若导出级别名称不为“one”，则会被拒绝。** `processing=` 和 `levels=` 会触发 `raw`、`debayered`、`radiance`、 `reflectance` 或 `display` （或别名，例如 `preview`；`processing=` 以及 `"all"`）。 在触摸相机之前，任何其他名称均会被拒绝（HTTP 400 `UNKNOWN_LEVEL`：`capture()` 会触发 `ChlorosConnectError`、`grab()` / `frame_stream()` `FrameFetchError`）；1.2.0 版本对此保存或返回了 `raw`。
> - **LATTICE 捕获不支持“纹理感知”功能。** 在包含 LATTICE 捕获的项目中，`configure(debayer="Texture Aware (Slow, Highest Quality)")` 会触发 `ChlorosConfigurationError`，而在导入前，当 `configure()` 要求 在导入前曾请求此设置。项目先前保存的“纹理感知”值被设置为“标准”，并触发了 `UserWarning` 错误。LATTICE 拍摄素材始终采用标准去马赛克处理，因此 1.2.0 版本接受了该设置并予以忽略；Chloros 应用对此类素材也不提供该选项。 参见 [支持的值](#supported-values)。
> - **相机将恢复为初始状态——除非遇到 `keep_settings=True`。** `ArraySession.disconnect()` / `force_disconnect()`，每个瞬态函数的结尾（`capture_cameras_transient()`, `start_multi_camera_stream()`、`test_array_sync()`、`calibrate_alignment_profile()`、`capture_aligned_multiband()`、`start_aligned_stream()`、 `capture_live_index()`）的结束以及`start_camera_continuous()`的停止，会将每台相机恢复至初始状态：包括其触发器和同步线接线、PTP设置，以及本次会话中更改的所有相机设置（曝光和增益、 自动曝光目标、限制值和阻尼、白平衡、黑电平、像素格式、像素合并和感兴趣区域、翻转、帧率）。 而 `keep_settings=True` 则将每台摄像机的设置保存到摄像机自身的内存中，以便开机时直接采用这些设置；`clear_settings=True` 则将该内存重置为出厂设置。 无论哪种方式，相机都将保持自由运行状态。项目在连接相机时，会根据自身保存的设置来配置相机。参见 [相机内存](#camera-memory--keep_settings--clear_settings)。
> - **所有模式下，项目的保存方式均相同。** 对项目下连接的摄像头、阵列或光传感器所做的任何更改——无论是通过SDK、应用程序、CLI还是MCP服务器进行的——都会保存到该项目的`cameras.json` / `sensors.json` 目录中保存；在任何模式下打开该项目都会根据其中设置配置设备；而 `ChlorosProject` 会将自身文件夹命名为后端。此前，只有应用程序会保存项目。参见 [一个项目记录，所有模式](#one-project-record-every-mode)。
> - **由多台相机生成的文件会标注每台相机的信息。** 多波段堆栈、由多台相机波段生成的实时索引以及数组捕获的 `composite/` 图像，均以列表形式携带每台源相机的序列号、型号和校准包（`XMP-mapir:SourceCameraSerials`、 `SourceCameraModels`, `SourceCalBundleSha256s`, …)，而 EXIF 记录中会以逗号分隔的形式列出序列号列表。此前，这些文件要么不包含相机信息，要么仅包含第一台相机的序列号。 参见 [由多台相机生成的文件](#files-made-from-several-cameras)。
> - **仅 `raw` 文件包含相机的校准标签。** 与 MicaSense 兼容的文件集（`BlackLevel`、`RadiometricCalibration`、各-通道的 `Band{R,G,B}RadiometricCalibration`、`MicaSenseVignetting*` 数据集）以及 `CalibrationJSON` 有效载荷描述了传感器的原始值，因此它们仅包含在 `raw`捕获文件中传输。其他所有文件——即捕获`debayered`、`display`、`radiance`、`reflectance`以及索引文件、连续录制和多摄像头流帧，以及所有 `process()` 产品 ——都包含摄像机的标识（`XMP-mapir:CameraSerial`、`CameraModel`、`FilterType`、`LensModel`、 `CalBundleSha256`, ...），并在每次拍摄时包含该帧本身的记录（`XMP-mapir:FrameJSON`：曝光、增益、温度、像素格式、拍摄时间）。在 1.2.0 版本中，这些文件还包含原始数据集，相关处理工具（如 Pix4D、Metashape）会将其应用于非原始值的像素。Processing 通过序列号和 `CalBundleSha256` 来确定此类文件的校准信息。
>
> **其他变更：**
>
> - **数组的自动帧率将根据数组的实际输出进行调整。** 若某帧率丢失的触发信号超过 1%，则会降低该帧率（此判断基于多次长时间监测而非单次监测），且系统会不定期重新尝试恢复其先前被调低的帧率。用户手动设置的帧率则保持不变。
> - **自动曝光更稳定了。** 它在光线恒定时保持恒定，光线变化后数秒内稳定下来，忽略微小的高光，并能恢复在强阳光下停止响应的相机。参见 [自动曝光的工作原理](#how-auto-exposure-behaves)。
> - **`import_images(recursive=True)` 仅统计您的图像和录像。** 包含项目 `camera_cal/` 的文件夹中，这些校准文件将不再被计入图像和录像的总数；而已将这些文件纳入的项目则会将其移除。
> - **`configure(radiometric_output=...)` 生效。** 现在它会设置项目的 `Export radiance` / `Export reflectance` 输出格式；此前，每个项目自身的输出设置会优先于此， 因此 `radiance`、`sensor-response` 和 `all` 均导出了辐射度和反射度。
> - **`test_array_sync()` 根据摄像机的开机状态进行处理。** 它会在运行期间开启摄像机的 PTP 功能，并在首次触发（`ptp_lock`）前等待其时钟锁定并稳定——每台摄像机从通电起需 120 秒（约 2 分钟）才能完成锁定 （`ptp_settle_s=` 设置该参数），因为刚锁定的时钟在约 2 分钟内误差可达 ~200 微秒 ——并将每台摄像机的 PTP 恢复至初始状态。对于仍处于先前测试锁定状态的摄像机，不再等待其重新锁定。此前在单交换机网络中（无 DAQ-E 响应时），该程序会保持 PTP 关闭，并将每台摄像机的自由运行时钟视为无法锁定的 PTP 时钟 （“PTP未锁定”，每次运行均如此）。
> - **`.daq`记录会标明其名称是经选择还是默认生成的。** DAQ-U、 DAQ-M 和 DAQ-E 出厂时未配备盖帽，且 Chloros 无法检测是否安装了盖帽，因此未指定盖帽名称的传感器默认会被命名为 `sunshine_cosine`，这属于*推定*。每个 Chloros 记录（模式 1.30）现在会在 `cap_id` 旁边存储 `cap_id_source`：当您为封装命名时（包括 `none`）， 当其为默认值时，则为 `auto_default`。处理一个电容器被默认设定的记录时，若项目的 DAQ 电容器保持为“自动”，则会输出一个名为 `[daq-dls] WARNING` 的文件： 在阳光校正条件下，未加滤波器的 DAQ-U 读数约高出 30 倍。连接时**请为所安装的滤波器命名**（`connect_daq_sensor(cap_id=...)` 或 `DAQSensorSession.set_cap()`、`chloros-cli daq pool-connect --cap-id` / `daq pool-set-cap`，或GUI中的电容下拉菜单），或者设置项目的DAQ电容，警告就会消失。
> - **Windows 系统上不再出现 pip 控制台窗口。** 在安装了 Python 的 PC 上，后端及其每个图像处理进程在启动时会各运行一次 `pip`，并各自打开一个控制台窗口。现在不再出现这种情况。
> - **实时 `.daq` 录制数据可在断电后保留。**录制数据在每次提交时都会在传感器后台通过日志和完整磁盘同步写入，因此即使磁盘速度较慢，也会延迟文件写入，但绝不会延迟读数。 若记录在提交过程中中断（断电、强制终止），可能会在原文件旁生成一个 `<name>.daq-journal` 文件：**请将这两个文件合并。**Chloros在首次读取文件时会回滚日志，因此记录文件打开时包含停止前已提交的所有数据（约最后3秒的数据会丢失）；在1.2.0版本中，若在错误时刻发生断电，可能会导致整个文件无法读取（“数据库磁盘映像格式错误”）。
> - **在繁忙磁盘上，`DAQSensorSession.record_start()` 不再导致传感器卡死**： 读数会被排队并由独立线程写入，未能写入的读数将计入 `status()` 的 `record_errors` 中（与 `record_last_error` 一起）。
> - **直接 `LatticeCamera`：**没有块曝光/增益的帧现在报告相机的**真实** 曝光和增益（回退机制会读取缓存值，该值在整个自动曝光运行过程中可能保持不变），而 `apply_settings(trigger_source='LineN')` / `set_trigger_source('LineN')` 则应用了与 Chloros 线控从机相同的 5 µs 输入去毛刺处理（未经过滤，即从机在未收到触发信号时仍会触发帧）。
> - **软件触发相机不再丢弃每隔一次的触发信号。** 处于软件触发模式的相机现在会将读出期间收到的触发信号加入队列（或Chloros等待其准备就绪），这与有线同步相机此前的工作方式一致。
> - **`discover_daq_sensors()` 用于命名 DAQ-E-S `DAQ-E-S`。** 对于每个以太网传感器，其 `model` 此前均被标记为 `DAQ-E`；若某传感器在约 2 秒内未响应身份验证，仍会以 `DAQ-E` 的形式列出。
> - **新增：`chloros-mcp`，一款用于 AI 助手的 MCP 服务器。** Claude Desktop、Claude Code、Cursor 及其他 MCP 客户端可处理图像（创建项目、导入捕获数据、选择设置、进行处理、报告运行结果）并控制 LATTICE 相机、阵列及 DAQ 光传感器：连接、预览、设置曝光和色彩、捕获、记录。 其工具功能涵盖了该SDK的功能。它随SDK一同安装，Linux的.deb包会将其安装在`/usr/bin`目录下。参见 [MCP服务器（AI助手）](#mcp-server-ai-assistants)。
> - **新增：相机实用工具**，`chloros-cli lattice` 命令作为 SDK 调用：色彩配置文件、饱和度/对比度、高光、白平衡（单次、 自动、固定比例）白平衡、预设、滤镜模型、固件状态与更新、电源（检测、休眠、唤醒）、目标校准、网络适配器（检测、估算、修复）、文件暗角校正、DLS读数以及主机检测。参见 [相机实用工具](#camera-utilities)。
> - **新增：其余CLI功能均以SDK调用形式实现**，因此CLI、SDK和`chloros-mcp`提供相同的功能（出于安全考虑，CLI仅保留登录、DAQ-E的配置以及直接DAQ服务器功能）：通过单条命令打开的相机（`start_camera_continuous()`、`start_multi_camera_stream()`、`capture_cameras_transient()`、`test_array_sync()`、`detect_array_master()`, `analyze_array_models()`），对齐配置文件（`calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `alignment_profile_info()`、`reorder_alignment_profile()`）、索引（`index_catalog()`、 `compute_index()`、`capture_live_index()`）、PTP 主节点（`time_sync_status()` 及其相关文件）、项目文件夹（`project_folder()`、 `set_project_folder()`、`reset_project_folder()`）以及配方（`run_project_recipe()`，即CLI自带的运行器）。光传感器相关文件：`connect_daq_sensor(cap_id=...)`、`DAQSensorSession.set_cap()`、`daq_bindings()` / `set_daq_bindings()`、`daq_band_presets()` 和 `light_sensor_reflectance()`（两个传感器的环境反射率、物体反射率及折射率）；用于摄像头： `CameraSession.status()` 和 `capture_repeated()`；对于阵列：`ArraySession.capability()` 和 `set_member_settings()`（单个成员的曝光、增益等）。请参阅 [瞬态数组、对齐配置文件与指数](#transient-arrays-alignment-profiles--indices)、[项目文件夹与配方](#project-folder--recipes) 以及 [DAQ 传感器会话](#daq-sensor-sessions)。
> - **改进的 LATTICE 指数产品**（NDVI 及其他指数）源自 `process()`，适用于所有处理模式。
> - **LATTICE OCN相机：在阳光下，辐射度、反射率及指数产品的质量得到提升。** 波段解混后会应用机队校正——包括实时处理、捕获数据以及`process()`；RGN相机保持不变。 参见 [辐射测量输出](#radiometric-output-lattice-multispectral-pipeline)。
> - **每次捕获都会保存其使用的 DAQ 读数，与 GUI 行为一致。** `CameraSession.capture()`、`ArraySession.capture()` 和 `continuous()` 会写入 `.daq` 及其 `.match.json`；在 1.2 版本中，0 版本中，原始或去拜耳化捕获仅通过 `force_daq=True` 保存一项。`daq=False` 将其关闭，而 `daq_tolerance_ms=` 设置匹配容差；两项结果均说明了 `.daq_files` 和 `.daq_matches` 中保存了什么。现在，捕获操作会等待 DAQ 的下一次读数，以便混合曝光前后两侧的读数。参见 [捕获操作保存的 DAQ 读数](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches)。
> - **`ArraySession` 包含实时帧和服务器端的连续捕获。** `grab()` 和 `frame_stream()` 返回来自一个触发器的 numpy 帧 （与项目 API 中的 `ArrayHandle.grab()` / `frame_stream()` 读取操作相同），而 `continuous()` 会将每个同步组保存到后端，并返回一个 [`ContinuousHandle`](#continuoushandle)。参见 [在您自己的管道中使用实时帧](#live-frames-in-your-own-pipeline)。
> - **数组 `grab()` 被固定到一个触发器上**（`ArrayHandle.grab()` 以及新的 `ArraySession.grab()`）。在 1.2.0 版本中，20 个四- 摄像头 `ArrayHandle.grab()` 的读取数据横跨两个触发器。现在每个帧都包含 `group_id`，而数组 `frame_stream()` 使每个触发器仅产生一次数据。 参见 [每次读取一个触发器](#one-trigger-per-read--group_id)。
> - **按波段和对齐的帧：** `grab()` / `frame_stream()` 采用 `band=`， `align=`、`bit_depth=` 和 `refl_max=` 作为相机和阵列句柄。参见 [导出单个波段](#exporting-a-single-band--band)。
> - **新的 `ArraySession` 方法和参数：** `set_target_fps()`、`force_disconnect()`、`capture(after_seq=..., orient=...)` （配合 `CaptureResult.group_seq` 和 `.failed` 使用）、`continuous()`、`connect_array(packed=...)`；`capture_repeated()` 会将连续的组串联起来，并在驱动满载时停止。
> - **`ArraySession.record()` 在未打开 GUI 的情况下进行记录。** 当 GUI 的组合视图关闭时，后端会自行渲染视图：数组的NDVI，或您作为`expr=`传递的索引。`RecorderHandle.stop()`会在录制失败（即未写入任何帧）时触发`ChlorosConnectError`，并将停止结果保留在`.result` 上保存停止结果（若留有 `with` 代码块则不会抛出异常：请检查 `rec.result`）。参见 [`RecorderHandle`](#recorderhandle)。
> - **新的 `DAQSensorSession` 方法：** `enable_ptp()`、`force_disconnect()`，以及用于 DAQ-E 自身设置的 `device_info()`、`set_network()`、`set_auto_stream()`、`reboot()` 以及 `update_firmware()`。 `record_stop()` 会返回 `warning` 和 `gaps`。如果重启操作关闭了某个录制任务，则会在其响应中命名该录制任务；若重启失败，则 在 `ChlorosConnectError`（`recording_closed`；参见 [DAQ-E 设备设置](#daq-e-device-settings)）中。 参见 [`DAQSensorSession` 方法](#daqsensorsession-methods)。
> - **停止传输的池化 DAQ-E 将由后端重新连接**，并保留其原有设置；该设备上的记录将在同一 `.daq` 中继续进行，文件中会标记出数据缺口； `record_stop()` 将间隙作为 `gaps` 返回。在 1.2.0 版本中，传感器会保持静默状态，直到被断开并重新连接，此时记录才会终止。请参阅 CLI 参考文档中的 [停止传输数据的DAQ-E](cli-reference.md#a-daq-e-that-stops-delivering)。
> - **`open_project()`需基于已安装的SDK运行。** 安装程序（和 PyPI）现已包含所需的项目加载器；此前，`open_project()` 会触发 `ImportError` 异常，并指明 `project_cameras` / `project_sensors`。`connect_all()` 通过正在运行的后端 （光传感器通过其DAQ池）连接已保存的摄像头、数组和光传感器；而对于无法查找的已保存设备，系统会在`errors`下列出具体原因，而非不作说明地直接跳过。对于已在后端（GUI、 或`chloros-cli project connect`的）已连接并保持运行，而此前这会被报告为错误（“摄像头<sn>已存在于数组中<id>”）。参见[`ChlorosProject`方法](#chlorosproject-methods)。
> - **`ChlorosLocal()`、`process_folder()` 和 `process_lattice_capture()` 可在 Windows 上查找 Chloros。** 它们会查找安装程序放置后端的位置（默认是 `<install folder>\backend\`、`C:\Program Files\Chloros\backend\`， 来自安装程序自身记录的自定义文件夹），并直接连接到已运行的后端，而不会进行任何搜索。在 1.2.0 版本中，除非传入 `backend_exe=`，否则它们会在已安装的机器上报错“未找到 Chloros 后端”。
> - **后端在启动时崩溃会立即报告。** 当 `ChlorosLocal()` 或智能连接调用启动了一个后端，而该后端在响应前就退出了， 该调用将在数秒内抛出异常：`ChlorosBackendError` 由 `ChlorosLocal` 引发，而 `ChlorosConnectError` 则由智能-connect 调用中分别抛出 `ChlorosLocal` 和 `ChlorosConnectError` 异常，并附带退出代码、后端执行的命令以及后端输出的最后几行内容。1.2.0 版本会等待 60 秒的启动超时，但不会说明原因。参见 [后端自动启动](#backend-auto-start)。
> - **后端在脚本之间持续运行。** 退出 `with ChlorosLocal(...)` 代码块后， `process_folder()` / `process_lattice_capture()` 返回以及 `chloros-mcp` 退出，不再会停止由 SDK 启动的后端； 下一个脚本会复用该后端。它会在 `logout()` 时停止（除非 Chloros 应用处于打开状态或正在运行），在 `shutdown_backend()` 时停止，或者在后端环境中没有任何进程使用它 30 分钟后自动停止 （后端环境中的 `CHLOROS_BACKEND_IDLE_MIN` 会改变这一行为；`0`：永不）。
> - **由 SDK 启动的后端会被要求关闭，而非被强制终止。** `shutdown_backend()` 允许后端释放其摄像头和光传感器，并完成任何录制；只有在 10 秒后仍处于运行状态时，才会终止该后端。1.2.0 版本会立即终止它 （在 Windows 上，这相当于强制终止），这可能会导致连接到该后端的 DAQ 光传感器无法响应。SDK 发现正在运行的后端仍会被保留。
> - **在 Linux 上，自动启动功能绝不会从当前文件夹运行 `backend_server.py`。** 此前，若未安装后端程序，但在包含后端程序的文件夹中启动脚本时会运行该程序；现在仅启动已安装的后端程序（对于 `ChlorosLocal`，则启动 `backend_exe=` 指定的后端）。
> - **`configure(indices=...)`**会将每个预设添加到其摄像头的波段项目中，并包含一个新项目；若无法添加某个名称，则会发出警告（`UserWarning`）。**`import_images()`** 将 `.daq` 录制文件报告为 `scan_count`，并且不再针对 LATTICE 捕获文件夹发出警告。
> - **`read_image_audit_tags()` 可通过 pip 安装运行。** 它会自行读取图像的元数据；在 1.2.0 版本中，当 Python 路径中不包含桌面安装版的内部组件时，它会返回 `{}`。
> - **紧随 `connect_array()` 之后的 `capture()`** 会等待最多 20 秒以获取数组的首个帧组，而非因 HTTP 503 错误“数组仍在预热中”而失败。
> - **`status()` 报告系统停滞。** 新的 `stalled` 字段在抓取循环运行期间为 True，但不会发布任何数据，随后 `actual_fps` 变为 0。在 1.2.0 版本中，`actual_fps` 在 ，因此停滞的数组会显示 `actual_fps > 0`，而捕获操作则会因 503 错误“无新的同步帧组”而失败。
> - **不完整的帧集和捕获 503 错误减少：** 现在每台从属相机的触发输入都经过了去抖处理。
> - **压缩 12 位传输**（`connect_array(packed="auto")`）将全传感器阵列的帧率提高了约三分之一。参见 [压缩 12 位传输](#packed-12-bit-transport-packed)。
> - **ROI 范围内的阵列曝光：** 较小的 ROI 区域不再将成员的曝光时间限制在传感器的自由运行周期内（768 × 304 分辨率下为 4.9 毫秒）。参见 [阵列成员的曝光](#exposure-on-array-members)。
> - **自动曝光不再降低数组的自动帧率。** 通过电缆同步的阵列以网络和相机读出允许的速率运行；自动曝光会先填满该速率所对应的周期，然后提高增益。如果手动曝光时间超过该周期，系统会减慢触发速度以适应该时间，而不是拒绝该请求。参见 [阵列成员的曝光](#exposure-on-array-members)。
> - **`read_nodes()`**在自动曝光运行期间会添加 `frame_values` 和 `notes`（参见 [自动曝光下的 `read_nodes()`](#read_nodes-under-auto-exposure--frame_values-and-notes))。当相机拒绝某次曝光或增益时，**`set_settings()`**会说明其可接受的范围，并恢复此前因该值而关闭的自动模式； `set_settings(target_brightness_auto=True)` 是新代码。
> - **`orient=`** 在 `ArraySession.capture()` / `capture_fastest()` / `continuous()` / `burst()` 和 `ArrayHandle.capture()` 上 / `capture_smart()` 会根据相机方向旋转产品。请参阅 [`ArraySession` 方法](#arraysession-methods)。
> - **`DLS()` 需要传感器自身的 `mac_address`。** 不再有默认的 MAC 地址（它曾是一个单元的地址）；若未指定 MAC 地址，`DLS()` 将触发 `DLSError`。 参见 [DAQ-M 频段幅度](#daq-m-band-magnitudes--dlsband_magnitudes)。
> - **Windows安装程序**提供“启用Windows长路径”和“将Chloros后端排除在Microsoft Defender扫描之外”选项，两者默认均已勾选。参见[步骤1](#step-1--install-the-chloros-platform-package)。
> - **Linux .deb 包** 会为系统安装 Python SDK（`python3` （此前仅将 wheel 包放置在磁盘上），会增加摄像头流所需的内核套接字接收缓冲区，并添加一个以您的用户身份运行的后端服务。Jetson 软件包为 `chloros_1.3.0_arm64_jp6.deb`，适用于 JetPack 6， 并包含 JetPack 的 CUDA PyTorch。请参阅 [步骤 1](#step-1--install-the-chloros-platform-package) 和 [步骤 2](#linux-deb)。

---

## 60 秒速览架构

该 SDK 是在 Chloros 后端（与桌面 GUI 和 CLI 使用的 Flask 服务器相同）之上构建的轻量级 Python 层。若需实现自动化，请导入 `chloros_sdk` 并调用高级方法；在底层， 每次调用都会转化为发往本地后端（端口 5000）的 HTTP 请求 — `http://127.0.0.1:5000/api/...` （刻意未采用 `localhost`，因为在 Windows 系统上该名称会首先解析为 `::1`，且针对仅支持 IPv4 的后端，每次请求耗时约 2 秒）。 后端管理硬件池——包括相机、数据采集传感器、对齐配置文件和帧缓冲区——因此SDK脚本可以与GUI共存，而无需争夺串行端口或网卡带宽。

您将使用以下三个接口：

1. **`ChlorosLocal` + 自由函数** (`process_folder`, `process_lattice_capture`) — 图像处理管道。通过一次Python调用，即可对整个文件夹执行校准/去拜耳化/索引导出操作。
2. **智能连接控件** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — 为实时硬件建立持久后端会话。与图形界面相同的“智能预处理”流程：网络探测、层级自动选择、PTP、AE 初始化、GPIO 触发配置。
3. **`ChlorosProject` / `open_project`** — 加载已保存的项目（包含 `cameras.json` + `sensors.json` + `project.json` 的文件夹），一次性连接其中的摄像头、阵列和光传感器，并通过命名句柄驱动捕获操作。

如果尚未有后端在监听（即 GUI/CLI 启动的同一捆绑二进制文件），则界面 1 和 2 会 **自动启动本地后端** —— 因此，在全新终端中运行一个简单的脚本即可工作，无需您先启动后端：`ChlorosLocal` （以及 `process_folder()` / `process_lattice_capture()`）、`connect_camera()`、 `connect_array()`、`connect_daq_sensor()`、 `discover_lattice_cameras()` 和 `discover_daq_sensors()`。`list_cameras()`、`list_arrays()`、`list_daq_sensors()`、 `attach_array()` 和 `analyze_array_network()` 永远不会启动：它们需要一个正在运行的后端。传递 `auto_start_backend=False` 以选择退出 （例如，当指向一个远程后端时，该后端永远不会被启动）。参见 [后端自动启动](#backend-auto-start)。Surface 3 的行为有所不同：`open_project()` 不接受 `auto_start_backend` 参数，且 `connect_all()` 绝不会启动后端 ——它会向 `http://127.0.0.1:5000` 进行一次探测，若无人响应，则回退到直接（无后端）的 `lattice_sdk` 设备控制。这需要 Arena SDK 运行时以及 OpenCV （`cv2`）可被Python导入，而SDK wheel 包并未安装这些组件； 若缺少这些组件，每个已保存的摄像头和数组都会被报告为错误，并附带具体原因。项目的光传感器没有无后端路径：若无后端运行，每个已保存的传感器都会被报告为错误，并附带具体原因（参见 [`SensorHandle`](#sensorhandle)）。只有 `proj.process()` 和 `stream(..., overlays=True)` 会延迟构建一个 `ChlorosLocal()`（该组件支持自动启动）。

这三者均需授权：在该机器上运行一次 `chloros-cli login`，或通过桌面图形界面登录。 在没有有效会话的情况下调用该函数会引发什么异常，取决于具体接口（参见 [异常](#exceptions)）：`ChlorosAuthenticationError` 来自 `ChlorosLocal`（以及 `process_folder()` / `process_lattice_capture()`)、来自智能-connect 调用及其会话中，`status: "error"` 与来自 `analyze_array_network()` 的 `error_code` 相关，以及 `connect_all()`报告中的一条记录。

要求：
- Python 3.7+（如包所声明；在 3.10 上开发/测试）
- 本地已安装 Chloros Desktop（后端二进制文件包含在安装程序中）
- 持有 **任何付费 Chloros+ 套餐（Copper 或更高）**的有效 Chloros+ 登录账号；免费**Iron**套餐不支持 SDK/CLI 访问。此限制由**服务器端**强制执行：每个带有SDK/CLI标记的请求必须同时包含有效的会话和付费套餐。低于“Copper”级别的套餐将收到包含`403`和`error_code: PLAN_UPGRADE_REQUIRED`的错误（`ChlorosLicenseError`来自`ChlorosLocal`，smart-connect调用则为`ChlorosConnectError`）。已注销的调用者将获得`401` / `AUTH_REQUIRED`， （`ChlorosAuthenticationError` 来自 `ChlorosLocal`，`ChlorosConnectError` 来自 Smart-Connect 呼叫）——这两者有所不同，因为重新运行 `chloros-cli login` 可修复 `401`，但无法修复 `403`，后者需要升级套餐。
- 在套餐宽限期内支持离线使用：许可层级将从服务器验证缓存（5 分钟）或已签名且与设备绑定的许可缓存（月度套餐为 30 天，年度套餐为订阅到期日）中读取。当宽限期届满后，套餐将转为免费版，且 SDK/CLI 访问将暂停，直至设备能成功连接服务器一次。`chloros-cli status`（`GET /api/license-status`）在免费套餐下仍可访问，因此可查看具体原因。
- Windows 10/11 64 位、**Ubuntu 22.04 LTS 或更高版本**，或 Jetson（JetPack 6）。**不**支持 Ubuntu 20.04： `.deb` 的依赖项源自后端链接的库（包括 `libc6 (>= 2.35)`），而 Focal 系统自带的 glibc 版本为 2.31。

---

## 安装

Python SDK 是在 Chloros 后端之上构建的轻量级 Python 层。除非您在另一台机器上驱动后端（参见 [远程后端模式](#remote-backend-mode-pip-only-host-via-tunnel)），否则您需要在本地安装 **Chloros 桌面软件包** （Windows 安装程序或 Linux `.deb`），其中包含 DAQ 光传感器。该软件包提供了后端二进制文件、适用于 LATTICE 相机的 Arena SDK 运行时以及校准包。

最新下载：[`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### 步骤 1 — 安装 Chloros 平台软件包

#### Windows（.exe）

1. 从下载页面下载 `Chloros Installer 1.3.0.exe`。
2. 运行安装程序并按照向导操作。默认安装路径为 `C:\Program Files\Chloros\`。
3. 至少启动一次 Chloros，并使用您的 Chloros+ 账户登录。

安装程序会允许后端（`chloros-backend.exe`， 仅此而已）通过Windows防火墙：UDP入站（用于摄像头流、PTP、mDNS和DAQ-E流），以及仅限本地子网的TCP入站（用于DAQ-E在固件更新期间与计算机建立的回连） ([`update_firmware()`](#daq-e-device-settings))。卸载程序会移除这两条规则。无论如何，后端的 HTTP API 均保留在 `127.0.0.1` 上。

向导中的**系统设置**组包含两个选项，默认均已勾选：

| 选项 | 功能说明 | 退出静默安装 |
| --- | --- | --- |
| **启用 Windows 长路径** | 允许项目文件夹位于超过 Windows 260 个字符路径限制的更深层级。如果长路径功能处于关闭状态（Windows 默认设置），则在处理任何内容之前，系统会拒绝路径过深的项目文件夹——请参阅 [故障排除](#故障排除提示) | `/MERGETASKS="!longpaths"` |
| **将Chloros后端排除在Microsoft Defender扫描之外** | 阻止Defender对实时扫描。在高捕获速率下，扫描会占用约 30% 的速率：在 9 台摄像头阵列上以 ROI 512 进行测试，连续原始数据捕获时，开启扫描的速率为 52.7 帧/秒，而排除后端时为 77.7 帧/秒。 | `/MERGETASKS="!defenderexclusion"` |

连续捕获会报告 Defender 的负载为 `defender_cores`（参见 [`ContinuousHandle`](#continuoushandle)）。

#### Linux amd64 (.deb)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

请使用 `apt install ./…` 进行安装，而非 `dpkg -i`：apt 还会安装 Python SDK 的依赖项，而该软件包仅推荐而非强制要求这些依赖项（参见 [步骤 2](#linux-deb)）。

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

适用于运行 JetPack 6（L4T R36）的 Jetson Orin / Orin NX / Orin Nano。 其 PyTorch 是 JetPack 6 的 CUDA 构建版本，因此 GPU 处理在 Orin 的 GPU 上运行；它使用 JetPack 6 安装的 CUDA 和 cuDNN 库（在缺少这些库的镜像上，请参见 `sudo apt install nvidia-jetpack`）。

#### Linux：接收缓冲区与后端服务

这两个 .deb 包均将 LATTICE 摄像头流所需的内核套接字接收缓冲区大小提升至 32 MB（`/etc/sysctl.d/60-chloros-gige.conf`）。在内核默认设置下， 摄像头数据包会在内核中被丢弃，导致数组丢失帧和帧率；当缓冲区不足时，`chloros-cli lattice network` 会报告该情况。请参阅 CLI 参考文档中的 [Linux 主机 — 套接字接收缓冲区](cli-reference.md#linux-hosts--socket-receive-buffers)。

在无显示器的机器上，请以 **您的用户身份** 将后端作为服务运行，以便脚本能检测到正在运行的后端（`connect_all()` 不会启动后端）：

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
```

此时后端将以您的身份运行：它写入的捕获数据属于您，并且使用您的 `chloros-cli login`。对于已启用该功能的机器，root 的 `chloros-backend.service` 仍会安装，但 root 后端写入的每个文件都归 root 所有。 请参阅CLI参考文档中的[作为服务运行后端](cli-reference.md#running-the-backend-as-a-service-headless)。

该服务运行的后端是共享的，绝不会被客户端停止：SDK脚本和`chloros-cli`会按其发现的状态使用它 （`ChlorosLocal.shutdown_backend()` 仅会停止由 `ChlorosLocal` 自身启动的后端），而 `GET /api/status` 将该单元命名为 `service`。 请使用 `sudo systemctl restart chloros-backend@$USER.service` 重新启动它。升级软件包会在安装过程中停止后端服务， 因此正在运行的脚本会失去其摄像头和传感器，随后会重新启动所有已启用的设备；卸载该软件包会停止这些服务并禁用已启用的设备。由于同一时间仅能运行一个后端，因此请仅启用一个单元：第二个单元将无法启动并保持 `failed` （退出状态 98，未重启），其 `systemctl status` 会说明原因。若要切换单元，请禁用当前未使用的那个。</id></sn>

### 步骤 2 — 安装 Python SDK

**Chloros 安装程序会随附一个与之匹配的 SDK wheel 包。** 每个 Windows 安装程序和 Linux .deb 包都会在磁盘上放置一个 `chloros_sdk-X.Y.Z-py3-none-any.whl` 文件，其版本与 GUI / CLI / 后端版本完全一致。您无需特意关注 PyPI 来保持版本同步。

#### Windows

安装程序会自动使用您的系统 Python 运行 `pip install` 并加载捆绑的 wheel 包（优先使用 `py.exe` 启动器，若不可用则回退至 `python -m pip`）。无需任何操作——安装成功后，`import chloros_sdk` 即可在您的 Python 环境中运行。如果系统上没有 Python，安装程序会静默跳过此步骤，GUI 和 CLI 仍可正常工作。

#### Linux（.deb）

两个 .deb 包（x86_64 和 Jetson）均会安装适用于系统 `python3` 的 SDK，因此 `import chloros_sdk` 在安装完成后即可运行，无需 pip 且无需网络。这些软件包 （`chloros_sdk`、 `lattice_sdk`、`lattice_cam`、`arena_api` 以及项目加载器）将安装到 `/usr/lib/python3/dist-packages` 中， 而SDK的`chloros-mcp`命令（即[MCP服务器](#mcp-server-ai-assistants)）则位于`/usr/bin`中，所有这些均归该软件包所有：升级时会替换它们，移除软件包时则会将其移除。

SDK 自身的依赖项 `python3-numpy` 和 `python3-requests` 属于 **推荐而非必需**（GUI、CLI 和后端均为自包含组件）： `sudo apt install ./chloros_<version>_amd64.deb`（在 Jetson 上为 `…_arm64_jp6.deb`）会安装这些组件，而 `dpkg -i` 则不会。 安装输出将报告 `python3` 导入的 SDK 版本、任何缺失的依赖项， 以及位于 `python3` 路径下其他位置、会覆盖该软件包副本的 `chloros_sdk`（例如之前通过 `pip install --user` 安装的版本）。

在无法访问系统软件包的虚拟环境中，请将捆绑的 wheel 安装到其中：

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

对于隔离部署的 Jetson，此过程完全离线进行——SDK 和 wheel 已存在于磁盘上。

#### 公共 PyPI

对于仅支持 pip 的主机（未安装 Chloros 桌面包；此时 SDK 会在另一台机器上驱动后端，参见 [远程后端模式](#remote-backend-mode-pip-only-host-via-tunnel)）：

```bash
pip install chloros-sdk
```

PyPI 提供最新稳定版本。请确保 SDK 与它所驱动的 Chloros 后端保持同一版本（参见 [版本控制与兼容性](#versioning--compatibility)）。 DAQ-M 辅助程序 `DLS` 还需 `bleak`：`pip install "chloros-sdk[camera]"`（或 `pip install bleak`）。

#### 验证

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **需要 Chloros+ 订阅。** 所有 SDK 调用都需要有效的 Chloros+ 登录凭据。请在每台机器上运行一次 `chloros-cli login user@example.com 'YourPassword'`；凭据将缓存于 `~/.chloros/` 中。

### 我需要桌面包吗？

对于大多数工作流而言，仅靠 pip 包是**不够的**。以下是每个 SDK 接口所需的组件：

| SDK 接口 | 是否需要桌面包？ | 原因 |
| --- | --- | --- |
| `ChlorosLocal`、`process_folder`、`process_lattice_capture` | **是**（用于启动一个）**/ 否** （后端已可访问） | 使用正在运行的后端（桌面应用的，或另一个已启动的脚本）；否则自动启动已安装的后端：`/usr/lib/chloros/chloros-backend`（Linux）或 `<install folder>\backend\chloros-backend.exe` （Windows；默认使用 `C:\Program Files\Chloros\backend\`，自定义安装文件夹将从安装程序记录中读取）。传递 `backend_exe=` 可自行命名。 |
| `chloros-mcp` ([MCP 服务器](#mcp-server-ai-assistants)) | **是**（启动一个）**/ 否**（后端已可达） | 基于 `ChlorosLocal` 构建，因此与上一行相同。 |
| `read_image_audit_tags` | **否** | 读取已处理图像自身的元数据（仅限 Python 标准库）。 |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **是**（本地）**/ 否**（远程） | 通过后端运行的纯HTTP客户端。本地后端 → 需要桌面包。远程后端 → `backend_url=`**通过隧道**（参见“远程后端模式”——随附的后端仅绑定回环）。 |
| `ChlorosProject` / `open_project` | **是** | 通过后端驱动已保存的项目。 |
| 直接调用 LATTICE 类（`LatticeCamera`、`CameraPool`、`Calibration`、`DLS`、…） | **是** | 需要 Arena SDK 运行时（包含在桌面版软件包中），以及 Python 环境中的 OpenCV (`cv2`)；而桌面版软件包和 SDK 均未安装 (`pip install opencv-python`)。否则，`CAMERA_AVAILABLE` 在导入时即为 `False`。 `DLS` 还需 `bleak`。 |

### 远程后端模式（仅 pip 主机，通过隧道）

> **无法通过局域网访问已安装的后端。** 它仅监听
> 回环端口（`127.0.0.1` / `::1`），因此 `backend_url="http://<lan-ip>:5000"`
> **无法连接到已安装的 Chloros**。 若要驱动另一台
> 机器上的后端，请自行转发其环回端口，并将SDK指向
> 隧道：

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"     # the tunnel endpoint
serials = ["213800234", "214000533"]   # master first

# auto_start_backend=False: the tunnel URL looks local, so without it a tunnel
# that is down would start this machine's own backend, if one is installed.
arr = chloros_sdk.connect_array(serials, backend_url=BACKEND,
                                auto_start_backend=False)
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                     backend_url=BACKEND,
                                     auto_start_backend=False)

# The backend opens every path on ITS machine: pass absolute paths valid there.
# (A relative path would be made absolute against this machine's working folder.)
arr.capture("/home/chloros/captures/run1", processing="raw")
```

无头/CI/机器人主机可以保留一台安装了完整桌面的机器作为“Chloros服务器”，其余所有机器均使用`pip install chloros-sdk`——但它们之间的传输必须通过上述用户自行配置的隧道，而非直接的局域网URL。

> **仅运行 pip 的主机上运行 `ChlorosLocal`。** `ChlorosLocal(backend_url=BACKEND)` 会连接到无需桌面安装即可访问的后端：它仅在必须启动后端时才会搜索本地后端二进制文件（无响应时触发 `auto_start_backend=True`）。在 1.2.0 及更早版本中，它会先搜索二进制文件，即使后端可达，在仅安装 pip 的主机上仍会触发 `ChlorosBackendError` 错误（“未找到 Chloros 后端…”）。您传递的文件夹 (`process_folder`, `import_images`) 指定的文件夹会在该机器上进行检查并由后端读取，因此使用远程后端时，两端必须指向同一个文件夹（例如共享驱动器）。

### 仅数据采集（DAQ）工作流（仅 pip 主机）

仅靠 `pip install chloros-sdk` 无法驱动 DAQ 传感器：DAQ 传感器始终需通过带有 `connect_daq_sensor()` 的 Chloros 后端进行驱动：

- **本机上的传感器：** 在此处安装 Chloros 桌面软件包。若未运行后端，`connect_daq_sensor()` 将自动启动其后端。
- **另一台机器上的传感器：**该机器需要安装桌面软件包；通过 [远程后端模式](#remote-backend-mode-pip-only-host-via-tunnel) 中的隧道访问其后端。后端会在其本机上打开传感器， 因此 DAQ-U 需连接至该机器，DAQ-M 需在其蓝牙覆盖范围内，而 DAQ-E 需位于该机器可访问的网络中。

```python
import chloros_sdk

# http://127.0.0.1:5000 is the local desktop install's backend, or the tunnel.
with chloros_sdk.connect_daq_sensor() as daq:   # smart-detect USB / BLE / ETH
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

无论哪种方式，后端都会强制执行Chloros+方案：在运行后端的机器（`chloros-cli login` 或桌面应用）上使用任何付费套餐（Copper及以上）登录。后端未登录或套餐等级低于“Copper”时，都会触发 `ChlorosConnectError` 错误，并携带后端对应的错误原因（`AUTH_REQUIRED` / `PLAN_UPGRADE_REQUIRED`）。

---

## 快速入门

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## 顶级 API 索引

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by CameraSession.capture / ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job
chloros_sdk.ContinuousHandle                      # handle for an array continuous() capture
chloros_sdk.GrabResult                            # dict returned by grab() / yielded by frame_stream()
chloros_sdk.FrameFetchError                       # raised by grab()/frame_stream() on a refusal

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()
chloros_sdk.daq_bindings() / set_daq_bindings(cameras=, arrays=)   # which DAQ feeds which camera
chloros_sdk.daq_band_presets(filter_code=None)    # band sets for Ambient + Object reflectance
chloros_sdk.light_sensor_reflectance(ambient_id, object_id, ...)   # two DAQs -> reflectance + indices
chloros_sdk.time_sync_status() / time_sync_peers() / time_sync_cameras()   # PTP grandmaster
chloros_sdk.restart_time_sync() / set_time_sync_priority(priority1=, priority2=)

# === Camera utilities (the CLI's lattice commands; by serial) ===
chloros_sdk.camera_color_profile(serial) / set_camera_color_profile(serial, ...)
chloros_sdk.camera_color_adjust(serial) / set_camera_color_adjust(serial, ...)
chloros_sdk.camera_highlights(serial) / set_camera_highlights(serial, value)
chloros_sdk.camera_white_balance_once(serial) / set_camera_white_balance(serial, ...)
chloros_sdk.camera_presets() / apply_camera_preset(serial, preset)
chloros_sdk.camera_filter_models() / camera_filter_model_info(name)
chloros_sdk.camera_firmware_status() / update_camera_firmware(serial)
chloros_sdk.camera_power_probe(serial) / sleep_camera(serial) / wake_camera(serial)
chloros_sdk.calibrate_camera_with_target(serial, ...)
chloros_sdk.network_adapters(...) / fix_network_settings()
chloros_sdk.apply_vignette_correction(input_dir, output_dir, ...)
chloros_sdk.read_dls(mac, ...) / probe_system(...) / system_info()

# === Transient cameras, alignment profiles, indices (cameras opened for one command) ===
chloros_sdk.start_camera_continuous(serial, ..., daq=None, daq_tolerance_ms=None)  # → CliJob; a DAQ bound to the camera records one .daq per frame (daq=False: none)
chloros_sdk.start_multi_camera_stream(serials, ...)   # → CliJob
chloros_sdk.capture_cameras_transient(serials, ...)
chloros_sdk.test_array_sync(serials, count, ptp_settle_s=None) / detect_array_master(line) / analyze_array_models(models)
chloros_sdk.calibrate_alignment_profile(serials, save_path, ...)
chloros_sdk.capture_aligned_multiband(profile_path, ...) / start_aligned_stream(profile_path, ...)  # → dict / CliJob
chloros_sdk.alignment_profile_info(path) / reorder_alignment_profile(path, ...)
chloros_sdk.index_catalog() / compute_index(input_path, ...) / capture_live_index(profile_path, ...)
chloros_sdk.CliJob                                # stats() / stop() / wait()

# === Project lifecycle ===
chloros_sdk.open_project(path, relink=None)       # → ChlorosProject
chloros_sdk.project_folder() / set_project_folder(path) / reset_project_folder()
chloros_sdk.run_project_recipe(recipe, project_path)  # `chloros-cli project run`
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics               # DLS: DAQ-M over Bluetooth; needs bleak

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosProjectFilesMissingError      # a ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True when chloros_sdk.project imported; open_project()
                                 # still raises ImportError without the project loaders
```

---

## 图像处理 — `ChlorosLocal`

核心管道类。在构造时启动后端（如果无后端响应且 `auto_start_backend=True`），创建/配置项目，监控进度，并返回运行后摘要。

### 构造函数

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### 方法

| 方法 | 描述 |
| --- | --- |
| `create_project(project_name, camera=None)` | 创建一个新项目（可选地使用相机模板，如 `"Survey3N_RGN"`）。 |
| `open_project(project_path, relink=None)` | 打开已保存的项目（其文件夹，或项目文件夹中的项目名称），并将其设为当前打开的项目。 已移动项目的路径将重新指向其当前所在的文件夹；`relink` 指定其他缺失文件当前所在的文件夹。通过 `source_files` 返回后端的响应； 若无法找到任何文件，则抛出 `ChlorosProjectFilesMissingError` 异常。参见下文的 **已移动的项目**。 |
| `import_images(folder_path, recursive=False)` | 导入 RAW/TIF/JPG/DNG 图像 **以及 `.daq` 光传感器记录**。返回 `count` （图像）和 `scan_count`（记录）。仅当文件夹中既无图像也无记录时才会发出警告。 |
| `export_light_sensor(daq=True, csv=True)` | 针对项目中的每条光传感器记录，写入经过校准的 `.daq` + `.csv`， 并将其写入 `<project>/Light Sensor/`。参见 [光传感器记录](#light-sensor-recordings--calibrated-daq--csv)。 |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | 设置处理参数。 `indices` 是预设名称（`NDVI`、`GNDVI`、 `OSAVI`, …)，每个名称均根据其摄像头的波段添加到项目中；若名称不是预设名称，或该名称对应的波段组合无法由现有摄像头提供（例如 `NDRE` 需要 RE 和 NIR 波段，而单台摄像头无法同时提供这两种波段），则会生成一个名为 `UserWarning` 的文件，且不包含索引图像。`daq_log_path` 用于命名光传感器记录 （即 `.daq` 或 `.csv` 文件，或包含此类文件的文件夹），LATTICE 和 Survey3 会将每帧图像与该记录进行匹配，而非使用随图像导入的记录； 若指定路径不存在或未包含任何记录，则使用导入的记录。 |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | 运行处理流程。返回 `{"status": "complete", "async": False}`， 以及当后端提供时生成的 `summary` 键——参见 [运行后摘要与提示](#post-run-summary--hints) ——当无法验证运行的导出结果时返回 `export_check_failure`，丢失图像时返回 `image_failure`。`wait=False` 会同时返回 `{"status": "started", "async": True}`。 |
| `get_config()` / `get_status()` / `status()` | 检查后端状态。 |
| `logout()` | 注销：清除缓存的凭据并释放该机器的设备槽位（结果中显示为 `device_deregistration`）。 正在进行的处理运行不会被停止：它将以启动时的模式完成（结果中显示为 `run_in_progress: true` 和 `run_mode`），而下一次运行将采用标准处理流程。 |
| `shutdown_backend()` | 如果该实例启动了后端，则停止该后端；若发现后端正在运行，则不予干预。遇到 `with` 代码块或该实例被垃圾回收时，不会停止后端： 后端会继续运行以处理下一个脚本，并在无人使用的情况下于 30 分钟后自动停止。系统会要求后端关闭，因此它会先释放其摄像头和光传感器，并完成任何录制任务； 只有在 10 秒后仍处于运行状态时，才会终止该后端。 |
| `discover_cameras()` | **通过该实例的后端**发现 LATTICE 摄像头 (`/api/camera/discover`)。返回一个字典列表（`serial`、`model`、 `ip`, …) — 与 GUI/CLI 所见结构相同。若未发现任何摄像头或后端不可达，则返回空列表。 |
| `camera_capture(output_dir, format="tiff", **settings)` |**通过后端**捕获单帧图像（由该句柄自动启动），使其获得与 GUI/CLI 相同的预处理设置（默认 12 位，池资源复用，嵌入校准元数据）。使用 `serial=` 或 `device_index=` 解析目标； 将 `exposure`/`gain`/`pixel_format`/`preset` 作为 `**settings` 传递。返回旧版元数据字典（`filepath`、`width`、 `height`、`pixel_format`、`exposure_time`、`gain`、`timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | 从聚合摄像头生成叠加合成预览帧——通过后端 `/api/camera/<serial>/stream-annotated` 路径的轻量级 MJPEG 客户端 （服务器端绘制的斑马线/网格/十字线/直方图/峰值/光斑）。`decode=True` 返回 BGR 数组，需要 OpenCV（`cv2`），而SDK未安装该库；`False`返回原始JPEG字节。也可按项目通过`ChlorosProject.stream(overlays=True)`访问。 |

**正在运行的进程会保留其项目。** 在处理运行进行期间——无论是本脚本、应用程序、`chloros-cli process` 还是 `chloros-mcp` 的运行——后端均拒绝创建、打开、导入项目或更改项目的处理设置 （HTTP 409 `PROCESSING_IN_PROGRESS`）：`create_project()` 和 `import_images()` 会触发 `ChlorosProcessingError`， 而 `configure()` 会触发 `ChlorosConfigurationError`，此外，当 `process()` 的模式与当前运行模式不同时，会触发 `ChlorosProcessingError` （一次运行会保留其启动时的模式），每次都附带后端的消息。保存到项目中的捕获会在运行结束时加入该项目（其回复中的 `project_add` 如此说明）。等待运行结束（`get_status()`），然后继续。**已移动的项目。** 项目的 `project.json` 会根据完整路径为每个源文件（每张图像和光传感器记录）命名，因此移动项目文件夹——或将整个项目文件夹移至另一驱动器——会导致这些文件仍保留旧位置的路径。 每次打开项目时，系统会自动修复可修复的部分：原本位于项目文件夹内的文件会在项目当前所在的文件夹中被查找，并重新指向该文件夹中找到的每个路径，然后保存 （`project.json` 会先被备份为 `project.json.before-relink-<date>-<time>`）。路径重定向仅针对实际存在的文件进行。 位于项目文件夹之外的文件——例如 Hub 会话或“添加文件夹”导入的文件——仅能通过 `relink=<the folder they are in now>`（或其上级的任何文件夹）找到。`open_project()` 返回 `source_files`： `total`、`found`、`missing`、`missing_roots`（缺失文件所在的文件夹）， `missing_outside_project` 和 `relink`（被重新指向的文件，从何处指向何处）。包含部分缺失文件的项目会与其余文件一同打开；而未找到任何文件的项目则无法打开， 而 `open_project()` 会触发 `ChlorosProjectFilesMissingError`（`.missing_roots`、`.audit`； HTTP 409 `PROJECT_SOURCES_MISSING`）。CLI中的`project open --relink`和`chloros-mcp`'`open_project`（`relink_folder`）也会产生同样的结果。

用作上下文管理器以确保清理：

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### 光传感器记录 — 已校准的 `.daq` + `.csv`

DAQ-U / DAQ-M / DAQ-E 可以在**不**使用其校准包的情况下进行记录。这正是
公开的 [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
记录器（`record_daq.py`）默认的做法：它们写入原始传感器计数数据，并在
文件中添加时间戳，以便 Chloros 首先从本地缓存中获取该传感器的出厂校准数据 **按序列号** ——
，随后从MAPIR云端获取——并在导入时应用校准。

Chloros将结果写回，每条记录生成两个产品，位于
`<project>/Light Sensor/`下：

| 产品 | 内容说明 |
| --- | --- |
| `<name>_calibrated.daq` | 可重新处理的存档文件——与实时记录采用相同的架构，但现在声明了生成该文件的软件包。重新导入该文件时，**不会**进行第二次校准。 |
| `<name>_calibrated.csv` | 基于传感器自身波长网格的谱辐照度（单位：W/m²/nm），每行对应一个读数，外加光度学列（总功率、明视/暗视勒克斯、PPFD 及其蓝/绿/红分量、峰值波长）。 |
| `<name>_raw.daq` / `<name>_raw.csv` | **仅限无捆绑包的传感器（DAQ-A）.** 原始光谱传感器计数值 — *非*辐照度。详见下文。 |

`<name>` 是记录本身的文件名。位于不同文件夹中的两个记录可以共享
一个——由两台相机在
同一毫秒内捕获的读数（`<capture>/daq/daq_data_<sensor id>_<ts>.daq`）——且各自仍会生成独立的处理结果：第一个保留
`<name>`，而下一条则在文件名**之后**添加区分其所属文件夹的后缀
(`<name>__<capture>_daq_calibrated.daq`)，因此记录的日期和时间仍位于首位。

`process()`将此导出作为其工作流的其中一个阶段。它**不需要**图像：
单独搭载的光传感器本身就是一种一流的工作流，而此类项目按设计
本身就不包含任何图像。

**DAQ-A 记录以原始计数形式导出。** DAQ-A 系列早于按序列号
分拣的捆绑系统，因此无需获取任何捆绑包——它是在现场通过
反射率标靶进行校准的，这就是它从未需要捆绑包的原因。 这些记录
以 `_raw` 为前缀导出，而非 `_calibrated`：采用不同的文件名而非文件内的标记，
因为该声明必须在作为纯文件名通过电子邮件转发时保持有效。
`.csv` 文件头中显示的是 `raw spectral sensor counts (NOT irradiance)`，并提示这些
数值仅在**同一**文件内可比——这正是基于目标校准所
的用途——而非跨传感器比较。与功率相关的光度学列（总功率、
明视/暗视勒克斯、PPFD）返回**NULL**，而非根据计数进行积分计算。

对于无法获取数据包的 DAQ-U / DAQ-M / DAQ-E，系统仍会**跳过**该设备，
而非写入原始数据：这种情况下数据包确实存在，“重新连接并重新处理”是切实可行的建议。

旧版 **v1.01 / v1.02** 记录（由 DAQ-A-SD 写入）不包含每次读数的纪元，
仅包含文件的写入时间。图像↔下行光匹配器仍会拒绝这些数据——将
帧与写入时间进行匹配会导致隐性错误——但导出工具会读取这些数据，且
生成的 CSV 文件会打印 `clock=daq_created_on`，因此该产品会标明所采用的时钟类型。

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

若无法获取某条记录的校准包（离线状态，或传感器无
文件校准），系统将以 `skipped` 的形式报告该情况，并附带**原因说明**。 该记录绝不会
作为包含原始计数数据的“已校准”文件写出——请连接互联网并
重新运行，导出操作即可完成。

### 进度回调

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### 运行后摘要与提示

完成后，`process()` 会获取 `GET /api/processing-summary` 并将正文作为 `result["summary"]` 附加。该获取操作仅尽最大努力，绝不会阻塞成功返回 ——如果摘要不可用，`process()` 将回退到普通的 `{"status": "complete", "async": False}` 结构。`summary["hints"]` 中的每一条条目——包含建议补救措施的完整句子， 例如为何某次运行产生了零输出——也会作为Python格式（`UserWarning`）重新输出，因此即使您从未检查过该字典，零输出的运行也能实现自我诊断：

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` 是机器可读的部分：

| 键 | 统计内容 |
| --- | --- |
| `models` | 运行中的相机组。 |
| `images_in_groups` | 这些组中的源图像。 |
| `targets_found` | 检测到的反射率目标。 |
| `images_calibrated` | 该运行任务生成反射率产品文件的图像。 |
| `images_failed` | 处理失败且未导出的图像。 |
| `exported_files` | **该运行任务生成的图像产品文件**：该运行生成的所有产品（去拜耳化、预览、辐射度、反射率、索引、目标渲染），涵盖所有格式。 |
| `reflectance_files` | 其中反射率产品 (`Reflectance_Calibrated_Images`)，无论是由目标还是光传感器（DAQ）的辐照度进行校准的。如果没有目标但仍写入了反射率数据，运行提示中会注明；只有在未写入任何反射率数据的情况下，系统才会提示反射率输出缺失。 |
| `clamp_masks_written` | 其旁标注的 [解混钳位掩膜](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on）——这些是辅助信息而非最终结果，因此不会出现在 `exported_files` 中（CLI 同样将它们单独计数）。 |
| `captures_left_out` | 项目中位于 `captures` 文件夹内、本次运行未捕获的原始帧。 每次运行开始时都会等待前一次捕获的数据，因此除非在运行期间保存了捕获数据或等待超时，否则此处为 0。`summary["captures_left_out_note"]` 说明了如何处理这些数据。 |
| `daq_recordings_exported` / `daq_recordings_skipped` | 光传感器记录，特意单独计数 ——它们来自不同的阶段，且存在于完全没有图像的运行中，因此若将其合并，会导致仅进行数据采集（DAQ）的运行看起来像是导出了图像。 |

此外还有：`summary["output_dirs"]`（写入过的每个目录），
`summary["light_sensor_export"]`、`summary["stopped"]`（当用户中断
运行时为真， 因此部分计数不会被误判为产出不足的已完成运行），以及
`summary["groups"]`（按组细分）。

`exported_files` 由管道在**写入时**记录，而非事后从
项目的映像对象中扫描获取。并行和 GPU 策略会构建各自的映像
对象（在 GPU 路径的工作子进程中），因此旧版扫描会为
`0 file(s) written`，随后会发出零导出提示——而这些运行
实际上一切正常。如果您根据该数字编写脚本，现在正常的并行运行
会报告非零计数。

“轻量级传感器”跳过报告会显示读取器针对每个文件实际确定的原因——
不可读的模式、缺失的捆绑包、写入错误——这些原因会经过**去重处理**，因此因同一原因被跳过的二十个文件
会被归为一个原因，而不是重复显示二十次。

> **当运行未生成任何图像时，`process()` 不会触发。** 这是 SDK 和
> CLI 刻意设计不同的两个方面之一：`chloros-cli process` 将“请求了产品，但未
> 写入任何产品”视为失败并以非零状态退出，而 SDK 则正常返回并通过
> 通过 `summary` / hints 报告该状况。如果您的管道在空运行时应停止，请自行检查
> ——检查 `summary` （或统计项目文件夹下的文件数量），而不是依赖于
> 是否抛出异常。常见原因包括：输入文件夹未被识别为
> 捕获源，以及某些产物因不适用于当前摄像头而被跳过（例如仅支持RGB的
> 摄像头生成的辐射度数据）。
>
> 当无法验证运行的导出结果时，`process()` 也不会触发：它会发出警告并设置
> `result["export_check_failure"]`。生成文件仍会被写入。若要在此情况下停止，请检查该键值。 一个
> 丢失图像的运行情况也是如此：它会发出警告并设置 `result["image_failure"]`，而 `chloros-cli process`
> 在该标志非零时会退出。

### 便利函数

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "GNDVI", "OSAVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# process_folder() / process_lattice_capture() also take input_level=,
# array_alignment=, array_alignment_crop=, array_alignment_interpolation=,
# progress_callback=, poll_interval=, timeout=, processing_stuck_timeout=,
# backend_exe= and api_url= (NOT backend_url=). indices defaults to ["NDVI"].

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

每个运行都在其自身的 `with ChlorosLocal(...)` 中进行；它必须启动的后端在返回时仍会继续运行， 以便为下一次调用做好准备，并在 30 分钟内无人使用时自动停止。这两个函数均不接受 `auto_start_backend`、`radiometric_output` 或 `daq_log_path`；请使用 `ChlorosLocal`和`configure()`来处理上述情况。 `process_lattice_capture()` 不接受 `camera`、`debayer` 或 `ppk`：它会跳过面板检测并使用标准去拜耳算法。

### 支持的值

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"TIFF (8-bit)"            # all-8-bit images are written as this; never raised
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer for Survey3 imagery, Chloros+ only.
                                         # Survey3 only: on a project holding LATTICE captures
                                         # (which always use the standard demosaic) configure()
                                         # or process() raises ChlorosConfigurationError; a
                                         # Texture Aware value the project saved earlier is set
                                         # to Standard with a UserWarning, as the app does. Needs a
                                         # CUDA GPU the backend's PyTorch can use (an NVIDIA
                                         # GPU, or a Jetson Orin's). Without one, process()
                                         # uses the standard debayer, and the backend prints
                                         # one "[DEBAYER] ... needs a CUDA GPU" line saying why.
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as Chloros's saved debayered (3-channel) product
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### 辐射度输出（LATTICE 多光谱处理管道）

`process` 处理流程的 LATTICE 多光谱（M3C/M3M）导出级别——`reflectance`（默认）， `radiance`、`sensor-response` 或 `all`（每张图像的每个适用模式）——用于设置项目中**“导出辐射度”**和**“导出反射率”** 产品：`radiance` 仅导出辐射度，`reflectance` 仅导出反射率，`sensor-response` 不导出二者（仅导出去拜耳化及预览产品），`all` 同时导出两者。去拜耳化和预览产品不在该级别可修改范围内。`configure()` 对此设有专用关键字：

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

高级应急方案——通过 `custom_settings` 写入产品——仍然有效，但请记住它会替换整个设置块 （参见下方的警告）。写入 `"Export <product>"` 键：项目自身的 `"Export radiance"` / `"Export reflectance"` 设置将覆盖较旧的 `"Radiometric output"` 键，因此仅该键本身不会产生任何变化。

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Export radiance": True, "Export reflectance": False},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` （默认）将摄像机辐射度除以**与时间戳匹配的 DAQ 下行辐射**，该值由记录的 `.daq`（DAQ-U/M/E）**或与影像一同发现的 DAQ-M 原生 `.csv`**；任何本地缺失的单相机或 DAQ 校准包，将在首次使用时**从 AWS 自动获取**。CLI 根据`chloros-cli process`：`--radiance`/`--no-radiance`、`--reflectance`/`--no-reflectance`、`--debayered`、`--preview`。**这是谁的读数。**捕获数据中保存的 `.daq` 代表其所描述的相机（或多台相机）的读数——这些相机在旁边的 `<file>.daq.cameras.json` 辅助文件中命名 （单帧、同步及数组捕获；原始连拍会在其清单中列出每个成员的读数）——且**仅与来自这些摄像头的图像**匹配。因此，没有自身读数的独立摄像头将不再借用其他摄像头的数据采集（DAQ）；需将其与数据采集绑定，或进行录制。数据采集自身的录制 （`chloros-cli daq pool-record`，或在GUI中启动的录制），或任何不包含该sidecar文件的`.daq`（包括较早的捕获数据），仍可服务于所有相机。**阳光下的OCN相机。** 每个OCN校准包都是在窄带光下构建的（每波段一个LED），而阳光会混合各波段。Chloros 1.3.0在波段解混后应用了基于阳光的批量校正，这改善了阳光下OCN的辐射度、反射率和折射率产品。 对于支持该功能的OCN校准包，该校正将在所有进行波段分离的场景中生效——包括`process()`、捕获数据、`grab()` / `frame_stream()`以及实时预览；RGN相机则保持不变。 使用早期版本拍摄的OCN图像，若在存储了其拍摄时所用校准数据的计算机上进行处理（Chloros在相机连接后会保留其校准数据），也会应用此功能；否则仍按原有方式处理。因此，OCN的辐射度、反射率及指数值将与1.2.0版本不同。

> `custom_settings` **将**整个计算设置块替换掉 （按设计，它会绕过 `configure()` 的其他关键字和验证）。使用时，请包含您关心的所有 `Project Settings` 关键字，如上例所示。

#### 读取指数像素 （NDVI 及其他植被指数）

**整数指数栅格是对每幅图像进行的对比度拉伸处理，而非 −1…1 映射。** `TIFF (16-bit)`、`TIFF (8-bit)`、 `PNG (8-bit)` 和 `JPG (8-bit)` 指数产品——包括 8 位TIFF（8 位捕获数据以该格式写入）——存储

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

其中 `IndexMin` / `IndexMax` 是 **该帧自身的最低和最高有限索引值**。 DN 0 并非 −1，满量程也并非 +1。

Chloros（Chloros）会在两个端点上都打上标记，以确保映射保持可逆：

| 标签 | 含义 |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — 如下所示的约定 |
| `Chloros:IndexMin` | 作为 DN 0 存储的索引值 |
| `Chloros:IndexMax` | 作为全量程存储的索引值 |

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

读取时无需 `-config pix4d.config` —— exiftool 可以自行读取这些 XMP 属性。该配置仅在*写入*时才需要。

> **`IndexMin == IndexMax`** 表示该帧的每个像素都具有相同的索引值。 栅格填充至中等范围，且无需除以任何值——直接取端点值即可。

> **若无 `Chloros:Index*` 标签，则表示未声明映射关系。** LUT 导出是一种彩色渲染，其像素为根据您的 LUT 阈值窗口生成的渐变色；而没有有限索引像素的帧则没有范围可言。无论哪种情况，都不要假设存在映射关系——这与缺少 `PixelScale` 的情况遵循相同的规则。**由于拉伸是按图像进行的，因此同一飞行任务中的两帧图像在数字数值（DN）上不可比**——每帧都有其自身的端点，因此相同的NDVI值在不同图像中对应的DN值会不同。若需对整个飞行任务进行定量分析（如分区统计、固定阈值、时间序列），请导出**`TIFF (32-bit, Percent)`**：索引栅格将以 float32 格式写入其中，即**索引值本身**，无需重新缩放且无需标签。尽管格式名称如此，索引产品并非 ×100 —— 该缩放仅适用于整数数据类型的反射率。

#### 解混剪切掩膜 — 某个波段在哪些像素上被剪切

3×3 光谱解混可能会导致某个波段变为负值 — 例如当光线几乎不包含该波段，或者场景超出相机解混矩阵描述的范围时。这并非真实值，因此导出时将其固定为 0 ——实时预览中也会显示相同的 0。一个波段可能仅在少数像素上被钳位，也可能在几乎所有像素上被钳位；钳位掩码会记录这两种情况。`Chloros:Band{R,G,B}UnmixClampFraction` 表示该现象在帧中占多大比例；而钳位掩码则指出**具体是哪些像素**：

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— 位于产品文件夹**旁边的** `clamp_masks/` 文件夹中， 绝不位于其内部（`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`）——适用于每个至少有一个像素被截断的辐射率、反射率和指数产品。 单波段 `uint8`，基于该产品自身的像素网格（包含方向、 ROI、像素合并及数组对齐变形均已包含）。每个掩码包含其对应产品的相机名称（`Make` / `Model` / 序列号）、拍摄时间及镜头标签，并列出了其位所代表的波段 （`XMP-Camera:BandName`，以第 0 位为首）；其 `ImageDescription` 和 `Software` 标记表明其为钳位掩码，且不包含校准信息。
- **位**：第 0 位 = 产品 R 通道中的波段（`Chloros:BandRName`），第 1 位 = G（`BandGName`），第 2 位 = B (`BandBName`)。位被设置表示该通道中该像素的值已被钳位为 0 —— 请将其以及由此构建的任何索引视为低可信度。
- **`Chloros:UnmixClampMask`** 标记在每个运行过解混操作的产品上：该标记表示相对于产品自身文件夹的蒙版路径（`../clamp_masks/<product folder>/<product name>_clampmask.tif`，始终以 `/`分隔——将其与图像文件夹拼接即可打开蒙版），或在未对像素进行钳位时为 `none`。若无该标记，则表示该产品从未运行过解混操作（RGB或单色相机，raw/去拜耳化处理的色阶）。
- 捕获结果将其作为帧的 **`clamp_mask`** 携带（即蒙版的路径，若无像素被截断则为 `None`） — 在 `CameraSession.capture()` 结果中，`ArraySession.capture()`'`CaptureResult`帧以及HTTP捕获响应中。离线构建将其蒙版列为`clamp_masks`。
- 实时索引的多波段堆栈（`/api/camera/cli-array-index` 与 `save_multiband`）为每台源摄像机生成一个蒙版，位于输出文件夹内的 `clamp_masks/` 中——请参阅 CLI 参考'中的 `lattice index`。视频不包含遮罩：离线构建的结果会报告每帧的钳位比例。

由于任何层级的产物文件夹中都不包含遮罩，因此可以将产物文件夹（包括其子文件夹）原样传递给摄影测量软件——该软件仅处理图像。请直接提供产物文件夹， 而非其上级文件夹。Chloros 自身的导入功能会跳过遮罩。

```python
import os, numpy as np, tifffile

mask = tifffile.imread(
    "clamp_masks/radiance/capture_<ts>_SN<serial>-<filter>_radiance_clampmask.tif")
# or, from the image: os.path.join(os.path.dirname(image_path), tag), with
# tag = the image's Chloros:UnmixClampMask
r_clamped = (mask & 1) != 0      # the band Chloros:BandRName names
g_clamped = (mask & 2) != 0
b_clamped = (mask & 4) != 0
```

---

## 适用于 LATTICE 相机的 Smart-Connect

针对实时硬件的持久后端会话。使用与 GUI 相同的端点，因此 SDK、CLI 和 GUI 之间的行为完全一致。

### 单台相机 — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

此处设置的曝光、增益和目标值将作为您的设置保留：Chloros 的自动曝光控制（包括 DAQ 测量的曝光）不会更改这些设置。`cam.set_settings(exposure_auto="Continuous")`（或 `gain_auto=`）将曝光（增益）交还， 而`cam.set_settings(target_brightness_auto=True)`将目标参数交还给Chloros的自动目标控制。

#### `connect_camera()` 签名

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

- **返回时数据流已启动。** 后端会进行等待，且有时间限制（约 15 秒等待流启动，随后 10 秒等待其首个完整缓冲区——而非首个预览帧，后者约在 1.5 秒后出现）， 因此紧接着 `status()` 之后便是 `streaming`。该会话的 `connect_stream` 即为响应中的 `stream` 数据块： `state` `streaming`, `no_frames`（数据流已启动但未传输任何数据 ——`note` 已明确指出这一点；对于使用巨型数据包的摄像头，需以 1500 字节的数据包大小重新连接），`starting` 或 `start_failed`，其中 `waited_s`。`None` 用于已连接的摄像头。预览 （`get_camera_preview`、`/api/camera/jpeg/<serial>`）在等待（上限为4秒）摄像头发送首个预览帧后才予以响应，而非直接返回 “尚无帧可用”的响应；而流尚未传输任何数据的摄像头（`no_frames`，或已为硬件触发器做好准备）仍会立即响应。
- **数据包大小由您自行设定。** `settings={"gev_packet_size": 1500, "gev_packet_size_auto": False}` 以 1500 字节的数据包打开摄像头，未进行探测（`chloros-cli lattice cam-connect --packet-size standard` 发送相同数据包）。
- **每次仅建立一个连接以保持摄像头池可用**：连接建立、包大小探测、首次校准数据下载（对于该计算机首次连接的摄像头约需 70 秒），以及流传输开始。若此时有请求需要使用摄像头（`read_nodes`， `set_settings`、捕获操作、第二次连接）将最多等待5秒，随后返回HTTP 503错误（`CAMERA_POOL_BUSY`），并指明正在连接的摄像头（`ChlorosConnectError`）；稍后 片刻后再试。列表项从不等待：`list_cameras()`（`GET /api/camera/pool`）、`list_arrays()` 和 `CameraSession.status()` 会立即响应。 连接中的摄像头被列在池响应的 `connecting` 下 (`serial`、`phase` — `opening the camera`、`rebalancing the pool bandwidth`、 `loading its calibration`、`starting its stream`、`waiting for its stream` — 以及 `elapsed_s`)，其 `status()` 显示 `connecting: True`，而 `connecting: True` 又显示 `phase`； 其余摄像头在资源池空闲时进行实时读取，否则返回上次读取的结果（`live: False`）。

#### `CameraSession` 方法

| 方法 | 描述 |
| --- | --- |
| `status(timeout=15.0)` | 摄像头的实时状态，由 `chloros-cli lattice status` 读取： `streaming`、`stream_state`（`starting`、 `no_frames`、`streaming`、`start_failed` 或 `stopped`）、`device`（型号、序列号、固件版本、温度、`is_mono`）及其当前设置。 |
| `read_nodes(names, enum_names=(), timeout=30.0)` | 读取 GenICam 节点；返回 `{nodes, errors, enums, device, was_in_pool, in_array_id}`， 以及 `frame_values` 和 `notes`（在自动曝光运行期间，详见下文）。 |
| `set_settings(**kwargs)` | 按友好名称写入节点（`exposure_time`、`gain`、 `pixel_format`、`width`、`height`、 `target_brightness`、`target_brightness_auto`、`ae_damping`、`ae_upper_limit`、`trigger_mode`、 `trigger_source`, …）。`exposure_time`（`gain`）为手动曝光（增益）：若未标注`exposure_auto` (`gain_auto`) 时，需先关闭自动模式，以免该值被正在运行的自动模式拒绝。生成的 `applied` 保存了相机实际拍摄的参数； 被相机拒绝的写入操作记录在 `errors` 中，且不会被保留。该错误指明了相机当前接受的范围（`ExposureTime` 的最大值取决于相机的帧率）， 且仅针对被拒绝的该值而关闭的自动模式将被恢复； `restored` 列出了该范围（例如 `{"ExposureAuto": "Continuous"}`）。 |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, daq=None, daq_tolerance_ms=None, settings=None, timeout=None)` | 捕获 **单帧** 帧。返回一个 `CaptureResult`：包含帧元数据字典的列表（每个保存级别一个），其中包含 `.skipped`、`.failed`、 `.daq_files` 和 `.daq_matches`。（连拍/多帧捕获功能已被移除——如需捕获系列帧，请在循环中调用 `capture()`。） |
| `capture_repeated(output_dir="output", *, count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | 来自该摄像头的连续（`interval_s=0`）或间隔捕获， 需配合 `count` 捕获和/或 `duration_s`（至少一个，以便结束），如同 `ArraySession.capture_repeated()` 处理数组那样。 返回 `CaptureResult` 的列表；当捕获报告硬盘已满后，该操作即停止。`on_capture(i, result)` 在每次捕获后运行；其他关键字则转至 `capture()`。 |
| `disconnect(keep_settings=False, clear_settings=False)` | 从池中释放，将相机恢复至连接时的初始状态（触发器接线、PTP、设置）；`keep_settings=True`则将其设置保存至自身内存中，`clear_settings=True`则重置该内存（参见 [相机内存](#camera-memory--keep_settings--clear_settings))；当被请求执行上述任一操作时，返回 `camera_memory`。作为数组成员的相机将释放整个数组。若连接到已打开的会话，则不执行任何操作。 当后端确认释放后，或无内容可释放时（摄像头已不存在，或无后端正在运行）；若释放超时或被后端拒绝，则抛出 `ChlorosConnectError` 异常，因为摄像头可能仍被占用，且下一次连接会与其发生冲突。`with` 代码块的退出过程绝不会抛出异常。 |

属性：`serial`、`device`（摄像头的发现记录）和 `backend_url`。

`capture()` 导出控制（与数组 + GUI 相同模型）：

- `processing` / `levels` — `processing="all"` 保存所有适用的导出类型；`levels=["raw","radiance"]` 仅保存其中部分（覆盖 `processing`）。若省略这两项，则采用后端默认设置。
- `daq` — 用于读取捕获数据的 DAQ 默认会以 `.daq` 旁路文件的形式保存，其中包含仅原始数据的抓取内容，以便后续可将帧重新处理为反射率/折射率数据。 当没有任何级别使用它时，`daq=False` 会跳过它。`force_daq=True` 是默认值的旧拼写形式。无 DAQ， 不支持 `.daq`。
- `daq_tolerance_ms` — 读数与曝光时间之间的最大允许偏差（默认 500）。仅判定结果会发生变化。

#### 自动曝光下的 `read_nodes()` — `frame_values` 和 `notes`

在 `ExposureAuto` (`GainAuto`) 运行期间，`ExposureTime` (`Gain`) 节点会读回**上次写入的值**，而非相机当前的实际状态。`nodes` 仍保留该节点的值。 当您在自动模式并非 `Off` 的已连接相机上查询 `ExposureTime` 或 `Gain` 时，响应中会额外包含：

- `frame_values` — 摄像机最新帧携带的值，以该节点的名称为标识（`ExposureTime` 以微秒为单位，`Gain` 以分贝为单位）；
- `notes` — 针对此类节点，每行包含两个值的描述。

```python
r = cam.read_nodes(["ExposureTime", "Gain"])
print(r["nodes"]["ExposureTime"])                  # last value written, e.g. 2000.448
print(r.get("frame_values", {}).get("ExposureTime"))  # what the newest frame used, e.g. 19600.0
for name, note in r.get("notes", {}).items():
    print(note)
# ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448);
# the camera's newest frame was at 19600 us.
```

当自动模式为 `Off` 时（此时该节点即为摄像机的的实际值），或相机未连接至后端时，这两个键均不存在。对于数组，`ArraySession.status()["member_ae"]`会一次性返回每个成员的帧值。

#### DAQ 测光曝光上限（HTTP）

当连接了 DAQ 光传感器时，Chloros 会根据测得的日光强度限制摄像头的曝光 × 增益，以确保白色参考板不会出现剪切。 该上限仅在校准目标处于该摄像头的视野内时生效（即在过去 10 秒内出现在其画面中）；当视野中无目标时，将改由摄像头自身的常规自动曝光（带有高光保护功能）运行，以确保场景的其余部分不会被不必要地压暗。 该功能没有SDK封装；需通过`GET` / `POST /api/camera/<serial>/daq-exposure-cap`针对每台相机单独读取或切换（POST请求正文中的`{"enabled": false}`可关闭该相机的此功能）。 响应中包含 `enabled`、`cap_us`（实时上限值，`None` 表示该功能处于非活动状态，包括未检测到校准目标时）， `k_exp_g`（用于计算上限的常数）以及：

| 字段 | 含义 |
| --- | --- |
| `k_exp_g_source` | 正在使用的常量：`camera` — 该相机自身的常量，由其校准包中的实验室点计算得出 （RGB相机）；`fleet` — 机队常数，用于校准包中无实验室点的场景；`override` — 在后端设置的字段覆盖值。 |
| `k_exp_g_fleet` | 机队常数，用于比较。 |

### 同步阵列 — `ArraySession` (Smart-Prep)

`connect_array` 是多摄像头配置的 **推荐入门方案**。它在后台运行完整的图形用户界面 (GUI) Smart-Prep 工作流程：

1. **网络分析**（`/api/camera/array/recommend`）——设置**完整传感器** （无像素合并，无ROI裁剪），并让帧率降至该主机和网络所能提供的水平，该水平是根据 Connect 将使用的像素格式和封装方式推算得出的。系统绝不会缩小帧尺寸以维持帧率：传递较小的 `width` / `height` 或 `binning=2` / `4` 参数，以牺牲分辨率换取帧率（该端点的的 `alternatives` 列表中列出了这些选项及其预期的帧率）。
2. **分级自动选择** — `sim-capture-sim-emit`（所有相机在同一触发信号下曝光，以共享总线的方式控制发射节奏，共享带宽缩减时帧率随之下降），适用于任何可对相机进行节奏控制以共享总线的网络； `slip-emit-and-capture`（相机轮流使用总线，不同时曝光）仅在无法共享时启用。
3. 默认**启用 PTP**— 跨摄像机的时间戳会以**约 1 毫秒**的精度对齐到一个共享时钟上，后端的 [PTP 主时钟](#ptp-grandmaster--time_sync_status)。同时曝光由 M8 硬件触发器（**< 100 µs** 模块间延迟）实现，而非由 PTP 控制：PTP 仅对齐*时间戳*，而非曝光。
4. **按相机自动选择像素格式** — 单色 → `Mono12`，彩色 → `BayerRG12`。
5. **自动曝光初始化** — 记录每台相机当前的自动曝光状态，以防止连接时在拍摄过程中重置曝光。
6. **GPIO 触发配置** — `connect_array` 使每台相机进入就绪状态（`TriggerMode=On`、`TriggerSource=Line2`），以便主机的的脉冲能通过 M8 电缆驱动从机。此步骤仅适用于阵列：若仅通过 `LatticeCamera` 打开单个摄像头，则该摄像头将进入自由运行模式。释放阵列（`disconnect()` / `force_disconnect()`、`chloros-cli lattice array-disconnect`）将所有相机重置为自由运行模式（`TriggerMode=Off`），其余设置均保持连接时检测到的状态：触发和同步线设置 （`TriggerSource`、`TriggerActivation`、`TriggerOverlap`、`TriggerDelay`、 `LineMode` / `LineSource`；主设备的同步线恢复为输入模式），PTP，以及拍摄会话中更改过的相机设置——除非 `keep_settings=True` / `clear_settings=True`（参见[相机内存](#camera-memory--keep_settings--clear_settings)）。

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

属性：`array_id`、`master_serial`、`slave_serials`、`ptp_enabled`、 `sync_mode`、`packed_transport` 和 `packed_reason` （最后四个是来自 `attach_array` 的句柄上的 `None`），`connect_stream` 和 `backend_url`。 `connect_stream` 是连接应答的 `stream` 块，即单次连接的形状加上 `members` (`{serial: {state, why?}}`)：当数组中的第一个同步组到达时（后端会等待该组，有时间限制，约10秒）， 或者当每个成员的最后一次读取失败作为其 `why` 时，`no_frames` 会触发，对于巨型数据包， `note`中的1500字节备用值。`None`位于`attach_array`句柄上。

#### `connect_array()` 签名

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # pin the master trigger rate (auto if None); see set_target_fps
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    packed="auto",                        # packed 12-bit transport: "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=600.0,                        # s; also the backend's deadline (see below)
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`timeout` 表示调用等待连接的时间长度，该值将作为
截止时间发送至后端。 若届时连接尚未完成，则操作将被撤销：所有已
打开的摄像头将恢复至初始状态并关闭，没有任何数组保持连接，且
该调用将抛出 `ChlorosConnectError` （HTTP 504 `CLIENT_DEADLINE_PASSED`，指明
连接所处的步骤）。因此，引发异常的调用绝不会留下
。一个包含9台摄像头的阵列连接大约需要一分钟；在
摄像机通电后，可能需要几分钟。一个已启动但尚未
同步帧组的阵列会被返回，并伴随错误代码 `connect_stream["state"]`
`no_frames`。后端日志中，每个连接步骤
（`[array-connect <id>] <step>: <s> s …`）对应一行时间记录，最后一行标明最慢的步骤。

`force_tier` 值：
- `"sim-capture-sim-emit"` — 真正的同步（所有摄像头在同一时钟沿触发）。
- `"sim-capture-ftd-stagger"` — 灵活的时间域交错（各摄像头在略有偏移的时间点发射，因此数据包在总线上呈串行化）。
- `"slip-emit-and-capture"` — 按摄像头顺序捕获 （无时间同步）。仅当无法将相机速率调低到足以同时共享总线时，才会自动选择此模式。

`wire_ceiling_mbps` 会覆盖 **主机的持续总线带宽预算**（单位为 MB/s）——这是
整个阵列资源分配所依据的唯一数值。 保留默认值 `None` 以使用自动检测的
值。当阵列报告 GVSP 帧损坏时，请降低该值： 自动生成的值源自
网卡报告的链路速率，该值会高估 USB 适配器、带宽有限的 PCIe 通道以及
繁忙的共享互连结构——而这种高估通常表现为帧损坏，而非
肉眼可见的链路速度变慢。 该值会保存在项目的阵列捕获块中，因此
重新打开项目或后续设置 `connect_array` 时，它会像其他阵列设置一样被恢复。
参见 [阵列健康状况](#array-health--which-subsystem-is-losing-frames)。

#### 12 位紧凑传输 (`packed=`)

在传感器满载时，数组的帧率取决于每台相机发送
帧的速度。12 位格式（`BayerRG12`、 `Mono12`）通常以16位
字的形式传输；紧凑传输则将相同的12位数据以每像素1.5字节的形式发送，
并在主机端进行解包，因此每帧的传输时间缩短了25%——大约
**在传感器满载时每秒可多传输**33%的帧**（实测：9台相机阵列上从10.7帧/秒提升至14.0帧/秒）。**保存的数据完全一致**：数值相同、16位
文件格式相同、元数据相同。

- `packed="auto"`（默认）仅在压缩有益且**且**主机能够
  承担解包负担时才进行压缩：连接时，后端会测量自身的解包速度、
  CPU 核心数、当前负载和网络带宽。 拥有闲置核心的台式机
  在全传感器模式下进行压缩；而繁忙或低功耗的主机（单板计算机、
  嵌入式控制器）则以未压缩形式流式传输。较小的感兴趣区域（ROI）和分桶帧的收益微乎其微。
- `packed="off"`（或 `False`）从不进行压缩——用于在诊断时排除该选项，
  或用于必须将 CPU 资源保留给其他任务的主机。
- `packed="on"`（或 `True`）始终对 12 位格式进行打包。
- 任何其他值都会触发 `ValueError`。8 位和 16 位格式绝不会
  被打包。

`analyze_array_network(..., packed="auto")` 采用相同的决策，
因此其 fps 即为数组运行的速率（响应中包含
`packed_transport` 和 `packed_reason`）； 其`pixel_format`默认采用
连接所应用的设置（Mono12 / BayerRG12），而非8位。

该决策及其理由记录在会话中：

```python
arr = chloros_sdk.connect_array(serials)          # packed="auto"
print(arr.packed_transport, arr.packed_reason)
# True  auto: projected 11.0 -> 14.4 fps, unpack 0.16 of 1.50 budgeted cores

arr = chloros_sdk.connect_array(serials, packed="off")
```

`ChlorosProject.connect_all(packed=...)` 会对项目中的每个
数组应用相同的覆盖设置（默认值为 `None`，这意味着 `"auto"`：项目不会为
每个数组保存独立的压缩设置）。
若要设置全局默认值，请在
后端环境中设置 `CHLOROS_ARRAY_PACKED12=off`（或 `on`）； 但显式的 `packed="on"` / `"off"` 设置仍具有优先级。
`/api/camera/array/status` 会报告 `packed_transport` 和 `packed_decision`。

#### 预期帧率

测得的阵列帧率（硬件同步，12 位；全传感器 = 2048 × 1536）。 可利用这些数据对 `status()["actual_fps"]` 进行合理性验证；`analyze_array_network()` 则可据此推算您自身主机和网络的帧率。

| 配置 | 阵列帧率 |
| --- | --- |
| 全传感器，PCIe 10 GbE 主机网卡，打包传输（台式机上的 `packed="auto"`） | ~14 fps |
| 全传感器，PCIe 10 GbE 主机网卡，未打包传输（`packed="off"`） | ~10.5 fps |
| 全传感器，8 台摄像头连接至 USB 10 GbE 适配器 | ~3.8 fps |
| 全传感器模式，1 GbE 主机链路 | 1.2–1.7 fps |
| 2 × 2 像素合并（`binning=2`），10 GbE 主机网卡 | ~30–33 fps |

- 在 PCIe 10 GbE 连接下，全传感器模式下的性能瓶颈在于**摄像头**而非网络：4 台和 8 台摄像头的运行速率相同。
- **`target_fps` 代表触发速率，阵列运行速度绝不会超过该值。** `target_fps=1` 将阵列运行速率设为 1 fps；`None`（默认值）则将其运行速率设为主机和网络所能承载的最高速率。若某引脚的速率超过该值，则会被降至该速率 （参见 [`set_target_fps`](#interval-capture-for-drone-surveys)）。
- **`link_speed_mbps` 中的 `4294` 并非链路速率。** 这意味着 Windows 无法读取适配器的实际速率（驱动程序报告了最大的 32 位值），这是 USB 以太网适配器的典型特征。无论如何，USB 适配器在 Windows 上的可用速率上限均为 200 MB/s（参见 [阵列健康状况](#array-health--which-subsystem-is-losing-frames])；如需进行调优，请参阅 CLI 参考文档中 [主机网卡设置与调优](cli-reference.md#host-nic-setup--tuning-lattice-arrays) 下的“USB 10GbE 适配器”部分。
- **Linux 系统同样存在 USB 速率限制：x86_64 架构下为 400 MB/s，NVIDIA Jetson 架构下为 200 MB/s。** 适配器通过 sysfs 进行识别（若无法识别则回退至 `ethtool -i`），因此 USB 5 GbE 或 10 GbE 适配器的带宽分配将以该上限为准，而非其标称的 500 / ~1060 MB/s。 x86_64 平台的数据是在 Linux 上测得的：同一款适配器在 Windows 上超过 200 MB/s 时会导致帧数据损坏，但在 Linux 上，其运行 4 台摄像头阵列时，在高达 480 MB/s 的所有上限速度下均能保持数据完整。 在 NVIDIA Jetson 上，同一网卡仅在 200 MB/s 时传输无误，超过该速度后会间歇性地丢失帧。如果您的主机在未启用 `gvsp_corrupt` 帧的情况下能处理更高的数据量， 请通过 `wire_ceiling_mbps` 提高该阈值。在 Linux 系统中，连接至 USB 2.0 端口的适配器带宽上限为 30 MB/s；请将其插入 USB 3.0 端口。

#### 超额订阅（每台摄像头的下限）

阵列可在任何网络上运行；帧率取决于网络性能。 模拟发射调速机制会将防冲突的带宽配额平均分配给每台摄像机，且所有摄像机仍会在同一触发信号下输出图像，因此在低速网络中，每台摄像机的带宽份额——以及帧率——都很低：8台分辨率12位摄像机在未启用巨型帧的1 GbE网络上，每台可获得6.5 MB/s，约**1 fps**（压缩后约1.3 fps）。该份额的下限为摄像机可调速到的最低速率 (`per_cam_floor_bps`：连接时从每台摄像机的 GevSCPD 范围中读取，且绝不低于 1 MB/s)。 如果网络链路要求更低速率，连接将回退到**串行**发送模式（`slip-emit-and-capture`）：相机将以全链路速率轮流传输，因此每个帧都能到达，但成员数据**不会同时暴露**——响应的层级和阻塞器 (`SERIAL: ...`) 会明确指出这一点；传递 `force_tier="sim-capture-sim-emit"` 可拒绝此降级机制。帧大小不影响资源共享，因此**像素合并和 ROI 虽能提高帧率，但不会改变数组是否能适配**。只有当链路无法以任何时序（无论是同时还是交替）传输该数组时，才会发生**超订阅**——其故障模式是GVSP数据包丢失，而非帧率降低。解决方法：减少摄像头数量、端到端使用巨型帧，或使用更快的网卡。

- `analyze_array_network()` 和 `/api/camera/array/connect` 的响应中包含 `oversubscribed`、`aggregate_demand_bps`、`collision_safe_ceiling_bps`、`max_cams_collision_safe` 和 `per_cam_floor_bps`。当 `oversubscribed` 为真时，投影会 **将 fps 字段清零**（`achievable_fps_max` / `fps_bright` / `fps_dark`），而不是报告一个误导性的“速度虽慢但仍可工作”的帧率。
- `POST /api/camera/array/connect` 接受一个 `pin_resolution` 正文参数（**仅限 HTTP — 并非 SDK 关键字参数**；`connect_array` 未公开该参数）。该模式为严格模式：若在 `pin_resolution` 设置为启用状态时建立超额订阅连接，系统将**直接拒绝**，并返回包含所有补救措施的错误信息。 若未设置该参数，连接将照常进行，但会发出警告，提示数组将丢失数据包。无论哪种情况，连接都会保留请求的帧大小——它绝不会自行进行分桶或裁剪。
- 仅用于测试： 在后端环境中设置 `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1`，将拒绝行为降级为醒目的警告——您仍可建立连接，但需接受数据包丢失。

#### 数组健康状况——哪个子系统正在丢失帧

`CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` (`GET /api/camera/array/<array_id>/capability`) 在已连接的阵列上承载一个活跃的 `health` 块，
该指标在滚动 **10 秒** 滚动窗口内重新评估。它将帧丢失
细分为两种需要相反修复措施的原因，而不是一个未指明具体原因的“不完整”率：

| 字段 | 含义 | 涉及子系统 |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct`（按序列号） | 帧**已到达但结构损坏**— GVSP 数据包丢失。 |**网络**：带宽预算、速率控制、网卡接收环、MTU |
| `never_arrived_rate_pct`（按序列号） | 帧**根本未到达**—— 相机未触发，或未输出任何数据。 |**触发/同步**：M8电缆、`line=`、`TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | 每台摄像头的最差传输速率 。 | — |
| `per_cam_rate_pct` | 每台摄像头的综合不完整率（两种原因合计）。 | — |
| `stable_for_seconds` | 每台摄像头处于 0.01% 以下的时间长度。 | — |

除 `health` 外，同一记录还报告了整个分配所占用的数量：

| 字段 | 含义 |
| --- | --- |
| `wire_ceiling_mbps` | 主机当前生效的持续带宽配额，MB/s。 |
| `wire_ceiling_source` | 该数值来源的文字说明——例如 `USB-capped 200 MB/s (was theoretical 1062; …)` 或 `user override 120 MB/s (auto said 200)`。 |
| `wire_ceiling_is_user_set` | 当由 `wire_ceiling_mbps=` 设置时，该值为 `true`。 |
| `nic_is_usb` | 对于 USB 以太网适配器，该值为 `true`。|

```python
import chloros_sdk

serials = ["213800234", "214000533"]           # master first
arr = chloros_sdk.attach_array(serials)
h = arr.capability()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    # force_disconnect(), not disconnect(): this handle came from attach_array,
    # and disconnect() never tears down an array the session only attached to.
    # It returns only once the backend confirms the release; a teardown that
    # timed out or was refused raises ChlorosConnectError instead, so the
    # reconnect below never races a half-released array.
    arr.force_disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**解读：**当 `gvsp_corrupt_rate_pct` 不为零且 `never_arrived_rate_pct` 为 0 时，
表示触发和电缆同步完全正常，100% 的损耗源于网络路径——将
`wire_ceiling_mbps` 并重新连接。反向模式则表明问题出在同步线缆或
触发线上。

> **`target_fps` 并非解决帧损坏问题的关键。** GevSCPD 时序在
> 连接时仅设置一次，因此降低触发率会改变占空比，而不会改变
> 同时发射的突发速率。实测将需求削减 5 倍未见改善，而
> 将线速上限从 240 MB/s 降至 200 MB/s 后，同一测试平台的帧损坏率从 10.4% 降至
> 0.00%。

> **在 LATTICE（M3C / M3M）固件环境下，运行中的阵列无法自行重新规划。** 它无法
> 自行修复此问题；请断开并重新连接，以便连接时间选择器根据
> 新的上限重新规划。

**USB 以太网适配器** 无论其
标称规格：将链路速率转换为持续传输速率的效率表源自
PCIe，而 USB 网卡虽会广告其以太网链路速率，但实际受限于
USB 总线及其驱动程序。 该上限是绝对值，而非比例——USB 1 GbE 适配器
可达到约 80 MB/s，且不受此限制。该限制适用于 Windows 和 Linux（包括 Jetson）；
具体测量结果请参见 [预期帧率](#expected-frame-rates)。

#### `ArraySession` 方法

| 方法 | 描述 |
| --- | --- |
| `status(timeout=10.0)` | 运行中数组的实时状态：`target_fps` / `target_fps_auto`（触发速率及速率控制器是否拥有该数组），`target_fps_clamped_reason`（固定速率被压低的原因，否则为 `None`）， `actual_fps`（实测值；一旦抓取循环放弃或卡死，则为 0），`recommended_fps`，`streaming` （一旦抓取循环放弃则为 False ——请检查此项，而不仅仅是 `last_error`； 在设置重启后仍保持为True，而`restarting`会标记此状态），`stalled`（当抓取循环正在运行但约十帧周期内（2–30秒）未发布任何帧组时为True； `last_error`仅在60秒后才报告此情况)，`last_frame_age_s`，`last_error`，`serials`， `ptp_enabled`（`True`仅在数组通过PTP连接时出现 **且**每个成员上次读取的是`PtpEnable`；这与`ptp_epoch`所测量的“锁定”状态不同），`ptp_configured`（连接请求的内容），`ptp_off_members` / `ptp_unread_members`（最后读取状态为“`PtpEnable` off”的成员 / 未响应的成员），`member_recovery` （`None`，或已断电或链路中断的成员：`state`、`waiting`——尽管其未响应，`reopening`， 随后是 `recovered`，以及 `members` 和 `why` —— 当该节点再次响应时，数组会重新将其纳入并恢复其连接时的 PTP、 触发器和同步线接线、帧率及相机设置，随后再次进行流传输（无新的 `connect_array`），以及 `ptp_epoch`（实测值）、`sync_mode` / `sync_tier`、`packed_transport` / `packed_decision`、`per_cam_emit_mbps`、`link_speed_mbps`（`4294` 表示 Windows 无法读取适配器的速度 — 参见 [预期帧率](#expected-frame-rates))，`trigger_pipeline` / `pipeline_tune`、`radiometric_chain`（表示各波段的辐射链是否可用，以及是哪一成员导致链断裂），以及 `band_clamp` ——同时实时预览会渲染该数组：每个成员中，解混算式将亮度值限制为0的亮像素所占比例 （`members`：按相机R、G、B波段顺序排列的`{serial: [band0, band1, band2]}`），以及合成图像中各波段显示的 （`composite`：`{band name: share}`），其中 `warn` 表示 GUI 标记为“低可信度波段数据”的比例 （约每秒测量一次；当未运行实时预览时为 `None`），以及 `member_ae` — 针对每个成员，其最后发布的帧所携带的曝光值（`exposure_us`）和增益 （`gain_db`）——这些数据取自帧本身（`read_nodes` 中的 `ExposureTime` / `Gain` 返回自动曝光运行期间写入的最后一个值，而非当前操作状态）。 |
| `set_member_settings(serial, **kwargs)` | 写入某台成员相机的设置，`CameraSession.set_settings()` 会将其 （曝光、增益、目标亮度等）；该成员仍保留在数组中。若序列不是数组成员，则触发 `ChlorosConnectError` 异常。参见 [数组成员的曝光设置](#exposure-on-array-members)。 |
| `capability(timeout=10.0)` | 智能预设最终确定的参数（采集层、帧大小、`pixel_format`）、实时 `health` 模块以及主机的带宽预算。 参见 [阵列健康状况](#array-health--which-subsystem-is-losing-frames)。 |
| `set_target_fps(fps, timeout=10.0)` | 在阵列运行时更改触发率，无需重新连接。某个数值将其固定（速率控制器不予干预）； `None` 将其恢复为自动模式。返回 `{requested, target_fps, target_fps_auto, clamped}` — 当速率降低至主机和网络可承载的水平时，`clamped` 为 True。 参见 [无人机测绘的间隔采集](#interval-capture-for-drone-surveys)。 |
| `grab(processing="debayered", with_metadata=False, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, bit_depth=None, band=None, align=False, refl_max=None)` | 每台相机各获取一帧图像，**均由单次触发生成**，以 numpy 数组形式返回（不写入文件）。返回一个 `GrabResult`（一个 `dict`）：`{serial: array}`，或包含 `{serial: (array, metadata)}` 和 `with_metadata=True` 的 `{serial: array}`。 `.group_id`是帧共享的触发器；`.errors`将每个被排除的相机映射到原因（无法在该触发器下处理的相机将被省略，绝不会由其他相机填补）。 `processing` 等于 `raw` / `debayered` / `radiance` / `reflectance` / `display`。其读取方式与项目 API 的 [`ArrayHandle.grab()`](#one-trigger-per-read--group_id) ——一个共享的实现——因此那里的每个关键字（`band`、`align`、 `bit_depth`、`refl_max`) 均可在此处使用。需要 `numpy`。 |
| `frame_stream(*, processing="debayered", fps=5.0, duration=None, count=None, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, with_metadata=False, bit_depth=None, band=None, align=False, refl_max=None)` | 每 tick 返回一个 `grab()` 结果的迭代器（仅限关键字参数），按 `fps` 进行限速， 受 `duration`（秒）和/或 `count` 限制（`None` = 无界）。**每个产出的集合都是一个新的触发器：** 若某个时钟周期中最新触发的正是该集合（该集合已被产出），则该时钟周期将等待下一个触发而非重复该触发，因此数据流将以 `fps` 与数组自身帧率中的较低者运行。此时会抛出 `FrameFetchError` 异常而非保持静默： 当所有摄像机均因某种原因拒绝处理（导致下一个帧将重复）时，在连续 60 秒的帧中无帧出现，以及在 60 秒内无新触发器时（“60 秒内无新触发器”），会立即抛出 `FrameFetchError` 异常。 实现方式与 `ArrayHandle.frame_stream()` 相同。参见 [在您自己的管道中使用实时帧](#live-frames-in-your-own-pipeline)。 |
| `continuous(output_dir="output", *, format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, max_captures=None, duration_s=None, daq=None, daq_tolerance_ms=None, timeout=30.0)` | 启动后端自身的连续捕获（即 GUI 中的“连续”模式）→ [`ContinuousHandle`](#continuoushandle)。会保存数组发布的**每个**同步组，保存方式与 `capture()` 相同（采用相同的导出控制），直至被停止或直到 `max_captures` / `duration_s` （`None` = 直到停止）。如果该数组已有进程正在运行，则会触发 `ChlorosConnectError` 异常。 |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, smart=False, after_seq=None, daq=None, daq_tolerance_ms=None, timeout=300.0)` | 一个同步的捕获组。返回一个 `CaptureResult` （帧字典列表 + `.skipped` + `.failed` + `.group_seq` + `.daq_files` + `.daq_matches` + `.alignment` + `.oriented`)。将上一个结果’`group_seq` 作为 `after_seq`，以获取其后的下一个同步组，而非等待新组。 以下为导出控制选项。 |
| `capture(..., smart=True)` | **智能捕获** — 等待所有摄像头的 AE 值稳定后触发。 |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, orient=None, timeout=120.0)` | 最快捕获：仅原始数据 + 分配的 DAQ 读数 （+ 自由组合索引）。与 GUI 中的“最快捕获”按钮功能一致。 |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | 在一个有界循环中执行单次/连续/间隔捕获。 返回 `list[CaptureResult]`。**需要 `count` 和/或 `duration_s`** 才能终止（SDK 不支持 Ctrl+C）。此外，在通过后， 当某轮次报告磁盘已满（即包含 `disk_full` 的 `.failed` 条目）时，该轮次即为最终结果。 |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, expr=None, timeout=30.0)` | 开始将实时组合索引视图录制为视频/GIF → `RecorderHandle`。视频为 Motion-JPEG 格式 `.avi` （无需ffmpeg）。若未打开任何GUI视图，后端将渲染：`expr`（数组波段名称中的索引表达式，例如`"(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)"`），或默认渲染数组的NDVI。 每个数组对应一个合成记录器。 |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, orient=None, timeout=30.0)` | 启动高帧率原始拜耳连拍 → `RecorderHandle`。使用 `build_video()` 进行离线重新处理。`orient=True`：`build_video()`生成的产品会根据相机方向进行旋转（单台相机的产品按其自身方向旋转，组合索引按主相机的方向旋转）；原始帧保持原生格式，连拍清单会记录每个成员的方向。 |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | 将已保存的原始连拍数据离线重新处理为校准后的视频。 该任务将阻塞直至完成（`wait=True`），并返回构建结果：`{success, outputs, errors, skipped, substitutions, notes, degraded, combined}` — 参见[构建结果](#the-build_video-result) 。 |
| `build_video_status(job_id, timeout=15.0)` | 轮询离线构建任务：`{running, result, error, burst_dir}`。 |
| `disconnect(keep_settings=False, clear_settings=False)` | 释放整个阵列 — **如果本次会话曾打开它** (`connect_array`)：此时即为 `force_disconnect()`，若释放未获确认，则抛出 `ChlorosConnectError` （`with` 代码块的退出时绝不会抛出异常）。对于来自 `attach_array` 的句柄，它会让数组继续运行，并向 stderr 打印一行信息说明此情况：该数组可能属于 GUI 或其他脚本。 每台摄像机均保持自由运行状态，且状态与连接时一致；`keep_settings=True` 将其设置保存到摄像机自身的内存中，`clear_settings=True` 将该内存重置为出厂设置（参见 [摄像机内存](#camera-memory--keep_settings--clear_settings))）。|
| `force_disconnect(timeout=60.0, keep_settings=False, clear_settings=False)` | 释放整个数组，无论由谁打开。当您需要终止数组时（例如，为了使用不同设置重新连接），请在已连接的句柄上使用此函数。 当后端确认释放后，或无内容可释放时（数组已不存在，或无后端正在运行），该函数将返回；若释放超时或被后端拒绝，则抛出 `ChlorosConnectError` 异常。无论使用哪种标志，该函数都会返回每台摄像机内存的 (`camera_memory`)，否则返回 `None`。 |

`capture()` 导出控制（与 GUI/CLI 使用的端点相同）：

- `processing` / `levels` — `processing="all"`（或 `levels=["raw","radiance",…]`）会保存每台摄像头的所有适用导出类型； 而单个 `processing` 值仅保存该级别。
- `aligned` — **默认启用（`None`），因为 Chloros 应用对同一数组的捕获**（其“按数组**对齐**”的捕获设置）。每个成员的非原始导出数据会通过数组的[对齐配置文件]（#array-alignment）映射到主相机的帧中，并裁剪至各成员的共同重叠区域，因此这些文件经过像素配准，其尺寸、裁剪范围和 `Alignment*` 标签均与该应用程序生成的结果一致（在 ROI 范围内，变换会重新基于窗口进行调整）；`XMP-mapir:AlignmentJSON` 表示 `"applied": true` 及其 `crop_roi`。 原始文件保持原生尺寸且未进行变形，但会在元数据中携带变换信息。`aligned=False`将每个文件以原生尺寸保存，且未进行变形；`aligned=True`是明确指定的默认设置。`continuous()`、`capture(smart=True)`、`capture_repeated()` 以及项目级别的 `ArrayHandle.capture()` / `capture_smart()` 均采用相同的默认设置。 无论是否应用了畸变，一旦数组具有对齐配置文件，每个成员文件都会记录其摄像机的 `Alignment*` 变换——但配置文件未对齐的摄像机除外 （被省略，或因求解失败而被迫设为单位变换），以及在没有其他相机对齐时的参考相机：这些相机不携带 `Alignment*` 标签。 如果数组上没有配置文件，则捕获数据将以未对齐状态保存，结果也会注明这一点：`result.alignment` 即为 `{"requested": True, "applied": False, "warnings": [...]}`，每个导致无法应用变形的原因（无配置文件、成员分辨率不同、数组无法，或帧无法映射到配置文件上）。只有当所有请求畸变处理的导出文件均成功处理后，`applied` 才会变为 `True`。 数组的自动对齐功能会在曝光稳定后生成配置文件；`ArrayHandle.calibrate_alignment()` / CLI 的 `align-calibrate` 可按需生成配置文件。 `result.oriented` 用于指示 `orient=True` 是否已处理相关产品。
- `render_index=False` — 跳过按相机生成的植被指数叠加层；默认情况下，会在配置的位置渲染该叠加层。
- `daq` / `daq_tolerance_ms` — DAQ读数默认将作为`.daq`副文件保存，即使所选层级无需该数据；`daq=False`则跳过此操作， 而 `daq_tolerance_ms` 用于设置匹配容差（默认值为 500）。`force_daq=True` 是该默认值的旧拼写形式。参见 [捕获保存的 DAQ 读数]（#the-daq-reading-a-capture-saves--daq_files-and-daq_matches）。
- `orient=True` — 将每个成员的**乘积**（`display`、`radiance`、`reflectance`、各摄像机索引及合成图像）乘以摄像机方向，该操作在对齐变形之后进行： 若在变换至主相机坐标系时，采用**主相机**的姿态（`aligned`的默认设置——成员文件保持共注册），否则采用其自身相机的姿态。**默认：每个成员文件保持原生格式**，因为数组捕获始终如此；无论哪种情况，`raw` 和 `debayered` 均保持原生格式。每个文件都会记录其像素所在帧的姿态 （`XMP-Camera:ChlorosRotation` / `ChlorosFlip*` / `ChlorosOrientationApplied` — 原始文件，采用其自身相机的坐标系)，而 `Alignment*` XMP 文件则保留原生帧变换（在方向变换之前应用）； 携带该变换的原始文件还会记录 `ChlorosAligned*`（主帧的），即被变换到主帧中的产品的方向。处理过程会根据像素所属的方向旋转每个产品：当对原始文件进行变换时（启用数组对齐），使用对齐后的方向；当 不启用时则采用相机自身的变换。 `capture_repeated()` 和 `capture_fastest()` 会直接传递该变换；项目级别的 `ArrayHandle.capture()` / `capture_smart()` 也会采用该变换。

**TIFF 压缩 （HTTP 专用控件）：**`ArraySession.capture()` 不发送 `compression` 键， 因此采用后端默认设置——`POST /api/camera/array/capture` 读取 `compression` 正文参数，默认情况下： 未压缩，完全保留传感器读取的原始像素数据（每帧全分辨率图像约 6.3 MB）。 `"deflate"`（无损 zlib L1 + 水平预测器，约 4.1 MB/帧）写入的文件较小但速度较慢——这两种格式均为无损，且在导入时读取结果完全相同。 SDK 未为此提供任何关键字参数；解决方法是使用 `chloros-cli lattice array-capture --compression deflate` 或原始 HTTP。DEFLATE 会占用 Python 的 GIL，因此压缩写入无法在各摄像头写入线程间并行处理——若要以传感器速率持续进行全分辨率捕获，则需使用未压缩的默认设置。详情： [CLI参考 → array-capture](cli-reference.md)。**连续数据组（HTTP）：** 该端点返回 `group_seq`；若在循环中将其作为 `after_seq` 发回至下一次请求，即可获取下一个同步数据组——包括在前一次捕获保存期间发布的那个 ——而非等待请求之后发布的组，而 `skipped_groups` 会报告循环中遗漏的任何组。`capture_repeated`、 CLI 中的 `--continuous` 以及 GUI 的“连续模式”均采用此机制；间隔周期内的数据在每个预定时刻均保持最新状态。**按成员进行的导出覆盖（仅限 HTTP）：**同一端点还接受 `exclude_serials`（列表——从保存的集合中移除成员；该数组仍作为一个同步组触发，被排除的成员将在 `excluded` 中返回）、`serial_levels` （`{serial: [level tokens]}` 按摄像头级别的覆盖设置）以及 `serial_index`（`{serial: bool}` 按摄像头索引叠加的覆盖设置）。 这些是与 GUI 对应的正文参数，**目前还不是 SDK 关键字参数**；映射中缺失的成员将回退到全数组范围的 `levels` / `render_index`。

##### 检查被跳过的摄像头 — `CaptureResult.skipped`

`ArraySession.capture()` 返回一个 `CaptureResult`，该对象是 `list` 的子类：对其进行迭代、索引， `len()` 操作——所有现有模式均可正常运行。新代码可检查 `.skipped` 属性，以查看哪些摄像头被排除以及原因。 最常见的情况是，当您请求 `processing="radiance"` 或 `"reflectance"` 时，混合滤光阵列中的 RGB 摄像头 ——对于宽带传感器而言，按拜耳阵列划分的辐射度数据毫无意义，因此后端会跳过这些摄像头，而非生成无意义的数据。

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

原因标记遵循 `<level>-not-applicable-to-rgb-cam` 的模式（每个被跳过的级别对应一条记录，每条记录包含 `level`）。 未加载校准包的相机将跳过 `radiance` / `reflectance`，并使用 `<level>-needs-calibration (connect the camera so its cal pack loads)`， 若无可用的数据采集（DAQ）反射率读数，则跳过 `reflectance-skipped-no-dls`。其他与反射率相关的跳过情况包括：`reflectance-skipped-no-fresh-dls`（无新的下行辐射读数可用）、`reflectance-skipped-bound-daq-unavailable (…)` （无法访问绑定的DAQ），以及 `dls-uncalibrated-band-<nm>` —— 该波段大部分位于DAQ光传感器辐射校准范围之外（~374–974 nm），因此基于DAQ的绝对反射率划分被拒绝，该帧会明确降级为传感器响应模式。在已上市的SKU中，仅F988会触发此情况；该相机的支持路径是反射率面板工作流。

##### 无法写入的文件 — `CaptureResult.failed`

即使捕获操作仅能写入部分文件，仍会返回： 该列表包含所有**已**写入的文件，而 `.failed` 则列出了其余文件，每个文件对应一个字典，其中 `serial` （`None` 用于组合图像及数组的 `.daq` 辅助文件），`level` （`raw`、`radiance`、……、`index`、 `composite`、`daq`、 或 `match` —— 这是 `.daq` 的 `.match.json` 来源记录，表示 `.daq` 已被写入但其 sidecar 未被写入）以及 `error`。在已满的磁盘上，该条目还会包含 `disk_full: True`；此后对该磁盘的每次捕获都会以同样的方式失败，这就是为什么 `capture_repeated()` 到此为止。 部分捕获不会引发异常，且 `.failed` 中的文件不会以半写状态保留在磁盘上。在正常捕获中，`.failed` 为空。

```python
result = arr.capture("output/", processing="all")
for f in result.failed:
    print(f"SN:{f['serial']} {f['level']} not written: {f['error']}")
if any(f.get("disk_full") for f in result.failed):
    raise SystemExit("drive full -- free space and capture again")
```

##### 读取捕获数据的 DAQ 会保存 — `.daq_files` 和 `.daq_matches`

每当 DAQ 与相机或阵列绑定，或被纳入池中时，每次捕获都会保存其使用的下行读数： 一个 `.daq` 文件，位于捕获文件夹的 `daq/` 子文件夹中，并包含其 `.match.json` 和 `.cameras.json` （即该读数所属的文件）。这同样适用于原始或去拜耳化的捕获，且无论捕获操作来自GUI、CLI还是SDK，生成的文件都是一样的。对于未使用DAQ级别的捕获，`daq=False`会跳过该操作 （但 `radiance`、`reflectance` 或 `display` 电平仍会写入该值）；`daq_tolerance_ms` 会调整该 捕获的容差线，绝不会移动实际使用的读数。具体使用哪个DAQ：是绑定到相机或阵列的那个（GUI中的光传感器选择、项目的绑定，或`chloros-cli daq pool-bind`），还是后端拥有的最新那个。 CLI中将这些控件命名为`--no-daq`和`--daq-tolerance-ms`（[CLI参考](cli-reference.md#the-daq-reading-a-capture-saves)）。

```python
r = cam.capture("out", levels=["raw"])      # a CameraSession; ArraySession results match
for m in r.daq_matches:                      # one per .daq in r.daq_files
    print(m["daq_file"], m["gap_ms"], m["in_tolerance"])
```

一次捕获会将所用的光传感器读数保存为 `.daq`，并附带一个 `<file>.daq.match.json`，说明该读数与曝光的关联方式：`method`、`gap_ms`（从曝光到最近的DAQ读数）以及`in_tolerance`（在500毫秒内，即捕获的`daq_tolerance_ms`）。反射率文件的XMP中包含与`Chloros:DlsMatchMethod`、`Chloros:DlsMatchGapS`和`Chloros:DlsInTolerance`相同的判定结果。

捕获会等待曝光后数据采集卡（DAQ）的首次读数，以便融合其两侧的读数（`method` `interpolated`）。 DAQ-E在日光下每0.4秒发送一次读数，在室内每1-2秒发送一次，因此`capture()`会在这些间隔的约一个半之后返回； 曝光本身并无延迟。对于已停止发送数据的DAQ，系统不会等待其恢复。DAQ-E的读数是自上次读数以来该时间间隔内光照的平均值，因此该读数被放置在该时间间隔的中间位置： `gap_ms` 的测量值以该中间点为准，而 `.match.json`'的 `bracket_stamps_ns` 则提供读数本身的时间戳。`ArraySession.continuous()` 会进行等待，但不会减慢采集速度。`capture_repeated()` 的回往返过程不会等待，因此能保持其速率，并匹配到曝光前最接近的读数。

读数永不过期，因此不会因读数延迟而丢失帧——取而代之的是，该读数会被标记为：

- **数据采集卡（DAQ）流卡顿** —— 系统将使用其最新读数，且 `gap_ms` 会标明该读数的时滞（`in_tolerance: false` 超过 500 毫秒）。
- **无数据采集卡可读取**—— 已断开连接（`disconnect()`、`chloros-cli daq pool-disconnect`），或无法访问已绑定的数据采集卡 —— 此时将复用相机的上一次读数，并将其描述为**本次** 曝光：`method`、`retained`、`gap_ms`来自该曝光， `in_tolerance` 经重新计算，而 `retained_from_epoch_ns` 则标明其最初匹配的曝光。从未获取过读数的相机仍显示无反射率，结果中会说明原因 （`reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`，或捕获失败）。

捕获文件若保存在 Chloros 中已打开的项目文件夹内，则会加入该项目的文件列表，这与在应用内进行捕获时相同（保存在其他位置的文件则不受影响）： `CameraSession.capture()`、`ArraySession.capture()`（`smart=True`也是如此） 以及 `capture_cameras_transient()`。结果在 `.project_add` 中有所体现（`capture_cameras_transient()`：字典中的 `project_add`） ——`held`、`files` 以及后端的 `message`；`held` 是 `True`，当处理运行正在进行时，文件会在处理结束后合并。当项目中没有文件时，返回 `None`。

`CameraSession.capture()` 和 `ArraySession.capture()` 将其作为结果返回为 `.daq_files` 和 `.daq_matches`， 作为HTTP捕获响应（`/api/camera/<serial>/cli-capture`、`/api/camera/capture`、`/api/camera/capture-all-synced`、`/api/camera/array/capture`） 则包含 `daq_files` 和 `daq_matches`，每个 `.daq` 对应一个：`daq_file`， `cameras`、`method`、`gap_ms`、 `in_tolerance`、`tolerance_ms` 和 `stream_fault` ——这是数据采集卡（DAQ）自身的流故障，仅在读数超出容差范围时才会被检出。在依赖捕获数据的反射率之前，请先检查其 `.match.json` 或 XMP 中的 `in_tolerance`，或数据采集卡（DAQ）的 [`status()`](#daqsensorsession-methods)。

`processing` 级别：

| 级别 | 输出 |
| --- | --- |
| `"raw"` | 直接来自传感器的单通道拜耳（单色摄像头：单波段）数据。 |
| `"debayered"` *（SDK默认）* | 3通道彩色：从原始数据中扣除校准包的暗偏移（DSNU），然后使用OpenCV的边缘感知去马赛克——不包含线性LUT、平场校正或解混 （单色相机：1通道灰度）。 保存的TIFF文件采用**RGB**通道顺序（光度RGB，因此任何图像查看器都能显示真实颜色）；而`grab()` / `frame_stream()`返回的内存数组采用OpenCV的**BGR** 顺序——若要与保存的文件进行比较，请反转通道顺序（`frame[..., ::-1]`）。`chloros-cli process` 写入的结果与 `Debayered_Images` 相同。 |
| `"radiance"` | 通过完整的辐射测量链计算得出的 float32 W/m²/sr/nm。仅限多光谱模式——RGB相机将被跳过。 |
| `"reflectance"` | uint16 0..32768（兼容 Pix4D）；需要实时数据采集（DAQ）配对以获取绝对参考。仅限多光谱。 |
| `"display"` | 完整校准链，与 GUI 预览一致（根据相机配置文件进行色彩校正、白平衡和伽马校正）。 |
| `"all"` | **每台相机针对每个适用级别**生成一个文件（与 GUI 的“捕获全部”/CLI默认设置一致）。返回的 `CaptureResult` 文件中，每个 `(cam, level)` 对应一个帧字典，每个字典中包含该级别信息； 不适用的级别则出现在 `.skipped` 中。用于读取捕获数据的 DAQ 会被保存为 `.daq` 旁路文件。 |

> **注意 — 默认值与CLI不同。** `ArraySession.capture()` 的默认值为 `processing="debayered"`；`chloros-cli lattice array-capture` 命令的默认值为 `processing="all"`。 请从 SDK 显式传递 `processing="all"`，以模拟 CLI/GUI 的多级保存功能。

##### 数组捕获文件名

每个级别都保存在 `output_dir` 的独立子文件夹中（`raw/`、`debayered/`、 `radiance/`、`reflectance/`、 `preview/` 对应 `display`、`index/`、`composite/`）。每个文件将相机命名为 `SN<serial>-<filter>`：先是序列号，然后是相机型号中的滤镜部分——例如，`SN214000533-FOCN` 对应 `M3C-L87-FOCN`， `SN261902142-F450` 对应 `M3M-L87-F450`，`-` 对应 `_`， 因此文件名与之前一样，由 `_` 分隔的字段组成。 无法读取型号的相机仅显示为 `SN<serial>`，而早期版本的捕获文件名中不包含过滤器。文件名取决于成员被要求提供多少级信息：

| 成员被要求提供 | 文件名 | 示例（`reflectance`，默认 `format="tiff"`） |
| --- | --- | --- |
| **一级**（`processing="reflectance"`，或仅含一项的 `levels`） | `sync_<timestamp>_SN<serial>-<filter>.<format>` — 与早期版本一样，没有级别后缀，因此现有脚本仍可正常运行 | `reflectance/sync_<ts>_SN214000533-FOCN.tiff` |
| **多级**（`processing="all"`，或包含多个项目的 `levels`） | `sync_<timestamp>_SN<serial>-<filter>_<level>.<format>` | `reflectance/sync_<ts>_SN214000533-FOCN_reflectance.tiff` |

[钳位掩码](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) 紧随产品名称之后，`clamp_masks/<product folder>/<product name>_clampmask.tif`：单级反射率捕获会写入 `clamp_masks/<product folder>/<product name>_clampmask.tif`，而 `processing="all"` 会写入 `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`。 后缀规则根据每个成员所请求的级别单独确定，因此跳过的级别（例如RGB相机上的反射率）不会改变该规则。请从返回的帧中读取路径，而不是自行构建。单相机捕获 （`CameraSession.capture()`）总是会添加一个级别：`capture_<ts>_SN<serial>-<filter>_<level>.tiff`。

##### 仅接受PNG/JPEG格式的显示和索引产品

PNG 或 JPEG 每通道包含 8 位。若请求的色阶超出其支持范围，系统将在捕获任何数据前拒绝该请求，并显示一条包含具体色阶名称及支持该色阶的 TIFF 格式的提示信息：

- **`radiance`**(float32) 和**`reflectance`**(uint16，`32768` = 1.0) ——因此 `processing="all"` 包含上述两种格式。 SDK会在参数上检查这一点：`capture()`会引发**`ValueError`** 异常，而未包含请求（`CameraSession`、`ArraySession`、`CameraHandle`、 `ArrayHandle`、`capture_smart`）。
- **来自流媒体位深度超过 8 位的摄像头的 `raw` / `debayered`**（12 位 M3C、Mono12 M3M）——后端在知晓每台摄像头的位深度后会拒绝这些请求： HTTP 400 状态码（涉及 `error_code` 和 `FORMAT_CANNOT_HOLD_LEVEL`），由 `ChlorosConnectError` 触发 (`CameraSession` / `ArraySession`) 或 `RuntimeError` (`CameraHandle` / `ArrayHandle`)。单级 `debayered` 捕获模式下，单台摄像头抓取的 8 位帧（用于 PNG/JPEG 格式）不会被拒绝。

将辐射度数据保存为 `format="tiff"`（`ext=".tiff"`）；对于 `display` 和索引渲染，请使用 PNG/JPEG 格式。

#### 自动曝光的工作原理

- **先曝光，后增益。** 当增益设为“自动”时，增益将保持在 0 dB，直到曝光达到允许的最大值。
- **手动曝光后，增益仍保持在“自动”模式**，直到您更改增益或将其设为手动。切回“自动”模式后，增益的限制值将恢复为之前的数值。
- **微小的高光不会使场景变暗。** 占画面面积不超过 1% 的镜面高光将被忽略；在已知校准目标的面板中，单个面板内的高光占比不超过 0.5% 时亦被忽略。
- **数据采集（DAQ）的曝光上限仅在校准目标处于视野内时生效。** 该限制可防止白色面板过曝；当未检测到面板时，相机将根据场景进行曝光。
- **恒定光照下参数保持稳定，光照变化后数秒内稳定。** 在恒定光照条件下，曝光值及其上限保持不变；光照发生大幅变化后，画面将在几秒内稳定下来。 过曝不再永久限制曝光，且当感兴趣区域（ROI）发生变化时，系统会重新调整曝光。
- **亮度目标始终在可控范围内。**当相机处于曝光极限且增益无法提升时，亮度目标不会上升，因此光线恢复时也不会导致画面过曝。
- **自动曝光停止响应的相机将保持在当前光照条件下的曝光水平。**在强阳光下，相机可能会降至最短曝光时间并保持该状态，画面变暗，但所有设置读数均显示正常。Chloros会持续监测画面，将该相机保持在适应当前光照的曝光水平，并在光线变化时释放控制。 光传感器被遮挡或处于黑暗中也不会影响这一过程。

#### 阵列成员的曝光

LATTICE 相机将 `ExposureTime` 的上限限制为其帧率的一个周期。当阵列进行数据流传输时，Chloros 会以略高于触发频率的速率运行每个启用了 `AcquisitionFrameRateEnable` 的成员， 其运行速率略高于触发速率，因此单个成员的曝光上限在**任何ROI**处都略低于触发周期：

- 单个成员的帧周期等于触发周期减去周期15%的冗余时间（至少3毫秒，至多为周期的一半），其曝光上限则等于该帧周期减去约0.1毫秒。 示例：10 fps 的触发率（100 ms）对应的成员速率约为 11.8 fps，曝光上限约为 85 ms。
- **若手动设置的 `ExposureTime` 时间超过该上限，则会减慢触发频率以适应该设置。**Chloros 首先将该成员的上限值设为该数值，并让较低的速率运行几个触发周期（写入操作可能需要多花几秒钟），然后进行写入， 且触发频率不会超过曝光允许的上限。在自动速率数组中，当所有因手动曝光而导致触发变慢的成员恢复为自动曝光后，该数组将恢复至之前的运行速率。 相机本身无法接受的数值仍会被拒绝，此时错误信息中会显示其当前可接受的范围，且如果该成员的自动曝光仅因此次写入操作而被关闭，则会恢复其自动曝光。在 1.3.0 版本之前，一个小的 ROI 将每个成员的上限限制在传感器自身的自由运行周期内， 无论触发速率如何：在 768 × 304 分辨率下为 4.9 毫秒，因此无法提高任何成员的亮度。
- Chloros 绝不会为了适应更快的触发率而缩短手动曝光时间。自动触发率会保持在手动曝光允许的范围内；若固定触发率快于曝光允许的速率，该成员将保留足够长的曝光上限以适应该速率，且其接受的触发频率不会超过其曝光允许的上限。当每个成员都必须响应每次触发时，请降低曝光时间或固定触发率。
- **读回：**读取成员的节点显示 `AcquisitionFrameRateEnable` 为 True，且 `AcquisitionFrameRate` 以该成员的速率显示；当数组停止时，使能信号会重新关闭。
- **该数组拥有这两个节点。请勿在数组成员上写入 `AcquisitionFrameRateEnable` 或 `AcquisitionFrameRate`** —— 请改用 `target_fps` / `set_target_fps()` 来设置数组速率时，请改用 `target_fps` / `set_target_fps()`。

使用 `arr.set_member_settings(serial, exposure_time=20000)` 写入成员的暴露值（该命令的关键字与 `CameraSession.set_settings()` 相同；`chloros-cli lattice exposure --serial` 功能相同）。 该成员仍保留在数组中。请勿使用 `connect_camera()` 打开数组成员： connect 会拒绝已处于数组中的相机，而它返回的 `CameraSession` 在断开连接时会将相机从其数组中移除。`ArraySession.status()["member_ae"]` 会报告每个成员的的曝光值和增益。

### 一个项目记录，适用于所有模式

一个项目文件夹包含所有设备的设置：`cameras.json` ——包括每台摄像头的设置（`cameras[].settings`，适用于单台摄像机及每个阵列成员）和每个阵列的全阵列设置（`arrays[].capture`： 格式、ROI、像素合并、触发率、带宽）——以及 `sensors.json`（每个光传感器的积分时间、帧平均值、上限）。应用程序、CLI、SDK 和 MCP 服务器均以相同的方式读写相同的文件：

- **连接到某个项目下的设备会被保存到该项目中，并将其中的每项更改记录到该项目中。** “属于某个项目”是指在后端（即应用程序）中打开了该项目，或者——通过 `ChlorosProject`（`open_project()`）将自身文件夹的名称传递给后端 (`project_path`)。当项目尚未包含的摄像头、阵列或光传感器连接时，它们会连同其设置一并添加到项目文件中（`connect_camera()`、`connect_array()`、光传感器连接、CLI、应用程序、 `chloros-mcp`），并附带其设置。从任何模式（`set_settings()`、`ArraySession.set_member_settings()`、 `set_target_fps()`、数组设置、`set_cap()`、CLI、 应用程序控制面板）所做的任何更改，都会在片刻后保存到该项目的文件中。在相机上设置的曝光、增益或自动曝光（AE）目标会被保存为“保持”状态，因此重新打开项目时会再次保留这些设置。
- **打开项目时，系统会根据该项目设置其设备**，无论处于何种模式：未设置自身参数的相机、阵列或光传感器将采用项目保存的设置（在项目打开时为 `lattice cam-connect` / `array-connect`，关闭项目后为 `chloros-cli project connect`、`open_project().connect_all()`，以及应用重新打开时的状态）。 以新 ID 连接的阵列会更新项目中已保存的包含相同摄像头的阵列；包含其他摄像头的阵列则会并行添加。
- **应用程序自身的保存操作绝不会用其较旧的副本覆盖设备的实时设置**，并会保留通过 CLI 或 SDK 调用者保存的每个成员的覆盖设置。
- **您设置的像素格式即为项目所保留的格式。** 在“数组设置”中、通过 `--pixel-format` 或 `connect_array(pixel_format=)` 设置的格式（包括 8 位格式）都会被精确保存并重新连接。 若未设置，每台相机将以 12 位默认格式连接（单色传感器上为 `BayerRG12`、`Mono12`）。格式名称代表其位深度； 而“系列”指代传感器类型，因此彩色摄像机绝不会被设置为单色格式，单色摄像机也不会被设置为拜耳格式， 且当格式被映射时，行 `[PixelFormat <serial>] ... written` 会明确标注这一点。8 位捕获的导出数据为 8 位（包含反射率）：经过重新缩放并标注其 `PixelScale`。
- 在无项目下连接的设备 则不记录任何内容。
- **录制操作将跟随其后创建的项目；而打开操作则保留其原项目。**当创建项目时 ——无论是 `create_project()`、`chloros-mcp` 的 `create_project`，还是应用程序的**新建项目**——或复制（应用的**复制**功能，该功能会打开副本）时，如果录制仍在向之前打开的项目中写入数据，则录制会继续进行，所写入的内容最终会保存在新项目中，位于该项目内的相同位置 （如下所示）。打开现有项目——`ChlorosProject.process()` （这会在后台打开该项目），`chloros-cli process`，即应用程序的**打开项目**——此时正在录制的项目仍会继续录制：录制完成后，其文件将加入该项目的文件列表（应用程序在打开项目前会停止所有正在进行的录制）。
  - `ArraySession.record()` 和 `ArraySession.burst()`：录制停止时，视频文件或包含所有帧的连拍文件夹会整体移动到该位置。停止操作的结果会报告新路径 (`out_dir`, `video_path`)，并可据此生成连拍视频。 若驱动器拒绝移动，录制文件将保留在原始位置，结果会相应显示（`relocation_error`）。已停止的连拍的原始帧和 `.daq` 将加入其最终所属项目的文件列表；视频则不会被添加到 项目文件列表中。
  - `ArraySession.continuous()`：变更后的每次捕获都会保存到新项目中；变更前保存的则保留原位置。
  - `DAQSensorSession.record_start()`（以及应用自带的光传感器录制）：当录制停止时，`.daq` 会移动到新项目的 `light_sensor` 文件夹中；`record_stop()` 返回新路径，且该文件将加入该项目的文件列表。

  录制到打开项目之外的文件夹中的内容将保留在原处，而 `start_camera_continuous()` / 多流任务将保留其指定的文件夹。
- **在应用中，新建项目后的捕获操作会进入该新项目。** 创建或复制项目不会中断任何正在进行的捕获运行和录制，此后进行的每次捕获（“捕获全部”、“开始捕获”）都会保存到新项目中，并加入其文件列表。 在应用中打开现有项目会首先停止所有捕获任务。
- **项目在录制过程中不会被移动。**将打开的项目移动到另一个工作目录（应用的**设置 → 工作目录**；没有任何 SDK 调用会移动项目， 但HTTP客户端向`/api/move-project`发送请求则会移动）将因HTTP 409 `PROJECT_RECORDING`错误被拒绝，前提是数组录制或连拍、数组连续捕获、相机连续/多流任务或光传感器录制正在向该项目写入数据： `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.`

这与摄像头在断开连接时保留的状态无关（参见 [摄像头内存](#camera-memory--keep_settings--clear_settings)）：项目在打开时会自行设置其摄像头。

### 摄像机内存 — `keep_settings` / `clear_settings`

当阵列连接时，其摄像机将采用阵列的设置，之后您可以对其进行修改。 当Chloros释放相机时 — `ArraySession.disconnect()` / `force_disconnect()`，即瞬态函数结束时，或`start_camera_continuous()`停止时 ——系统会将每台摄像机恢复到连接前的状态：包括其触发器和同步线接线、PTP 以及摄像机设置——`ExposureAuto` / `GainAuto` （以及手动模式 `ExposureTime` / `Gain`）、`TargetBrightness` 以及 AE 限值和阻尼、白平衡、黑电平、`PixelFormat`、 像素合并和感兴趣区域（ROI）、色彩处理流程（`GammaEnable` / `Gamma`、`LUTEnable`、`ColorTransformationEnable`）、`ReverseX` / `ReverseY`、`UserSetSelector`、PTP的`PtpSlaveOnly` / `PtpDomainNumber`、 数据包大小（`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`），速率控制 （`DeviceLinkThroughputLimitMode` / `DeviceLinkThroughputLimit`、`GevSCPD`、`GevSCFTD`）、数据块 （`ChunkModeActive`，其下的每个 `ChunkEnable` 均属于 `ChunkSelector`）、触发输入去毛刺——一个数组为从机的同步线提供 （`LineFilterSelector` / `LineFilterWidth`）以及采集帧率 （仅当`AcquisitionFrameRate`的使能位为开时有效：若使能位关闭，读取的帧率为时序允许的最高值，而非设定值）。仅写入差异部分：相机首先停止，随后是曝光前所需的时序和帧率（这些参数的帧周期需根据具体情况确定）。

若要在项目外使用相机，有两个参数会写入相机的**自有内存**——即相机开机时的初始状态，无论在何种软件环境下（在 `disconnect()` / `force_disconnect()`、所有瞬态函数、`start_camera_continuous()` 和 `ChlorosProject.disconnect_all()`；MCP：与 `disconnect_device` 及瞬态工具相同的参数； CLI：`--keep-settings` / `--clear-settings`）：

- **`keep_settings=True`** 将每台摄像机的设置按会话结束时的状态保存至其内存中（UserSet2，即启动设置）。 摄像头仍保持自由运行状态（`TriggerMode` `Off`），同步线接线保持原状。 没有 UserSet2 的摄像机将设置保存到 UserSet1 中；这台计算机会记住这一点，因此 Chloros 的阵列自愈功能——该功能会重置偏离 Chloros 基准值的 UserSet1——不会对其进行干预。
- **`clear_settings=True`** 将每台摄像头的内存重置为出厂设置：即 Chloros 的基准值，该值由其阵列自愈机制确定 ——通过`TriggerMode`、`Off`、`TriggerSource`、`Line2`固化的出厂默认值，自动曝光，以及RGB摄像机校准后的白平衡 ——在 UserSet1 中保留，并将其设为启动集。
- 不可同时使用 （`ValueError`；后端返回400）。响应中包含`camera_memory`，每台相机对应`{"saved_to": "UserSet2" | "UserSet1" | None, "errors": [...]}`或`{"cleared": bool, "errors": [...]}` ——`force_disconnect()`将其返回；对于任务，则随附`stop()`。

该应用不提供这些。

**每条退出路径都会将摄像头恢复原状。**恢复原状操作会在每条退出路径上执行：`CameraSession.disconnect()`（应用程序的摄像头**X**）、应用程序的**X** 位于数组中或任何数组成员上（整个数组将被释放），关闭应用程序（后端有最多 20 秒的时间将所有摄像头、数组和正在运行的连续录制恢复原状；仍在运行的临时函数或单次调用将在下一个安全点停止，首先将摄像头恢复原状， 最多持续8秒，并返回HTTP 503 `SHUTTING_DOWN`；仍在运行的`connect_array()`会在处理下一台摄像头之前停止，并先恢复其已打开的摄像头，最多持续10秒，并抛出`ChlorosConnectError`（HTTP 503 `SHUTTING_DOWN`）；而在关机期间到达的请求则不会打开任何摄像头：HTTP 503 `SHUTTING_DOWN`），以及被终止或终端关闭的 `chloros-cli` 命令（其任务会被停止，且摄像头会首先被还原）。发生崩溃或被强制终止的后端无法还原任何内容；重启会将每个摄像头恢复到其内存中。**项目会自行设置其摄像头。** 当一个数组通过项目连接时——无论是应用程序的打开项目，还是一个 `ChlorosProject`， 该项目会向后端（`project_path`）声明自身名称——每个成员都会获得项目为其保存的设置：曝光、增益或自动曝光（AE）目标（被设为“保持”状态，因此自动曝光功能不会对其进行调整），以及其白平衡模式（Chloros 的校正； 相机自身的白平衡将保持关闭），无论相机内存中存储了什么内容。

### 拍摄模式与录制器

数组界面与 GUI 拍摄面板相对应：单张 / 连拍 / 间隔 / 最快快门模式，外加两个录制器（实时合成视频和原始连拍 → 离线重处理）。

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Continuous — every synced group for 60 s, saved by the backend itself
    with arr.continuous("run1/", processing="raw", duration_s=60) as run:
        run.wait()
    print(run.result["captures"], run.result["captures_per_s"],
          run.result["stop_reason"])

    # Combined-index video/GIF recorder: the GUI's combined view, or with no
    # GUI open the array's NDVI (pass expr="..." for another index)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    if rec.result and rec.result.get("error"):   # leaving the block does not raise
        print("recording failed:", rec.result["error"])
    else:
        print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**是 SDK 的连续/间隔循环。由于没有 `Ctrl+C` 可在脚本中中断该循环， 您**必须**传入 `count` 和/或 `duration_s`（达到其中任意一个即停止）。`interval_s` 从每次循环的开始处开始计时 （与 GUI 一致）。连续迭代（`interval_s=0`）会串联 `group_seq` → `after_seq`，因此它们是数组中连续的同步组速率。其余关键字参数将直接传递给 `capture()`。
- **`continuous`**是后端自身的连续捕获功能——即 GUI 中的“连续”模式。**当您希望以高帧率将每个触发事件记录到磁盘时，请使用此选项而非 `capture_repeated()`：** `capture_repeated()` 会从您的脚本中针对每个组发起一次 HTTP 请求，而 `continuous()` 则在后端内部保持两个捕获任务同时进行，组之间无需往返传输。 在 9 台摄像头阵列上以 ROI 512 进行测试：73–77 次捕获/秒，而该阵列的发布速率约为 78 次。请保留 `capture_repeated()` 用于间隔捕获（`interval_s`） 以及用于逐帧检查的捕获。 在 Windows 系统上，为实现高帧率，请将后端从 Microsoft Defender 中排除（参见 [步骤 1](#step-1--install-the-chloros-platform-package)）。
- **`record`** 属于 *监控级*： 它记录渲染后的实时组合索引复合视图。 当 GUI 的组合视图打开时，它会记录显示的视图内容；若未打开任何视图，后端会自行渲染视图：若您传入视图，则为 `expr`；否则，即为数组的 NDVI（最接近 660 nm 和 850 nm 的红光和 NIR 波段， 正如实时视图所选取的那样），一旦数组对齐后。在 `expr` 中按数组命名的名称命名波段： 对于 M3C 相机，请按数组中的命名方式命名波段（`<band>_<filter>`，`NIR_850_RGN`、`Red_660_RGN`、`Orange_615_OCN`）， `Mono_<filter>` 用于 M3M（`Mono_850`）；简短颜色名称（`NIR`、`Red`） 仅在数组中有一条波段具有该名称时才有效。录制过程中打开的GUI视图会在其保持打开状态期间被录入。若录制未写入任何帧，则会因`error`和`"no-frames: …"`报错（数组可能仍在对齐或稳定曝光）， 此时会触发 `stop()` 异常（参见 [`RecorderHandle`](#recorderhandle)）。每个阵列仅允许运行一个复合记录器（若已有记录器正在运行，则会触发异常）。
- **`burst` → `build_video`** 属于 *分析级*： `burst` 以抓取循环的满速率写入原始帧 + 每帧清单 + 每个独立 DLS 读取对应一个 `.daq`（位于 `<output>/bursts/<base>/` 之下）（无链式处理， 不使用exiftool，无实时预览）。`build_video`将每帧与最接近的`.daq`进行时间匹配，并重新运行导入管道中的辐射度/反射率/指数处理链。`products` 是一个 `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` 列表（默认：组合指数）。`burst().stop()` 还会自动触发一次尽最大努力的组合-指数构建，并在停止结果中作为 `build_job` 返回。

##### `build_video` 结果

| 键 | 含义 |
| --- | --- |
| `success` | 仅当所有请求的产品均按要求构建完成时，`True` 才会出现：至少有一个输出，且 `skipped` 中无内容， 且不包含 `degraded`。 |
| `outputs` | 已写入的视频（和 GIF）路径。 |
| `errors` | 问题描述（文本形式）。 |
| `skipped` | 对于每个未生成视频的请求产品，生成一个 `{"serial", "level", "reason"}` —— 可能是摄像头的 （该相机无校准包、无反射率的DLS读数、处理链失败）或组合索引的（包含`"kind": "combined"`、`serial`和`None` （各相机中无红光/NIR波段组合，且无同时包含这两个波段辐射度数据的组）。任何条目都会生成 `success` `False`；其余视频仍会被写入并列出。 |
| `substitutions` | 相机渲染级别与请求不符，及原因说明——当RGB相机请求辐射度/反射率时，若其校准包可用，则返回经过白平衡处理的显示视频。这不属于错误。 |
| `notes` | 信息性文本，非错误——例如：未指定单个摄像机索引表达式的摄像机没有索引视频，或RGB摄像机的显示视频未进行白平衡（无校准包，或校准包中未包含出厂白平衡），并标明摄像机名称。 这不属于错误。 |
| `degraded` | 当已生成产品但与请求不符时：`True` 表示离线组合指数为内置的红光/NIR归一化差值， 因此配置的索引表达式不会被评估。导致 `success` `False`。 |
| `combined` | 组合索引摘要： 使用的波段、LUT范围、生成的帧。 |
| `clamp_masks` | 每个帧校准 TIFF 文件（`save_tiffs=True`）的 [去混色钳位掩膜](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) 路径，位于 TIFF 文件所在 `<level>/` 文件夹旁的 `clamp_masks/<level>/` 中，每个发生像素钳位的 TIFF 对应一个路径。视频文件不包含此类信息。 |
| `clamp_fractions` | 视频中替代蒙版的信息：每个视频产品对应一个 `{"kind", "serial", "level", "frames"}` （`kind` `"combined"` 用于组合索引中的成员相机），`frames` 是一个列表，其中包含 `[frame, R, G, B]` —— 该帧中，解混操作在 R/G/B 通道所处的色带内被裁剪的像素比例 (`Chloros:Band{R,G,B}UnmixClampFraction`)。|

#### `RecorderHandle`

由 `ArraySession.record()` 和 `ArraySession.burst()` 返回。可将其用作上下文管理器，以便在范围退出时自动停止，或手动控制。

| 成员 | 描述 |
| --- | --- |
| `job_id` | 后端作业 ID（字符串）。 |
| `kind` | `"composite"` （来自 `record`）或 `"raw"`（来自 `burst`）。 |
| `start_stats` | `start` 调用返回的字典。 |
| `result` | 运行期间返回 `None`；停止后返回最终的停止结果字典。 |
| `stats(timeout=10.0)` | 实时任务统计信息（写入帧数、实际帧率、已用时间）。 |
| `stop(timeout=60.0)` | 停止录制器；返回并缓存最终结果（若停止请求本身失败，则 `.result` 保持为 `None`）。 对于失败的 `record()` 录制——即未包含任何已写入帧的录制（`error` `"no-frames: …"`） ——它会将结果缓存到 `.result`，随后抛出 `ChlorosConnectError` 异常。该操作具有幂等性（第二次调用将返回缓存结果，且不抛出异常）。 离开 `with` 代码块会停止记录器而不会抛出异常，因此请在其之后检查 `rec.result.get("error")`。 |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

#### `ContinuousHandle`

由 `ArraySession.continuous()` 返回。将其用作上下文管理器 ——若此时仍处于运行状态，退出 `with` 代码块将停止捕获——或通过 `status()` / `wait()` / `stop()` 进行控制。每次仅针对每个数组运行一次连续捕获。

| 成员 | 描述 |
| --- | --- |
| `start_status` | 启动调用返回的状态字典。 |
| `result` | 运行期间为 `None`；捕获结束后的最终状态字典（由 `stop()` 设置， 或由察觉到捕获结束的 `status()` / `wait()` 设置）。 |
| `status(timeout=10.0)` | 实时状态字典（字段如下）。 |
| `wait(timeout=None, poll_s=0.5)` | 阻塞直至捕获自行结束——达到 `max_captures` 或 `duration_s`、数组耗尽、磁盘已满或发生错误——并返回最终状态。 使用 `timeout` 时，即使捕获仍在运行，也会返回指定秒数后的最新状态：请参阅 `running`。 |
| `stop(timeout=30.0)` | 停止捕获并返回其最终状态。幂等（第二次调用将返回缓存的结果）。 |

状态字段：

| 字段 | 含义 |
| --- | --- |
| `running` | 捕获运行期间为真。 |
| `captures` | 已保存同步组。 |
| `files` | 已写入文件。 |
| `failed_files` | 无法写入的文件（如 [`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed) 所示）. |
| `failed_captures` | 整体失败的捕获尝试，包括捕获重试的情况（数组未能及时生成新组时出现 HTTP 503 错误）。 |
| `skipped_groups` | 数组已发布但未保存的组（捕获进度滞后）。 |
| `captures_per_s` | 自开始以来的平均保存率。 |
| `elapsed_s` / `started_at` | 当前运行时间（秒）；开始时间（Unix 纪元秒）。 |
| `last_group_seq` | 最新已同步并保存的组。 |
| `last_error` | 最新错误，或 `None`。 |
| `stop_reason` | 已结束一次：`count`（达到`max_captures`），`duration` （已到达 `duration_s`），`stopped`（调用了 `stop()`），`array_gone`（数组已断开连接），`error`（连续20次失败，或后端拒绝的请求），`disk_full`（驱动器无空间容纳的文件）。`None`在运行中。 |
| `max_captures` / `duration_s` | 启动时的边界（`0` = 无）。 |
| `defender_cores` | 过去 10 秒内 Microsoft Defender 占用的 CPU 核心数；`None` 直到测得该值，且在未安装 Defender 的主机上始终为 `None`。 |
| `defender_warned` | 一旦捕获日志记录到 Defender 导致其运行变慢（占用 1 个或更多核心），则返回 true。将后端排除在 Defender 扫描之外——请参阅 [步骤 1](#step-1--install-the-chloros-platform-package)。 |

```python
run = arr.continuous("captures/run1", processing="raw", max_captures=5000)
try:
    while True:
        st = run.wait(timeout=5)              # returns every 5 s while running
        print(f"{st['captures']} captures, {st['captures_per_s']:.1f}/s, "
              f"{st['skipped_groups']} skipped")
        if not st["running"]:
            break
finally:
    final = run.stop()                        # no-op once it has ended
if final["failed_files"] or final["stop_reason"] in ("disk_full", "error"):
    raise SystemExit(f"continuous capture: {final['stop_reason']}: "
                     f"{final['last_error']}")
```

### 在您自己的处理管道中使用实时帧

若要将同步帧输入到您自己的代码中（例如机器学习、自定义查看器或自建录像机），请使用 `ArraySession.frame_stream()`。每个时钟周期都会生成一个 `GrabResult`： `{serial: numpy array}`，**全部由单次触发生成**，且该触发的 ID 存储在 `.group_id` 中。其实现方式与项目 API 中的 `ArrayHandle.frame_stream()` 相同， 因此 [每次读取一个触发器](#one-trigger-per-read--group_id)、[`band`](#exporting-a-single-band--band)、[`align`](#aligning-the-streams--align) 以及 [`bit_depth`](#bit_depth--trading-precision-for-bandwidth) 下的所有内容均适用于二者。

```python
import chloros_sdk
from chloros_sdk import FrameFetchError

serials = ["213800234", "214000533", "214701288", "214701292"]   # master first

with chloros_sdk.connect_array(serials) as arr:
    try:
        for frames in arr.frame_stream(processing="raw", fps=10, duration=300,
                                       with_metadata=True):
            if not frames:              # no camera answered this tick
                continue
            if frames.errors:           # cameras missing from this trigger
                print("left out:", {s: str(e) for s, e in frames.errors.items()})
            for serial, (img, hdr) in frames.items():
                # img: numpy array (raw: one Bayer or mono plane)
                # hdr: group_id, frame_id, timestamp_ns, exposure_us, gain_db, ...
                my_pipeline(frames.group_id, serial, img, hdr["timestamp_ns"])
    except FrameFetchError as exc:      # the stream cannot continue
        print("stream ended:", exc.status, exc)
```

- **每个输出的集合都是一个新的触发器。** `frames.group_id` 在流中绝不会重复；当您请求的 `fps` 数量超过数组的运行速度时，流会等待下一个触发器，因此运行速度将以两者中较慢者为准。
- **缺失的摄像头会被标注名称，不会被补全。** 在该触发点无法被服务的摄像头将从字典中缺失，并列在 `frames.errors` 中。
- **它会以明显的方式结束。** 若发生拒绝，`FrameFetchError` 将在下一个时间步长重复；或在 60 秒内未收到帧，或 60 秒内未收到新触发信号后，该流将终止。**性能表现。**`frame_stream()` 通过本地 HTTP 连接传输完整的 NumPy 帧，峰值速率接近**170 MB/s**。 这足以满足原始帧、分箱数据或 ROI 数组的实时处理需求，但无法满足以全分辨率处理每个触发信号的要求——若要保留**每个** 触发信号，需使用 [`continuous()`](#continuoushandle)。测试基于四摄像头阵列（FOCN、FRGN、FRGB、F450），采用紧凑传输模式；每个时钟周期均为一组完整的单触发数据：

| | 全传感器（阵列 14.2–14.4 fps） | 2 × 2 像素合并（阵列 ~33 fps） |
| --- | --- | --- |
| `frame_stream(processing="raw")` | 6.7 组/s | 22.2 组/s |
| `frame_stream(processing="debayered")` | 2.3 组/秒 | 9.3 组/秒 |
| `frame_stream(processing="display")` | 0.8 组/秒 | 3.0 组/秒 |
| 每台摄像机 MJPEG， 720p 4路流 | 每路13.5帧/秒；每帧均与触发信号匹配 | 每路约25帧/秒；4路流中约45%的触发信号存在 |

请使用 `processing="raw"` 进行吞吐量处理和本地去马赛克；`debayered` 级别还会去除校准包中的暗偏移， 而 `raw` 的去马赛克处理则不会。

#### 每台摄像机的 MJPEG 流（不支持 RTSP）

**不支持 RTSP。** 流媒体接口为基于 HTTP 的 MJPEG（`multipart/x-mixed-replace`，每部分一个 JPEG，每部分带有 `Content-Length`），每台摄像头一个 URL：

| URL | 提供内容 |
| --- | --- |
| `GET /api/camera/stream/<serial>` | 8 位预览，最高约 30 fps。对于阵列成员，`max_h` （默认 720，范围 64–2160）和 `jpeg_q`（默认 80）分别用于设置尺寸和质量。 |
| `GET /api/camera/<serial>/stream-annotated` | 相同的预览图像，但叠加了 GUI 辅助线（斑马线、网格、十字线、直方图、峰值显示、点测光）。`ChlorosLocal.camera_stream()` 会为您读取这些信息。 |
| `GET /api/camera/<serial>/stream-reflectance` | 每波段 8 位反射率 — 参见 [实时反射率流](#live-reflectance-stream-mjpeg)。 |

- **触发头。**对于数组成员，三个数据流的每个部分都包含 `X-Chloros-Group-Id`、 `X-Chloros-Frame-Id` 和 `X-Chloros-Timestamp-Ns`。来自不同相机但具有相同 `X-Chloros-Group-Id` 的片段属于同一触发器： 收集各部分并按该标头分组以重新组装成一组。独立摄像机没有分组：其预览部分不携带任何这些标头 （其反射率部件携带一个空的 `X-Chloros-Group-Id`）。
- **预览流在负载下会丢帧。** 它提供最新帧并跳过任何未能及时发送的帧，因此在高数组速率下，并非每个触发信号都能到达每台摄像机的流中（参见上表）。对于每个触发信号，请使用 `frame_stream()` 或 `continuous()`。
- **当阵列的合并实时流处于打开状态时，`/api/camera/stream/<serial>` 会暂停**（`/api/camera/array/stream/<array_id>` — 由 GUI 的阵列视图打开该流）。随后，每个成员的流会发送一个“已暂停（阵列流活动）”占位帧，该帧不包含 `X-Chloros-*` 头， 此后直至组合流关闭前不再发送任何数据；随后流会自动恢复。`stream-annotated` 和 `stream-reflectance` 不会暂停。当脚本读取各摄像头的预览流时，请关闭 GUI 的阵列视图。
- **若需获取数值（而非图片），请使用 `frame_stream()`。** JPEG 是有损压缩且为 8 位格式；若路径中未包含编解码器，`frame_stream()` 将返回相同的结果。

### 无人机测绘的间隔捕获

测绘飞行需要每飞行几米生成一组同步图像，而非数组能生成的每一帧。有两个设置控制此功能：**`target_fps`**用于设置相机拍摄帧的频率，而**`capture_repeated(interval_s=…)`** 用于设置保存帧的频率。**相机在两次保存之间会持续进行流式传输。** 每次间隔会保存最新的同步图像组；其间的帧不会被写入。自动曝光会在每帧进行调整，因此保存发生时曝光值已趋于稳定。不存在相机在拍摄间隙停止运行、每次拍摄时才被唤醒的模式：否则每次拍摄都必须等待数据流启动并等待曝光稳定。**将帧率固定为间隔所需的速度。**保持自动模式时，数组将以主机和网络所能支持的最高速率运行。将其固定为较低速率可降低摄像头发热、网络负载和主机 CPU 占用，同时不改变保存的内容。选择一个能达到**每个间隔 2–4 帧** 的速率：

- 每次保存都会在请求后等待下一帧，因此保存的帧组最终会在计划时间点之后最多一个帧周期内完成。更高的帧率能将这一时间差缩小。
- 自动曝光每帧调整一次，因此过低的帧率会减缓曝光对移动云层的响应速度。

**根据飞行计划计算间隔：**

```
spacing    = along-track footprint × (1 − forward overlap)
interval_s = spacing / ground speed
```

例如，60 米的拍摄范围，前向重叠率为 80%，间距为 12 米；以 10 米/秒的速度飞行时，参数为 `interval_s=1.2`， 而固定为 `target_fps=3` 时，每个间隔内可获得 3 至 4 帧。拍摄基于时间触发：SDK 没有距离或 GPS 触发模式，因此在测绘航线上需保持稳定的地面速度。

**在飞行器上保存原始数据，稍后进行校准。** `interval_s`是从每次保存开始时测算的，因此如果某次保存耗时超过间隔时间，会导致下次保存延迟开始，从而造成间距漂移。 同时保存原始数据和光传感器读数（`processing="raw"`；该读数默认会被保存）可确保每次保存时间较短，并使后续每帧数据都能转换为反射率。

```python
import chloros_sdk

INTERVAL_S = 1.2      # from the flight plan
FLIGHT_S = 20 * 60    # bounds the loop; it also stops at count= if given

with chloros_sdk.connect_array(serials, target_fps=3.0) as arr:
    groups = arr.capture_repeated(
        "survey/", interval_s=INTERVAL_S, duration_s=FLIGHT_S,
        processing="raw", render_index=False,
        on_capture=lambda i, r: print(f"group {i}: {len(r)} frames"))
```

- **`connect_array(target_fps=…)`**从开始时就锁定速率；**`arr.set_target_fps(fps)`**可在飞行途中更改速率而无需重新连接，而**`arr.set_target_fps(None)`** 将其恢复为自动模式。若速率超过主机和网络的承载能力，则会降至该上限——请在结果中检查 `clamped`——且最低速率为 0.1 fps。
- **请勿使用 `burst()` 来稀疏帧。** 它会写入数组生成的每一帧；虽然它确实会跟随一个固定的 `target_fps`，因此只有在需要每一帧时，它才是合适的工具。
- **`target_fps` 无法修复损坏的帧。** 降低该值会减少生成的帧数，而非降低每帧的传输速度；若遇到 GVSP 损坏的帧，请改降低 `wire_ceiling_mbps`（参见 [数组健康状况](#array-health--which-subsystem-is-losing-frames)）。

```python
r = arr.set_target_fps(5.0)
if r["clamped"]:
    print(f"asked for {r['requested']} fps, running at {r['target_fps']}")
```

### 连接到已连接的数组 — `attach_array`

如果数组已处于活动状态 （由GUI打开，或先前SDK会话已调用`connect_array`），请使用`attach_array`获取其句柄，而非重新连接。在此情况下，`connect_array`始终会报错“相机  <sn>已存在于数组<id>中”的错误，因为针对池中成员发送POST请求`/array/connect`不具备幂等性； `attach_array` 会读取 `/api/camera/array/list`，并通过 array_id 或序列号进行匹配。

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

模式：与桌面图形用户界面（GUI）共租户的SDK脚本应首先尝试`attach_array`，如果池中尚无数组，则回退到`connect_array`。

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **已连接的数组是借用的，而非拥有的。** 来自 `attach_array` 的句柄绝不会释放该阵列：无论是 `with` 代码块的退出，还是显式的 `disconnect()` 调用，都不会释放它（显式调用会在 stderr 上打印一行信息，说明 让数组继续运行）。这与 `CameraSession` / `DAQSensorSession` 的规则相同。因此，无论是否使用 `with`，与 GUI 共存都是安全的：
>
> ```python
> with chloros_sdk.attach_array(serials) as arr:
>     arr.capture("output/", processing="reflectance")
> # … the array stays up for the GUI
> ```
>
> 当你确实需要结束连接时——例如，为了使用另一个 `wire_ceiling_mbps` 重新连接——请调用 `arr.force_disconnect()`。如果在已连接的句柄上直接调用普通 `disconnect()` 后尝试重新连接，将会因 “相机<sn>已存在于数组中<id>”，因为资源尚未被释放。

### 网络分析辅助工具

在打开数组之前非常有用——可预测您拟定的设置是否适用：

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    # width / height are sensor pixels BEFORE binning (what connect_array
    # takes with binning=); out_width / out_height is what is streamed.
    print(f"Connect with width={r['width']} height={r['height']} "
          f"binning={r['binning']} ({r['out_width']}x{r['out_height']} streamed)")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

```python
analyze_array_network(
    master_serial,
    slave_serials=None,
    width=2048, height=1536,              # sensor pixels before binning
    pixel_format=None,                    # None = what connect applies (Mono12 / BayerRG12)
    binning=1,                            # 1 | 2 | 4
    force_tier=None,
    packed="auto",                        # "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",
    timeout=30.0,
) -> dict
```

它永远不会启动后端：若后端未响应，将触发 `ChlorosConnectError` 错误；而被拒绝的请求则会作为结果返回。错误的 `packed` 将触发 `ValueError` 错误。

`status` 是 `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` （否则为 `error` —— 包括被拒绝的请求：未登录，`error_code: AUTH_REQUIRED`，或套餐低于 SDK 最低标准，`PLAN_UPGRADE_REQUIRED`）。`auto_capped_fps` 表示所请求的分辨率仅在受限的触发速率下才适合 RX 环——保持该分辨率并将 `target_fps=result["recommended"]["recommended_target_fps"]` 传递至 `connect_array`（参见 [示例 6](#6-capability-probe-before-connecting-a-4-cam-array））。`auto_shrunk` 仅在无法以任何帧率传输所请求帧时出现；连接操作绝不会自行缩减帧，因此请自行应用 `recommended`。

**如何解读投影**（与GUI中的“阵列设置”面板采用相同模型）：

- **突发（`frame_bytes_total`）是按每台摄像头的实际像素格式进行逐台相加的。**单色**M3M**摄像头传输 Mono12（2 位/像素），或通过 `pixel_format` 采用其深度通道的单色格式 （`"BayerRG8"` 即 `Mono8`， 1 B/px)，因此若未启用该功能，由三台单色摄像头组成的4摄像头全分辨率帧大小为**~25 MB**，而非全8位假设下的~12.6 MB。后端会根据型号解析每台摄像头的格式。
- **通量 (`burst_fits_nic_ring`) 具有排水意识**， 而非“整突发与环路”的区别：当主机从接收环路中读取数据的速度快于摄像头填充环路的速度时，模拟发射机制便能适配。即使突发数据量超过环路容量，10G主机搭配1 GbE摄像头仍能**支持**全分辨率数据； 而 1 GbE 主机则会阻塞（`needs_force_slip` / `auto_shrunk`）。
- **`achievable_fps_max` 是一个保守的串行读取上限值** — `max(readout+emit, N×emit)` 将每个 CAM 的发射速率限制在 1 GbE 相机链路带宽内，且与曝光时间无关。 例如，对于 4 台摄像头的全分辨率 12 位阵列，帧率约为 2.8 fps（与运行时测得的 ~2.7–3.0 相符）。 完整模型：[CLI参考 → 阵列帧率与连拍模型](cli-reference.md#array-fps--burst-model)。
- **连续捕获结束后，投影结果反映其最终保持的帧率。** 30 秒或更长时间的自动速率捕获将记录该计算机在此阵列大小下最终稳定的帧率；帧率控制器对更高帧率的短暂尝试将被忽略。此后，根据 `array-connect`'行开始，当帧率低于模型设定值时，`connect_array()`的分析结果及“数组设置”面板均会引用该帧率，并说明在`cycle_limiter`中限制该帧率的因素：`"clock"` （数组以最终稳定的速率运行，抓取循环等待其触发时钟），`"loop"`（抓取循环时间耗尽）， `"capture"`（捕获滞后并跳过了若干组）或 `"measured"`（未明确说明具体情况）。模型的数值见 `wire_fps_max`。 该数组仍会攀升至线路所能承载的水平。
- **超额订阅（`oversubscribed: true`）意味着无论相机达到何种帧率，线路都无法承载该数组** ——帧率字段（`achievable_fps_max` / `fps_bright` / `fps_dark`）读数为0，且像素合并无法解决此问题 （该操作仅降低每帧字节数，而非每秒定时传输的字节数）。仅因网络带宽不足不会导致此问题：此时帧率会下降，若无法将摄像机定时速率调低至足够低，则状态将变为 `slip-emit-and-capture` （串行）状态。解决方法包括减少摄像头数量、使用巨型帧或更换更快的网卡；`max_cams_collision_safe` 报告了您的网络线缆和摄像头的性能上限。 响应中还包含 `aggregate_demand_bps`、`collision_safe_ceiling_bps` 和 `per_cam_floor_bps`。参见 [超额订阅](#over-subscription-the-per-cam-floor)。

### 发现与列出

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

它们各自接受仅含关键字的 `backend_url=` 和 `timeout=30.0`，并返回一个字典列表。 `discover_lattice_cameras()`（`serial`、`ip`、 `mac`、`model`、`lattice_model`、`vendor`、`connected`、 `array_id`, …) 也会调用 `auto_start_backend=True`；而 `list_*` 的调用则永远不会启动后端。

---

## 智能自动曝光（Smart-AE）/ 智能抓拍（Smart-Capture）

LATTICE 阵列一经连接便会在后台持续运行自动曝光（AE），但新对焦的场景需要片刻时间才能收敛。**智能捕获** 是一项便捷的封装功能：它会轮询每台摄像机的曝光值，等待阵列在指定时间窗口内保持稳定，然后触发捕获。这相当于图形用户界面（GUI）操作：桌面应用中的“智能”捕获按钮调用的正是同一个后端端点。

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

通过 `ChlorosProject`（下一节）进行控制时，您将获得更多调节选项：

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

智能自动曝光策略默认较为保守。对于要求严格的辐射测量工作，请收紧 `exposure_tolerance_pct` 参数；对于变化迅速的场景，若仅需“大致准确”的测量结果，则可放宽该参数。

---

## DAQ 传感器会话

用于光谱传感器的持久后端池（通过 USB 的 DAQ-U、通过 BLE 的 DAQ-M、通过以太网的 DAQ-E）。与相机功能一致：智能检测、池资源复用、幂等连接。

### 智能检测（零配置）

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

优先级：以太网 → BLE → USB。传递任意一个显式提示即可锁定传输协议。

### 锁定传输协议

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# A specific sensor by id (as list_daq_sensors() shows it; ':' or '-' alike)
daq = chloros_sdk.connect_daq_sensor(sensor_id="CB-7C-A7-5A-73")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

`sensor_id=`指定一个传感器，且仅该传感器会被打开。系统会将其与已打开的传感器进行匹配，随后向 DAQ-E 查询其主机名或 IP 地址（对于网络扫描中仅能通过 IP 识别的 DAQ-E，系统会以只读方式查询其名称、MAC 地址和序列号， 因此通过上述任一方式均可定位该设备），以及DAQ-M的BLE MAC地址，还有DAQ-U或DAQ-M在打开后报告的ID（为验证而打开的任何其他设备都会被重新关闭）。若未找到， `connect_daq_sensor`将触发`ChlorosConnectError`，列出所发现的设备；同时还会包含一个地址（`port=` / `mac=` / `eth_host=`）中包含不同传感器时，也会触发异常。该函数绝不会退而采用任何可见的传感器——这正是零标志调用的用途。</id></sn></id></sn>

### `connect_daq_sensor()` 签名

```python
connect_daq_sensor(
    *,                                    # every argument is keyword-only
    sensor_id=None,                       # one named sensor (see above)
    transport=None,                       # "usb" | "ble" | "eth"; None = smart-detect
    port=None,                            # DAQ-U serial port ("COM3", "/dev/ttyUSB0")
    mac=None,                             # DAQ-M BLE MAC (implies transport="ble")
    eth_host=None,                        # DAQ-E hostname or IP (implies transport="eth")
    integration_time=None,                # ms; None = the sensor's own
    frame_avg=None,
    enable_ae=None,
    cap_id=None,                          # cap correction: "sunshine_cosine", "fov_30", ..., "none" (bare)
    start_streaming=True,
    backend_url="http://127.0.0.1:5000",
    timeout=60.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> DAQSensorSession
```

已处于传感器池中的传感器将按原样返回（即“借用”状态）：`integration_time`、`frame_avg` 和 `enable_ae` 不会应用于该传感器。 请先断开连接，然后使用您的设置打开它。`cap_id` 是个例外：在已借用的传感器上，它通过 `set_cap()` 进行设置。若省略该参数，则采用后端的默认值（参见方法表下方的注释）。

### `DAQSensorSession` 方法

| 方法 | 描述 |
| --- | --- |
| `status(timeout=10.0)` | 数据池条目摘要（流式传输/记录状态、波长范围、校准 SHA、积分时间、frame_avg、AE 状态）。对于数据流已静默的 DAQ-E：`reconnect` （当后端处于自动重连状态时：`state`、`attempts`、`next_attempt_in_s`、`last_error`；`None` 表示正在传输数据），`stream_lost_s`、`reconnects` 和 `last_reconnect`，以及 `record_gaps` 表示 正在进行的录制；`rx_stream_fault`表示重连状态，直至收到第一个频谱帧。 |
| `latest(*, n=1, timeout=10.0)` | 返回最多 N 个最近的光谱帧（每个 `ts_ns`、`spectrum`、 `is_saturated`、`integration_time`、`x`、`y`、`z`）. |
| `stream_start()` / `stream_stop()` | 恢复 / 暂停流式传输 （句柄保持打开状态）。 |
| `record_start(*, output_dir=None, device_name=None, timeout=15.0)` | 开始录制 .daq 文件。返回文件路径。对于未配备 AWS 校准包的 DAQ-U/M 设备，此操作将被拒绝（DAQ-E 除外）。 |
| `record_stop()` | 停止录制。返回 `{path, rows, warning, gaps}`；对于干净的录制，`warning` 即为 `None`， 否则返回后端诊断信息（例如文件为空或流中存在间隙），该信息也会输出到 stderr。 `gaps` 列出了流未输出任何内容的每个时间段：`start_utc`、`end_utc`（`None`：在停止时仍丢失）， `duration_s`, `reconnected`, `marked_in_file`。连续记录时该字段为空。 |
| `set_cap(cap_id, timeout=15.0)` | 设置从下一个读数开始应用的峰值校正配置文件： `"sunshine_cosine"`、`"fov_15"` ... `"fov_90"`，或对于裸传感器时为 `"none"`（或 `None`）。返回传感器的摘要，可通过 `cap_id` 读取该摘要。DAQ-E-S 无论被查询什么，都会保持 `as_recorded` 的值不变（其扩散器为内置式）。若校准曲线 无法加载时触发 `ChlorosConnectError`。CLI：`chloros-cli daq pool-set-cap`。 |
| `enable_ptp(timeout=15.0)` | 仅限 DAQ-E：对于 PTP 处于关闭状态的设备，重新启用 PTP （`status()["rx_ptp_enabled"]`即为`False`；该设置保存在设备上，因此PTP将保持关闭状态直至重新启用）。 返回更新后的状态。对于 DAQ-U / DAQ-M 设备，或当设备未确认时，将触发 `ChlorosConnectError` 异常。Chloros 绝不会自行将设备重新启用，因为关闭状态可能是用户有意为之。 CLI：`chloros-cli daq pool-enable-ptp`。 |
| `device_info(timeout=15.0)` | 仅限 DAQ-E：设备自身的设置。`firmware` (`running`、`bundled`、`update_available`)， `ip` 和 `ip_mode`（`dhcp` / `linklocal` / `static`： 当前运行的设置），`network`（已保存的设置：`mode`，加上 `ip` / `mask` / `gateway`，处于静态时），`network_change_pending` （在设备运行包含 `set_network(..., reboot=False)` 的保存设置之前为真），`auto_stream`、`ptp`。 当设备版本无法读取时，`firmware["update_available"]`将变为`None`。当设备的流数据未到达本计算机时，系统会在建立新连接时读取该数据，词典会添加`stream_silent`（`True`）、`stream_fault` 和 `reconnect`；若设备未响应，则触发CLI：`pool-info`。 |
| `set_network(mode="auto", *, ip=None, mask=None, gateway=None, reboot=True, force=False, comeback_timeout=45.0)` | DAQ-E 仅限：DHCP (`"auto"`) 或静态地址。重启设备以应用设置并重新连接；当前会话将继续运行，且 `address` 将跟随该设备。`reboot=False` 仅保存该设置。 参见 [DAQ-E 设备设置](#daq-e-device-settings)。CLI：`pool-set-network`。|
| `set_auto_stream(enabled, timeout=15.0)` | 仅限 DAQ-E：开机时在未连接软件的情况下进行数据流传输（设置保存在设备上）。返回已确认的设置。CLI：`pool-set-auto-stream`。 |
| `reboot(*, comeback_timeout=45.0)` | 仅限 DAQ-E：重启设备并按其原有设置重新连接。CLI：`pool-reboot`。 |
| `update_firmware(*, comeback_timeout=45.0)` | 仅限 DAQ-E：将此 Chloros 版本随附的固件刷写到运行旧版本的设备上，然后重新连接（约需一分钟；请勿断电）。 返回 `updated`、`fw_before`、`fw_after`、`bundled` 以及 `warning`；当设备恢复速度较慢或恢复后显示为其他版本时；对于已更新至最新版本的设备，`updated` 对应 `False`。 当无法读取设备版本时，会触发 `ChlorosConnectError`。CLI：`pool-update-firmware`。 |
| `disconnect()` | 从池中释放。对于-op 操作。对于本会话打开的传感器，此操作为 `force_disconnect()`；若释放未获确认，则抛出 `ChlorosConnectError` 异常（`with` 代码块的退出 从不抛出异常）。 |
| `force_disconnect(timeout=30.0)` | 无论谁打开的传感器，均将其释放 ——例如，一个因脚本崩溃而留在池中的传感器，新脚本仅能将其作为已附加的句柄访问，其 `disconnect()` 不会执行任何操作。终止其他客户端的传感器将结束其正在进行的任何记录。在后端确认释放后，或当无内容可释放时 （传感器不在池中，或没有后端正在运行）；若释放超时或被后端拒绝，则抛出 `ChlorosConnectError` 异常。CLI：`chloros-cli daq pool-disconnect --sensor-id ID`。|

属性：`sensor_id`、`model`、`transport`、 `address`（紧随一个被 `set_network()` 重新寻址的 DAQ-E 之后）、`info` 和 `backend_url`。

> **请说明该电容的名称。** DAQ-U、DAQ-M和DAQ-E出厂时未配备电容，且Chloros无法检测是否安装了电容：未连接`cap_id`的传感器会获得`sunshine_cosine`，其记录数据显示这是基于 (`cap_id_source=auto_default`)。在该校正下，未安装保护帽的 DAQ-U 读数会高出约 30 倍，因此对于未安装保护帽的传感器，请传入 `cap_id="none"` （或 `set_cap("none")`），而安装其他盖子的测量曲线则不同。

### DAQ-E-S — 固定式扩散器和倾斜传感器

**DAQ-E-S** 是一款 DAQ-E，但有两处对软件而言至关重要的区别。其余所有功能——设备发现、连接、PTP、记录以及下文所述的设备设置——均与 DAQ-E 完全一致。**其扩散板为固定式，因此无需进行扩散板校正。** 该设备内置了 Sunshine 风格的扩散器，且在工厂校准测量时该扩散器已安装在设备上，因此该校准值已准确反映真正的向下辐射通量。 Chloros无论被要求什么，都会将其上限值设置为 `as_recorded`（无配置文件）：以该设备命名的上限值（`sunshine_cosine`、 `fov_*`、`none`）均会被忽略并显示提示信息，其记录数据中存储的是 `cap_id_source=model`，因此“假定存在校准文件”的警告对其永远不会触发。 也不要手动对其数据应用上限：若在顶部放置 `sunshine_cosine`，读数将高出约 11 倍。

**该传感器配备了一个三轴加速度计。** 阳光漫射器的余弦响应仅在设备水平放置时有效，因此倾斜的传感器会报告错误的辐照度——且反射率会继承该误差——且无其他症状。因此，每个 DAQ-E-S 读数都包含其姿态信息：

| 字段 | 含义 |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | 角度单位为度。当固件将角度标记为未定义（自由落体、轴失效）时，读数显示为 `None`。 |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | 以毫g为单位的加速度。 `mag_mg` 若数值远离 ~1000，则表示设备处于运动状态，该样本的角度数据无实际意义。 |
| `cal_applied` | 启用：坐标轴和角度以扩散器坐标系为基准（应用了设备的安装校准）。禁用：原始电路板坐标系，可能显示高达约 2.3° 的倾斜角，但这并非真实倾斜。 |
| `sample_age_ms` | 发送频谱时加速度计采样数据的采集时间。 |
| `temp_raw` | 未校准的芯片温度**趋势**——仅显示温差，绝非温度计读数。 |

`DAQSensorSession.latest()` 将其返回至每个帧的 `imu` 键下（在 DAQ-U、DAQ-M 或普通 DAQ-E 上为 `None`）。 从 DAQ-E-S 记录的 `.daq` 会将其相同值存储在 `imu_*` 列中（在 DAQ-U、DAQ-M 或普通 DAQ-E 上为 `imu_tilt_deg`、 `imu_roll_deg`、`imu_pitch_deg`、 `imu_mag_mg`、`imu_cal_applied`、…）中存储相同的数值，以便读者能够区分云层与三脚架被撞倒的情况。在其他所有型号中，这些列均为空。

姿态数据流在其自身的字段中进行报告，位于`status()` （`rx_imu_stream_fault`、`rx_imu_stream_faults`、`rx_last_imu_stream_fault`）中报告其自身字段，这些字段与光谱字段分开存储，因此加速度计故障绝不会被误读为记录中的缺口。 `discover_daq_sensors()` 和该会话的 `model` 显示为 `DAQ-E-S`。

### DAQ-E 设备设置

```python
with chloros_sdk.connect_daq_sensor(eth_host="daq-e-def330.local") as daq:
    info = daq.device_info()
    print(info["ip"], info["ip_mode"], info["firmware"])

    # A fixed address on the network this computer is on
    daq.set_network("static", ip="192.168.1.50", gateway="192.168.1.1")
    print(daq.address)              # 192.168.1.50: the session follows it

    if info["firmware"]["update_available"]:
        daq.update_firmware()
```

- **该静态地址必须位于本计算机的网络中。** `set_network` 会为子网中不包含本计算机：重启后将无法连接到传感器以将其恢复原状。仅当该计算机将被重新分配地址或路由至新子网时，才应通过 `force=True`。
- **`set_network`、`reboot` 和 `update_firmware` 会重启传感器**，并在记录期间触发 `ChlorosConnectError`， 且在 `comeback_timeout` 秒内未恢复时。传感器 ID 和 `daq-e-<id>.local` 主机名永远不会改变。
- **重启前刚开始的记录会被重启操作关闭并命名。** 当传感器恢复连接时，返回的字典中包含 `recording_closed`（文件路径） 以及一条说明此情况的 `warning`。若传感器未恢复连接，或固件传输失败，则由 `ChlorosConnectError` 进行相应提示：其消息以警告结尾，且会设置 `e.recording_closed` / `e.warning` 被设置（若未关闭任何录制任务，则同时设置 `None`）。待传感器恢复连接后，请重新启动录制。
- **仅传感器本身重新连接。** 重启后，设备必须响应其所在位置 ——并在 `set_network("auto")` 之后，还需以其 `.local` 名称——通过其原有的网络适配器（MAC）地址，报告其名称、主机名和型号； `address` 将成为响应的 IP 地址。以该名称发出的响应仅与该 MAC 地址相关联，因此，无论是通过路由器连接到传感器的计算机，还是来自 WSL2 的默认 NAT 网络，被 DHCP 移动的传感器都不会被重新连接 （`ChlorosConnectError`；请通过其新地址重新连接）。
- **固件更新需要传感器重新建立连接。** `update_firmware()` 向设备发出请求，设备会通过操作系统选定的端口向该计算机建立 TCP 连接以获取镜像。后端在可连接该设备的网卡上监听（若无法确定具体网卡，则在所有网卡上监听），并仅为该设备提供服务： 在持续等待期间，将拒绝任何其他连接（后端输出中显示 `[DAQ-FW] … refused a firmware connection from <ip>`）。若防火墙阻断了该连接，更新将因 `device never connected back on TCP <port>` 而失败。 Windows安装程序会为其添加相应规则（参见[步骤 1](#windows-exe)）；在其他防火墙上，请允许来自传感器子网的入站 TCP 连接至后端。
- **在 WSL2 环境下，请使用镜像网络**（`networkingMode=mirrored` 及 `.wslconfig`）：在默认的 NAT 模式下，所有静态地址都会被拒绝（WSL 自身的地址不在传感器的网络中），数据流无法到达 WSL，导致固件更新无法完成。
- 图形界面会与传感器保持独立连接：通过脚本重启会终止图形界面与传感器的会话。图形界面的设置与之相同（“光传感器”选项卡、传感器设置、网络）。

### 发现 — 查找用于连接的地址

`discover_daq_sensors()` 会扫描 USB / BLE / ETH 接口，查找您*可能*能打开的传感器。它是 `discover_lattice_cameras()` 的 DAQ 对应版本，也是获取 **DAQ-M 的 BLE MAC 地址** 的唯一途径 ——DAQ-E 具有主机名，DAQ-U 具有 COM 端口，但 MAC 地址既不会印在设备上，也不会由操作系统列出。

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               DAQ-U  {'vid': '0403', 'pid': '6001', 'manufacturer': 'FTDI'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| 字段 | 描述 |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM 端口 / BLE MAC / 主机名 — 传递给 `connect_daq_sensor` 作为 `port=` / `mac=` / `eth_host=`。 |
| `display` | 易于阅读的标签。 |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` \| `DAQ-E-S`， 或 `None`。以太网传感器为 `DAQ-E-S` 或 `DAQ-E`，因为在扫描期间，当被询问时，它会报告自身信息（只读）， （若未能及时响应，则显示为 `DAQ-E`）。当 USB 串行端口的桥接芯片是 DAQ-U 所使用的类型（FTDI、CH340、CP210x、Prolific）时，其读数为 `DAQ-U` —— 这可从端口描述中看出， 而非探针——而其他任何 USB 设备则显示为 `None`，该设备仍被列出，因此您可以将其作为 `port=` 传递。 没有 USB 供应商 ID 且不包含此类桥接芯片的串行端口（例如主板上的 UART，如 Linux 中的 `/dev/ttyS*`、蓝牙串行链接、Intel AMT）不能被视为 DAQ-U，因此未被列出。 |
| `extra` | 按传输方式的详细信息 (BLE 广播名称；USB `vid`、`pid`、`manufacturer`、`serial_number`；DAQ-E IP/固件/…）。空值被省略。 |

| 参数 | 默认值 | 描述 |
| --- | --- | --- |
| `transports` | 全部三个 | 限制扫描的序列（或 CSV 字符串）。当您明确知道所需内容时，建议传入该参数——BLE 是速度较慢的一环。 |
| `scan_timeout` | 5 | 每种传输方式的扫描窗口（以秒为单位）；后端会将其限制在 1–20 之间。 |
| `timeout` | 60.0 | 整个调用的 HTTP 上限（与 SDK 其他地方一致）。 |
| `auto_start_backend` | `True` | 若未运行任何后端，则启动本地后端。对于远程 `backend_url` 绝不会启动后端。 |

> **池中已打开的传感器不会显示。** 已连接的 BLE 外设会停止广播，且无法探测已打开的 COM 端口，因此发现功能仅列出*可连接*的设备。 刚连接设备后出现空结果是正常的——若需获取已连接设备的信息，请使用 `list_daq_sensors()`。无法运行扫描的传输协议（未安装 bleak / zeroconf）会被跳过而非触发异常，因此未启用蓝牙的机器仍能获取其 USB 和以太网的响应。

### 代码示例

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### 哪个数据采集器（DAQ）连接哪台摄像头 — `daq_bindings()`

捕获数据的反射率及其保存的 `.daq` 值，会使用与该相机或阵列绑定的 DAQ，否则使用后端中最新的 DAQ。GUI 中的光传感器选择以及 `chloros-cli daq pool-bind` 会设置此参数； 以下操作同样会进行设置：

```python
chloros_sdk.set_daq_bindings(cameras={"214701281": "COM7"},          # a sensor id or an address
                             arrays={"array-1779862544497": "daq-e-def330"})
chloros_sdk.set_daq_bindings(cameras={"214701281": None})            # None clears it
for serial, b in chloros_sdk.daq_bindings()["cameras"].items():
    print(serial, b["binding"], b["source"], b["resolved"]["in_pool"])
```

`daq_bindings()` 返回 `{"cameras": {serial: b}, "arrays": {array_id: b}}`：每个 `b` 包含 `binding`（已设值）、`source`（`project`， 来自该项目的 `sensors.json`，或 `runtime`），`known` / `known_as` （无论相机或数组键是否命名了Chloros所知的内容，以及如何命名：`connected`、`project` —— 已打开项目的`cameras.json` ——或者，对于摄像头，`seen`：此前曾在此计算机上打开过；`known: False` 表示未命名任何对象的键) 以及 `resolved` （它当前映射到的池传感器：`in_pool`、`sensor_id`、 `address`、`model`、`streaming`）。运行时绑定优先于项目；项目文件不会被修改。断开相机、数组或数据采集卡的连接会解除该硬件的所有绑定，无论由谁设置（数组的成员绑定与数组本身绑定）。

`set_daq_bindings()` 仅绑定已连接的相机（该相机需在当前打开的项目中，或此前曾在此计算机上打开过），以及已连接或位于当前打开项目中的数组（或已绑定的键）。任何其他键都会触发名为 `ChlorosConnectError` 的错误 （`400`、`code`、`UNKNOWN_BINDING_TARGET`，以及`unknown`键），且该调用中**没有任何**内容会被绑定。 `None` 会清除所有键。

### 环境光 + 目标反射率 — `light_sensor_reflectance()`

两个合并的数据采集（DAQ），一个朝上观测天空（环境），一个朝下观测目标（物体），可从其最新读数中提供目标的反射率和植被指数：即 GUI 中的“环境 + 物体”配对，以及 `chloros-cli daq reflectance`，而无需再次打开传感器。 两者必须均已连接并处于数据流传输状态。

```python
r = chloros_sdk.light_sensor_reflectance("daq-e-def330", "daq-e-54b774")
print(r["reflectance_type"])                       # "Apparent Reflectance" when both are calibrated
for name, band in r["bands"].items():
    print(name, band["rho"] if band["valid"] else band["reason"])
for idx in r["indices"]:
    print(idx["name"], idx["value"])
```

```python
light_sensor_reflectance(
    ambient_sensor_id, object_sensor_id, *,
    bands=None,          # [{"name", "center_nm", "fwhm_nm", "role"}]; role blue / green / red / nir
    preset=None,         # a preset id from daq_band_presets(); default: the suggested one, else "GENERIC"
    expressions=None,    # custom indices: [{"name": "MyNDVI", "expr": "(NIR_850 - Red_660) / (NIR_850 + Red_660)"}]
    builtins=True,       # include the built-in indices
    backend_url="http://127.0.0.1:5000", timeout=30.0,
) -> dict
```

- **波段。** 每个波段的反射率是目标波段积分辐照度。`daq_band_presets(filter_code=None)` 列出了波段集：每个 LATTICE M3C 滤光片对应一组（`RGN`、`OCN`、 `NGB`，即相机自身的波段模型，因此DAQ NDVI与相机NDVI是同一物理量）以及`GENERIC`宽带波段。 其`suggested`是与`filter_code`匹配的预设值，否则即为首个连接相机的滤光片 （由 `suggested_from` 指定）。
- **返回值** `bands` （按波段 `rho`、`valid`、`reason`、`e_ambient`、 `e_object`)，`indices`（内置-ins），`custom`（各配有`value`，或`valid: False`和`error`），`invalid_bands`、`reflectance_type`、`preset`以及这两个读数的`timestamps` （`ts_ns`）。环境传感器无法测量的波段（超出其测量范围或处于噪声底限）将被视为无效，并注明原因；所有依赖该数据的指标均被排除在外，绝不会以零为基准进行计算。
- **表观值或相对值。** 与量程相关的指数（SAVI、OSAVI、EVI、MSAVI2、RDVI、TDVI、LAI、NLI、MNLI、FCI、GEMI）需要绝对比值，因此仅在两个传感器均经过功率校准时才会生成 （每个传感器均具有 `calibration_bundle_sha`）：`"Apparent Reflectance"`。否则，结果为 `"Relative Reflectance"`，仅包含尺度不变的指标。
- **两个不同的波长网格** 根据其重叠部分进行比较：环境传感器的数据点位于物体的测距范围内，物体的光谱被插值到这些数据点上。
- **必须是两个传感器，绝不能只有一个。** 将一个传感器同时命名为两个（无论拼写形式如何，只要池子接受，ID相同）会引发错误 `ChlorosConnectError`：传感器在自身上的读数在每个波段均为 1.0，这意味着无法测量任何数据。 后端也会拒绝该操作（`POST /api/daq/indices` 响应 `400`， `code` `SAME_SENSOR`，而SDK发送的`ambient_sensor_id`和`object_sensor_id`属于同一个池化传感器，例如其 ID及其端口）。

### 与CLI及其他脚本的资源共享

后端的 DAQ 传感器池由该机器上的每个 CLI 和 SDK 客户端共享。如果传感器已存在于池中（由 `chloros-cli daq pool-connect` 或其他脚本打开），则从 Python 调用 `connect_daq_sensor(port="COM3")` 将返回一个借用的句柄：其 `disconnect()` 操作无实际效果，因此当脚本退出时，不会将传感器从最初打开它的客户端手中“抢走”。 对于 `sensor_id=`（通过 `:` 或 `-` 调用，两者均可匹配）也是如此，这适用于 DAQ-E 传感器，该传感器在通过主机名打开时由其 IP 命名（反之亦然），以及当后端返回其已持有的传感器时，带有零标志的智能检测——`with` 代码块只会释放它自己打开的传感器。

Chloros图形界面**不**属于此池：其DAQ面板会自行建立与传感器的连接，因此`connect_daq_sensor`绝不会借用图形界面已打开的传感器。 请勿同时在 GUI 和脚本中使用同一个传感器。GUI 持有的 DAQ-U 或 DAQ-M 通常会无法打开， 但 DAQ-E 设备会接受第二个会话：脚本的连接操作会更改 GUI 下的设备采集设置，且其 `disconnect()` 会停止设备的数据流，这将终止 GUI 的实时预览及其正在进行的任何录制。请先在 GUI 中关闭传感器。

### PTP 主时钟 — `time_sync_status()`

后端运行 PTP 主时钟，LATTICE 相机和 DAQ-E 设备从中获取时间戳。它按需启动：`connect_camera()`、 若其未运行，则由 `connect_array()`、`connect_daq_sensor()` 和 `connect_all()` 启动它 ——包括随后连接失败的情况，例如 `connect_all()` 尝试连接不存在的 DAQ-E 设备——`test_array_sync()` 以及所有为单条命令打开摄像头的调用 （`capture_cameras_transient()`、`start_multi_camera_stream()`、对齐调用、`capture_live_index()`）。

- **其广播位置。** 在连接摄像头的有线网络适配器上（包括一个 `169.254.x.x` 链路本地地址），否则在默认路由的适配器上（如果该适配器是有线的）。绝不会在 Wi-Fi 或虚拟适配器（Tailscale、WSL）上：因为其后方没有摄像头或 DAQ-E。 如果没有此类网卡处于活动状态，它将运行在无网卡绑定的状态下（`health` `NO_SOCKETS`），并在下一台相机或DAQ连接时自动绑定该网卡。
- **链接中断后。** 当摄像头适配器丢失链接并恢复连接时（例如拔插网线、更换交换机、USB适配器重新枚举），主控节点会自动重建套接字，无需重启， 并重新检查所有已连接的 DAQ-E，若发现有设备因连接中断而卡住，则将其重新激活。此外，当其服务的所有摄像头和 DAQ-E 连续 15 秒无响应时，它也会自动重建连接。
- **是否已连接到线缆？** 请参阅 `time_sync_status()`（`GET /api/time-sync/status`）。`state` 仅表示主控器线程正在运行；`health` 则显示其数据包是否成功发送： `OK`（未检测到故障）、`STOPPED`、`NO_SOCKETS`、`SENDS_FAILING`、 `DEGRADED`（部分适配器故障）或 `PEERS_SILENT`（其所服务的摄像头和 DAQ-E 设备停止响应），并在 `health_detail` 中显示一条提示信息。 `announce_sent` / `sync_sent` / `delay_resps_sent` 仅统计已发送的消息； `announce_send_errors`、`sync_send_errors`、`follow_up_send_errors`、`delay_resp_send_errors`、`last_send_error`、 `failing_interfaces` 和 `last_delay_req_age_s` 描述了未发送成功的消息，而 `rebinds` / `last_rebind` 则描述了重传操作。 `sync_sends_slow` 统计发送时间超过 200 微秒的同步操作，而 `sync_send_max_us` 则表示耗时最长的同步操作；每个 Follow_Up 的源时间最多计入其发送过程的 200 微秒。 主节点的运行位于其独立进程中（`own_process`、`process_pid`），在Windows系统上具有高优先级（`process_priority`）， 因此繁忙的后端无法延迟时间戳的生成，也无法一次性调整所有摄像头的时钟；如果该进程无法启动，则像以前一样在后端内部运行（`own_process` false）。

```python
st = chloros_sdk.time_sync_status()
if st.get("health") not in (None, "OK", "STOPPED"):
    print("PTP:", st["health"], "-", st.get("health_detail"))
    chloros_sdk.restart_time_sync()
```

| 函数 | 功能说明 |
| --- | --- |
| `time_sync_status()` | 主时钟的状态、健康状况、时钟标识、优先级和消息计数器（如上所述）。`chloros-cli time-sync status`。 |
| `time_sync_peers()` | 其监听到的 PTP 从时钟 （摄像头、DAQ-E）：`peers`、每个 `port_identity`、`last_seen_ip`、`last_delay_req_age_ms`、 `delay_req_count`。`time-sync peers`。 |
| `time_sync_cameras()` | 每台已连接的阵列摄像头的 `PtpStatus`、 `PtpOffsetFromMaster` 和 `PtpMeanPathDelay`（固件中没有 PTP 节点的相机仅保留其序列号）。`time-sync cameras`。 |
| `restart_time_sync()` | 停止并重启主控器；之后将重新检查每个已纳入数据池的 DAQ-E 的 PTP（`daq_ptp_rechecked`）。当 未能恢复时触发 `ChlorosConnectError`。`time-sync restart`。 |
| `set_time_sync_priority(priority1=None, priority2=None)` | 其 PTP 优先级（0-255，数值越低在最佳主节点选举中获胜），适用于存在另一个 PTP 主节点的网络。需要该主节点正在运行。`time-sync set-priority`。 |

它们分别接收 `backend_url=` 和 `timeout=`，并启动一个尚未运行的本地后端。

---

## 相机实用程序

`chloros-cli lattice` 除了连接和捕获之外，还执行 `chloros_sdk`（模块 `chloros_sdk.camera_utils`）中的基本功能。它们各自调用 CLI 命令所使用的后端路由，并遵循 CLI 的规则， 因此两者的行为一致。它们接受摄像头**序列号**（而非会话），并使用与智能连接调用相同的 `backend_url=` / `timeout=` 关键字； 如果后端未运行，则会像 `connect_camera()` 那样启动它。若调用被拒绝，则会触发 `ChlorosConnectError` 并返回后端给出的拒绝原因。

```python
import chloros_sdk as c

c.set_camera_color_profile("214701281", profile="natural")
c.set_camera_highlights("214701281", -40)
print(c.camera_firmware_status()["cameras"])
st = c.network_adapters(estimate=True, cameras=4, pixel_format="BayerRG12")
```

| 函数 | 功能说明 |
| --- | --- |
| `camera_color_profile(serial)` / `set_camera_color_profile(serial, profile=, custom_cct_k=)` | 显示颜色配置文件： `raw`、`linear`、`natural`、 `enhanced` 或 `custom_temp`，及其色温（限制在 2000-10000 K 范围内；当配置文件为 `custom_temp` 时读取）。 |
| `camera_color_adjust(serial)` / `set_camera_color_adjust(serial, saturation=, contrast=, reset=False)` | 显示饱和度和对比度，各为 0-200 %（100 表示不变）；`reset=True` 将两者均恢复为 100。 |
| `camera_highlights(serial)` / `set_camera_highlights(serial, value)` | 降低显示渲染中最亮、最白的像素值：-100..0（向零截断，然后进行钳位； 0 = 关闭）。 |
| `camera_white_balance_once(serial, settle_s=None)` | 单次白平衡：相机短暂传输数据，待参数收敛后，返回 `red_ratio` / `blue_ratio`（若无法报告，则返回 `ratios_read` False）。它将保留已计算出的白平衡值；其 open 函数写入的其他所有设置均按原始值返回。 |
| `set_camera_white_balance(serial, mode=None, red=None, blue=None)` | `mode="auto"` （连续）或 `"off"`；或固定的 `red` / `blue` 白平衡比例，这些比例会首先关闭自动模式。 相机拒绝的比率在 `errors` 下命名。|
| `camera_presets()` / `apply_camera_preset(serial, preset)` | 相机预设，应用其中一个时：相机将保持预设名称所对应的设置 （响应中的 `settings`，即相机随后读取的参数，例如其运行的帧率；`None` 表示预设中未命名的字段），而“open”写入的其他所有设置将恢复为原始状态。 `"default"` 将摄像机重置为出厂默认值（即出厂设置，包含自由运行的自动曝光、链接吞吐量限制和缺陷校正关闭，同时保留RGB摄像机的校准白平衡），且不会保存至其内存中： 响应中包含 `factory_defaults: True` 以及 `settings` 的所有字段。 在 `camera_presets()` 中，`default` 即为 `{factory_defaults: True, description}`。 |
| `camera_filter_models()` / `camera_filter_model_info(name)` | 滤光片模型，以及其中一个模型的详细信息（波段、光谱响应条件数）。相机的滤光片模型是从其型号字符串中读取的；不能手动设置。 |
| `camera_firmware_status(serial=None)` | 每台相机的固件：`current_version`、`available_version`、`update_offered`、`needs_minimum_update` 以及 `minimum_firmware`。 |
| `update_camera_firmware(serial, force=False)` | 将此Chloros版本的固件刷入一台摄像头：恢复出厂设置、刷写、重启，Chloros基准设置被重写； 相机的校准数据将保留。耗时数分钟；相机必须保持通电。`force=True`用于重新刷写当前相机的固件。 |
| `camera_power_probe(serial)` | 相机的电源和热节点及其温度。 |
| `sleep_camera(serial)` / `wake_camera(serial)` | 相机的低功耗空闲模式（相机将保持休眠状态直至被唤醒： 其链路吞吐量限制已启用（`DeviceLinkThroughputLimitMode` `On`）且处于下限，若相机具备节能模式则该模式已开启），并显示唤醒前后的温度； 唤醒后，链路状态将恢复至休眠时的状态（包括限值、模式以及数据包延迟 `GevSCPD`——当限值启用时，摄像机将该值保持为 0；该记录保存在本计算机上，因此后续后端进行唤醒时也能获取该信息）， 或者，对于这台计算机未将其置于睡眠状态但看起来已进入睡眠状态的摄像头（其限值在阈值处开启），此时关闭限值，并将`GevSCPD`保留为0（出厂设置）； 若限制值设定在阈值以上，则保持原样。仅保留电源状态；其他所有设置在写入时均恢复为原始值。 |
| `calibrate_camera_with_target(serial, filter=None, attempts=30, settings=None)` | 持续捕获帧，直到MAPIR校准目标进入视野，并根据其面板数据拟合各通道系数：`target_id`、`panel_dns`、 `coefficients`。 |
| `network_adapters(estimate=False, cameras=1, pixel_format="BayerRG8")` | 该计算机用于 GigE Vision 的以太网适配器（MTU、链路速率、巨型帧）以及任何 `issues`； `estimate=True` 会补充 `cameras` 相机将获得的帧率和带宽。 |
| `fix_network_settings()` | 将 GigE Vision 适配器设置应用到这台计算机的的相机适配器。该操作将更改适配器设置，且仅在Chloros以管理员身份运行时有效；否则将显示检测结果日志并退出。 |
| `apply_vignette_correction(input_dir, output_dir, lens_model=None)` | 对文件夹中的图像进行暗角校正，每张图像均使用其对应镜头的平场图：`processed`， `lens_models`（用于校正该文件的镜头）、`skipped`（附带校正原因）。若未提供`lens_model`，则每个文件的镜头信息将来自其EXIF“型号”字段； `lens_model` 适用于型号名称中未指定相机的文件，而型号信息与其不符的文件将被跳过。仅限文件。 |
| `read_dls(mac, filter=None, camera_serial=None)` | 来自 DLS（蓝牙下行光传感器）的一次读数： 光谱、各波段辐照度以及通过相机自身响应获得的波段幅度（当提供 `camera_serial` 时）。`discover_daq_sensors(transports=["ble"])` 用于查找 MAC 地址。 |
| `probe_system(pixel_format="BayerRG12", discover=True)` / `system_info()` | 主机用于摄像头的硬件及其推荐配置（`report_text`），以及系统的 CPU / GPU / PyTorch 运行时。 |

- **有些需要独占摄像头。** `camera_white_balance_once()`, `apply_camera_preset()`、`sleep_camera()` / `wake_camera()`、`calibrate_camera_with_target()` 和 `update_camera_firmware()` 会打开 摄像头。当Chloros正在持有该摄像头时（在应用中已连接、作为数组成员与`connect_camera()`关联，或当前正被其他命令使用， 例如数组测试或捕获操作正在打开其摄像头），后端会返回HTTP 409 `CAMERA_HELD`错误（在`code`和`error_code`中均会发生），并说明了如何释放该资源。颜色设置、高光、`camera_power_probe()`以及读取操作在已连接的摄像头上均可正常工作。
- **颜色、高光和白平衡仅适用于RGB摄像头。** 在单波段单色（M3M）相机上，这些设置函数不会更改任何参数，并返回 `{"skipped": True, "reason": ...}`，这与 CLI 的行为一致。它们是显示设置：仅涉及实时预览和显示级别的捕获图像，绝不涉及辐射度或反射率。

---

## 瞬态数组、对齐配置文件与索引

在没有持久连接的情况下，`chloros-cli lattice` 的行为与 `chloros_sdk`（模块 `chloros_sdk.array_utils`）中的函数：`continuous`、`multi-capture`、`multi-stream`、`multi-test`、`multi-detect`, `analyze-array`、`align-calibrate`、`align-apply`、`align-stream`、`align-info`、 `align-reorder` 和 `index`。每个命令都会发送CLI命令所发送的请求，并采用CLI的默认设置。

```python
import chloros_sdk as c

prof = c.calibrate_alignment_profile(["214000535", "214000551", "214000556"],
                                     "D:/rig/alignment_profile.json", method="orb", model="rigid")
print({sn: t["reprojection_error"] for sn, t in prof["transforms"].items()})

c.capture_aligned_multiband("D:/rig/alignment_profile.json", "D:/rig/out")
idx = c.capture_live_index("D:/rig/alignment_profile.json", preset="NDVI",
                           output_dir="D:/rig/ndvi", capture_level="reflectance")

with c.start_multi_camera_stream(["214000535", "214000551"], "D:/rig/stream", fps=5, count=50) as job:
    print(job.wait()["final_stats"])          # waits for the 50 groups, then closes the cameras
```

- **它们会自行打开摄像头。** 每个命令都会打开其对应的摄像头，执行相应操作后再关闭，因此需要使用Chloros未持有的摄像头（在应用中已连接， 通过 `connect_camera()` 连接，或作为数组成员）；后端会返回 HTTP 409 错误（`CAMERA_HELD`）并说明如何释放摄像头。临时打开操作需要一定时间（几台摄像头约需 40 秒）。对于重复捕获操作， `connect_array()` 仅执行一次是快速路径，且适用于所有导出类型（而临时捕获仅保存去拜耳化图像）。`calibrate_alignment_profile()` 是例外：对于已连接的此类摄像头数组，它会从数组自身的帧中进行处理 （`used_live_array`）。
- **`serials`** 是要打开的相机，第一个相机将触发拍摄；`None` 则打开网络上的所有相机。
- **`settings`** 是一个用于覆盖摄像机设置的字典，其中字段由CLI的标志参数设定：`exposure_time`（微秒）， `gain`（dB）、`pixel_format`、 `width`、`height`、`target_brightness`、`ae_damping`、`ae_upper_limit`、 `frame_rate`、`gev_packet_size` 以及 `preset`（一个按名称命名的相机预设）。指定曝光值或增益会关闭自动控制功能，这与标志选项的作用相同。
- **`gpu`**：`True` 强制使用 GPU 变换，`False` 强制使用 CPU （`--gpu` / `--no-gpu`）；`None` 则由后端自行选择。
- **`keep_settings`**/**`clear_settings`**（所有打开摄像头的函数，默认值为 `False`）：运行结束时，每台摄像机都会恢复到初始状态；`keep_settings=True` 则将本次运行的设置保存到摄像机自身的内存中（自由运行模式）， `clear_settings=True` 将该内存重置为出厂设置 — 参见 [摄像头内存](#camera-memory--keep_settings--clear_settings)。 响应（针对某项任务，即 `stop()` 的）包含 `camera_memory`。
- **路径属于后端**：当后端位于本机时，相对路径会相对于您的工作目录转换为绝对路径。对齐配置文件相关函数（`alignment_profile_info()`、`reorder_alignment_profile()`）会像CLI一样，从本机读取该文件。

| 函数 | 功能说明 |
| --- | --- |
| `start_camera_continuous(serial, output_dir="output", *, format="tiff", jpeg_quality=95, queue_depth=64, settings=None)` | 持续录制一台摄像头的画面，直至停止。 返回一个 `CliJob`。一次 `tiff` 运行生成的帧，即 `raw` 文件，是由该摄像机通过数组捕获写入的（传感器上一个马赛克平面的原始值，未压缩）； `png` 和 `jpg` 各包含 8 位，因此这些运行生成的帧是经过去马赛克处理的、可供查看的图像。绑定或池化的 DAQ 会将运行中的每帧数据写入一个 `.daq` 文件，并存入该运行的的 `daq/` 文件夹中（其统计信息中显示为 `daq_files`）。每帧图像都会连同完整的元数据（型号、拍摄时间、镜头、波段、基于原始帧的校准）写入一次； `stop()` 会在所有已捕获的帧写入完毕后（最长 25 秒）返回一次，此时才会关闭相机。若相机处于硬件触发待机状态 （`TriggerMode` `On`）的相机，在本次运行期间将设置为自由运行模式（起始响应中的 `trigger` 会说明检测结果）。 停止时，相机将恢复至被发现时的状态——包括其触发设置以及运行过程中更改的所有设置——除非使用 `keep_settings=True`（将运行设置保存到相机自身的内存中，即自由运行模式）， 或使用 `clear_settings=True` 重置该内存。未录制任何帧的运行将在 10 秒后自动终止，此时会触发 `stop_reason` 和 `no_frames`，具体原因记录在 `last_error`中记录，其`stop()`会将该原因返回为`error`。因写入器无法跟上而丢失的帧会被计数，具体原因由`drop_reason`说明。对于 Chloros持有的摄像头，请使用 `CameraSession.capture_repeated()`。`lattice continuous`。 |
| `start_multi_camera_stream(serials=None, output_dir="output", *, fps=3.0, count=None, format="tiff", jpeg_quality=95, settings=None)` | 在 `fps`，持续处理 `count` 个帧组或直至停止。返回一个 `CliJob` （其 `start_reply` 包含 `n_cams` 及线缆 `pacing`）； 其最终统计数据包含已捕获、已写入和已丢弃的组，写入器无法写入的文件，以及相机未提供任何数据的组。 `lattice multi-stream`。 |
| `capture_cameras_transient(serials=None, output_dir="output", *, format="tiff", jpeg_quality=95, settings=None)` | 一次同步捕获，每台相机一张去拜耳化图像：`results` (`filepath`, `serial`, `frame_id`) 以及 `pacing`。 当某组缺少一台相机时，系统会提示并首先列出已保存的文件数量。未连接阵列的 `lattice multi-capture`。 |
| `test_array_sync(serials=None, count=10, ptp_settle_s=None, timeout=None)` | `count` 以固定曝光值进行同步抓拍： `sync_groups`、`master`，每触发一次对应一行 `group_timestamps`（缺失一次拍摄的相机在其行中缺席），以及 `sync`：每台相机相对于主相机的偏移量，按每次触发计算（`offsets_us`：每台相机分别为 `mean_us`、`min_us`、 `max_us`、`jitter_us`、 `groups`)，基于一个时钟（`clock_domain`：`ptp`，或 `device` ——PTP关闭，每台相机使用自身的运行时间时钟，因此其偏移量取决于其启动时间，仅通过抖动来测量同步）。时间戳混合了设备时钟和PTP（PTP关闭或仍在锁定中）的触发事件会被排除并计数（`groups_mixed_domain`），同样被计数的还有PTP时钟与主时钟仍相差超过`agree_within_us`的触发事件 (`groups_clocks_disagree`)；`notes`对此有说明。`clock_offset_mean_us` / `clock_offset_jitter_us` 分别表示每台摄像机的最大均值和抖动值。 当触发信号被遗漏、同一时钟下每台摄像机的触发次数少于两次，或者时钟从未同步时，`success` 为 false，而 `error` 则为 true （`sync.problems`）时；无论哪种情况，这些行都会被返回。 如果 PTP 主时钟尚未运行，则启动它（`time_sync` 表示 `grandmaster_running` 和 `grandmaster_started_by_this_test`），开启每台摄像机的PTP（如同`connect_array()`所做的那样），并在首次触发前等待每台摄像机的PTP时钟锁定至主时钟并稳定下来： `PtpStatus` 作为从机连接到一个主时钟，与主时钟的偏差在 100 微秒以内且稳定 3 秒，且每台摄像机均以这种方式锁定， 且无中断，持续120秒——即从上电起约2分钟。刚锁定的时钟在读取“从属”状态后的约2分钟内，误差可达~200微秒，而相机自身的`PtpOffsetFromMaster`并不会显示这一情况； 120秒后，相机时钟将稳定在与主时钟相差约10微秒的范围内。若某台相机自上次测试后一直保持锁定状态（且PTP功能保持开启，按此后端所见），则计时将从该锁定时刻开始， 因此连续进行的测试无需再次等待；而测试中刚开启的相机，或被发现处于PTP开启状态但该后端尚未观察到其锁定（可能在其他地方开启，或在后端重启前开启）的相机，则需等待120秒；在包含这两类相机的数组中，刚刚锁定的相机仍需等待其预定时间。 若摄像机脱离从属状态、超时超过100微秒，或在等待期间连续跟随另一台主时钟进行3次读数 （1.5 秒）的，将重新开始计数；此类读数会记录一两次（`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`），计数继续进行，而无法读取的主时钟 ID 将被视为未知，而非状态变化。 所有摄像头的计数同时重置（在一秒内）表明是主时钟的时间发生了变化，而非摄像头自身：后端日志显示为 `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`，`ptp_lock` 在 `grandmaster_steps` 中列出该值，随后超时的等待会以 `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …` 开始记录错误。该等待的时限不会因此延长。`ptp_settle_s`（0-600；若无稳定等待则为`0`）设置了本次运行的稳定时间；若设置其他值将被拒绝（HTTP 400）。 该等待的上限为结算时间加 90 秒（默认 210 秒，至少 90 秒），且调用将等待后端 `timeout` 秒：默认值为 240 秒加上结算时间（360 秒）， 这与 `chloros-mcp` 的 `test_array_sync` 行为一致。`ptp_lock` 包含 `converged`、 `waited_s`、`timeout_s`、`settle_s`、`locked_s` （每台相机已锁定并保持稳定的时间长度）、`found_settled`（被发现时相机已锁定达到稳定时间）以及每台相机的 `status`、`offsets_ns`、 `slave_s` （已锁定时长；“无”：未锁定）以及 `remaining_s`（当无摄像机的 PTP 处于开启状态时显示“无”）。后端日志每 5 秒记录一行，包含每台摄像机的状态、已锁定时长以及剩余锁定时长。 每台摄像机开机时 PTP 均处于关闭状态；随后时钟约在 10 秒内完成同步。此后，每台摄像机将恢复至测试发现时的状态——其 PTP、自动曝光和增益均保持测试期间设定的“关闭”状态 （`ExposureAuto` / `GainAuto`，以及手动设置的 `ExposureTime` / `Gain`）， 其余情况则恢复原状，除非是 `keep_settings=True` —— 因此，若某台相机 PTP 处于开启状态（`PtpEnable=True`，或运行时设置为 `keep_settings=True`） 将保持锁定状态，且一旦该后端检测到其已锁定并达到稳定时间，后续测试将不再等待该相机。如果时钟未能稳定，则在任何触发之前 `success` 为假，而 `error` 说明了应采取的措施： 从未锁定的相机（`not_slave`）意味着主控无法与其建立连接（`time_sync_status()`）；持续掉线的相机将按其锁定时长命名 （`remaining_s`）；时钟仍在收敛（`far`）：检查 `time_sync_status()`，若要在测试前使它们锁定，请开启 PTP 并保持开启 （每台摄像机上设置为 `PtpEnable=True`，或 `keep_settings=True`；测试会将发现的设置恢复原状，因此该功能将保持开启状态），然后运行测试。`lattice multi-test`。 |
| `detect_array_master(line=None)` | 同步线上的摄像机和默认主控设备：`master_serial`， `slave_serials`，以及被探测的 `line`。`lattice multi-detect`。 |
| `analyze_array_models(camera_models=None, *, width=2048, height=1536, n_active=None, force_tier=None, binning=1, packed="auto", pixel_format=None)` | 这些摄像头型号（产品名称、 `"LATT-M3C-L87-FOCN"`）将出现在这台计算机的网络中：`tier`、`achievable_fps_max`，线缆预算（`max_cams_collision_safe`、 `oversubscribed`）、`cam_caps` 和 `fix_suggestions`。默认情况下，发现的摄像头型号； 均未打开。有关您拥有的摄像头及其连接所需的设置，请参阅 [`analyze_array_network()`](#network-analysis-helper)。`lattice analyze-array`。|
| `calibrate_alignment_profile(serials=None, save_path="alignment_profile.json", *, frames=3, method="orb", model="rigid", reference_serial=None, checkerboard=None, control_points=None, preview=False, vignette=False, roi=None, mask_path=None, rms_threshold_px=None, per_cam_overrides=None, settings=None, ...)` | 根据实时帧生成对齐配置文件并保存。 `method`：`orb`、`akaze`、`phase`、`checkerboard` （配合 `checkerboard="9x6"`，即内角）或 `manual`（配合 `control_points={serial: [[x, y], ...]}`，即参考摄像机的数据）。`model`： `translation`、`rigid`、`affine` 或 `homography`。 更精细的选项是CLI的标志：`max_features`、`ratio_threshold`、`matcher`（`bf` / `flann`）、`knn_k`、`prefilter`（`none` / `gradient` / `clahe` / `blur` / `hist_match`)，`ransac_threshold_px`， `ransac_max_iters`, `ransac_confidence`、`averaging`、`lock_rotation`、`lock_scale`、`lock_axis`； `per_cam_overrides` 即为 `{serial: {option: value}}`。`rms_threshold_px` 拒绝保存较差的解。 返回 `save_path`、`output_width` / `output_height`、`crop_roi`、 `reference_serial`、`transforms`（按相机分类：`reprojection_error`、 `num_matches`) 以及 `preview_path`。`lattice align-calibrate`。|
| `capture_aligned_multiband(profile_path, output_dir="output", *, serials=None, format="tiff", bit_depth=None, gpu=None, no_crop=False, bands=None, order=None, vignette=False, per_band=False, per_camera=False, interpolation="linear", border_mode="constant", border_value=0, settings=None)` | 通过该配置文件生成的单张对齐多波段图像：`files`（该多波段TIFF文件及其`bands`和尺寸， 外加每波段一个文件（`per_band`）和每台相机一个文件（`per_camera`））以及 `vignette_corrected`。 `bands` 选择频段，`order` 按名称（列表或逗号分隔的字符串）对频段进行排序。`lattice align-apply`。 |
| `start_aligned_stream(profile_path, output_dir="output", *, serials=None, fps=3.0, count=None, ...)` | 流对齐多波段图像（与 `capture_aligned_multiband()` 相同的变换选项），处理 `count` 或直至停止。 返回一个 `CliJob`；其最终统计数据包括 `frames_written` 以及任何 `frame_groups_incomplete`。`lattice align-stream`。 |
| `alignment_profile_info(path)` | 汇总为 `align-info` 的配置文件会将其打印出来：`name`、`created`、`method`、`reference_serial`、 `sensor_resolution`、`output_width` / `output_height`、`crop_roi`、`camera_serials`、 `transforms` （每台摄像机 `model`、`reprojection_error`、`num_matches`、`reference`）, `camera_models`, `layer_order`（每台摄像机对应 `band_names`, `wavelengths_nm`, `enabled`）；`profile`是整个文件。 |
| `reorder_alignment_profile(path, order=None, enable=None, disable=None)` | 就地修改配置文件：`order` 是频段名称（每个频段按此顺序将对应摄像头的图层向上移动）， `enable` / `disable` 表示序列号。返回新的 `layer_order`。`lattice align-reorder`。 |
| `index_catalog()` | 索引预设（`presets`：每个 `name`、`channels`、 `formula`）以及颜色 `gradients`. `lattice index --list-presets` / `--list-gradients`。 |
| `compute_index(input_path, *, preset=None, formula=None, channel_map=None, output=None, capture_level="unknown", output_format="all", colorize=False, gradient="RdYlGn", vmin=None, vmax=None, percentile=None, bg_mode="clip")` | 对齐的多波段TIFF文件的索引：一个 `preset` (`NDVI`、`NDRE`、 ……）或一个 `formula`，该索引映射到符号 `channel_map` 对应的波段（`{"NIR": "NIR_850", "Red": 0}`，按名称或索引）。一个预设'符号 `channel_map` 会省略该文件中名称以该符号开头的那个波段（`red` -> `Red_660`， `nir` → `NIR_850_RGN`），其名称由文件本身决定（即其波段列表，或Chloros捕获数据/产品的`Chloros:Band{R,G,B}Name`）， 因此，Chloros反射率捕获中的NDVI无需`channel_map`； 两个对应同一符号的波段会被拒绝，而非被推测。`capture_level`说明了波段包含的内容：`reflectance`或`radiance`使数值具有定量意义； 未经校准的波段堆栈（`align-apply`）提供定性值； `"unknown"`从文件中读取数据，其中包含Chloros文件的`Chloros:ProcessingLevel`。返回`stats` （形状、1-99百分位数范围、查找表范围、所用的`capture_level`）以及写在输入旁边的`files` (`<input stem>_<preset>.tif`、`..._LUT.png`) 旁边，或在 `output` 处写入一个路径：若文件带有图像扩展名，则根据其根目录命名 （`output="D:/out/ndvi_FRGN.png"` 会写入 `ndvi_FRGN.tif` 和 `ndvi_FRGN_LUT.png`），若无扩展名，则作为前缀 （`D:/out/ndvi` → `ndvi.tif`， `ndvi_LUT.png`）；从 `files` 读取路径。这些文件包含输入的相机记录（品牌/型号/序列号、GPS、镜头、UTC 时间、校准标识符，但绝不包含其 raw-DN 校准）、`Chloros:ProcessingLevel=index`，以及索引作为其唯一波段（`XMP-Camera:BandName`，TIFF和LUT PNG均如此）。仅文件；无相机信息。`lattice index --input`。 |
| `capture_live_index(profile_path, *, preset=None, formula=None, channel_map=None, serials=None, output_dir="output", output_prefix="index", capture_level="reflectance", save_multiband=False, ...)` | 拍摄、校准（`reflectance` 或 `radiance`； `raw` 会跳过此步骤），对齐并一步计算索引；`save_multiband=True` 保留校准后的图像堆栈，`compute_index()` 稍后可调用该堆栈。 返回 `stats`（附带其所使用的频段的 `serials`）和 `files`。 它会打开 `serials`（第一个触发器），否则会打开配置文件中 `layer_order` 条目已启用的相机，先启用参考相机 （即使其波段被禁用也会被打开）；波段被禁用的摄像机绝不会被校准。`channel_map`仅在堆栈无法响应某个符号时才需要， 例如在`compute_index()`中：预设的符号会选用名称以该符号开头且已启用的单一波段，该检查在任何相机开启前进行（若无相机响应或多个相机响应，则拒绝并列出其名称）；自定义的`formula`则需要每个符号。 `index.tif` 和 LUT 携带堆栈中的相机、拍摄时间（参考相机的曝光时间，UTC）和镜头信息，二者均将索引作为其唯一波段（例如 `XMP-Camera:BandName`， `NDVI`：彩色PNG文件以索引为名称，而非其颜色通道）。`lattice index --live`。 |

### `CliJob`

三个流返回的句柄。该任务在后端运行，并在被停止之前一直拥有其摄像头。

| 成员 | 描述 |
| --- | --- |
| `stats(timeout=10.0)` | `running`、`state`、`stats`（已保存/ 写入、丢弃、`fps`、`elapsed`、写入失败、`stop_reason`；对于连续任务还包括 `frames_grabbed` 和 `frames_queued`——每个捕获的帧均被写入、排入队列、丢弃或写入失败——以及 `drop_reason`、 `writers` 和 `write_ms`（单帧写入的平均耗时）以及 `output_dir`。`state` 即 `running`，`ended`（该进程已自行停止但尚未被终止），`stopping`（一个停止中的进程正在写出其队列： 实时统计数据，以及 `stopped_by`）或 `finished`（其最终统计数据，停止任务的 `error` 和 `stopped_by`）。每个客户端都会将已停止作业的记录保留 10 分钟（最多保留最近的 32 条）：另一个脚本或 `chloros-mcp` 会读取相同的状态和最终统计数据。一个作业 `chloros-cli` 已启动 (`lattice continuous`、`multi-stream`、`align-stream`) 当CLI连续30秒未进行状态报到时，该任务会被后端停止——被终止，其机器处于休眠状态——后端会生成`stop_reason`、`client_gone`和`stopped_by`； 从这里读取其统计信息无法使其保持运行状态。由SDK启动的任务将持续运行直至被停止。每次启动都会将其`timeout`作为后端的截止时间（`client_deadline_s`）：若在此之后打开的摄像头（此时该摄像头仍在启动中）被恢复并关闭，且未启动任何任务，则该调用会抛出 `ChlorosConnectError`（`CLIENT_DEADLINE_PASSED`）。任务持有的摄像头会被拒绝给其他所有尝试打开的 （`connect_camera()`， 另一次启动、瞬时捕获）时，会返回HTTP 409错误（`CAMERA_HELD`），并注明该任务名称及发起者。 |
| `stop(timeout=30.0)` | 停止该任务并关闭其摄像头： `final_stats`、`output_dir` 以及当停止操作未顺利完成时的 `error`。第二次调用将返回相同的响应， 从另一个客户端发起的停止请求也是如此：当第一个请求仍在写出其队列时发送的请求会等待它完成（最长25秒；超过该时间将触发异常，最终统计信息见`stats()`）。 |
| `wait(timeout=None, poll_s=0.5)` | 等待其自行结束（达到 `count` 状态、磁盘已满或发生错误），然后通过 `stop()` 调用其以收集最终计数。 `timeout` 在其仍在运行时触发 `TimeoutError`；没有 `count` 的作业只有在被停止时才会结束。 |
| `job_id`、`kind`、`output_dir`、`start_reply`、 `stopped` | 其 ID 和类型（`continuous`、 `multi-stream`, `align-stream`），其写入位置、起始路由的响应，以及 `stop()` 是否已运行。 |

将其用作上下文管理器，以便在作用域退出时停止该进程。

---

## 项目文件夹与配方

**项目文件夹**是创建新项目的文件夹，由Chloros应用、CLI和SDK（模块`chloros_sdk.system_utils`；`chloros-cli project-folder`）共享：

```python
print(chloros_sdk.project_folder())                # the configured folder, else the default
chloros_sdk.set_project_folder("D:/Flights")       # created if missing
chloros_sdk.reset_project_folder()                 # -> {"path": <default>, "was_set": True}
```

如果已配置的文件夹不再存在，则会像应用程序一样回退到默认文件夹（`Documents/MAPIR/Chloros Projects`）。 当后端位于本机时，提供给 `set_project_folder()` 的相对路径将以您的工作目录为基准。

**配方**是指 `chloros-cli project run PATH RECIPE` 针对已保存项目执行的 YAML/JSON 操作列表。`run_project_recipe()` 则使用 CLI'（随SDK安装），因此同一配方在以下两种方式下的行为完全一致：

```python
r = chloros_sdk.run_project_recipe("D:/campaign/morning.yaml", "D:/Projects/Field_A")
print(r["ok"], r["actions"])
print(r["log"])                                    # what the CLI would have printed

chloros_sdk.run_project_recipe({"actions": [{"apply": {"name": "FrontLeft", "settings": {"exposure_us": 5000}}},
                                            {"wait": "2s"},
                                            {"capture": {"name": "FrontLeft", "output": "pose_a/", "format": "tiff"}}]},
                               "D:/Projects/Field_A")
```

```python
run_project_recipe(recipe, project_path, *, dry_run=False, echo=False,
                   backend_url="http://127.0.0.1:5000") -> dict
```

- **`recipe`** 是一个 `.yaml` / `.yml` / `.json` 文件，或是配方本身：一个包含 `actions`（`apply`、 `capture`、`burst`、`stream`、 `sensor`、`wait`) 组成的字典，并可选地包含 `overrides` 和 `stop_on_error`（默认值为 true： 运行将在第一个失败的操作处停止）。这些操作即CLI参考文档中的[Recipe DSL](cli-reference.md#recipe-dsl)；相对路径`output`将根据脚本的工作目录进行解析。
- **返回值** `ok`（所有操作均成功）、`actions`（成功数量）、`log` （运行器的输出；`echo=True` 在运行时也会打印该内容）以及 `validation`。如果配方文件缺失或无法解析，则会触发 `ChlorosConnectError`。
- **在运行前已检查。**`validation` 即 `{"ok", "errors", "warnings", "steps"}`：每个操作的键必须且仅能是一个已知的动词（`{"capture": {"name": ...}}`， 而非 `{"action": "capture"}` 或 `{"type": "capture"}`），该动词需指代项目中类型正确的相机、数组或传感器。存在错误的配方将**不**执行任何操作（`ok` false， `log`中的错误），无论`stop_on_error`如何规定。未使用的键、 以及该配方所控制但当前未连接的任何数组或光传感器，均属于 `warnings`。配方不会连接设备：请通过项目中的首个（`ChlorosProject.connect_all()`、`chloros-cli project connect`、 或 MCP 工具 `connect_project_devices`）来连接设备。`dry_run=True` 不执行任何操作，并返回 `{"recipe": ..., "validation": ...}`。
- 一个进程中每次只能运行一个配方。YAML 配方需要 PyYAML。

---

## 项目自动化 — `ChlorosProject`

已保存的Chloros项目是一个文件夹，其中包含`cameras.json` + `sensors.json` + `project.json`。`open_project(path, relink=None)`加载清单文件（`relink`：这是已移动项目中缺失源文件当前所在的文件夹，后端在加载该项目时会使用该文件夹——`process()`——参见**已移动的项目** （位于 `ChlorosLocal` 下方）），而 `connect_all` 会将所有已保存的摄像头、阵列和光传感器及其保存的设置恢复在线状态——这与图形界面生成的硬件状态完全一致。光传感器通过后端的 [DAQ 传感器池](#daq-sensor-sessions)，因此需要后端正在运行（参见 [`SensorHandle`](#sensorhandle)）。

### 最小示例

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Light sensors are addressable by name OR slot_id
reading = proj.sensors["Sky"].read()   # (spectrum, is_saturated, ...) or None

# One frame from every camera and array, one spectrum from every sensor
proj.capture_all("./out")

proj.disconnect_all()
```

或者作为上下文管理器：

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### `ChlorosProject` 方法

| 方法 | 描述 |
| --- | --- |
| `connect_all(*, cameras=True, arrays=True, sensors=True, verbose=False, align=None, packed=None)` | 发现并连接所有已保存的摄像头、阵列和光传感器。返回按类划分的连接报告 （`{"cameras": {...}, "arrays": {...}, "sensors": {...}}`，其中包含 `connected`、`missing` 和 `errors`；`arrays` 还包含 `alignment`)。当后端在 `127.0.0.1:5000` 上监听时，将通过该后端查找并连接已保存的摄像头和阵列。若无后端，该过程将通过 `lattice_sdk` 端口自行查找并驱动它们，该端口需要 Arena SDK 运行时和 OpenCV （`cv2`）的支持——SDK安装包不会安装这两者；若无法安装，所有已保存的摄像头和阵列将连同原因一并列在 `errors` 下，而 `missing` 将保持为空。 一个正在运行但拒绝SDK的后端（注销状态：HTTP 401；Copper以下版本：403）并不属于“无后端”情况：每个已保存的摄像头、数组和光传感器都会在`errors`下列出，并附有后端的原因， 且该进程中没有任何内容处于打开状态。每次调用都会重新检查后端，因此登录后或启动 Chloros 后，`connect_all()` 会连接到同一项目对象。 后端中已存在的、包含相同摄像头的已保存数组（来自 GUI 或 `chloros-cli project connect` 的）将原样附加，不会重新应用其保存的设置， 且 `disconnect_all()` 会让其保持运行状态。光传感器始终通过后端处理（参见 [`SensorHandle`](#sensorhandle)）。它绝不会创建新的后端。 |
| `disconnect_all()` | 拆除所有内容。首先释放所有设备；然后，如果后端未确认相机、阵列或光传感器已释放（拆除超时或被拒绝）， 则分别生成一个 `ChlorosConnectError` 条目。对于已关闭的设备、 或无后端运行时，将保持静默。其他客户端已打开的阵列或光传感器将继续运行。该项目的 `with` 退出操作绝不会抛出异常。 |
| `capture_all(output_dir=".")` | 从每个已连接的摄像头和阵列各采集一帧图像， 以及每个已连接光传感器的一组光谱（`SensorHandle.read()`），按顺序采集（先摄像头，再阵列，最后传感器），而非在同一时刻采集。 |
| `stream(camera, overlays=False, fps=10.0)` | 从指定摄像头（或阵列）生成 BGR 格式的 `numpy` 帧。`overlays=False` 使用句柄`grab()` 在 `debayered` 层级： 当项目通过后端连接时，数据通过后端传输；否则采用直接的 `lattice_sdk` 抓取循环（数组生成 `{serial: frame}` 字典）。`overlays=True` 通过 `ChlorosLocal.camera_stream()` → 后端的 `/api/camera/<serial>/stream-annotated` MJPEG 数据流， 并将摄像头保存的 `ui.overlay` 块作为查询参数传递。需要后端模式、OpenCV (`cv2`) 以及一台 **独立摄像头**： 直接模式摄像头会触发 `RuntimeError`（后端无法获取该进程拥有的摄像头），而数组会触发 `NotImplementedError`（按摄像头合成叠加——按名称流式传输成员）。单次调用等效形式： `CameraHandle.capture(annotated=True)`。 |
| `align_arrays(align=True, verbose=False)` | 对当前连接的每个数组运行对齐操作。 |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | 对项目的图像运行校准/索引处理管道（包含 `ChlorosLocal.process`；这四个是**唯一**被接受的关键词参数——`indices=` 等 会引发 `TypeError`；通过 `ChlorosLocal.configure()` 设置索引）。懒加载方式构建一个 `ChlorosLocal()`，该对象会自动启动后端。 |

属性：
- `proj.cameras` — 以名称和序列号为键的 `Dict[str, CameraHandle]`。
- `proj.arrays` — `Dict[str, ArrayHandle]`，以名称和 array_id 为键。
- `proj.sensors` — `Dict[str, SensorHandle]`，以名称和 slot_id 为键。
- `proj.config` — `project.json["config"]` 字典。

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**处理层级。** `capture()`、`grab()` 和 `frame_stream()` 均采用相同的 `processing`
令牌，且该链是累积的——每个级别都会运行其上方的所有内容：

| 级别 | 输出 | 备注 |
| --- | --- | --- |
| `raw` | 1通道拜耳模式，传感器原生 | 不进行去马赛克处理。此级别不支持叠加功能。 |
| `debayered` | 3 通道彩色（**默认** 用于 `CameraHandle` 以及所有 `grab()` / `frame_stream()`；`ArrayHandle.capture()`、`capture_smart()` 和 `stream()` 默认采用 `raw`)：内存中为 BGR 格式（`grab()` / `frame_stream()`），由 `capture()` 保存为 RGB 格式 | 在后端模式下，作为捕获操作： 先应用校准包的暗场偏移（DSNU）对原始数据进行校正，然后使用 OpenCV 的边缘感知去马赛克算法——不使用线性化查找表（LUT）、平场校正或解混色。唯一无需后端模式即可正常工作的级别，该模式下采用的是相机库的普通双线性去马赛克 （无暗偏移）。保存的TIFF文件为光度RGB格式，因此需反转内存帧的通道 (`frame[..., ::-1]`) 以便与文件进行比较。 |
| `radiance` | float32, W/m²/sr/nm, **每波段一个平面** | 完整的辐射度链：去马赛克 + 3×3 解混（多光谱） + DSNU + 平场校正 + NIST 标度，其中已剔除曝光量 × 增益，因此数值为绝对值。 |
| `reflectance` | uint16，32768 = 1.0（或 uint8，255 = 1.0 —— 参见 `bit_depth`），**每波段一个平面** | 辐射度除以降射辐照度（ρ = π·L/E）。需要 DLS/DAQ 读数 — 参见下文注释。 |
| `display` | 8 位 sRGB 风格 | 与 GUI 渲染效果相当：通过相机的活动色彩配置文件实现 CCM + 白平衡 + 伽马校正。 |

仅在直接（无后端）模式下，`debayered` 可用：`grab()` 在另一层级上会触发
`NotImplementedError`，`capture()` 忽略 `processing`，而 `frame_stream()` 会触发
`RuntimeError`。 `reflectance` 需要一个可用的下行读数——帧终点会自动将
汇总的 DAQ 拉入相机的 DLS 插槽，但如果未绑定 DAQ，该链路将拒绝
反射率输出，并在返回的元数据中如实标记降级情况，而不是默默地
返回一个低级别的结果。

> **反射率DN标度——请勿硬编码。** LATTICE反射率使用 `32768` = ρ 1.0 并标记
> XMP `Chloros:PixelScale=32768`；Survey3反射率使用 `65535` = ρ 1.0，且不包含
> `Chloros:*` 标签。读取该标签并以此值进行除法运算。该值定义在 uint16 域中，因此对于所有进行重新缩放的格式（16 位 TIFF、8 位 TIFF、
> 8 位 PNG/JPG、32 位百分比），其值均保持为 `32768`
> — 首先将存储的数据类型归一化回 uint16（从 8 位乘以 257，从浮点数乘以 65535）。唯一
> 的例外是 1.3.0 之前版本生成的文件：当时的 8 位 TIFF 会被 *裁剪* 而不是重新缩放，因此
> 无法用任何比例来描述，且不包含 `PixelScale` 或 MicaSense 元组。对于
> LATTICE 反射率文件中缺失的标签，应将其视为“无有效比例”，而非默认值。

**校准标签：仅限 `raw`。** 捕获数据的 `raw` 文件包含相机的校准数据包： `CalibrationJSON` 有效载荷以及与 MicaSense 兼容的标签（`BlackLevel`、`RadiometricCalibration`、`Band{R,G,B}RadiometricCalibration`、 `MicaSenseVignetting*`），这些标签描述了传感器的原始值。其他各级别的文件（`debayered`、`display`、`radiance`、`reflectance`、`index/`渲染文件）以及每个连续录制/多机位流的帧都包含摄像头的标识 （`XMP-mapir:CameraSerial`、`CameraModel`、`FilterType`、`LensModel`、 `CalBundleSha256`，……），以及帧本身的记录信息 `XMP-mapir:FrameJSON`（曝光、增益、 温度、像素格式、捕获时间、DAQ匹配），因为每个 `process()` 产品都包含该标识。处理过程会根据该文件的序列号和 `CalBundleSha256` 在 Chloros 的校准缓存中查找其校准信息，如果该计算机缺少该校准数据，则会下载一次该包。

#### 由多台相机生成的文件

由多台相机的波段组合而成的文件——多波段叠加图像 (`capture_aligned_multiband()`、`start_aligned_stream()`、`capture_live_index(save_multiband=True)`、`Registration.save_multiband()`)、由多台相机生成的实时索引（`index.tif`、`index_LUT.png`），以及阵列捕获生成的 `composite/` 图像——文件名中会列出所有参与生成的相机：

- **按-摄像机列表，按波段顺序排列**（合成图像：背景层，然后是前景层；索引层， 其波段来源的相机）：`XMP-mapir:SourceCameraSerials`、`SourceCameraModels`、`SourceSensorProfiles`、`SourceProductLines`、`SourceFilterTypes`、`SourceLensModels`、`SourceCalDateUTCs`、`SourceCalBundleSha256s` （即每台相机各波段所经过的校准包，否则——为`capture_aligned_multiband()` / `start_aligned_stream()`堆栈，其波段未经过任何校准包——即相机包Chloros已缓存的， 即其捕获文件的名称），`SourceLightboxCertIds`、`SourceHalogenLampCertIds`、`SourcePackTiers` ——并列列表，每个列表中的第 *n* 个条目描述同一台相机；`-` 表示某台相机无相关值（无校准包，镜头未知）。每台相机仅对应 **一个** 条目：若其包中某个字段有多个值——例如包中的卤素灯证书 — 这些值通过 `;`（`F-1479-1;F-1480-1;F-1481-1`）连接，绝不使用逗号，因此无论列表如何显示，每台相机都对应一个条目。
- **EXIF `SerialNumber`** 包含相同的序列号，以逗号分隔（`214000535,214000551`）。
- 该字段**不**包含单台摄像机的标识（`XMP-mapir:CameraSerial`、`CalBundleSha256`、……）：这些标识分别对应单台摄像机及其数据包，处理过程将据此解析文件的校准参数。由多台摄像机的波段计算得出的索引 （`index.tif`、`index_LUT.png`）计算得出的索引包含 EXIF 数据 `Model` `LATTICE`（即拍摄装置）， 而非单台相机的 `LATT-…` 型号。**单台**相机的波段文件——无论是按波段划分的TIFF文件、`capture_aligned_multiband(per_camera=True)`帧文件，还是单台相机的堆栈或实时索引——都包含该相机自身的标识标签， 正如其他所有捕获文件一样。`capture_aligned_multiband()`的堆栈、各波段文件和各相机文件，以及每个`start_aligned_stream()`堆栈及其各波段文件，都包含其所属组的捕获时间 （即参考相机的曝光时间，UTC），而单色相机的 `per_camera` 帧仅包含一个平面，其格式与 `bit_depth` 堆栈相同。**相机方向（翻转/旋转）。** 来自 `grab()` / `frame_stream()` 的帧在各个层级上始终保持
相机的原始方向。保存的捕获数据遵循相机的方向
设置（GUI 中的“水平镜像”、“垂直镜像”或“旋转”）：

- **`raw` 和 `debayered` 文件完全反映了传感器读取的数据**，无论设置如何——旋转或翻转的拜耳
  马赛克将形成不同的彩色滤光片图案，而校准图是
  基于传感器自身的网格。
- **`display`、`radiance`、`reflectance` 及索引文件均经过定向处理**——在完成所有
  校准运算后旋转了一圈。其钳位掩膜位于同一网格上。
- **每个拍摄文件都会在XMP中记录该设置**：`XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical`（`true`/`false`)、`ChlorosRotation`（0 / 90 / 180 / 270，顺时针，翻转后
  翻转之后）以及 `ChlorosOrientationApplied` — `false` 适用于原始和去拜耳化图像，`true` 适用于定向
  图像。 `ChlorosProcessing` 列出了对该文件像素所应用的操作（原生格式时为
  `none`）。 这包括连续录制（相机自身的设置，
  未应用）和多机位流（身份：其成员帧为原生格式，如同
  数组捕获一样）的每一帧。
- **处理项目时会将记录的定向**应用于其从原始数据生成的产品
  （首先在传感器坐标系中进行校准，接着进行阵列对齐变形，最后应用方向），
  因此这些产物与实时捕获的产物相匹配——包括未经校准的传感器响应/已校正暗角的
  备用产物；其去拜耳化产物保持原生状态。
- **Chloros图像查看器会显示根据其记录旋转过的原始或去拜耳化文件** —— 仅限显示，
  文件本身不会被修改；没有记录的文件则按存储时的状态显示。
- **没有记录的捕获将按原生格式进行处理。** 在 1.3.0 版本之前，单摄像头捕获
  会自行转换其原始文件和去拜耳化文件；Chloros 无法区分此类原始文件与原生文件，
  也不会进行猜测。
- **数组捕获默认将每个成员文件保持为原生格式**，并记录其标识；
  `ArraySession.capture(orient=True)` 会处理其生成结果（参见数组捕获控制项）。

#### 按波段输出与混合数组

在 `radiance` 和 `reflectance` 级别，相机返回 **该相机每个波段对应一个平面** —— M3C 相机为三个 （其三个未混合波段，BGR通道顺序）则有三个平面，M3M（其
单个窄带滤光片）则有一个平面。单色相机的复制平面会被合并，因此**混合
M3C/M3M 阵列在每个序列中会产生不同的阵列形状**。请从帧
元数据中读取 `bands` 键值以了解每个平面的具体信息；切勿根据通道数量推断：

```python
for serial, (arr, hdr) in arr_handle.grab(processing="reflectance",
                                          with_metadata=True).items():
    print(serial, arr.shape, hdr["bands"])   # ('…', (1536, 2048, 3), ['nir','green','red'])
                                             # ('…', (1536, 2048, 1), ['F685'])
    rho = arr.astype("float32") / hdr["pixel_scale"]
```

在使用这些值之前，请务必检查 `hdr["processing_level"] == "reflectance"`。 降级
（无数据采集读数、无校准包、未知频段布局）会在 `processing_level` 和
`radiometric_refused` 中报告，而非提升——您获得的阵列仍然是一个有效的帧， 只是
并非您所请求的数据。被降级的已保存捕获数据在其 XMP 中包含与 `Chloros:RadiometricRefused`
相同的代码 （例如 `no-band-layout`、`no-dls-snapshot`）。

#### 每次读取一个触发信号 — `group_id`

**数组成员的数据来自数组的同步帧组，而非每台摄像机自身的
抓取缓冲区。** 硬件同步数组中的摄像机会共同输出图像，但每台摄像机的 N 次读取
并不会一起*采样*——读取每台摄像机最新缓冲帧会跨越触发
边界，而单帧丢失便会导致整个阵列失步，且无人察觉。

从组中提供数据是必要的，但仅靠这一点还不够：每个请求仍会
在自身所在的瞬间对*最新的*组进行快照，而抓取循环可能在此期间发布新帧。
在实际的4摄像头阵列上测得，**20次并行读取中有12次跨越了两个组**——且始终
恰好相邻。因此读取操作被**固定**：

1. 每帧都携带 `group_id`，即阵列的单调发布计数器。
2. `grab()` 读取主摄像头（或当主摄像头无法响应时，读取下一个可响应的摄像头），
   随后将剩余摄像头“固定”到其落脚的组 ID 上（HTTP
   端点上的 `?group_id=N`）。
3. 后端会将最近的组保留**至少半秒**，因此已被后续发布超越的同级组
   仍会响应该触发器——这段时间足以让每台摄像头的
   反射率导出数据完成往返传输。对于后端不再
  持有的绑定，系统会返回 `409` 和 `group_aged_out: true` 错误，而非静默地
   并返回不同的触发信号——`grab()`会重新读取该集合。

启用固定后重新测量：**20次读取中有20次返回了一个触发信号。** `grab()`会自动为您完成此操作；
仅当直接调用HTTP端点时，才需要使用`group_id=`。

- 无法在固定组中服务的摄像头——因未包含在该组中，或拒绝请求
  （例如它不具备的 `band`，或无法生成的电平）——将被 **省略而非通过其他触发器进行补全**
  ，而其他摄像头仍会被返回。
- `grab()` 返回的字典是一个 `GrabResult` （一个普通的 `dict` 子类；
  `from chloros_sdk import GrabResult`）： `.errors` 将每个
  被省略的序列映射到其读取操作引发的异常，而 `.group_id` 则是共享组。一个空的
  `.errors` 表示所有摄像头都属于该组。
- 独立摄像机没有组，报告 `group_id: None`，并且会**拒绝** `group_id=`
  引脚，而不是忽略它。

```python
frames = arr_handle.grab(processing="reflectance")
for serial, err in frames.errors.items():
    print(f"SN:{serial} omitted: {err}")       # e.g. status=400 ... band 'F685' not in [...]

frames = arr_handle.grab(processing="reflectance", with_metadata=True)
ids = {hdr["group_id"] for _a, hdr in frames.values()}
assert len(ids) == 1, f"straddled triggers: {ids}"     # belt and braces
```

该计数器会在数组成员的**每个摄像机级 MJPEG 流**中发布——包括
预览流（`/api/camera/stream/<serial>`）、标注预览流（`/stream-annotated`）以及
反射率流（`/stream-reflectance`）——在每个部分中均以`X-Chloros-Group-Id`的形式发布， 此外还有
`X-Chloros-Frame-Id` 和 `X-Chloros-Timestamp-Ns`。流是连续的而非
特定时间点的，因此通过收集各部分并基于该头信息进行分组来实现匹配——无需
设置引脚。独立摄像头不属于任何组。 参见
[按摄像机划分的 MJPEG 流](#per-camera-mjpeg-streams-no-rtsp)。

**`frame_stream()` 绝不会陷入静默状态。** 因可自行消除的原因而失败的 tick
会被跳过——摄像机流对此不返回任何结果，数组流则返回那些
已响应的摄像机 （可能为零）。这些原因包括：尚未收到帧；摄像头的抓取线程暂停了一
两秒（`404 not streaming`，当另一台摄像头连接
或断开，或设置变更导致池重新平衡时，后端会返回此状态）；后端重启或 `5xx`；对齐
仍在稳定中 （`409 … retry shortly`）；以及因数据采集卡（DAQ）尚未获得读数
而导致的反射率拒绝（`no-dls-snapshot`，过期或空读数）。 若在下一个时钟周期再次发生拒绝，情况只会重演——例如
未知错误 `band`、基于辐射度的 `bit_depth=8`、缺乏辐射度标度的校准包，以及因**完全没有数据采集设备*
而导致的反射率拒绝*（既未绑定到相机或其阵列，也未连接到
后端： 该拒绝的 `dls_source` 即为 `"none"`），任何其他 `4xx` ——
都会触发 `chloros_sdk.FrameFetchError` （即一个携带 `.status` 和 `.body` 的 `RuntimeError`）
。同样，**60 秒**的跳过时钟周期（期间无帧数据）， 无论该 `fps` 如何——时间
足够长以度过对齐稳定期和后端重启，又足够短，使得一台
真的已失效的摄像头会终止流。该放弃错误也是 `FrameFetchError`，无论是针对单个摄像头还是
摄像头数组，最后一次读取产生的自身异常则为 `.errors` （对于相机，则为 `__cause__`）。对于
数组，“拒绝”意味着所有相机均被拒绝；异常代码 `.errors` 包含每台相机的
具体原因。

**数组 `frame_stream()` 会将每个触发器返回一次**（`ArrayHandle` 和 `ArraySession` 均如此）。若
某个时钟周期中最新的一组数据正是其已输出的触发条件，则该周期将等待下一个触发条件，而非
再次输出同一组数据，因此数据流的运行速率将取 `fps` 与数组自身帧
，且连续的数据集绝不会共享同一个 `group_id`。停止发布数据的数组会触发
`FrameFetchError`（“60 秒内无新触发器”），而非重复其最后一个数据集。若无此机制，当
流请求的数据量超过数组发布量——或在连接后立即开始——则会分发
重复数据： 在一项实测中，20组数据包含11个不同的组ID。

```python
from chloros_sdk import FrameFetchError

try:
    for arr in cam.frame_stream(processing="radiance", bit_depth=8):
        my_analysis(arr)
except FrameFetchError as exc:
    print(exc.status, exc.body)        # the HTTP status and the backend's reason
```

#### 导出单个频段 — `band`

`grab()` / `frame_stream()` 配合 `band="<name>"` 可将摄像机的某个波段导出为
单通道 `(H, W, 1)` 数组 ——其形状与单色相机生成的相同，因此无论单波段输出源自 M3M 还是通过拆分 M3C 获得，其
处理方式均完全一致。FRGN 的红波段：

```python
red = cam.grab(processing="reflectance", band="red")        # (1536, 2048, 1)
```

可接受的命名形式包括完整的布局名称（`Red_660`、`NIR_850`、`Green_550`）或其简写
前缀 （`red`、`nir`、`green`），不区分大小写。若名称未知，系统将拒绝该名称并列出可用的
名称。

> **选择依据是名称，而非通道编号——这是刻意为之。** “通道 1”在
> 数组的 BGR 通道顺序中是 `Green_550`，但在滤波器代码的字母顺序中为*红色*（`RGN`），而两者
> 不一致。若选错，将产生一个完全合理的错误频段帧，而
> 下游任何设备都无法检测到。 正因如此，这里没有数字索引。

#### 流对齐 — `align`

`align=True` 将帧变换为其数组的通用参考帧。每台摄像头的变换
均采用**相同的**配置文件分辨率和裁剪 ROI， 因此独立打开的各相机数据流
输出时已实现**相互配准**——无需同时请求它们：

```python
frames = arr_handle.grab(processing="reflectance", align=True)   # {serial: array}
# or one MJPEG stream per camera, each co-registered on its own:
#   GET /api/camera/<serial>/stream-reflectance?align=1
```

要求与行为：

- 摄像头必须位于具有对齐
  配置文件的**硬件同步**阵列（M8同步线缆）中——共注册假设阵列成员同时曝光，而软件或
  自由运行阵列无法满足。否则将因 `409` 报错而被拒绝。
- **请求将被拒绝，绝不会在未对齐时静默通过。** 捕获路径可能会在发出警告后保存未对齐的数据，
  因为文件会保留其来源标签，并可在稍后重新注册；而实时流则没有
  这样的第二次机会，因此缺少配置文件属于错误，而非退而求其次的处理方式。
- **请求对齐会启动该过程。** 在没有配置文件的阵列上请求 `align` 会触发
  校准，并返回 `409` 以及当前等待的状态（例如 *“等待曝光
  在 214000551 (1/3) 处稳定”*）——重试后通常会在几十秒内成功。
  这在无头模式下尤为重要：连接时的自动对齐会在 30 秒后放弃，且在
  都会被跳过，因此对于曝光
  收敛较慢的数组，以往整个会话都无法生成曲线，且无法
  在不打开桌面组合视图的情况下获取曲线。`alignment_status` 还会报告
  `ae_align_holdouts`，其中列出了仍在稳定中的相机及其进度。
- 需为同一阵列的每个数据流赋予**相同的 `max_h`**，否则其裁剪区域将落在不同的像素
  网格上。
- 对齐操作在位深度编码*之前*进行，因此变换会插值全精度
  值。

#### `bit_depth` — 以精度换取带宽

`grab()` 和 `frame_stream()` 支持 `bit_depth=16`（默认）或 `bit_depth=8`：

| `bit_depth` | 反射率编码 | 开销 |
| --- | --- | --- |
| `16` | uint16，32768 = ρ 1.0， 留有余量至 ρ 2.0 | 全精度。 |
| `8` | uint8 覆盖 `0..refl_max` （默认 ρ 1.0） | 字节数减半；在默认上限处约 0.004 ρ 的量化。超过上限的值将被截断——详见下文。 |

##### 8 位上限 — `refl_max`

`refl_max` 设置 ρ 的 8 位范围， 因此 `pixel_scale = 255 / refl_max` 和 ρ 仍为
`pixel / pixel_scale`。`grab()` 和 `frame_stream()` 将其作为关键字应用于摄像机和
数组句柄上均将其视为关键字（HTTP 端点上的 `?refl_max=`）。提高该值可为镜面
高光预留动态范围，但步长会相应变粗：

```python
frame = cam.grab(processing="reflectance", bit_depth=8, refl_max=1.5)   # 0..1.5 in 0..255
```

| `refl_max` | `pixel_scale` | 步长 |
| --- | --- | --- |
| `1.0`（默认）| 255 | 0.0039 ρ |
| `1.5` | 170 | 0.0059 ρ |
| `2.0` | 127.5 | 0.0078 ρ |

> **自动曝光无法修复 8 位剪切问题，尝试使用它反而会使数据质量变差。**
> ρ = π·L/E（已剔除曝光和增益因素），因此 ρ 是场景的属性，而非
> 相机设置的属性——在实时阵列上测量发现，将曝光减半会使平均 ρ 变化 1.3%。提高
> 曝光量以“适应”动态范围，反而会使像素进入**传感器**饱和状态，这
> 会导致 L 被低估，并使 ρ 产生**偏低**的偏差，且不会触发错误提示（实测：最大 ρ 从 1.203 降至 0.985，
> 曝光量为 2× 时，平均值下降 −23%）。 请使用 `refl_max` 选择动态范围；避免传感器
> 饱和本就是自动曝光（AE）的功能，这保障的是 ρ 的 *有效性* 而非其动态范围。

**请阅读 `reflectance_clipped_fraction`， 而非该标志。** `reflectance_clipped_at_1_0` 仅因
单个像素触发——在真实场景中，它报告了 `True`，而实际比例为 `0.000001`，即一百万个像素
。该分数值正是决定是否需要提高上限的依据。

**`bit_depth=8` 仅表示反射率。** 若请求 `radiance` 则会被拒绝而非
响应： 反射率具有0..1的固定范围，可在8位编码中保持不变；而辐射度则没有
固定范围，因此要将其压缩到8位，需要对每帧进行百分位拉伸，其映射关系
每帧都会变化——这只是一张图片，而非测量值。

#### 实时反射率流（MJPEG）

`GET /api/camera/<serial>/stream-reflectance` 以
`multipart/x-mixed-replace` 的形式提供每波段 8 位反射率——M3C 格式为 3 通道 JPEG，M3M 格式为灰度。查询参数：
`fps`（默认 10）、`jpeg_q`（默认 95）、 `max_h`（0 = 原生模式），`band`（一个命名波段，如
上所述），`align` （`1` 用于协同注册——规则与 SDK 的 `align` 相同），`refl_max`（8 位
编码上限，如上所述）。每个部分包含
`X-Chloros-Group-Id`、`X-Chloros-Frame-Id`、`X-Chloros-Timestamp-Ns`、`X-Chloros-Bands`、
`X-Chloros-Pixel-Scale`、`X-Chloros-Aligned` 和 `X-Chloros-Reflectance-Clipped`。
**来自不同摄像头但携带相同 `X-Chloros-Group-Id` 的数据包属于同一个触发器** —
这就是同一数组中每台相机的多个数据流如何被重新组装的，并且由于数组成员来自同步帧组（参见
[每次读取一个触发器](#one-trigger-per-read--group_id)）。当连接时无法获取反射率时，它会返回 `409`，
若运行过程中反射率不可用则终止流，因此流停滞
意味着“不再有反射率”，而非“仍正常，只是静止”。

> **JPEG 是有损压缩的 — 这些像素代表测量值。** 压缩误差会直接体现在
> 反射率值上，且在边缘处最为严重，而边缘正是材料边界所在。因此，质量
> 默认设置为 95。若需完整保留数值，请使用
> `grab(processing="reflectance")` / `frame_stream(...)`，它们会返回相同的结果，且路径中不包含
> 编解码器（若受限于布线成本而非编解码器，则使用 `bit_depth=8`）。

> **EXIF 信息将保留在导出文件中。** `process()` 将源捕获文件的 GPS 数据块
> **及其 ExifIFD** 复制到每个产品中，因此导出文件包含 `FocalLength`、`FNumber`、
> `ExposureTime`、`ISO`、`DateTimeOriginal` 和 `CameraSerialNumber`，以及
> 地理参考信息。`FocalLength`是Pix4D用于计算地面采样距离的依据——若缺少该数据，
> 重建结果将退化为严重失真的比例尺（曾有实测案例将411米长的场地
> 误算为47.8公里长）。该副本特意未采用 `-all:all`：IFD0 的结构标签会破坏
> LATTICE 的输出，而 `ExifImageWidth`/`Height` 则被排除在外，因为它们描述的是源
> 采集数据，而非导出的栅格。

采集阶段子标志（适用于辐射级——`radiance`、`reflectance`、 `display`）：

| 标志 | 默认值 | 含义 |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + 平场校正 + 3x3 解混 + NIST 辐射级标度。 |
| `apply_white_balance` | `True` | 白平衡查找表（WB LUT）。当数据采集（DAQ）与相机绑定时，支持 DLS。 |
| `apply_index` | `False` | 植被指数评估。 |
| `index_expression` | `None` | 覆盖公式。非为空 → 自动启用指数。 |
| `annotated` | `False` | 叠加 GUI 装饰（斑马线/网格/峰值）。不适用于 `raw`。 |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **返回值是一个 `chloros_sdk.CapturePaths`， 类型为 `CapturePathMap`，而非 `Dict[str, str]`。**
> `chloros_sdk.CapturePathMap` 是 `Dict[str, Union[str, List[str]]]`：单级
> `processing` 为每个序列号提供一条路径，而 `processing="all"` 则提供
> **有序列表**，其中包含该相机保存的所有产品。`ArrayHandle.capture()` 接受
> 不接受 `levels=`，并且会忽略其不认识的关键词参数（`levels`、 `daq`、
> `render_index`、`force_daq`），因此请仅传递此处显示的参数 （`aligned=False`
> 用于保存本机文件，与 `ArraySession.capture()` 相同）；对于这些控件，请使用
> `ArraySession.capture()`。如果存在实时组合复合流，则该流将以额外的
> `"combined"` 键下，而非序列键下。假设使用 `str` 的代码在
> 列表形式下会报错，且没有任何类型检查器提出异议——注释中提到 `Dict[str, str]`
> 这一标注，因此才有了该别名。若需使用扁平形式，请进行
> 规范化处理：
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

> **无法这样写：`.failed`。** 映射 `capture()` 返回的是一个
> `chloros_sdk.CapturePaths` ——一个普通的 `dict` 子类，因此上述所有内容
> 无需修改即可正常工作——该类还包含 `.failed`：即后端无法写入的文件，
> 其条目与 [`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed)
> （`serial`、`level`、 `error` 以及 `disk_full: True`，这些条目出现在磁盘已满的情况下）。 部分
> 捕获仍会返回所有已写入的路径；在磁盘已满的情况下，映射表为
> 空，且 `.failed` 说明了原因。 `.skipped` 列出了每台摄像头的跳过项，
> `.group_seq` 列出了同步组， 而 `.alignment` / `.oriented` 则是
> 后端的，如 `CaptureResult` 所示。`capture_smart()` 将它们返回为
> `result["frames"].failed` 和 `result["failed"]`。
>
> ```python
> paths = arr.capture("./out", processing="all")
> if any(f.get("disk_full") for f in paths.failed):
>     raise SystemExit("drive full -- free space and capture again")
> ```

### 数组对齐

`ArrayHandle` 公开了完整的对齐表面。默认情况下，分析结果仅在会话内有效——若要持久化，请显式调用 `export_alignment()`。

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Manual control points: each camera's pixel coordinates of the same
# features, the reference camera's included (>= 2 per camera). Every one
# of num_frames is solved from these points.
arr.calibrate_alignment(method="manual", model="rigid", control_points={
    "214701173": [[412, 300], [1630, 310], [1020, 1180]],   # reference (master)
    "214701292": [[418, 297], [1637, 306], [1027, 1177]],
})

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### 连接时对齐

`connect_all(align=...)` 可在连接时自动对齐每个数组：

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

若未指定，则回退到 `project.json["config"]["auto_align_on_connect"]`。

### `SensorHandle`

一个已保存的光传感器（DAQ-U、DAQ-M 或 DAQ-E），由 `connect_all()` 在后端的 [DAQ 传感器池](#daq-sensor-sessions) 中由 `connect_all()` 打开的已保存光传感器——该池与 `connect_daq_sensor()` 和 `chloros-cli daq pool-connect` 所使用的池相同。

```python
reading = proj.sensors["Sky"].read()
if reading is not None:
    spectrum, is_saturated, integration_time, x, y, z = reading
```

- `read()` 等待传感器的 **下一个** 光谱——调用时已缓冲的光谱不计入 ——并返回 `(spectrum, is_saturated, integration_time, x, y, z)`；若 5 秒内未收到任何数据，则返回 `None`。 已暂停的数据流被恢复。当传感器经过校准时，`spectrum` 的单位为 W/m²/nm。
- `.sensor` 是传感器的 [`DAQSensorSession`](#数据采集传感器会话)，对于其他所有情况：`latest(n=...)`、`status()`、`record_start()` / `record_stop()`。
- `.slot_id`、`.model`、 `.serial`、`.name` 和 `.settings` 是从 `sensors.json` 保存的条目。

**`connect_all` 恢复的内容。** 每个传感器都会在其保存的地址（即其 COM 端口、蓝牙地址或 DAQ-E 主机）处被打开；如果项目记录了传感器的 ID，则必须是该传感器：该地址处的其他设备将被拒绝，视为未连接。其保存的积分时间、帧均值和自动曝光参数将被应用，同时其帽值校正也将生效：

| 已保存的校准配置 | 连接方式 |
| --- | --- |
| 校准配置文件（例如 `fov_30`） | 该配置文件。 |
| 未选择 | `sunshine_cosine`（连接默认值），记录为假定值（`cap_id_source=auto_default`），因此处理过程会发出警告。传感器出厂时未安装保护盖：请选择已安装的保护盖。 |
| `none` — 在图形用户界面中选择为 **无（裸传感器）** | 无盖校正。 |

DAQ-E-S 传感器不采用任何盖帽配置文件，无论之前保存了什么：其扩散器是校准的一部分（参见 [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor)）。由早期Chloros保存的项目将裸传感器记录为无选项，因此它会重新连接到`sunshine_cosine`；请在GUI中再次选择**无（裸传感器）**并保存项目。

如果传感器已在传感器池中被其他客户端打开（通过CLI或另一个脚本），则直接连接该传感器，无需重新应用设置， 且`disconnect_all()`会让其保持运行；而由`connect_all()`打开的传感器则会被释放。与`connect_daq_sensor()`的情况类似，Chloros图形界面**不**属于传感器池的一部分： 请在通过脚本连接项目之前，先在 GUI 中关闭该传感器。**要求与报告。** 光传感器需要一个正在运行的Chloros后端，且该后端需登录付费的Chloros+套餐（Copper及以上级别），而`connect_all()`从未启动过此类后端。若无后端运行，所有已保存的传感器都会列在`report["sensors"]["errors"]`下，并标注此情况。 后端无法打开的传感器也会列在此处，并附有后端给出的原因；对于光传感器，`report["sensors"]["missing"]` 将保持为空。

---

## 直接硬件（无需后端）

若希望 LATTICE 摄像头（CI、无头机器人、嵌入式设备）完全不依赖后端，请直接导入 `lattice_sdk` —— 该模块由 `chloros_sdk` 重新导出。 关于 `CAMERA_AVAILABLE` 的注意事项： `lattice_sdk` 虽包含在 SDK 安装包中，但需要 Arena SDK 运行时和 OpenCV（`cv2`）的存在，而该安装包并未安装这两者。光传感器需通过后端处理—— [`connect_daq_sensor()`](#daq-sensor-sessions) 或项目中的 `proj.sensors` —— 仅有一个例外，即 DAQ-M 频带幅度辅助函数 [`DLS`](#daq-m-band-magnitudes--dlsband_magnitudes)，该助手还需 `bleak`。

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### 预设与触发器

四个预设中有三个采用**自由运行**模式：相机持续曝光，且
`capture()`会返回下一帧。`triggered`是例外 ——它会使
相机处于待机状态，等待第2线的硬件沿信号，因此在信号出现前不会进行任何拍摄。

| 预设 | 触发方式 | 适用场景 |
| --- | --- | --- |
| `default` | 自由运行 | 通用 |
| `high_speed` | 自由运行 | 8 位，60 fps 上限，短曝光 |
| `high_quality` | 自由运行 | 12 位，无帧率限制 — 静态照片的常规选择 |
| `triggered` | **就绪，第 2 行** | 相机通过 M8 同步线连接，由其他设备触发 |

如果您选择 `triggered`（或自行设置为 `trigger_mode="On"`），且
第 2 线未被任何设备驱动，则每个 `capture()` 都会超时 ——这是正确的，因为您要求
相机等待。SDK在发生此情况时会进行说明；请参阅
[捕获期间出现 SC_ERR_TIMEOUT](#direct-hardware-backend-free)。

> **注意 — 连接时的“GVSP probe”/`SC_ERR_TIMEOUT -1011`消息并非错误。**
> 连接时，SDK会尝试协商**巨帧**（9000字节的GVSP数据包）以获得更高的吞吐量。 在直接点对点的网卡链路上（例如链路本地 `169.254.x.x` 地址），网络通常无法传输巨帧，因此该探测会超时并记录如下内容：
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> 这是**设计中的备用方案**：SDK会自动恢复为标准的 1500 字节数据包，摄像头仍会正常连接（随后的 `[chunk-enable …]` 行属于正常的连接序列）。抓取功能仍然有效。
>
> 您可以跳过此探测，但**它不仅仅是一个日志抑制器——它会关闭巨型帧功能。** 无论您的网络质量如何，摄像机对“禁止分片”（Don't-Fragment）ping 请求的响应大小上限始终为 1500 字节，因此仅靠 ping 测试永远无法检测到巨帧；只有此探测命令才能实现。若将其禁用，摄像机将永远使用标准的 1500 字节数据包， 无论在何种网络环境下：
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> 仅在您*确知*网络无法承载巨帧的情况下才值得启用，此时每台摄像头的连接时间可节省约一秒。 由于这是一种实质性的权衡而非表面上的调整，SDK现在会在您使用时明确提示：
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **除非有充分理由，否则请勿更改此设置。**若保持启用状态，每个后端会话中会针对每台摄像头进行一次测量，并在其链路发生变化时（例如更换网卡、链路速率改变，或因交换机切换导致链路中断）再次测量；重新连接时将复用该结果。**若交换机后端发生变化但交换机仍保持在线，请重启后端以重新进行测量。** 首次连接时，此项检测偶尔可能需要几分钟，日志中会记录相关信息。
>
> 若您*希望*获得巨包吞吐量，请启用端到端巨包支持 （网卡 MTU 设为 9000 且交换机支持传输）功能，或者在确认链路支持时使用 `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` 将其固定——不过建议采用按命令设置的 `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` 而非永久设置， 因为固定大小的设置会跳过探测过程，并停止对前端网络的自适应调整。路径中的**每个**设备都必须支持巨包传输——包括任何 PoE 分路器或注入器，这通常是本应支持巨包的设置却无法传输巨包的原因。

> **在 `capture()` / `grab*()` 期间出现的 `SC_ERR_TIMEOUT -1011` 属于另一个问题 ——那才是真正的错误。**
> 上述说明仅涉及由**连接时间探针**记录的 `-1011`。如果**捕获**中出现相同的错误，则意味着摄像头已成功连接，但未发送任何图像：
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> 关键线索在于：摄像机的*控制*通道状态正常——发现功能正常，设置和 `[chunk-enable …]` 写入操作均成功——但*每一帧*都会超时。
>
> **通常的原因是相机已启用硬件触发模式。** 出现 `trigger_mode="On"` 和 `trigger_source="Line2"` 错误时，在 M8 同步线缆上接收到电平变化之前，相机完全不会发送任何数据。 如果该线路没有驱动线缆，每次抓取都会无限期等待。相机没有故障，网络也正常 ——它只是完全按照指令在运行。
>
> `CameraSettings()` 以及 `default` / `high_speed` / `high_quality` 预设为自由运行模式，当处于就绪状态时抓取超时，系统会显示具体原因，而非仅输出简单的 `-1011`。`PRESETS["triggered"]` 按设计会使 Line2 进入就绪状态。
>
> 要强制任何摄像头进入自由运行模式：
>
> ```python
> import copy
> settings = copy.deepcopy(PRESETS["high_quality"])   # PRESETS entries are shared
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> 如果使用 `trigger_mode="Off"` 时仍超时，则说明相机确实未传输数据——请将日志和 `ip link show` 发送给我们。

#### 色彩配置文件（RGB 实时预览）— `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` 用于选择 RGB 摄像头上 **实时预览** 的显示色彩配置文件（多光谱摄像头将忽略此设置）：

| 配置文件 | 含义 |
| --- | --- |
| `raw` | 完全绕过辐射度链。 |
| `linear` | DSNU + 平滑校正 + 白平衡，无 CCM，无伽马校正。 |
| `natural` | 线性 + 测量得到的CCM + sRGB伽马，仅采用基础处理（色度平滑 + 高光去饱和） ——这是逼真的默认设置。 |
| `enhanced` | `natural` 加上完整的 Hub-Parity 后期处理（去色散、鲜活度、CLAHE 局部对比度）。画面更丰富，但**每帧后期处理成本约为两倍**，因此实时帧率较低。 |
| `custom_temp` | `natural`，但白平衡固定为 `custom_cct_k` 开尔文 （忽略 DLS；后端侧限制在 2000–10000 K 之间）。 |

该配置文件仅作为**实时预览**的速率/画质调节控件：保存的截图始终会获得完整的丰富后期处理效果，无论选定何种配置文件，因此选择 `natural` 以换取帧时间并不会降低最终写入磁盘的画质。 未知配置文件会触发 `ValueError`；当可访问 chloros 后端时，该更改也会被 POST 至后端，以便下一帧预览反映该更改（未连接后端的直接 SDK 用户仍会收到设置变更）。

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### 高光（RGB显示） — `set_highlights`

`LatticeCamera.set_highlights(value)` 会提取**显示器**上最亮、最白的像素 渲染效果——这是针对明亮白色表面渲染成扁平白色块的修复方案。`value` 即 `-100`..`0`（已限制）；`0` 处于关闭状态，此时渲染效果与未启用该设置时完全一致。更强的设置会降低显示器的白色亮度（在 `-100` 时降至约 60%）并拓宽色调曲线， 因此，那些被预览的自动亮度功能推至纯白的区域将恢复其细节。

这虽然是一个显示控制参数，但与饱和度和对比度（仅改变实时预览效果）不同，它也会改变您保存的内容：实时预览和**显示级**的拍摄文件都会遵循该设置，因此这些文件也会显得更暗（它们包含 `Chloros:DisplayHighlights`）； 辐射度和反射度导出文件则永远不会受此影响，且该设置无法恢复已剪切的传感器数据——对此请降低曝光值。多光谱和单光谱相机将忽略此设置。该值存储在 `settings.highlights` 中，并 POST 到可访问的 chloros 后端， 这与 `set_color_profile` 完全相同；其 CLI 等效命令为 `lattice highlights --set N`。

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_highlights(-60)                      # 0 = off, -100 = strongest
```

#### 单色 (M3M) 相机与 `Calibration`

单色 **M3M** 相机（`M3M-<lens>-F<wavelength>`）为单波段： 仅有一个灰度平面，无拜耳马赛克，无 3×3 光谱串扰矩阵。`Calibration` 可识别该特性，并暴露一个 `is_mono` 标志。 反射率仍作为每波段的辐射测量图适用（解混矩阵为单位矩阵），但在单台相机上进行多波段运算会产生结果而非返回无意义数据：

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

若要利用单色硬件构建植被指数，请将不同波长的多台 M3M 相机组合成对齐的多波段堆栈（参见 [阵列对齐](#array-alignment)），并在该堆栈上计算指数，而非仅针对单台相机。

#### DAQ-M 波段光度值 — `DLS.band_magnitudes`

`DLS(mac_address, filter_model="RGN-IMX265")` 通过其专用的 BLE MAC 地址经蓝牙连接至 DAQ-M。 这是唯一一个不通过后端而是直接与传感器通信的光传感器类，因此它需要直接相机类所需的组件（`CAMERA_AVAILABLE`：Arena SDK 运行时和 OpenCV），此外还需 `bleak` 用于蓝牙 （`pip install "chloros-sdk[camera]"` 或 `pip install bleak`；若缺少该组件，`DLS` 将触发 `DLSError`）。连接在其他位置的 DAQ-M （后端池、项目、GUI）将停止广播，因此请先在该处释放它。 **必须指定 `mac_address`**：由于每个设备都有自己的默认值，因此此处没有默认值，且若未指定该值，`DLS()` 会在尝试任何连接之前触发 `DLSError`。使用 XPR 查找 MAC 地址OTX003351（参见 [发现](#discovery--finding-an-address-to-connect-with)）或 `chloros-cli daq pool-discover --only ble` 查找 MAC 地址。

`DLS.band_magnitudes(spectrum=None, *, spectral_response=None)` 通过摄像头的三个通道响应整合 DAQ-M 频谱（当 `spectrum` 为 `None` 时的最新读数； 若尚无该值，则为 `DLSError`）与摄像头的三个通道响应进行融合，并返回 `{"red": …, "green": …, "nir": …}` ——键名按此顺序标识各通道；对于RGB摄像头，第三个曲线（`nir`）即为蓝色通道。

- 若未指定 `spectral_response`，则使用滤镜模型的默认曲线（即 `filter_model` 构建时所用的 `DLS`），该曲线是在单个设备上测得的。
- `spectral_response` 则采用相机**自身的**测量曲线： 一个字典 `{"wavelengths": [...], "R": [...], "G": [...], "B": [...]}`，该字典存储于该相机的校准包中（`color_correction.spectral_response_smoothed` — RGB相机；多光谱包不包含此类曲线）。若无法使用该字典，则回退到机队曲线。

```python
from chloros_sdk import DLS

# camera_curves: {"wavelengths": [...], "R": [...], "G": [...], "B": [...]}
with DLS(mac_address="AA:BB:CC:DD:EE:FF") as dls:
    reading = dls.acquire()
    fleet = dls.band_magnitudes(reading)                         # fleet curve
    own = dls.band_magnitudes(reading,
                              spectral_response=camera_curves)   # the camera's own
```

CLI 中的 `chloros-cli lattice dls --camera SERIAL` 通过后端执行相同操作，从已加载或缓存的校准包中查找相机的曲线，并说明使用了哪台相机的曲线。 在HTTP环境下，即为`POST /api/dls/cli-read`，并包含`mac` （必填：若缺此参数，响应将为 HTTP 400 错误，即 `code` `MAC_REQUIRED`），且正文中包含 `camera_serial`； 当使用该相机自身的曲线时，响应中的 `band_magnitudes_source` 变为 `camera <serial>`；若使用机队曲线（即相机无曲线：非 RGB，或未加载包 或未缓存）。

---

## 异常

捕获基类以处理“Chloros出现的任何错误”：

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` 和 `ChlorosConfigurationError` 是 `chloros_sdk` （以及 `chloros_sdk.exceptions`）的属性，但在 `__all__` 中不存在，因此 `from chloros_sdk import *` 不会引入它们：请按名称导入。`ChlorosError` 家族由 `ChlorosLocal` 及其构建的函数（`process_folder()`、`process_lattice_capture()`）继承； 智能连接表面会派生出 `ChlorosConnectError`。

层次结构：

```

ChlorosError
├── ChlorosBackendError           (backend binary not found / failed to start)
├── ChlorosConnectionError        (HTTP transport failure; no backend running
│                                  with auto_start_backend=False)
├── ChlorosLicenseError           (403 PLAN_UPGRADE_REQUIRED: plan below Copper)
├── ChlorosAuthenticationError    (401 AUTH_REQUIRED: not logged in)
├── ChlorosConfigurationError     (bad configure() inputs)
└── ChlorosProcessingError        (pipeline failed; create/import refused)
    └── ChlorosProjectFilesMissingError  (open_project(): none of the
                                  project's source files found, even after
                                  the relink; .missing_roots, .audit)

ChlorosConnectError                (the smart-connect surface: connect_*, attach_array,
                                    list_*, discover_*, every CameraSession /
                                    ArraySession / DAQSensorSession method,
                                    RecorderHandle, ContinuousHandle,
                                    disconnect_all(), and analyze_array_network()
                                    when no backend answers; 401 and 403 included.
                                    Derives from plain Exception, NOT from
                                    ChlorosError, so `except ChlorosError` will
                                    not catch it. .warning: a warning the
                                    backend sent beside its error (also at the
                                    end of the message); .recording_closed: the
                                    recording a failed DAQ-E restart closed;
                                    both None otherwise)

FrameFetchError (RuntimeError)     (grab() / frame_stream() refusals; .status,
                                    .body, .errors)
ValueError                         (capture() arguments: PNG/JPEG with a level they
                                    cannot hold; a bad packed=)
FileNotFoundError / ImportError    (open_project(): no such folder / no project
                                    loaders)
RuntimeError                       (CameraHandle / ArrayHandle: a backend refusal)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## 端到端示例

### 1. 使用自定义进度条处理文件夹

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. 实时 LATTICE 阵列 → 反射率 + DAQ 参考

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. 项目驱动的捕获活动

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. 多摄像头帧流 → NumPy 处理管道

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. 无头直接硬件（无后端）捕获脚本

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. 连接 4 摄像头阵列前的功能检测

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["width"], height=r["height"],     # before binning
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. 捕获配方等效实现（纯Python）

`run_project_recipe()` 会原样运行一个 CLI 配方（参见 [项目文件夹与配方](#project-folder--recipes)）。若改用 Python 编写，步骤如下：

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply: connect_all() applied each camera's saved settings. CameraHandle
    # has no settings method; a CameraSession on the same pooled camera does
    # (it is borrowed, so it never releases the camera)
    chloros_sdk.connect_camera(cam.serial).set_settings(
        exposure_time=5000, gain=6.0)

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## MCP 服务器（AI 助手）

`chloros-mcp` 允许支持 [模型上下文协议](https://modelcontextprotocol.io)（MCP）的 AI 助手（例如 Claude Desktop、Claude Code 或 Cursor）驱动 Chloros：处理图像（创建项目、导入捕获文件夹、 选择设置、进行处理、报告运行结果）并控制 LATTICE 相机、阵列和 DAQ 光传感器（连接、预览、设置曝光和色彩、捕获、记录、更新固件）。其工具涵盖了此 SDK 的功能。您只需用自然语言提出请求，助手便会调用服务器的工具：

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

The assistant calls:
  create_project          {"project_name": "Flight_2026-10-17"}
  import_images           {"folder_path": "D:\\Flights\\2026-10-17"}
  configure_processing    {"indices": ["NDVI"]}
  start_processing        {}
  get_processing_status   {}      (every 30-60 s until its state is "finished")

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.

The assistant calls:
  list_devices            {}
  get_camera_preview      {"array_id": "array-1779862544497"}
  start_capture_run       {"array_id": "array-1779862544497", "mode": "interval",
                           "interval_s": 10, "duration_s": 300,
                           "processing": "reflectance"}
  get_job_status          {"job_id": "..."}   (until its state is "finished")
```

服务器基于此SDK构建：处理工具调用[`ChlorosLocal`](#image-processing--chloroslocal)，而硬件工具则由 [smart-connect 调用](#smart-connect-for-lattice-cameras)、[DAQ会话](#daq-sensor-sessions)、`ChlorosProject`以及[相机实用程序](#camera-utilities)，所有这些都与Chloros应用所使用的同一后端进行通信。其请求携带`X-Chloros-Client: sdk`， 因此后端会应用SDK的登录和套餐验证机制，并随之执行SDK的相关规则（例如，LATTICE拍摄的画面不支持“纹理感知”功能）。

### 安装

服务器包含在 `chloros-sdk` 包中；无需安装其他内容。SDK 会安装一个 `chloros-mcp` 命令，其用法与 `python -m chloros_sdk.mcp_server` 相同：

```bash
pip install chloros-sdk        # not needed where the Chloros installer set up the SDK (Step 2)
chloros-mcp --help
```

- **Windows：** Chloros 安装程序会通过 pip 将 SDK 安装到您的 Python 环境中，并将 `chloros-mcp.exe` 放置在该 Python 的 `Scripts` 文件夹中。
- **Linux（.deb）：** 该软件包会将SDK安装到系统级的`python3`目录中，并将`chloros-mcp`命令放置在`/usr/bin`中，因此该命令会出现在每个用户的`PATH`中。在虚拟环境中，`pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`会将`chloros-mcp`添加到虚拟环境的`bin`中。

服务器仅使用 SDK（Python 3.7+）之上的标准库。MCP 客户端会启动它，并通过标准输入/标准输出与其通信；除使用 `--help` 外，您无需自行运行它。

### 客户端设置

**Claude Desktop：**将服务器添加到 `claude_desktop_config.json` 中（在 Windows 上为 `%APPDATA%\Claude\claude_desktop_config.json`；通过**设置 > 开发者 > 编辑配置** 可打开该窗口），然后重启 Claude Desktop：

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

**Claude 代码：**

```bash
claude mcp add chloros -- chloros-mcp
```

**光标：**与 Claude Desktop 相同的 `mcpServers` 条目，位于 `~/.cursor/mcp.json`（所有项目；Windows 系统中为 `%USERPROFILE%\.cursor\mcp.json`）或 `.cursor/mcp.json`中。

选项应放入`args`中，或置于Claude Code命令（`claude mcp add chloros -- chloros-mcp --no-auto-start`）之后。 若要使用特定的Python SDK 代替 `chloros-mcp` 中的 `PATH`， 请使用该Python运行模块（若该Python并非`PATH`上的首个Python，则将其完整路径作为`command`）：

```json
{
  "mcpServers": {
    "chloros": {"command": "python3", "args": ["-m", "chloros_sdk.mcp_server"]}
  }
}
```

客户端会在不使用您的 shell 的情况下启动服务器，因此如果客户端报告服务器启动失败，则 `chloros-mcp` 可能不在客户端所见的 `PATH` 上。 请改用完整路径（Windows 系统下为 `where chloros-mcp`，Linux 系统下为 `which chloros-mcp`，其中 .deb 包位于 `/usr/bin/chloros-mcp`），或者使用包含 SDK 的 Python 环境运行该模块。

### 选项

| 选项 | 默认值 | 功能说明 |
| --- | --- | --- |
| `--backend-url URL` | `http://127.0.0.1:5000` | 要使用的 Chloros 后端。 |
| `--no-auto-start` | off | 绝不启动后端：使用已运行的后端，例如 Chloros 应用程序的后端、Linux 后端服务，或通过隧道连接到另一台机器的 （参见 [远程后端模式](#remote-backend-mode-pip-only-host-via-tunnel)；此时两台机器上的文件夹路径必须指向同一文件夹）。 |
| `--backend-exe PATH` | Chloros 安装的位置 | 当没有后端运行时需启动的后端，适用于 Chloros 安装在 SDK 无法查找的位置的情况。 |
| `--log-level LEVEL` | `WARNING` | `DEBUG`、`INFO`、 `WARNING` 或 `ERROR`。日志输出至 stderr，大多数客户端会在其 MCP 日志中显示该内容；stdout 仅传输协议消息。 |

### 工具

每个工具的参数均与其封装的 SDK 调用相对应；客户端会列出每个参数及其完整的模式。路径均为绝对路径。

**处理**

| 工具 | SDK 调用 | 功能说明 |
| --- | --- | --- |
| `create_project` | `ChlorosLocal.create_project` | 创建一个项目并将其设为当前打开的项目，处理工具将基于该项目进行操作。若仅指定名称，则在默认项目文件夹中创建该文件夹；若指定绝对路径，则直接在该路径下创建。 |
| `open_project` | `ChlorosLocal.open_project` | 打开已保存的项目并将其设为当前打开的项目。 已移动项目的路径将重新指向其当前所在的文件夹；`relink_folder`（绝对路径）指定了其他缺失文件当前所在的文件夹。 返回 `source_files` 和 `image_count`。未找到：列出其位置时发生错误，并命名为 `relink_folder`；未打开任何文件。 |
| `import_images` | `import_images` | 导入Survey3的`.RAW`+`.JPG`数据对，LATTICE `.tif` 捕获数据以及 DAQ `.daq` 记录；返回 `count`（图像）和 `scan_count`（记录）。 |
| `configure_processing` | `configure` | 仅修改传入的设置，不修改其他设置（取值：[支持的值](#supported-values)）；`custom_settings` 发送完整的配置。 |
| `get_project_settings` | `get_config` | 读取已打开项目的保存的处理、导出和索引设置。 |
| `start_processing` | `process(wait=False)` | 开始处理已打开的项目并立即返回。 |
| `get_processing_status` | `get_status` | 报告 `state`（`idle`、`starting`、`processing` 或 `finished`）及进度； 完成后，若导出数据无法验证，则生成`summary`和`export_check_failure`；若图像丢失，则生成`image_failure`：其余产品均被写入；将该运行报告为失败. |
| `process_folder` | `process_folder` / `process_lattice_capture` | 为一个文件夹创建项目，将其导入，并通过一次调用完成配置并启动运行 （针对 LATTICE 捕获的 `lattice: true`）。当运行正在进行时，系统会在创建任何内容之前拒绝该请求，此时后端也会拒绝来自任何工具的项目更改。 |
| `export_light_sensor` | `export_light_sensor` | 为该项目的光传感器记录生成已校准的 `.daq` 和 `.csv` 文件 记录生成已校准的 `.daq` 和 `.csv` 文件，且不处理图像；若未缓存，则从 MAPIR Cloud 获取工厂校准数据（参见 [光传感器记录](#light-sensor-recordings--calibrated-daq--csv)）。 |
| `read_image_audit_tags` | `read_image_audit_tags` | 读取已处理图像的 Chloros 审核元数据（来源、校准、已应用的处理阶段）。 |

**相机和阵列**

| 工具 | SDK调用 | 功能说明 |
| --- | --- | --- |
| `list_devices` | `list_cameras`, `list_arrays`、`list_daq_sensors` | 当前在任何客户端中已连接的设备（每个客户端均通过 `connected_here` 连接）；以及服务器上正在运行的任务。 |
| `discover_devices` | `discover_lattice_cameras`, `discover_daq_sensors` | 可连接的摄像头和光传感器。 |
| `get_device_status` | `CameraSession.status`、`ArraySession.status` / `capability`, `DAQSensorSession.status` | 单个摄像头、摄像头阵列（含其功能和运行状态）或光传感器的实时状态。 |
| `analyze_array_network` | `analyze_array_network` | 该主机和网络可为拟建阵列承载的负载。 |
| `connect_camera` / `connect_array` | `connect_camera` / `connect_array` | 连接一台摄像头或一个同步阵列（主设备优先）. |
| `connect_project_devices` | `open_project(...).connect_all` | 连接已保存项目中记录的所有设备及其设置。 |
| `disconnect_device` | `disconnect` / `force_disconnect` | 释放摄像头、阵列或光传感器（参见“行为”）。在阵列上使用 `keep_settings: true` 会将每台摄像头的设置保存到其自身的内存中，`clear_settings: true` 则会重置该内存 （其结果包含 `camera_memory`）。 |
| `disconnect_project_devices`、`align_project_arrays`、`capture_project_devices` | `disconnect_all`, `align_arrays`, `capture_all` | 该项目的所有设备，汇总。 |
| `read_camera_settings` / `set_camera_settings` | `CameraSession.read_nodes` / `set_settings` | 通过友好名称或 GenIC节点名称读取或写入相机设置。 |
| `set_array_frame_rate` | `ArraySession.set_target_fps` | 固定数组的帧率，或将其恢复为自动帧率。 |
| `capture_camera` / `capture_array` | `CameraSession.capture` / `ArraySession.capture` | 单次捕获（`fastest` 和 `wait_for_exposure` 即 `capture_fastest` 和曝光稳定后的拍摄）。若无 `output_dir`，则保存至打开的项目的 `captures` 文件夹中，且文件将加入该项目（暂不处理，直至处理运行结束）；响应中的 `project_add` 会说明这一点。 |
| `start_capture_run` | `ArraySession.continuous`, `capture_repeated` | 作为任务进行的连续或间隔拍摄。 |
| `start_array_recording` | `ArraySession.record` / `burst` | 作为作业处理的组合索引视频或原始连拍数据。 |
| `build_burst_video` | `ArraySession.build_video` | 将连拍数据重新处理为校准视频，作为一项任务。 |
| `get_job_status` / `stop_job` | | 跟踪或停止任务。 |
| `get_camera_preview` | | 每台摄像机的实时预览，以图像形式返回。 |
| `grab_frames` | `ArraySession.grab`, `CameraHandle.grab` | 处理级别下的单帧（或一组同步帧），不保存：包含各平面的统计数据及每帧的PNG图像。 |
| `calibrate_array_alignment`, `get_array_alignment`, `clear_array_alignment`, `nudge_array_alignment`, `export_array_alignment`, `import_array_alignment` | `ArrayHandle.calibrate_alignment`, `alignment_status`, `clear_alignment`, `tweak_alignment`, `export_alignment`, `import_alignment` | 数组的对齐方式。 |

**光传感器**

| 工具 | SDK调用 | 功能说明 |
| --- | --- | --- |
| `connect_light_sensor` | `connect_daq_sensor` | 连接 DAQ-U、DAQ-M 或 DAQ-E（`cap_id` 用于设置其电容校正）。 |
| `get_light_sensor_reading` | `DAQSensorSession.latest` | 最新读数：时间戳、饱和度、光谱范围和峰值，以及按波长区间平均的光谱（`full_spectrum` 用于每个数据点）。 |
| `set_light_sensor_streaming` | `stream_start` / `stream_stop` | 启动或停止数据流。 |
| `start_light_sensor_recording` / `stop_light_sensor_recording` | `record_start` / `record_stop` | 以作业形式记录 `.daq` 文件； `duration_s` 会自动停止该流。 |
| `set_light_sensor_cap` | `DAQSensorSession.set_cap` | 电平校正 （`none` 适用于裸传感器）。 |
| `get_light_sensor_bindings` / `set_light_sensor_bindings` | `daq_bindings` / `set_daq_bindings` | 哪个传感器为各相机和阵列提供反射率数据。 |
| `get_light_sensor_band_presets` / `get_light_sensor_reflectance` | `daq_band_presets` / `light_sensor_reflectance` | 波段集，以及两个流式传感器的环境+物体反射率和折射率。 |
| `get_light_sensor_device_info`, `set_light_sensor_network`, `set_light_sensor_auto_stream`, `enable_light_sensor_ptp`, `reboot_light_sensor`, `update_light_sensor_firmware` | `device_info`, `set_network`、`set_auto_stream`、`enable_ptp`、`reboot`、 `update_firmware` | DAQ-E 自身的设置，保存在设备中。 |

**相机实用工具** ([相机实用工具](#camera-utilities))

| 工具 | SDK调用 | 功能说明 |
| --- | --- | --- |
| `get_camera_color_settings` / `set_camera_color_settings` | `camera_color_profile`、`camera_color_adjust`、`camera_highlights` 及其设置器 | RGB摄像头的显示颜色： 色彩配置文件和色温、饱和度/对比度、高光。 |
| `run_camera_white_balance` / `set_camera_white_balance` | `camera_white_balance_once` / `set_camera_white_balance` | 单次白平衡，或自动 / 关闭 / 固定比例。 |
| `list_camera_presets` / `apply_camera_preset` | `camera_presets` / `apply_camera_preset` | 相机预设。 |
| `list_filter_models` | `camera_filter_models`、`camera_filter_model_info` | 滤镜型号，或按名称逐一选择。 |
| `get_camera_firmware_status` / `update_camera_firmware` | `camera_firmware_status` / `update_camera_firmware` | 固件版本，以及一次闪光（默认情况下为一项任务）。 |
| `get_camera_power_info` / `set_camera_power` | `camera_power_probe`, `sleep_camera`, `wake_camera` | 电源和热状态；睡眠或唤醒。 |
| `calibrate_camera_with_target` | `calibrate_camera_with_target` | 当前视图中校准靶标提供的系数。 |
| `get_network_adapters` / `fix_network_settings` | `network_adapters` / `fix_network_settings` | 相机网络适配器、估计值及修复方案 （仅限管理员）。 |
| `apply_vignette_correction` | `apply_vignette_correction` | 对一整文件夹的图像进行暗角校正。 |
| `read_dls` | `read_dls` | 一次 DLS 读取。 |
| `probe_system` / `get_system_info` | `probe_system` / `system_info` | 主机的硬件和运行时环境。 |

**瞬态相机、对齐配置文件和索引** ([瞬态数组、对齐配置文件和索引](#transient-arrays-alignment-profiles--indices))。此处所有打开相机的工具均使用 `keep_settings` / `clear_settings`（默认值为 false）：参见 [相机内存](#camera-memory--keep_settings--clear_settings)。

| 工具 | SDK调用 | 功能说明 |
| --- | --- | --- |
| `start_camera_continuous` / `start_multi_camera_stream` | `start_camera_continuous` / `start_multi_camera_stream` | 作为一项任务录制一台摄像头，或同步流式传输多台摄像头 （`duration_s` 会自动停止该任务）。 |
| `capture_cameras_transient` | `capture_cameras_transient` | 无需连接数组即可进行一次同步捕获。 |
| `test_array_sync` / `detect_array_master` | `test_array_sync` / `detect_array_master` | 触发同步检查，以及哪些摄像机处于同步线上。 |
| `analyze_array_models` | `analyze_array_models` | 该网络中由这些型号组成的阵列将获得什么。 |
| `calibrate_alignment_profile` | `calibrate_alignment_profile` | 求解并保存对齐配置文件（`options` 包含更精细的求解器选项）。 |
| `capture_aligned_multiband` / `start_aligned_stream` | `capture_aligned_multiband` / `start_aligned_stream` | 一张对齐后的多波段图像，或作为任务的一系列此类图像。 |
| `read_alignment_profile` / `reorder_alignment_profile` | `alignment_profile_info` / `reorder_alignment_profile` | 配置文件的内容；其波段顺序和已启用的相机。 |
| `list_index_presets`, `compute_index`, `capture_live_index` | `index_catalog`, `compute_index`, `capture_live_index` | 索引预设和渐变；多波段TIFF文件的索引；一步完成“捕获到索引”操作。 |

**时间同步、项目文件夹和配方** ([PTP 主时钟](#ptp-grandmaster--time_sync_status), [项目文件夹与配方](#project-folder--recipes))

| 工具 | SDK调用 | 功能说明 |
| --- | --- | --- |
| `get_time_sync_status` | `time_sync_status`、`time_sync_peers`、 `time_sync_cameras` | PTP 主时钟的健康状态、其对等节点以及每台阵列相机的 PTP 状态。 |
| `restart_time_sync` / `set_time_sync_priority` | `restart_time_sync` / `set_time_sync_priority` | 重启它；设置其优先级。 |
| `get_project_folder` / `set_project_folder` / `reset_project_folder` | `project_folder` / `set_project_folder` / `reset_project_folder` | 创建新项目的文件夹。 |
| `run_project_recipe` | `run_project_recipe` | 以任务形式运行已保存项目的配方，并生成日志。 首先进行检查：存在错误的配方将被拒绝且不执行任何操作；对于未连接的数组/传感器会发出警告（不会连接它们：需先执行 `connect_project_devices`）。`dry_run` 返回检查结果且不执行任何操作。 |

没有工具可直接打开已保存的项目进行处理：每个处理任务均以 `create_project`（或 `process_folder`）开始。

### 要求

- **同一台计算机上已安装 Chloros。** 处理工作由 Chloros 后端完成。服务器也可以使用通过隧道连接的后端（`--backend-url` 配合 `--no-auto-start`，参见 [选项](#options)）。
- **需在该计算机上登录**，通过 Chloros 应用或使用 `chloros-cli login` 登录。服务器无法代您登录：若在未登录状态下调用该工具，系统将返回“未登录 Chloros”的提示并附带后端给出的原因，助手会向您说明。
- **任何付费的 Chloros+ 套餐（Copper 或更高）。** 服务器采用 SDK 访问方式，而免费的 Iron 套餐不支持此功能：在免费套餐中调用该工具时，系统会返回“此 Chloros 套餐中不可用”的提示，并附带后端给出的原因。
- 配备 SDK 的 Python 3.7 或更高版本，以及一个可启动本地（stdio）服务器的 MCP 客户端。

### 行为

- **后端。** 启动服务器或列出其工具时不会启动任何进程；首次调用工具时才会建立连接。如果 Chloros 应用（或 Linux 后端服务）正在运行，服务器将使用其后端并保持其运行状态。 如果没有后端正在运行，服务器将启动已安装的后端，且不显示窗口。当服务器退出时（您退出客户端，或客户端重启服务器），它会停止其启动的捕获任务和光传感器记录，并释放已连接的设备；后端继续运行， 因此未完成的处理任务将持续进行直至结束，若 30 分钟内无人使用，它将自动停止。在 `--no-auto-start` 版本中，服务器永远不会启动后端：当调用工具时若无后端运行，系统会提示您打开 Chloros 应用，待您打开后，下一次调用即可建立连接。
- **仅有一个打开的项目。** 后端仅有一个打开的项目，当Chloros应用打开时，该项目与该应用共享，因此请勿在助手运行期间在应用中处理其他项目。
- **仅传递的设置会发生变化。** `configure_processing` 会将未被传入的所有设置保留为已保存的值（SDK 的 `configure()` 在未传入相关参数时，会将去拜耳处理、暗角校正、反射率校准、PPK 和导出格式重置为默认值； 服务器会从项目中填入这些参数）。如果无法读取项目中保存的设置，则不会进行任何更改并会提示该情况，除非这五项参数均已提供。`indices` 会替换项目的索引列表。结果中会列出作为 `settings_sent` 列出所发送的数据，并在 `warnings` 下为相机波段无法生成的索引命名。
- **处理采用“启动后轮询”模式。** `start_processing` 在运行开始后立即返回，随后处理过程将耗时数分钟至数小时， 随后辅助程序会接着调用 `get_processing_status`。`state` 可能为 `idle`（无处理任务）、`starting`（后端尚未报告运行结果，最长可能延迟 20 秒）、`processing` 或 `finished`，其中包含该运行的 [摘要](#post-run-summary--hints)：包含导出的数据，以及当导出数据少于预期时的提示（无图像、未找到校准目标）。生成结果将写入项目文件夹，与应用程序中的操作一致。 在运行进行中，`start_processing` 将被拒绝；后端停止响应时，系统会将其报告为错误，而非已完成的运行。
- **助手所见内容。** SDK 的警告会以 `warnings` 的形式显示在工具结果中，而失败情况（已注销、计划问题、路径错误）则以错误文本形式呈现，助手可读取并将其传递给您。 超过 20 项的列表将截取前 20 项，其完整长度显示为 `<key>_total`。路径必须为绝对路径。
- **设备是共享的。** 摄像头、 阵列和光传感器均位于后端，因此助手可控制已连接至Chloros应用的设备，其状态变化会实时显示在应用中。`disconnect_device`仅发布服务器连接的设备状态；若设备由其他客户端（如应用、`chloros-cli`、 脚本、另一个`chloros-mcp`）所连接的设备，仅在调用`force: true`时才会被释放——该方法即SDK中的`force_disconnect`，且当后端知晓连接者身份时，会报告拒绝名称； 后端完全未持有的设备将被报告为未连接（`released: false`、`connected: false`，无设备可释放）。后端运行的录制或流媒体会通过 `get_job_status` 视图携带 `counts` — `frames_written`、`frames_dropped`、 `frames_failed`、`drop_reason`（多机位流的帧组） ——以及运行期间显示的一行信息 `summary`（“写入 220 帧，丢弃 55 帧，0 次失败：...”），以及运行完成后的最终数据；另一位客户对其的列表 （`other_clients_jobs`）显示的内容相同（`list_cameras()`、`list_arrays()` 和 `list_daq_sensors()` 分别显示各设备的的`connected_by`：`client`、`label`、 `name`、`at`）。 每个 `chloros-mcp` 都会向后端将自身命名为 `chloros-mcp (pid N)`（`chloros_sdk.connect.CLIENT_LABEL`，发送时显示为 `X-Chloros-Client-Label`；任何 SDK 脚本均可设置该值）。停止其他客户端的光线传感器录制同样需要`force`。
- **长时间运行即为任务。** `start_capture_run`， `start_array_recording`、`build_burst_video`、`start_light_sensor_recording`、`start_camera_continuous`、 `start_multi_camera_stream`、`start_aligned_stream`、`run_project_recipe`、 以及任何请求的捕获、求解或索引操作（如 `background`），都会立即返回一个 `job_id`；随后是 `get_job_status`，最后由 `stop_job` 结束 。除非您提供 `max_captures`、`duration_s`（或对于突发任务，提供 `max_frames`）；当驱动器剩余空间不足时，`min_free_gb`将拒绝启动。除非您设置限制，否则不设任何限制。后端运行的录制或流媒体任务（`start_camera_continuous`、 `start_multi_camera_stream`、`start_aligned_stream`）会保留后端的任务 ID，因此另一个 `chloros-mcp`（即稍后启动的任务）可通过该 ID 找到它： `get_job_status` 会跟随它（并列出诸如 `other_clients_jobs`、`GET /api/camera/cli-jobs` 之类的任务，每个任务都包含 `started_by`）， 而 `stop_job` 仅能通过 `force: true` 将其终止。任何其他作业 （捕获运行、数组记录或突发记录、后台捕获）都在启动它的 `chloros-mcp` 中运行，且只有该服务器知道这一点： 当另一个进程查询该任务时，系统会如此反馈。被中止的任务在写出其队列时读取 `stopping`，随后读取包含最终计数的 `finished`， 持续10分钟，并从任何`chloros-mcp`中： 发起该进程的服务器会以同样的方式察觉到其他服务器的强制停止。`get_job_status` 绝不会等待停止过程：无论是自身停止（其自身的 `stop_job`、其他客户端的，或是因任务自行结束而释放摄像头的停止操作），它会立即以 `stopping` 及实时计数进行响应；而处于停止状态的任务则通过 `stop_job` 表明此状态。已结束的任务会在 `stop_reason`中说明结束原因：自身原因（`no_frames`、`disk_full`、`member_missing`、 或对于数组连续运行，则为`count`、`duration`、`array_gone`），而`limit reached`仅在计数显示其`count`（或一个突发序列的`max_frames`、一个记录的的`duration_s`）达到时，`error`、`stopped` （带 `stopped_by`）当客户端停止它时，否则为 `finished`。
- **捕获文件的存储位置。** 保存至当前打开项目的 `captures` 文件夹中（应用程序会将截图保存于此），除非指定了 `output_dir`。
- **图像。** `get_camera_preview` 和 `grab_frames` 除了返回 JSON 数据外，还会返回助手可查看的图像。`grab_frames` 图像在显示时会进行对比度拉伸；其统计数据为真实值。
- **当服务器退出时**，其启动的任务将停止，且其启动的光传感器记录也将关闭。如果服务器启动了后端，它还会在停止后端之前释放已连接的设备；而在应用的后端中，这些设备将保持连接状态。
- **协议。** 仅支持 stdio（无 HTTP 端点），仅限工具（无资源或提示），返回文本和图像结果，支持 MCP 协议版本 2024-11-05 至 2025-11-25。**未包含的内容。** 登录或注销（账户归您所有；服务器无法登录），以及停止服务器未启动的后端。此 SDK 涉及处理和硬件的其他所有功能均由工具实现。（DAQ-E 的配置——其名称、认证令牌、 OTA 密码、签名要求、工厂重置——以及直接 DAQ 服务器均由 `chloros-cli` 专属处理，SDK 中亦包含相关功能：若名称或令牌有误，将导致所有 Chloros 安装无法访问该设备。）

---

## 后端自动启动

智能连接入口点——`connect_camera`、`connect_array`、 `connect_daq_sensor`、`discover_lattice_cameras` 和 `discover_daq_sensors` —— 是一组轻量级 HTTP 客户端，它们假设后端正在监听 `127.0.0.1:5000` （智能连接界面的默认 URL）上监听。当 GUI 或 CLI 已运行时，该后端即处于运行状态。若从纯脚本启动， 可能不存在后端进程——因此这些函数会在首次调用前**自动启动捆绑的后端二进制文件**（无窗口模式，与 `ChlorosLocal` 的方式相同），随后等待最多 60 秒直至其启动完成。 `list_cameras()`、`list_arrays()`、`list_daq_sensors()`、 `attach_array()` 和 `analyze_array_network()` 则永远不会启动后端进程。

规则：

- **仅会生成本地URL。** 指向 `localhost` / `127.0.0.1` / `[::1]` 的 `backend_url` 才符合条件；任何其他主机均被视为他人的机器，绝不会被启动。
- **如果后端在响应前退出，则等待立即终止。** 该调用会立即抛出异常，并返回退出代码、后端执行的命令以及后端输出的最后几行内容，而非等待 60 秒。退出代码 98 属于例外情况： 另一个后端已占用该端口或锁文件（可能是 Chloros 应用程序、后端服务，或是仍在启动中的另一个脚本），因此该调用会继续等待该后端响应。`ChlorosLocal` 的行为与此相同。
- **后端将保持运行状态以便重复使用**（与CLI相同）——脚本退出时不会隐式关闭后端。重新运行脚本将复用该活跃的后端。
- **若需禁用此功能，可在上述任何调用中使用 `auto_start_backend=False`**（例如，当您指向远程后端，或自行管理后端生命周期时）。

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

如果无法找到或启动捆绑的二进制文件，后续的 HTTP 调用会抛出一个可处理的、**平台感知型**的 `ChlorosConnectError` 异常，而不是简单的连接被拒绝跟踪信息 ——在 Windows 上，它会引导您使用桌面应用程序或 `chloros-cli` 命令；在 Linux（无 GUI）上，它会引导您使用 `chloros-cli` 命令或 `.deb`。

---

## 环境与标头

SDK 会为每个后端 HTTP 调用添加 `X-Chloros-Client: sdk` 标记。后端将应用 SDK/CLI 授权规则（需登录 **且** 订阅付费版 Chloros+ 套餐），而非 GUI 免费层级路径。 此设置会在导入时自动完成——您无需进行任何操作。

该标头将添加到每个 `requests` / `urllib` 请求中添加该标头，无论其 URL 包含 `localhost` 还是 `127.0.0.1`，且不限端口——包括您自己的本地服务 ——并会替换您自行设置的值。对其他主机的调用则保持不变。

通过传递 `backend_url=` 来覆盖后端 URL（`ChlorosLocal` 接受 `backend_url=` 或 `api_url=`；`process_folder()` 和 `process_lattice_capture()` 仅接受 `api_url=`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

（已安装的后端仅监听回环接口，因此局域网中的 `backend_url` 无法与其建立连接；有关隧道模式的详细信息，请参阅 [远程后端模式](#remote-backend-mode-pip-only-host-via-tunnel) 了解隧道模式。）

---

## 版本控制与兼容性

- SDK版本通过 `chloros_sdk.__version__` 对外暴露。
- SDK的行为与捆绑的后端版本绑定。将旧版SDK与新版后端混合使用通常可行（向前兼容的端点），但将新版SDK与旧版后端混合使用可能会在新端点上引发`404`错误——请将桌面应用升级至与之匹配的版本。
- 智能连接接口（`connect_camera` / `connect_array` / `connect_daq_sensor`）和网络分析端点返回稳定的JSON模式；新字段均为增量添加。

---

## 故障排除提示

- **`ChlorosAuthenticationError: Login required`**（源自 `ChlorosLocal`；智能连接调用因相同原因会触发 `ChlorosConnectError: Login required. …`） → 在该机器上运行一次 `chloros-cli login EMAIL PASSWORD`，或通过 Chloros 桌面应用登录。
- **`ChlorosProcessingError: Failed to start processing: … The project folder is too deep for this computer …`**（或由 `create_project()` 引发的 `Failed to create project: …`）→ 该计算机上已禁用 Windows 长路径（这是 Windows 的默认设置）， 因此路径长度不得超过 260 个字符，且产品路径位于项目文件夹下方约 130 个字符处。未进行任何处理。该消息给出了符合要求的最长项目文件夹：请使用更短的文件夹，或启用 Windows 长路径并重启 Chloros。 Windows安装程序中提供“启用Windows长路径”选项，默认已勾选；否则请使用组策略：*计算机配置 > 管理模板 > 系统 > 文件系统 > 启用 Win32 长路径*。
- **`ChlorosConnectError: No Chloros backend is running …`** → 智能连接会调用本地后端的自动启动功能，因此该错误仅在无法找到或启动捆绑的二进制文件时出现（例如，仅运行 pip专用主机未安装桌面软件包）。该提示信息具有平台适配性：在Windows上打开桌面应用或运行任意`chloros-cli`命令； 在 Linux 上运行 `chloros-cli` 命令（无 GUI 界面）或安装 `.deb`。对于远程后端，请传递 `backend_url=` （以及 `auto_start_backend=False`）。
- **`CAMERA_AVAILABLE == False`** 在导入时 → `lattice_sdk` 加载失败：Arena SDK 运行时，或 OpenCV (`cv2`) 无法在此Python环境中导入。后端驱动的表面（`connect_*`，通过后端进行渲染，`ChlorosLocal`）仍可正常工作。
- **数组在全传感器模式下连接，但帧率较低**→ 设计使然：自动模式会保持全传感器模式，并让帧率降至线路所能承载的水平。传递 `binning=2` / `4` 或更小的 `width` / `height` 以获得更高的帧率，或者启用巨型帧 / 升级链路。 像素合并和 ROI**无法**解决聚合超订阅问题（`oversubscribed: true`，fps 字段为 0）： 网络带宽无法承载过多摄像头的问题无法通过像素合并/ROI 解决——请减少摄像头数量、启用巨型帧，或更换更快的网卡（参见 [超订阅](#over-subscription-the-per-cam-floor)）。
- **`analyze_array_network()` 报告网卡接收环过小（~0.26 MB）/ 连接门上显示“FRAMES WILL DROP”** → 主机网卡的接收环处于默认大小（网卡驱动更新后通常会重置为 32）。在 Realtek USB 10GbE 适配器上，请设置 `ReceiveBufferLen=256` 和 `PendingReceives=64`（需提升权限），然后重启后端使其重新读取环。完整操作步骤： [CLI参考 → 主机网卡设置与调优](cli-reference.md#host-nic-setup--tuning-lattice-arrays)。
- **主机在重启/关机时卡死，随后出现WMI `Invalid class`错误 / 网卡无法启用** → 过时的 USB 10GbE 驱动程序导致 `DRIVER_POWER_STATE_FAILURE`（蓝屏 `0x9F`）。将适配器驱动程序更新至最新版本（≥ 2026），并重新应用接收环设置。 请参阅 [CLI 参考 → 主机网卡设置与调优](cli-reference.md#host-nic-setup--tuning-lattice-arrays)。
- **数组捕获失败，HTTP状态码503“无新同步帧组”** → 数组在等待期间未发布新的帧组。请检查`arr.status()`：`stalled: True` （或 `streaming: False`）与 `actual_fps` 相关，其中 0 表示抓取循环已停止发布 ——请断开并重新连接数组。如果 `stalled` 为 False 且 `actual_fps` 大于 0，请重试。 紧随 `connect_array()` 之后，捕获操作已为第一组数据等待长达 20 秒。
- **`status()` 显示 `link_speed_mbps: 4294`** → Windows 无法读取适配器的速率；这是 USB 以太网适配器的典型特征，并非故障。请参阅 [预期帧率](#expected-frame-rates)。
- **`update_firmware()` 因 `device never connected back on TCP <port>` 而失败** → 防火墙阻断了 DAQ-E 与该计算机的回连。Windows 安装程序会添加相应规则（参见 [DAQ-E 设备设置](#daq-e-device-settings)）；若使用其他防火墙，请允许来自传感器子网的入站 TCP 流量连接至后端。
- **反射率被拒绝** → 若要获取绝对量程的反射率，必须将实时 DAQ 绑定到摄像头（或阵列）。可通过图形界面进行绑定，或使用 `processing="radiance"`（W/m²/sr/nm），该模式无需配对传感器。
- **`smart=True` 采集时间长于预期** → AE 收敛取决于场景动态。使用 `ArrayHandle.capture_smart()` 时，请扩大 `exposure_tolerance_pct`， 缩短 `stability_window_s` 或降低 `settle_timeout_s` 值，以获得更快、更不稳定的触发。

---

## 另请参阅

- [CLI 参考](cli-reference.md) — 每个 CLI 子命令都对应一个 SDK 调用。
- [DAQ 光传感器](https://mapir.gitbook.io/chloros/daq-light-sensors)（见Chloros手册）——传感器、其规格与校准范围、数据记录、DAQ-E 网络连接。
- [LATTICE 相机](https://mapir.gitbook.io/chloros/lattice-cameras/lattice)（见 Chloros 手册）—— 连接、相机设置、捕获模式、阵列。
- 在线文档：`https://mapir.gitbook.io/chloros/api-python-sdk`
