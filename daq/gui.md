# Chloros 中的“DAQ”选项卡

“DAQ”选项卡（在 Chloros 侧边栏中标记为 **光传感器**）是 [DAQ-U、DAQ-M 和 DAQ-E 光传感器](README.md) 的实时控制界面： 可通过任何传输协议连接传感器，实时查看已校准的光谱，计算传感器对的实时反射率，并将 `.daq` 文件直接记录到您的项目中。

该选项卡在Chloros后端启动完成后即可使用。该选项卡中的图表数据由Chloros的DAQ服务通过实时连接提供，若连接中断，系统会自动重新连接（重试间隔为2–10秒）；当服务不可达时，传感器的“状态”行将显示为**无服务器**。

界面布局包括一个 **传感器侧边栏**（每个已连接传感器占一行）和一个**图表区域**（每个传感器或组对应一个图表块）。

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## 连接传感器

点击侧边栏顶部的 **连接传感器**。连接对话框将在主区域中打开（或在添加另一个传感器时以覆盖层形式显示——此时会出现“取消”按钮）。

| 控件 | 行为 |
| --- | --- |
| **设备类型** | `DAQ-U (USB)`（默认）、`DAQ-M (Bluetooth)` 或 `DAQ-E (Ethernet)`。切换后将重新扫描所选的新传输协议。 |
| **端口 / BLE 设备 / 主机名 / IP** | 将发现的设备列为 `device - description`；系统会自动选中首个被识别为传感器的条目。 扫描过程中会显示 `Scanning...`（USB）、带有 8 秒倒计时的 `Scanning (N)...`（BLE）或带有 5 秒倒计时的 `Discovering ethernet sensors (N)...` （以太网）。无结果时显示为 `No ports` / `No BLE devices` / `No ethernet sensors found`。 |
| **↻ 刷新** | 立即重新扫描所选传输协议（在 BLE/以太网扫描过程中此功能不可用）。 |
| **连接传感器** | 选定设备后启用；连接建立期间标签将变为 `Connecting...`。 |

发现功能仅在**连接对话框显示于屏幕上时**运行，并仅针对所选传输协议每 15 秒重复一次——仅打开标签页不会触发扫描。若连接失败，对话框将显示：*“连接失败。请尝试拔下并重新插入传感器，然后再次点击‘连接’。”*

当第一个传感器连接成功时，侧边栏会自动弹出。

{% hint style="info" %}
**DAQ-E 未显示？**DAQ-E 没有状态指示灯——请检查其连接的交换机或注入器端口上的 PoE/链路指示灯，并在通电后等待几秒钟使其完成启动。Chloros 设备必须位于同一广播域内（mDNS 无法穿透路由器）。 安装程序中的**Chloros后端（UDP-In）**防火墙规则允许mDNS（UDP 5353）、DAQ-E数据（UDP 5002）和PTP（UDP 319/320）通过； 第三方防火墙也需开放相同的端口。同一局域网内的两台 DAQ-E 设备会被分别识别，每台设备均拥有独立的 `daq-e-<id>.local` 主机名。
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>“设备类型”提供 DAQ-U（USB）、DAQ-M（蓝牙）和 DAQ-E（以太网）</figcaption></figure>

***

## 传感器侧边栏

每个已连接的传感器都会占据一行（此外，每个“环境+对象”组还会占用一行）。行可通过拖拽重新排序，其顺序也会相应地重新排列图表块。点击一行可将该传感器/组设为列表视图中的活动图表。

| 元素 | 含义 |
| --- | --- |
| 彩色左边框 | 传感器的图表颜色。 |
| 传输图标 | `DAQ-U` / `DAQ-M` / `DAQ-E`，若为 Ambient+Object 反射率组，则显示绿色 `REF` 图标。 |
| 设备名称 | 默认采用传感器的序列号（用于校准、`.daq` 文件名及导入匹配的稳定标识）；自定义名称在每个项目中均会保留。 |
| **已校准** 图标（绿色） | 当加载传感器的出厂校准包时显示，即光谱值为真实的 W/m²/nm。 |
| **有更新可用** 圆点（琥珀色，仅限 DAQ-E） | 当前运行的固件版本早于此 Chloros

构建包中捆绑的镜像。更新过程中会显示实时进度（`Flashing… N%`, `Restarting sensor…`，随后显示为 `Updated X → Y` 或 `Failed`）。 |
| 眼睛图标 | 切换该传感器在图表中的可见性。 |
| 齿轮 | 打开单个传感器的设置模态窗口（如下所示）。 |
| ✕（红色） | 断开传感器连接，或移除“环境+物体”组。 |

行上方有两个按钮：

* **连接传感器** — 打开连接对话框 （处理中时标签会变为 `Connecting...`）。
* **全部录制 / 全部停止**— 在**所有**已连接的传感器上启动或停止 `.daq` 录制。 需要至少一个传感器**以及一个已打开的项目**（工具提示：“打开项目以进行记录”）；在任何记录运行期间，该按钮会变为红色。

空状态下显示“未连接任何传感器”。

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## 单个传感器设置（齿轮模态窗口）

点击传感器行上的齿轮图标打开。内容按顺序排列：

* **信息行** — 设备类型（DAQ-U/M/E）、连接方式（`Serial (USB)` / `Bluetooth` / `Ethernet`）、端口（COM 端口、BLE 地址或主机）以及串行号。
* **校准报告：下载** — 获取该设备的 NIST 可追溯校准证书（PDF），并在您的 PDF 阅读器中打开。 在已知序列号后可用；证书在首次连接时会被缓存。
* **设备名称** — 点击铅笔图标可重命名；名称在每个项目中保持不变。
* **图表线颜色** — 颜色样本；颜色在每个项目中保持不变。
* **积分时间 (ms)**— 滑块 + 数值，**1–500 ms**，默认**32 ms**。当 AE 处于开启状态时，此选项不可用。
* **帧平均值**— 滑块 + 数值，**1–50 帧**，默认**20**。
* **AE：开/关**— 自动曝光开关；连接时**默认开启**。关闭该选项可手动设置积分时间。
* **停止流媒体传输 / 开始流媒体传输** — 暂停或恢复实时流媒体传输。
* **录制 / 停止录制** — 按传感器 `.daq` 进行录制（需要打开项目）。
* **Cap** — 顶部校正配置文件（下一节）。
* **实时信息行** — 积分时间（毫秒）、帧率（FPS）、采样数、录制状态（红色 `REC` 或 `Off`）以及状态（`Streaming` / `Paused` / `SATURATED` / `No Server`）。

### 仅限 DAQ-E：网络、固件和 PTP 行

* **主机名 / IP** — 设备的当前地址。
* **固件** — 实时固件版本，以及一个操作单元：<version\>

当此Chloros

构建包包含更新的DAQ-E固件镜像时，将显示</version\>

一个**“更新至\<version\>**”按钮。更新过程通过网络进行，耗时约30秒；传感器将自动重启并重新连接，若传输过程中断，当前固件将保持不变。 进度实时更新（`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`），当前状态下该单元显示为 `Up to date`。
* **PTP 同步** — 实时 PTP 状态（回退至 `unknown`）。DAQ-E 固件 v1.2.0 及以上版本以“仅从属时钟”身份参与 IEEE 1588 PTPv2 协议； Chloros

主机的后端是PTP主时钟，局域网（LAN）上的每台DAQ-E和LATTICE相机均在域0中对其进行从属同步，将时间戳误差控制在约1毫秒以内。
* **网络**— 设备运行的地址及其获取方式（DHCP、链路本地或静态），以及重启后生效的保存设置。**更改…**将打开一个表单：**自动（DHCP）**或**静态 IP**，其中包含 IP 地址、子网掩码以及可选的网关。**应用并重启** 将保存设置，重启传感器并重新连接。 如果设置的静态地址位于本计算机网络之外，系统将拒绝该设置，因为重启后 Chloros

将无法连接到传感器；此时会弹出一个确认框，允许用户在计算机地址将被重新分配或路由到该网络时仍继续设置。**开机即流式传输**用于设置传感器是否在通电后立即开始流式传输，即在任何软件连接之前。**重启传感器**将重启传感器并使用当前设置重新连接。传感器记录数据期间不允许重启。1.3.0 版本新增功能。

对于“环境+对象”组，齿轮模态窗口仅显示该组的源传感器、设备名称和图表线颜色。

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->



### 盖帽选择

**盖帽**下拉菜单用于告知 Chloros

传感器扩散器上安装的是哪种物理盖帽，并将该盖帽的出厂校准曲线应用于每个光谱。选项取决于型号：

| 型号 | 盖帽选项 |
| --- | --- |
| DAQ-U | 无（裸传感器）、视场角 15°、视场角 30°、视场角 45°、视场角 60°、视场角 90°、Sunshine（余弦校正器） |
| DAQ-M | 无（裸传感器）、Sunshine（余弦校正器） |
| DAQ-E | 无（裸传感器）、15°视场角、45°视场角、90°视场角、Sunshine（余弦校正器） |

**所有型号的默认设置均为“阳光”（余弦校正器）** — MAPIR

出厂时每台 DAQ 均已安装“阳光”盖，这是标准的户外配置： 180°半球视场，在60°以内余弦误差≤±4%，在70°以内≤±4.5%（不建议在太阳高度角低于约15°时使用），设计上具有衰减特性（约12倍）。您的选择将保留在项目中。

{% hint style="warning" %}
**盖板选择必须与实际安装的盖板一致。**无论是传感器还是软件，均无法检测已安装的盖板类型。该选择将同时影响实时校正以及写入每个 `.daq` 文件的标记——鉴于“阳光”盖板具有约 12 倍的衰减， 若未声明更换盖板，光谱校正误差将大致与此倍数相当。（拆下并重新安装同一盖板，误差可降至约1.5%。）仅当盖板已实际移除时，才选择**“无（裸传感器）”**； 在 DAQ-E 上，“无”仍会应用其凹陷式玻璃扩散器的出厂几何轮廓——这并非无操作——且裸露的 DAQ-E 属于台式配置，不属于受支持的现场配置。
{% endhint %}

{% hint style="info" %}
从早期手册升级说明：1.1.0 版本中浏览器端的“阳光扩散器已安装”切换按钮已移除。盖子处理现改为按传感器设置的盖子配置文件，并在服务器端应用。
{% endhint %}

***

## 图表区域

顶部固定栏包含一个 **列表 ⇄ 网格视图切换按钮**和一个**图表缩放** 滑块（图块大小 200–2000 像素）。当存在多个图表组时，视图会自动切换为网格模式；当仅有一个或更少图表组时，则自动切换回列表模式。 视图模式和图表大小按项目保存。

每个传感器的**光谱图表**显示：

* **X 轴** — 波长（nm）。 传感器网格范围为 340–1010 nm，间隔 5 nm（135 个数据点），显示时插值为 1 nm。
* **Y 轴**— 对于已校准的传感器（绿色**已校准**圆点），**功率 (W/m²/nm)**— 基于所有三种传输方式的辐射校准光谱辐照度 — 并根据峰值自动选择 SI 前缀 (m/µ/n)。若未加载校准包，传感器将以**强度 (计数)** 形式绘制原始读数，且不进行缩放。 当图表同时叠加这两种数据时，功率显示在左轴，计数显示在右轴。
* 单条曲线下方为彩虹色谱填充；同一图表中多个传感器以彩色线条叠加显示，填充色调较暗。
* **悬停**— 显示垂直光标，标注波长和各传感器的数值（已校准传感器显示为 W/m²/nm，否则显示为总计数）；**拖动**可缩放（缩放时会出现缩小按钮）。
* **+** 按钮（仅限网格视图），用于向该图表添加传感器或创建组（如下所述）。
* 设备名称居中显示在顶部，在收到第一帧数据前会显示加载转圈图标。

**饱和**状态不会在图表上直接标注：饱和的传感器会在实时数据表中显示红色状态文本 `SATURATED` 以及红色行 `Saturated: Yes`。降低积分时间或重新启用 AE 可消除该状态。

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->



***

## 实时数据表（列表视图）

位于图表下方的列表视图，每 500 毫秒刷新一次：

* **所有型号**：光色样本（基于 CIE XYZ 的 sRGB）、饱和（是/否）、CIE 1931 X/Y/Z、色度 x/y、CIE u′/v′、CCT（K）、 显色指数 (Ra)、主波长 (nm)、峰值波长 (nm)、激发纯度、Duv、CIE L\*/a\*/b\* 以及孟塞尔 H/V/C。
* **仅限已校准传感器**（DAQ-U / DAQ-M / DAQ-E 中的任意一款，只要已加载其出厂校准包——传感器行上的绿色**已校准** 标识即为标志）：总功率（W/m²）、明视照度（lx）、暗视照度（lx）、 S/P 比、PPFD 以及 PPFD Red

/Green

/Blue

（µmol/m²/s），以及各视锥细胞的辐照度——S-锥、黑视锥、红视锥、M-锥、L-锥（单位均为 W/m²）。

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## 反射率分组 （环境光 + 物体）

两个连接的传感器可以组合成实时反射率显示——无需摄像头：

1. 在网格视图中，点击图表方块上的 **+**，然后选择**组合环境光 + 物体**。
2. 选择一个 **环境光源**传感器和一个**物体扫描仪**传感器（两个不同的传感器），然后点击**创建**。

Chloros

会根据这两个实时数据流，按波长计算 R(λ) = 物体(λ) / 环境(λ)（当环境光强度 ≤ 0 时，结果为 0）。 该组的标签取决于传感器的校准类别：

* 两个传感器均已校准（已加载校准包）→ **“视反射率”**。
* 任一传感器未校准 → **“相对反射率”**。

该组在侧边栏中以绿色 `REF` 行显示，并拥有独立的图表（彩虹填充，悬停时显示 4 位小数，拖动缩放）。

**+**菜单还提供**添加新传感器** 功能，包含三种放置方式：*合并新传感器*（加入此图表）、*将现有传感器移至此处* 或 *查看新传感器*（单独图表）。

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->



### 植被指数表

在列表视图中，植被指数表位于反射率组的图表下方，该指数基于波段中心处的实时反射率计算得出：**蓝光 450 / 绿光 550 / 红光 670 / NIR

800 nm** 波段中心处的实时反射率计算得出（数值精确到小数点后4位，无法计算时显示为`---`；将鼠标悬停在指数名称上可查看其全称）：

* **始终显示**（尺度不变，适用于任何传感器组合）：NDVI

、GNDVI

、ENDVI、WDRVI

、GRVI

、 CVI

、GCI

、MSR。
* **仅当两个传感器均经过功率校准时显示**（两个传感器包均已加载）：EVI

、SAVI

、OSAVI

、GSAVI、GOSAVI、MSAVI2、RDVI、TDVI、LAI

、NLI、MNLI、FCI、GEMI

。

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

***

## 录制 `.daq` 文件

* 录制需要 **打开项目** —— 否则，“录制全部”（侧边栏）和各传感器的“录制”按钮均处于禁用状态。
* 文件将写入 **`<project folder>/light_sensor/`**；文件名包含传感器 ID 和时间戳，设备名称将随记录一起存储。
* 当录制停止时（点击“停止”、“全部停止”或录制过程中断开连接），生成的 `.daq` 文件将 **自动添加到当前打开的项目中** ——它会出现在项目的文件列表中，无需手动添加，可直接作为[反射率处理](README.md)的入射辐射数据使用。
* 录制过程中，设置模态窗口的实时行中会显示一个红色的 `REC` 指示符。

对于定量辐照度数值，请对至少 15 秒的数据进行平均处理——这是仪器特性，并非缺陷。

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->



***

## 多传感器布局与项目持久性

* 将多个传感器合并到一张图表中（共享坐标轴）、保持独立图表（自动网格布局）、在图表间移动传感器、拖动重新排列行/图块，以及通过“眼睛”图标切换按钮隐藏单个传感器。
* Chloros

会按项目保存以下信息：设备名称、图表颜色、图表大小、查看模式以及每个传感器的设置（积分时间、帧均值、AE 状态、盖帽选择）。
* **重新打开项目时，系统会根据地址自动重新连接传感器**——DAQ-U 通过 COM 端口，DAQ-M 通过 BLE 设备， 对于 DAQ-E 则为 mDNS 主机名（即使设备 IP 地址发生变化也能解析）——并重新应用每个传感器已保存的 cap 配置文件、帧平均、AE 状态和手动积分时间。***

## 相机配对（DLS）

无需进行配对。与无人机 DLS 工作流（需在前期将光传感器与相机绑定）不同，Chloros

在后续阶段将 DAQ 数据与图像进行匹配：在导入/处理时，`.daq` 的读数会被插值到每次拍摄的曝光时间戳上。 使用任何已连接的传感器进行记录（`.daq`会自动添加到项目中），反射率处理会根据时间自动匹配正确的读数——有关下行数据的使用方式，请参阅 [DAQ 光传感器](README.md)。 对于 **实时**LATTICE 捕获，请改在“相机”选项卡的**光传感器** 下拉菜单中选择传感器； 此时每次捕获都会保存其使用的读数，且该读数仅与该次捕获所用的摄像头相关联（参见 [反射率工作流](reflectance.md)）。</version\>
