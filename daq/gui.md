# Chloros 中的“DAQ”选项卡

“DAQ”选项卡——在 Chloros 侧边栏中标记为 **光传感器**——是 [DAQ-U、DAQ-M 和 DAQ-E 光传感器](README.md) 的实时控制界面： 可通过任何传输协议连接传感器，实时查看校准光谱，计算传感器对的实时反射率，并将 `.daq` 文件直接记录到您的项目中。

该选项卡在 Chloros 后端启动完成后即可使用。 该选项卡中的图表数据由 Chloros 的 DAQ 服务通过实时连接提供，若连接中断，系统将自动重新连接（重试间隔为 2–10 秒）；当无法连接该服务时，传感器的“状态”行将显示 **无服务器**。

布局由 **传感器侧边栏**（每个已连接传感器占一行）和**图表区域**（每个传感器或组对应一个图表块）组成。

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## 连接传感器

点击侧边栏顶部的 **连接传感器**。连接对话框将在主区域中打开（或在添加另一个传感器时以覆盖层形式显示——此时会出现“取消”按钮）。

| 控件 | 行为 |
| --- | --- |
| **设备类型** | `DAQ-U (USB)`（默认）、`DAQ-M (Bluetooth)` 或 `DAQ-E (Ethernet)`。切换后将重新扫描所选的新传输协议。 |
| **端口 / BLE 设备 / 主机名 / IP** | 将发现的设备列为 `device - description`；系统会自动选中首个被识别为传感器的条目。 扫描过程中会显示 `Scanning...`（USB）、带有 8 秒倒计时的 `Scanning (N)...`（BLE）或带有 5 秒倒计时的 `Discovering ethernet sensors (N)...` （以太网）。无结果时显示为 `No ports` / `No BLE devices` / `No ethernet sensors found`。 |
| **↻ 刷新** | 立即重新扫描所选传输协议（在 BLE/以太网扫描过程中此功能不可用）。 |
| **连接** | 选定设备后启用；建立连接时标签会变更为 `Connecting...`。 |

发现功能仅在**“连接”对话框显示于屏幕上时**运行，且仅针对所选传输协议每 15 秒重复一次——仅打开标签页不会触发扫描。若连接失败，对话框将显示：*“连接失败。请尝试拔出并重新插入传感器，然后再次点击‘连接’。”*

当第一个传感器连接成功时，侧边栏会自动弹出。

{% hint style="info" %}
**DAQ-E 未显示？** DAQ-E 没有状态指示灯——请检查其连接的交换机或注入器端口上的 PoE/链路指示灯，并在通电后等待几秒钟使其完成启动。 Chloros 设备必须位于同一广播域内（mDNS 无法穿透路由器）。 在 Windows 上，当 Chloros 首次绑定其组播套接字（mDNS UDP 5353、DAQ-E 数据 UDP 5002、 PTP UDP 319/320）。同一局域网上的两台 DAQ-E 设备会被分别发现，每台设备均拥有独立的 `daq-e-<id>.local` 主机名。
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>“设备类型”提供 DAQ-U（USB）、DAQ-M（蓝牙）和 DAQ-E（以太网）</figcaption></figure>***

## 传感器侧边栏

每个已连接的传感器都会占据一行（此外，每个“环境+对象”组还会额外占用一行）。行可通过拖拽重新排序，其顺序也会相应地重新排列图表块。点击某一行，即可将该传感器/组设为列表视图中的活动图表。

| 元素 | 含义 |
| --- | --- |
| 彩色左边框 | 传感器的图表颜色。 |
| 传输图标 | `DAQ-U` / `DAQ-M` / `DAQ-E`，若为 Ambient+Object 反射率组，则显示绿色 `REF` 图标。 |
| 设备名称 | 默认采用传感器的序列号（用于校准、`.daq` 文件名及导入匹配的稳定标识）；自定义名称在每个项目中保持不变。 |
| **已校准** 图标（绿色） | 当加载传感器的出厂校准包时显示，即光谱值为真实的 W/m²/nm。 |
| **有更新可用** 图标（琥珀色，仅限 DAQ-E） | 当前运行的固件版本早于此 Chloros 构建版本中捆绑的镜像。 更新过程中会显示实时进度（`Flashing… N%`、`Restarting sensor…`，随后为 `Updated X → Y` 或 `Failed`）。 |
| 眼睛图标 | 切换该传感器在图表中的可见性。 |
| 齿轮图标 | 打开单个传感器的设置弹窗（如下所示）。 |
| ✕（红色） | 断开传感器连接，或移除“环境+物体”组。 |

行上方有两个按钮：

* **连接传感器** — 打开连接对话框（连接过程中会重命名为 `Connecting...`）。
* **全部录制 / 全部停止**— 在**所有**已连接的传感器上启动或停止 `.daq` 录制。需要至少一个传感器**以及一个已打开的项目**（工具提示：“打开项目以进行录制”）；在任何录制运行期间，该按钮会变为红色。

空状态下显示“未连接任何传感器”。

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## 单个传感器设置（齿轮弹出窗口）

点击传感器行上的齿轮图标打开。内容按顺序排列：

* **信息行** — 设备类型（DAQ-U/M/E）、连接 （`Serial (USB)` / `Bluetooth` / `Ethernet`）、端口（COM 端口、BLE 地址或主机）以及序列号。
* **校准报告：下载** — 获取该设备的 NIST 可追溯校准证书（PDF），并在您的 PDF 阅读器中打开。在已知序列号后可用；首次连接时证书会被缓存。
* **设备名称** — 点击铅笔图标可重命名；名称在每个项目中保持不变。
* **图表线颜色** — 颜色样本；设置将按项目保存。
* **积分时间 (ms)**— 滑块 + 数值，**1–500 ms**，默认**32 ms**。当 AE 处于开启状态时，此选项不可用。
* **帧平均值**— 滑块 + 数值，**1–50 帧**，默认**20**。
* **AE：开/关**— 自动曝光开关；连接时**默认开启**。关闭此选项可手动设置曝光时间。
* **停止流媒体传输 / 开始流媒体传输** — 暂停或恢复实时流媒体传输。
* **录制 / 停止录制** — 按传感器 `.daq` 进行录制（需要打开项目）。
* **Cap** — 顶限校正配置文件 （详见下一节）。
* **实时信息行** — 积分时间（毫秒）、帧率（FPS）、采样数、录制状态（红色 `REC` 或 `Off`）以及状态 （`Streaming` / `Paused` / `SATURATED` / `No Server`）。

### 仅限 DAQ-E：网络、固件和 PTP 行

* **主机名 / IP** — 设备的当前地址。
* **固件** — 实时固件版本，附带一个操作单元：<version\>

当此 Chloros 构建包含较新的 DAQ-E 固件镜像时，会显示</version\>

一个 **更新至 \<version\>** 按钮。更新过程通过网络进行，约需 30 秒； 传感器将自动重启并重新连接，若传输中断，当前固件将保持不变。进度实时显示（`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`）， 且当前状态下，传感器读数为 `Up to date`。
* **PTP 同步** — 实时 PTP 状态（回退至 `unknown`）。DAQ-E 固件 v1.2.0 及以上版本以“仅从属时钟”身份参与 IEEE 1588 PTPv2； Chloros主机的后端是PTP总主时钟，局域网中的每台DAQ-E和LATTICE相机都在域0中对其进行从属同步，将时间戳误差控制在约1毫秒内。

对于“环境+物体”组，设备模式下仅显示该组的源传感器、设备名称和图表线颜色。

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->

### 盖板选择

**滤光片**下拉菜单用于告知 Chloros 传感器扩散片上安装的是哪种物理滤光片，并将该滤光片出厂测量的校正曲线应用于每个光谱。选项因型号而异：

| 型号 | 滤光片选项 |
| --- | --- |
| DAQ-U | 无（裸传感器）、视场角 15°、视场角 30°、视场角 45°、视场角 60°、视场角 90°、Sunshine（余弦校正器） |
| DAQ-M | 无（裸传感器）、Sunshine（余弦校正器） |
| DAQ-E | 无（裸传感器）、15°视场角、45°视场角、90°视场角、Sunshine（余弦校正器） |

**所有型号的默认设置均为“阳光”（余弦校正器）** — MAPIR 出厂时每台 DAQ 均已安装“阳光”盖，这是标准的户外配置： 180°半球视场，在60°以内余弦误差≤±4%，在70°以内≤±4.5%（不建议在太阳高度角低于约15°时使用），设计上具有衰减效果（约12倍）。您的选择将保留在项目中。

{% hint style="warning" %}
**遮光罩的选择必须与实际安装的遮光罩一致。**无论是传感器还是软件，均无法检测已安装的遮光罩类型。 该选择将同时影响实时校正以及写入每个 `.daq` 文件的标记——鉴于 Sunshine 遮光罩具有约 12 倍的衰减系数，未申报的遮光罩更换将导致光谱校正偏差大致与此系数相当。 （拆下并重新安装同一顶盖，误差可降至约 1.5%。） 仅当物理上移除盖子时才选择**“无（裸传感器）”**；在DAQ-E上，“无”选项仍会应用其凹陷式玻璃扩散器的出厂几何轮廓——这并非无操作——且裸露的DAQ-E属于台式配置，不属于受支持的现场配置。
{% endhint %}

{% hint style="info" %}
从早期手册升级说明：1.1.0 版本中浏览器端的“阳光扩散器已安装”开关已取消。 电容处理现采用这种按传感器设置的电容配置文件，并在服务器端应用。
{% endhint %}

***

## 图表区域

顶部固定栏包含一个 **列表 ⇄ 网格视图切换按钮**和一个**图表缩放** 滑块（图块大小 200–2000 像素）。当存在多个图表组时，视图会自动切换为网格模式；当图表组数量为一个或更少时，则自动切换回列表模式。 视图模式和图表大小按项目保存。

每个传感器的**光谱图表**显示：

* **X 轴** — 波长 (nm)。 传感器网格范围为 340–1010 nm，间隔 5 nm（135 个点），显示时插值为 1 nm。
* **Y 轴** — 功率（W/m²），根据峰值自动选择 SI 前缀（m/µ/n）。 在所有三种传输方式下，光谱均为经辐射校准的光谱辐照度（W/m²/nm）。
* 单条曲线下方采用彩虹色光谱填充；同一图表上的多个传感器以彩色线条叠加显示，填充色调较暗。
* **悬停**— 显示垂直光标，标注波长及各传感器的数值；**拖动**可缩放（缩放时会出现缩小按钮）。
* **+** 按钮（仅限网格视图），用于向该图表添加传感器或创建组（如下所述）。
* 设备名称居中显示在顶部，并在收到第一帧数据前显示加载转圈图标。

**饱和**状态不会在图表上直接标注：饱和的传感器会在实时数据表中显示红色 `SATURATED` 状态文本，且对应行显示为红色 `Saturated: Yes`。 降低积分时间或重新启用 AE 可清除该状态。

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->

***

## 实时数据表（列表视图）

位于图表下方的列表视图，每 500 毫秒刷新一次：

* **所有型号**： 光色样本（基于 CIE XYZ 的 sRGB）、饱和（是/否）、CIE 1931 X/Y/Z、色度 x/y、CIE u′/v′、CCT（K）、CRI（Ra）、主波长（nm）、 峰值波长（nm）、激发纯度、Duv、CIE L\*/a\*/b\* 以及孟塞尔 H/V/C。
* **仅限已校准的传感器**（加载出厂校准包后的任何 DAQ-U / DAQ-M / DAQ-E 传感器——传感器行中显示的绿色**已校准** 标识即为标志）： 总功率（W/m²）、明视照度（lx）、暗视照度（lx）、S/P比、PPFD 以及 PPFD Red/Green/Blue（µmol/m²/s），以及各视锥细胞的辐照度——S锥、黑视锥、红视锥、M锥、L锥（单位均为 W/m²）。

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## 反射率组 （环境光 + 物体）

两个相连的传感器可组合成实时反射率显示——无需摄像头：

1. 在网格视图中，点击图表磁贴上的 **+**，然后选择**组合环境光 + 物体**。
2. 选择一个 **环境光源**传感器和一个**物体扫描仪**传感器（两个不同的传感器），然后点击**创建**。

Chloros 根据两个实时数据流，按波长计算 R(λ) = 物体(λ) / 环境(λ)（当环境光 ≤ 0 时，R(λ) 为 0）。该组的标签遵循传感器的校准类别：

* 两个传感器均已校准（已加载校准包）→ **“视反射率”**。
* 任一传感器未校准 → **“相对反射率”**。

该组将以绿色 `REF` 行显示在侧边栏中，并拥有独立的图表（彩虹填充，悬停显示四位小数，拖动缩放）。

**+**菜单还提供**添加新传感器** 选项，有三种放置方式：*合并新传感器*（加入此图表）、*将现有传感器移至此处* 或 *查看新传感器*（单独图表）。

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->

### 植被指数表

在列表视图中，植被指数表位于反射率组图表下方，该指数基于各波段中心处的实时反射率计算得出：**蓝光 450 / 绿光 550 / 红光 670 / NIR 800 nm** （数值精确到小数点后4位，无法计算时显示为`---`；将鼠标悬停在指数名称上可查看其全称）：

* **始终显示**（尺度不变，适用于任何传感器组合）：NDVI、GNDVI、 ENDVI、WDRVI、GRVI、CVI、GCI、MSR。
* **仅当两个传感器均完成功率校准时**（两个数据包均已加载）：EVI、SAVI、OSAVI、GSAVI、GOSAVI、 MSAVI2、RDVI、TDVI、LAI、NLI、MNLI、FCI、GEMI。

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

***

## 录制 `.daq` 文件

* 录制需要 **打开项目** —— 否则，“全部录制”（侧边栏）和各传感器的“录制”按钮均处于禁用状态。
* 文件将写入 **`<project folder>/light_sensor/`**；文件名包含传感器 ID 和时间戳，设备名称将随录制数据一同存储。
* 当录制停止时（通过“停止”、“全部停止”或录制中途断开连接），已完成的 `.daq` 文件将 **自动添加到打开的项目中** ——它会出现在项目的文件列表中，无需手动添加，可直接作为[反射率处理](README.md)的入射辐射数据使用。
* 录制过程中，设置模态窗口的实时行中会显示一个红色的 `REC` 指示符。

对于定量辐照度数值，请至少取 15 秒的数据进行平均——这是仪器特性，而非缺陷。

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->

***

## 多传感器布局与项目持久性

* 可在同一图表中合并多个传感器（共享坐标轴），保持独立图表（自动网格布局），在图表间移动传感器，拖拽重新排列行/图块，并通过“眼睛”图标切换隐藏单个传感器。
* 每个项目中，Chloros 会保留以下信息：设备名称、图表颜色、图表大小、查看模式以及每个传感器的设置（积分时间、帧均值、AE 状态、电容选择）。
* **重新打开项目时，系统会根据地址自动重新连接其传感器**——DAQ-U 通过 COM 端口，DAQ-M 通过 BLE 设备， 对于 DAQ-E 则为 mDNS 主机名（即使设备 IP 地址发生变化也能解析）——并重新应用每个传感器已保存的 cap 配置文件、帧均值、AE 状态和手动积分时间。***

## 相机配对（DLS）

无需进行任何配对操作。 与无人机 DLS 工作流（需在前期将光传感器与相机绑定）不同，Chloros 在后续处理中将 DAQ 数据与图像进行匹配：在导入/处理时，`.daq` 的读数会被插值到每次拍摄的曝光时间戳上。 使用任何已连接的传感器进行记录（`.daq`会自动添加到项目中），反射率处理会根据时间找到正确的读数——有关下行数据的用法，请参阅[DAQ光传感器](README.md)。</version\>
