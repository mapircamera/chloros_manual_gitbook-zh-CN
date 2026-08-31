# 图片网格

将图片导入项目后，您会在主区域看到它们以网格形式排列。该网格用于选择**您正在查看的每张图片的哪个版本**——网格上方的按钮可一次性将所有缩略图在源文件与各处理后的版本之间切换。

## 缩略图大小

使用右上角的缩放滑块调整图像缩略图的大小。滑块的范围为 **64 像素 至 1200 像素**。

* **Ctrl + 鼠标滚轮** 也可调整缩略图的大小。
* **Ctrl + `+`**/**Ctrl + `=`**和**Ctrl + `−`** 每按一次，大小以 4 像素为单位递增。 键盘调整范围的下限为 64 像素，上限则为当前窗口中每行正好能容纳两个缩略图的任意尺寸。
* 您最终确定的尺寸会随项目一起保存（`UI → Grid thumbnail size` 位于 `project.json` 中，默认值为 `160`），因此重新打开项目时会恢复该设置。

<figure><img src="../.gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>缩略图的 *分辨率* 与缩略图的 *尺寸* 是两个独立的设置：请参阅 [项目设置](../project-settings/project-settings.md) 中的 **显示 → 图像缩略图分辨率**（默认值为长边 512 像素）。 “大小”指图块的绘制尺寸；“分辨率”则指为填充图块而获取的细节程度。***

## 网格工具栏

网格上方的按钮行最多包含三个组，从左到右依次为：

1. **按触发器 / 按摄像头** — 分组模式。 仅在包含 LATTICE 捕获数据的项目中显示。
2. **相机过滤按钮** — 每个 LATTICE 相机对应一个。仅在“按相机”模式下显示。
3. **导出/查看模式按钮** — 控制每个缩略图显示的产品。

当窗口过窄无法同时显示所有按钮时，各组会从右向左折叠为悬停下拉菜单：首先折叠的是导出/查看按钮，其次是摄像机按钮。折叠后的组会留下一个标有当前活动选项的按钮，将鼠标悬停在其上会将完整按钮集向下滑出。 **“按触发器”/“按相机”选项永不折叠。

<!-- SCREENSHOT-NEEDED: Image grid toolbar of a LATTICE array project at full width, showing all three button groups inline: Per Trigger / Per Camera, three camera filter buttons labelled "LATT-M3M (serial)", and the export/view buttons including TIFF, RAW (Original), RAW (Radiance), RAW (Reflectance). -->

*****

## 导出/查看按钮

这些按钮用于在网格缩略图中切换图像类型。**当所命名的产品存在时，相应按钮即会显示**——对于源文件而言，这意味着在导入时立即显示，而非处理完成后。 Chloros 会在处理运行过程中重新扫描项目中的产品，因此随着每个产品开始写入磁盘，按钮会在处理期间陆续出现。

### 基础按钮

最左侧的导出按钮标注的是 **您实际导入的内容**：

| 您导入的内容 | 按钮标签 |
| --- | --- |
| Survey3 RAW+JPG | `JPG` |
| LATTICE 捕获的文件，其旁带有与 RAW 帧并列的显示预览 | `PNG` 或 `TIFF`（取决于预览内容） |
| LATTICE 捕获的图像中，基础文件 **即** 为 RAW 帧 | *无按钮* — `RAW (Original)` 已显示该文件 |

在混合项目中，标签将采用大多数图像所使用的扩展名。

### 产品按钮

| 按钮 | 显示内容 | 出现时机 |
| --- | --- | --- |
| **目标** | 检测到校准目标的图像 | 在检测到目标的运行结束后 |
| **目标**| 已检测到校准目标的图像 | 在检测到目标的运行结束后 ||**反射率** | 已校准的反射率图像 | 仅限 Survey3 项目 — LATTICE 项目使用 `RAW (Reflectance)` 代替，因此网格中不会同时显示两个反射率按钮 |
| **白平衡** | 白平衡处理后的图像（RGB相机） | 处理完成后 |
| **暗角校正** | 未校准的暗角校正备选方案 | 在无法应用反射率校准且 *暗角校正* 处于开启状态的运行后 |
| **传感器响应** | 未校准的传感器响应备用方案 | 同上，但*暗角校正*已关闭 |
| **`RAW (<INDEX> Index)`** | 每个计算出的索引对应一个按钮 | 配置索引后的运行结果 |
| **`<INDEX> LUT`** | 每个色彩映射指数对应一个按钮 | 配置了 LUT 后的运行结果 |
| **`<Index> <Index\|LUT> <NNN>`** | 每个 [索引/LUT 沙盒](index-lut-sandbox.md) 导出运行对应一个按钮 | 沙盒导出完成时 |

### LATTICE 层级按钮

包含 LATTICE 捕获数据的项目会添加以下按钮，其标签为层级名称而非产品名称：

| 按钮 | 层级 |
| --- | --- |
| **RAW（原始）** | 导入时的源原始帧 |
| **RAW (辐射度)** | Float32 谱辐射度，W/m²/sr/nm |
| **RAW (反射率)** | uint16 反射率，32768 = ρ 1.0 |

`RAW (Original)` 在导入的瞬间即可用——无需任何处理。当 LATTICE 导入完全没有基础按钮时（每个捕获的基础文件即为其原始帧），网格会自动移动到第一个可用的级别按钮上，以便工具栏的高亮显示与您所看到的画面相匹配。

两级 Chloros 导出文件**不包含独立的网格按钮**：

* **去拜耳化** —— `RAW (Original)` 视图已渲染为去拜耳化图像，因此在视觉上完全相同的图像上添加第二个按钮只会造成干扰。 `RAW (Debayered)` 生成文件仍会写入磁盘，并且仍可从全屏图层下拉菜单中选中。
* **预览** — 在 RGB 相机上，预览被注册为 `White Balanced` 图层，该图层确实带有按钮。 在多光谱相机上，它被注册为 `RAW (Preview)`，并可通过全屏图层下拉菜单访问。

{% hint style="info" %}
这些级别按钮仅在项目实际包含 LATTICE 帧时才会显示。 Survey3 项目会注册一些相同的内部图层名称，系统会过滤掉这些图层对应的按钮，因此 Survey3 网格仍保留其熟悉的 `JPG / Targets / Reflectance` 图层集。
{% endhint %}

点击网格缩略图将打开全屏 [图像查看器](opening-an-image-full-screen.md)，显示**该网格所展示的同一产品** ——如果网格设置为 `Targets`，则缩略图将打开导出的目标图像。

<figure><img src="../.gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: This GIF predates the LATTICE level buttons and the toolbar group separators. Reshoot on a LATTICE project cycling base -> RAW (Original) -> RAW (Radiance) -> RAW (Reflectance) -> an index button, so the new button set and the level names are visible. -->

***

## LATTICE 项目的分组：按触发器分组 vs 按摄像头分组

阵列抓拍会生成来自不同摄像头模块的同一瞬间的多张图像。分组方式决定了网格如何排列这些图像。 两种模式均会显示全宽的可折叠标题栏；**每个分组初始状态均为展开**，且 Chloros 会记住您折叠过的分组。折叠状态在两种模式下分别记录，因此在“按摄像头”模式下折叠某个分组，不会影响“按触发器”模式下的任何分组。

### 按相机（默认）

每个相机模块对应一个分组。标题栏显示相机型号和序列号（`LATT-M3M — <serial>`）以及照片数量。分组内的图块按拍摄事件的时间顺序排列。

在此模式下，工具栏还会为每台相机提供一个**相机筛选按钮**，标记为 `MODEL (SERIAL)`。所有相机初始均处于选中状态；点击按钮可取消选中该相机，并将其组从网格中移除。这是快速查看整个飞行任务中单个波段图像的便捷方式。

### 按触发器

每个捕获事件对应一个组——即所有模块在同一触发器下拍摄的帧集合。标题栏显示捕获时间、参与拍摄的相机数量，以及该组中每种相机型号的标识。 组内的图块按相机序列号排序，因此同一波段在每个触发事件中都位于同一列。

<!-- SCREENSHOT-NEEDED: Image grid in Per Trigger mode for a 3-camera LATTICE array, showing two consecutive trigger groups with their header bars (chevron, capture timestamp, "3 cameras", and the three model badges) and one group collapsed to show the closed state. -->
混合项目中的非LATTICE图像不会被分组——它们会在组之后以普通图块的形式显示。

***

## 网格缩略图遵循 GSD 块大小

如果您已在“图像”选项卡侧边栏中设置了 **GSD (px)** 块大小，则网格缩略图将以相同的地面分辨率呈现——而不仅仅是全屏视图。 当块大小设为 8 时，应用程序中显示该图像的任何位置，每个显示像素都是 8 × 8 源像素块的平均值。

由于瓦片宽度原本只有几百像素，因此粗糙的块大小在网格中早于全屏视图就不再产生可见差异：绘制在 160 像素瓦片中的 4000 像素帧，其每个显示像素已对应约 25 个源像素。 有关该控件的详细信息，请参阅 [全屏打开图像](opening-an-image-full-screen.md#gsd-block-size)。

***

## 相关页面

* [**全屏打开图像**](opening-an-image-full-screen.md) — 全屏查看器、光标值和直方图
* [**图像图层**](image-layers.md) — 全屏查看器内的图层下拉菜单
* [**索引/LUT 沙盒**](index-lut-sandbox.md) — 构建和导出索引可视化效果
* [**项目设置**](../project-settings/project-settings.md) — 决定哪些产品会被导出的开关选项
