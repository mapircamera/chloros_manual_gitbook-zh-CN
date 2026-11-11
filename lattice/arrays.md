# 多摄像头阵列

LATTICE **阵列**是指将两台或更多台 LATTICE 摄像头连接成一个同步单元。其中一台摄像头作为**主摄像头**：它会在共享的同步线上（默认**Line2**）发送硬件 GPIO 触发脉冲， 从而确保每个成员在同一瞬间曝光。Chloros

增加了 PTP 时间同步、实时预览（按相机划分的图块或单个对齐的多波段合成图像）以及同步捕获功能——每次捕获过程都会生成一个 **帧组**，其中所有相机共享相同的时间戳和帧 ID （在采集输出中显示为 `fid:N`）。

数组是单色（M3M）相机生成植被指数的方式——每台相机贡献一个波段，数组将它们对齐成多波段堆栈。 参见 [单色相机与植被指数](mono-indices.md)。

连接阵列有三种等效方法，它们均运行相同的“smart-prep”流程：

| 界面 | 入口点 |
| --- | --- |
| GUI | “相机”选项卡 → **连接阵列**（蓝色按钮） |
| 命令行界面（CLI

） | `chloros-cli lattice array-connect --serials SN1,SN2,…`（第一个序列号 = 主节点） |
| Python

SDK

| `connect_array(serials=[…])` → `ArraySession` (第一个序列号 = 主节点) |

Smart-prep 执行以下操作：网络能力探测（ICMP DF ping + GVSP 探测）、同步层选择、按请求帧大小进行帧率预测、打包 12 位传输决策 （参见 [12 位压缩传输](#packed-12-bit-transport)）、PTP 启用、按摄像头自动选择像素格式、根据各摄像头保存的状态进行自动曝光初始化，以及在 Line2 上配置 GPIO 触发。

**默认使用完整传感器 （1.3.0 版新增功能）。** Connect 会保留您要求的帧尺寸——即完整的 2048×1536 传感器，不进行像素合并也不进行裁剪，除非您另行选择 ——并允许帧率降至主机和网络所能提供的水平。它绝不会自行进行像素合并或裁剪（1.2.0 版本曾在帧率低于 1.5 fps 时自动启用像素合并）。 是否进行像素合并或设置较小的感兴趣区域（ROI）由您决定：在“连接”对话框中选择一项，&quot;预计结果&quot;将显示相应的帧率；`array-connect`会打印出帧率更高的选项及其预计帧率。

{% hint style="info" %}
在执行上述任何操作之前，必须确保通过链路能够访问摄像机——请参阅 [连接摄像机](connecting.md) 了解设备发现、地址分配以及首次连接校准下载的相关内容。对于多摄像头系统，主机网卡的接收环设置与链路速度同样重要；完整的“症状→解决方案”对照表详见 [CLI

参考 § 主机网卡设置与调优](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays) 中。
{% endhint %}

## “阵列连接”对话框

在“摄像头”选项卡中 → 点击 **连接阵列**将打开一个三步向导：**选择 → 显示模式 → 设置**。

### 步骤 1 — 选择主摄像机和从属摄像机

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered (an FRGB among them). Table showing Camera / Serial / IP / Master radio / Slave checkbox columns in camera order (M3C first, FRGB at the top), with the FRGB pre-selected as master and the green "Sync cable found on N cameras. Any of them can be master; the RGB camera (else the first) is pre-selected." probe banner visible above the table. -->



对话框一打开就会立即扫描网络（“正在扫描网络...”），然后探测同步线缆（“正在探测同步线缆...”），以查找哪些摄像机连接在 M8 同步线上。您需要至少 **2 台摄像机** 才能构建一个阵列。

由于 M8 同步线缆汇聚于一个接头处，因此 **任何已连接线缆的相机均可作为主相机**。探测程序会预先分配角色——当RGB

（`-FRGB`）相机已连接时将其设为主相机，否则设为第一台已连接的相机，其余已连接的相机则设为从相机——并显示以下三种提示信息之一：

| 提示 | 含义 |
| --- | --- |
| “在 {n} 台相机上检测到同步线缆。其中任意一台均可作为主机；RGB

相机（否则为列表中的第一台）已被预选。”（绿色） | 探测器已在同步线上检测到相机；“主机”和“从机”复选框已自动勾选。 |
| “未检测到主相机——请检查 GPIO 线缆” (橙色) | 没有相机检测到触发脉冲；请检查同步线缆连接。角色仍为预设（RGB

相机，否则为列表中的第一台相机，作为主相机），您可以进行更改。 |
| “无同步线缆：{serials}”（橙色） | 列表中的相机未连接同步线缆。 |

相机表格包含以下列：**相机 / 串口号 / IP 地址 / 主相机（无线电） / 从相机（复选框）**。行按**相机顺序**排列——M3C（拜耳）相机排在 M3M（单色）之前；在 M3C 之后依次为 FRGB、FRGN、FOCN，然后是 FNGB； M3M 相机按滤光片波长排序，波长最短者优先（F450 排在 F808 之前）。侧边栏的相机列表和网格的默认布局均采用相同的排序方式。

* 请精确选择 **一个主相机**和**一个或多个从相机**。再次点击当前主相机的单选按钮可取消选中。选择另一个主相机只需单击一次：新主相机将带走所有从相机，而原主相机则成为从相机（除非其未连接同步线）。
* 标记为 **“无同步线”** 的相机绝不能被选为从机——没有触发线连接的从机将永远等待同步信号，从而输出无效画面。请将该相机作为独立相机连接。
* 已作为独立摄像机连接的摄像机*不会*被禁用：阵列连接会释放独立会话，并在阵列内重新启用该摄像机。

**下一步：显示模式 →**在选择了一个主摄像机和至少一台从属摄像机后生效。**重新扫描**将重新运行设备发现和布线检测。

{% hint style="warning" %}
在扫描或探测进行期间，**取消**按钮将不可用——在探测过程中取消操作可能会导致 LATTICE 相机固件上的相机 SDK

崩溃。请等待加载图标停止转动。
{% endhint %}

### 步骤 2 — 显示模式

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->



| 模式 | 显示效果 |
| --- | --- |
| **独立相机** | 每台相机一个实时磁贴，所有相机同时触发以确保帧同步。每台相机保持各自的颜色和设置。 |
| **合并摄像头** *(默认)* | 单个动态磁贴渲染对齐后的多波段NDVI

/指数合成图像。各摄像头共享数组颜色。 |

显示模式仅改变实时预览的呈现方式——两种模式下的采集行为完全相同。

### 步骤 3 — 数组设置与预期结果

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Trigger Rate / Wire Budget controls (1.3.0 has no "Pin resolution" checkbox), right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->



进入此场景时，Chloros

会向后端请求**建议**并加以应用：**全传感器模式——不进行像素合并，不进行ROI裁剪——只要帧率能够支持**，无论主机和传输线路提供何种速率。只有当任何速率都无法支持全传感器模式时，建议才会缩小帧尺寸， 此时系统会优先选择像素合并而非 ROI 裁剪，因为像素合并能保留完整的视野。从这一刻起，您需要在分辨率和帧率之间进行权衡。每次更改都会实时重新运行分析，并更新右侧的**预期结果** 面板。

左侧栏 — 设置：

| 控件 | 选项 | 默认值 | 备注 |
| --- | --- | --- | --- |
| **ROI（视场）** | 全景 (2048×1536) / 半景 (1024×768) / 四分之一 (512×384) | 全景 | 传感器裁剪：以原生像素间距将图像裁剪为半景或四分之一区域。 |
| **像素合并** | 1× / 2×（2×2 合并） / 4×（4×4 合并） | 1× | 硬件像素合并：2×2 = 以四分之一的总线成本获得全视场；4×4 = 以 1/16 的总线成本获得全视场。 若相机不支持像素合并，则隐藏。 |
| **线侧图像**（读出） | — | — | 实际通过线路传输的像素合并后宽度×高度，四舍五入为16的倍数（最小64）。 |
| **触发率** | 0.5–60 fps，步长 0.1 | 留空 = 自动 | 主控器的触发频率。 留空则由Chloros

自动推算。 |
| **总线带宽**| 20–2000 MB/s，步长 10 | 留空 = 自动 | 主机实际可处理的带宽，单位为 MB/s —**这是整个数组分配所依赖的关键数值。** 由网络适配器自动检测。如果数组报告帧损坏，请降低该值：检测到的数值往往高估了 USB 适配器和共享交换机的实际能力。更改该值将实时重新运行预测。 该字段初始显示的是上次连接数组时的带宽预算（随项目保存），因此您曾修正过的带宽值会被保留。 |

1.2.0 版本中的 **引脚分辨率** 复选框已移除：连接时系统将始终保留您选择的帧大小，并通过降低帧率来调整。

右侧栏 — **预测结果**：

* **同步层级** — “同时捕获”（绿色）、“同时捕获（FTD交错发射）”（绿色）、“交错捕获（100 毫秒漂移）”（琥珀色）或“配置过大” （红色）。Chloros

会自动选择同步层：只要网络带宽能够以相机可适应的速率共享（随着每台相机所占带宽的减少，帧率会相应降低），则采用“同步采集”；否则仅采用“交错采集”——因此，相机阵列可在任何网络上连接，并以该网络允许的帧率运行。
* **帧率预测** — 阵列预期运行的帧率。 在同时采集模式下，这是一个单一数值：帧率优先，自动曝光会填满其指定的时间段，随后提高增益，因此光线较暗的场景不会降低帧率（参见 [自动曝光如何分配曝光时间和增益](camera-settings.md)）。 在交错采集模式下，该参数以范围形式显示（“暗 → 亮”），因为在光线较暗的场景中，每台摄像机的曝光时间会延长交错间隔。
* **网卡线路** —— 链路速度和持续吞吐量（“网卡 {mbps} Mbps · 持续 {N} MB/s”）。
* **模拟发射突发检查** — 主机的网卡环路能否同时接收来自所有摄像头的突发数据（“模拟发射突发：X MB · 网卡环路可用带宽：Y MB ✓/✗”）。
* **带宽预算检查** —— 稳态总需求与防冲突带宽上限的对比（“带宽预算：{n} 台摄像机需求的 {demand} MB/s · 上限 {ceiling} MB/s ✓/✗ 超额订阅”）。
* **“该链路最大摄像头数量：{n} — 由每台摄像头的带宽下限决定，因此分组不会提高该上限。”** — 当您接近（或超过）摄像头数量上限时显示。
* **“在此设置下将丢帧。&quot;**— 红色警告，附带后端原因说明，以及阻塞因素列表和蓝色**解决方案建议**（“让该阵列适应网络” / “解锁同时捕获”）。**“应用并连接”** 功能在存在预测结果前处于禁用状态，其标签会说明拒绝的原因：

| 按钮标签 | 含义 | 实际解决方法 |
| --- | --- | --- |
| “正在分析...” | 分析仍在进行中。 | 请等待。 |
| **““此网络的摄像头过多”**| 该网络无法承载该阵列，即使每次仅传输一台摄像头（聚合检查失败）：摄像头数量远超其处理能力，或该网络上已有其他摄像头正在传输。 | 减少摄像头数量、端到端使用巨型帧，或使用更快的网卡。**缩小ROI无济于事** — 参见下文。 |
| **“缩小ROI以启用”** | 当前设置下帧会丢失（突发/环检查失败）。 | 缩小ROI、提高像素合并度，或修复网卡接收环。 |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->



连接过程中，可能会出现一个带绿色**校准下载面板**，其中显示按串口划分的进度条：当摄像头首次连接到机器时，Chloros

会通过GigE从摄像头下载约3.8 MB的出厂校准包（每台摄像头约需70秒）。 已缓存的相机不会显示此面板；当相机数量超过六台时，面板行会滚动显示。请参阅 [连接相机](connecting.md)。

### 打包的 12 位传输

12 位格式（`BayerRG12`、`Mono12`）通常以 16 位字的形式传输。**压缩**传输方式将相同的 12 位数据以每像素 1.5 字节的形式发送，并在主机端进行解包，因此每帧的传输时间减少 25% ——在传感器满载时，每秒帧数大约增加三分之一。**节省的数据内容完全相同。** 数组连接会自动决定（`auto`）：仅在压缩有益且主机有足够CPU资源进行解包时才进行压缩。图形界面中没有相关开关；在CLI

中请使用`array-connect --packed {auto,on,off}` （默认值为 `auto`）。8 位和 16 位格式绝不会被打包。

## 带宽：可支持多少台摄像头

阵列能承载多少台摄像头取决于网络链路，而非Chloros

，因此规划数据请参阅硬件手册：**[阵列带宽规划](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**。

Chloros

对此的处理方式：连接对话框会运行网络探测，预测可实现的帧率，并选择合适的层级。数组可在任何网络上连接：在低速链路上，每台摄像机的传输速率会根据其占用的带宽份额进行调节，帧率随之降低，而非直接拒绝数组连接或缩减其帧大小。 只有当网络连单台摄像头的带宽都无法承载时，该阵列才会被拒绝（**此网络摄像机数量过多**），而非默默丢弃数据包——详见上文所述的“预测结果”面板。

## 当帧丢失时

摄像机可能因两种截然不同的原因缺失于已发布的组中，
且它们需要截然相反的解决方法。Chloros

会分别统计这两种情况，而非报告一个
既未指明原因又未说明具体情况的“不完整”数值：

| 发生情况 | 含义 | 排查位置 |
| --- | --- | --- |
| **数据帧损坏**—— 帧已到达但结构损坏 | 网络路径上的 GVSP 数据包丢失 |**带宽预算**、网卡接收环、巨型帧、交换机 |
| **未到达**— 完全没有帧到达 | 摄像机未触发，或未发送任何数据 |**M8同步线**、同步线路、所有成员是否处于就绪状态 |

在阵列传输期间，系统每10秒重新评估分流情况。若超过5%，则
会记录具体数值，并针对每台摄像机，在首次发生时报告每个损坏的缓冲区，
随后每分钟汇总一次，以确保长时间会话的可读性。

**若损坏帧数为零且“未到达”数为零，则表明触发和线缆同步均正常**，所有丢失的帧均源于网络路径。解决方法是降低**线缆预算**并
重新连接。

{% hint style="warning" %}
**降低触发率对解决帧损坏问题无济于事。** 摄像机的数据包
分发节奏在连接时仅设置一次。降低触发率只会改变突发传输的
频率，而不会影响突发数据本身传输到网络上的速度。 在经过测量的 4 台摄像机系统中，
将触发率降低 5 倍没有任何效果，而将线速预算从 240 降至
200 MB/s，则使该系统的数据包损坏率从10.4%降至零。
{% endhint %}

正在运行的阵列无法自行重新规划——请断开并重新连接，以便连接时
的调度器能根据新的带宽预算进行工作。

### USB 网络适配器存在带宽上限

USB 以太网适配器虽标称其 *以太网* 链路速率，但其实际
可持续传输速率受限于 USB 总线及其驱动程序。一款 USB 10GbE 适配器曾被认为
具有约 1000 MB/s 的吞吐量 ——这一数值从未被实际测得——而将
四台摄像机按该虚幻的冗余带宽进行调速时，导致 6–18% 的帧出现损坏，而阵列
仍报告目标帧率正常。目前，USB 连接的适配器带宽上限为
Windows

和NVIDIA Jetson系统上为**200 MB/s**，在Linux

x86_64系统上为**400 MB/s**。该上限是绝对值而非百分比，因为限制因素在于
总线：一款USB 1 GbE适配器可获得约80 MB/s的速率，且不受此限制影响。

如果您的主机速度确实超过了该上限，请提高 **Wire Budget** 值以反映实际情况。

## PTP 时间同步

帧的 *同步* 来自硬件触发信号；**PTP**（IEEE 1588 PTPv2）为所有设备提供一致的 *时间戳*。 该功能在阵列连接时默认启用：

* **Chloros

主机后端运行PTP主时钟**。LATTICE相机和DAQ-E光传感器在域0中对其进行从属同步，因此图像时间戳和DAQ光谱数据基于同一时钟（约1毫秒）。
* `--no-ptp`（CLI

）可将其禁用以进行台式实验——此时不同相机之间的时间戳将**无法**相互比较。
* 通过 CLI

检查同步状态：

```bash
chloros-cli time-sync status     # grandmaster state, clock identity, health (OK, or why its packets are not reaching the network)
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

“相机”选项卡本身没有 PTP 指示器；该选项卡中显示的各相机同步信息包括只读的 **角色**（主/从）、**同步线** 以及阵列的“功能”层级。DAQ-E 的 PTP 状态显示在“光传感器”选项卡的传感器详细信息中。

## 实时阵列视图

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows in camera order (M3C before M3M, monos by wavelength), and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

主数据流区域提供两种布局（可在顶部栏中切换）：**网格视图**（每个图块是一个单元格； 当网格挂锁解锁时可通过拖拽重新排序）和**列表视图**（全宽行：**组合**阵列为一行——即其复合图块——而**独立**阵列则为**每个成员单独一行**，并带有各自的实时视频流和校准目标表； 数组下方，有一台独立相机——即当前活动相机）。**画面缩放**滑块用于调整图块大小；当单元格宽度小于 200 像素时，名称/帧率叠加层会自动隐藏。成员相机按相机顺序显示在侧边栏和网格的默认布局中 （M3C 优先于 M3M，FRGB 优先，单色相机按波长排序）；解锁网格并拖动仍可覆盖网格排序。

<!-- SCREENSHOT-NEEDED: list view of a connected SEPARATE array (1.3.0): one full-width row per member camera, each with its live feed and its own Calibration Target table below it. -->



**独立模式**下，每台相机显示一个图块。每个图块叠加显示：

* 相机名称（左上角），
* **帧率读数**（右上角）——这是后端报告的相机*实际抓取率*，而非预览轮询率（实时预览无论抓取率如何均限制在 30 fps），
* 状态点 — 绿色（流式传输）/ 琥珀色（加载中）/ 红色（错误），
* 当 2 秒内未收到新帧时显示的 **过期帧旋转图标** — 连接或断开后约 5 秒内属正常现象，此时后端正在各摄像头间重新平衡带宽分配。**组合模式**显示单个复合图块：后端进行去拜耳化、缩放、对齐、降噪，转换为各波段辐射度（若绑定了光传感器，则加上 DLS 反射率），评估数组的索引表达式， 应用查找表（LUT），并将结果以 MJPEG 格式流式传输。在渲染出第一个对齐帧之前，该图块会显示其状态：&quot;正在准备阵列……&quot;、 “校准对齐……”、“等待第一帧……”；或者——如果该图块的自动对齐重试（约一分钟）超时——则显示“需要对齐”并附带一个**校准对齐**按钮。

阵列图块的帧率读数也会报告流的状态：在更改设置（格式、像素合并、ROI）导致相机重启时显示 **“应用设置……”**；当阵列处于活动状态但已停止传输帧组时显示**“卡住”**；当阵列停止流式传输时显示**“已停止”**，表示已停止数据流传输——此时请重新连接阵列。当光谱解混处理不得不对合成图像中某个波段的大部分像素（25%或以上）进行钳位时，该图块会显示**“低可信度波段数据 · N%&quot;**：基于该波段生成的索引值可能有误（通常是某波段接收的光线过少）。

组合模式实用须知：

* 合成图像与**主**相机的帧对齐。在合成图像上进行的 AE-ROI 对焦和点测光，对主相机而言是精确的，对从属相机而言则是近似的；若需获得各相机像素级精确的图块，请使用**分屏视图**（阵列设置 → “显示成员相机”），无需额外打开相机连接。
* **显示图层**（阵列设置；默认关闭）允许您选择前景和背景图层——任何成员摄像机或**索引**。当前景 = 索引时，超出 LUT 最小/最大范围的像素将显示背景图层。
* **渲染分辨率**（默认 720p）同时设定实时流的高度 *和* 保存的合成图像导出尺寸。 每台摄像头的图像始终以全分辨率导出。
* 对齐情况按会话计算，且不会被持久保存。Chloros

会在**连接后自动对齐每个阵列（无论哪种显示模式）**，前提是成员摄像头的自动曝光已稳定（系统会持续尝试长达5分钟）。阵列设置面板的**对齐**部分会显示均方根残差以及**重新校准对齐** 按钮——该功能也适用于没有合成图块的独立阵列。

## 捕获：监控与分析

阵列采集表面可明确划分为 **监测级**（记录所见内容）和**分析级**（记录原始数据，稍后校准）：

| 工作流程 | 等级 | 保存内容 | GUI | CLI

|
| --- | --- | --- | --- | --- |
| **采集**（静帧） | 分析级 | 每轮扫描一个同步帧组；每个选定的导出级别（原始/去拜耳化/辐射度/反射率/预览/索引）下的每台相机文件 + `.daq` 辅助文件 |**“捕获全部”** 按钮 + 捕获设置 | `lattice array-capture` |
| **录制索引视频** | 监控 | 显示中的实时组合索引合成图像 — 8 位，预览分辨率，内嵌 LUT。 在未打开GUI视图的CLI

环境下，一旦阵列对齐，后端将渲染该阵列的NDVI

（或`--expr`）；若录制过程中未写入任何帧，则录制失败 | ● 录制指数视频（组合阵列） | `lattice array-record` |
| **原始连拍 → 生成视频**| 分析 | 以全抓取速率捕获原始传感器帧 + 清单 + `.daq`，随后离线重建为经过校准的辐射度/反射率/指数视频，并与数据采集（DAQ）读数进行时间同步 | ⦿ 录制原始连拍 →**生成视频** | `lattice array-burst` → `lattice array-build-video` |

经验法则：如果像素数据将用于*测量*，请使用“捕获”或“连拍”模式（分析级）；如果仅需*查看或演示*阵列所见内容，请录制索引视频（监控级）。

### 捕获设置（GUI）



<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->

**捕获全部**旁边的齿轮图标可打开“捕获设置”面板（需要打开项目——捕获数据将保存到该项目中）：

* **捕获模式**：**单次**（一次通过） /**连续**（连续捕获；受捕获帧数限制，默认 1 帧，或受时长限制，默认 10 秒） /**间隔** （延时摄影：每隔 X 间隔拍摄 N 张，共计 Y 张，默认每 5 秒拍摄 1 张，持续 1 分钟）。
* **每台相机的导出类型**：原始数据、去拜耳化、辐射度、反射率、预览、索引——所有适用的选项默认均开启。对于RGB

滤光片相机，辐射度/反射率选项会被隐藏； **“反射率”仅在相机具备 DAQ 光传感器时显示**（无论是相机自带的还是从阵列继承的）；“索引”需要配置索引表达式。
* **对齐**（按阵列设置，默认**开启**）：将成员导出数据变换为阵列的对齐配置文件，从而确保导出数据实现像素级对齐。 原始数据始终保持未变换状态，但会在元数据中携带变换信息。数组成员文件将按每台相机的原生方向保存（任何镜像/旋转设置仅记录在文件中，不会实际应用）。
* **最快采集**（开关）：仅采集原始数据 + 分配的 DAQ 读数 + 免费的组合指数合成数据，在采集时跳过校准计算以实现最高速率——稍后可从保存的 `.daq` 文件中重建辐射度/反射率/指数。
* 选择设置随项目保留。 隐藏或暂停的相机将被跳过。
* 数组中的 **连续** 模式在 Chloros

后端内运行（1.3.0 版新增）：当列表中的每台相机都属于一个数组时，后端会自行运行捕获循环，在写入一组数据的同时选择下一组，无需为每次捕获单独发送请求。 磁盘空间已满将停止运行（“已停止：磁盘已满。已保存 N，F 失败”）。 有关高捕获率的 Microsoft Defender 注意事项，请参阅 [捕获设置与模式](capture.md#capture-modes)。

等效的 CLI

（相同后端端点，相同语义）：

```bash
# One synced group, every applicable export level per camera (the default)
chloros-cli lattice array-capture -o output/

# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# 30-second monitoring clip of the combined index view, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# 5-second analysis-grade raw burst, then build the combined index video
chloros-cli lattice array-burst --duration 5 --build --products combined:index --fps 10 -o capture/
```

捕获的 TIFF

压缩格式为 `deflate`（无损，默认）或 `none` —— 完整的标志表、捕获文件夹结构以及重新处理规则详见 [CLI

参考](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess) 中。

## 配对 DAQ 光传感器

反射率和照度校正的预览需要来自 DAQ 传感器（在 **光传感器** 选项卡中连接）的向下照射光数据：

* 侧边栏的 **阵列行**显示一个**“DAQ · 开/关” 圆点**—— 当设置了阵列级光传感器**或** 任何成员摄像机拥有自己的光传感器时，该圆点显示为 *开*；其工具提示会精确列出哪个传感器为哪台摄像机提供数据。
* 在数组设置中进行全局分配 → **环境光传感器**→**光传感器**下拉菜单。该选择将随项目保留，并传播至每个成员相机，同时单个相机仍可通过其自身的传感器覆盖此设置。
* 其下方的状态行会报告实时状态： **关闭**→ “等待首个光谱……” →**“已激活 — 阵列中所有摄像机均已进行照度校正”** → 或者，如果过去 3 秒内未收到新的光谱，则显示过期提示 — 系统将继续使用上次读数（在捕获路径上，读数永不过期）。

分配传感器后：“反射率”导出类型将可用，实时预览将进行照度校正，预测性自动曝光可利用该光谱，且每次反射率采集都会将实际使用的 DAQ 读数作为 **`.daq` 文件** 写入采集的 `daq/`文件夹中，以便日后重新处理该捕获。此外，一个名为 `.daq.cameras.json` 的文件会标注该读数所对应的相机；重新处理时，Chloros

仅将该读数与来自这些相机的图像进行匹配。

## `array-connect` 命令行CLI

选项

| 标志 | 默认值 | 说明 |
| --- | --- | --- |
| `--serials SN1,SN2,…` | 自动发现所有 LATTICE 摄像头（需 ≥2 个） | **序列号最小的为MASTER。** |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO同步线。 |
| `--target-fps F` | auto | 主控触发器发射速率。 |
| `--binning {1,2,4}` | auto | 硬件像素合并。当任何帧率支持时，“auto”表示使用完整传感器（不进行像素合并）；若设置为“connect”，则始终不进行像素合并，仅为保持帧率。 |
| `--packed {auto,on,off}` | `auto` | [打包的 12 位传输](#packed-12-bit-transport): `auto` 仅在有助于提升性能且主机能够承担解包开销时才进行打包。 |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | 自动 | 专家级同步层选择器的覆盖设置。 |
| `--wire-ceiling-mbps MB_PER_S` | 自动检测 | 主机带宽预算（单位：MB/s）——即**带宽预算**字段的CLI

形式。若阵列报告帧损坏，请降低该值。该设置随项目保存，因此后续重新连接时将自动恢复。 |
| `--no-recommend` | 关闭 | 跳过网络分析步骤。 |
| `--no-ptp` | 关闭 | 禁用 PTP（此时不同摄像机之间的时间戳将无法比较）。 |

`lattice array-list`、`array-status` 和 `array-disconnect` 用于管理持久会话。 完整的子命令参考，包括对齐（`align-calibrate` / `align-apply`）和网络工具，请参见 [CLI

参考 § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice)；相应的 SDK

命令（`connect_array`、`ArraySession`、`attach_array`、`analyze_array_network`）则收录于 [SDK

参考](../reference/sdk-reference.md) 中。 在 Python

中，传输带宽配额为 `connect_array(..., wire_ceiling_mbps=120)`，而“有效数据/损坏数据/未到达数据”的划分详见 [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames)。
