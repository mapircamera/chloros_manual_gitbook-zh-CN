# 项目设置

Chloros 中的“项目设置”<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

侧边栏允许您配置项目的所有方面，包括图像处理、校准目标检测、多光谱指数计算以及导出选项。这些设置将随项目一起保存，并可保存为模板，以便在多个项目中重复使用。

## 访问项目设置

要访问项目设置：

1. 在 Chloros 中打开一个项目
2. 点击左侧边栏中的 **项目设置** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

选项卡
3. 设置面板将按类别显示所有可用的配置选项



<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->

{% hint style="info" %}
**依赖于其他设置的选项将显示为灰色。** 当父级开关导致某项设置无法进行时（例如，取消勾选 *反射率校准/白平衡* 会导致 *导出反射率* 无法进行），该从属控件将被禁用，其工具提示会标明需要更改的开关名称。
{% endhint %}

***

## 显示

### 图像缩略图分辨率

* **类型**：下拉菜单
* **选项**：`Default (512 px)`、`1024 px`、`2048 px`、`Full resolution`
* **默认值**：默认（512 像素）
* **说明**：图像网格缩略图的渲染分辨率（以最长边为单位，单位为像素）。数值越高，放大时显示越清晰，但加载速度越慢且占用更多内存。全分辨率即原始图像尺寸。
* **注意**： 仅用于显示 — 这绝不会影响处理或导出的文件。***

## 目标检测

这些设置控制 Chloros 如何在图像中检测和处理校准目标。这两项设置仅在启用 **反射率校准 / 白平衡** 时才生效（否则它们将显示为灰色，因为此时会完全跳过目标检测）。

### 最小校准样本面积（像素）

* **类型**：数字
* **范围**：0 至 10,000 像素
* **默认值**：25 像素
* **说明**：设置被检测区域被视为有效校准目标样本所需的最小面积（以像素为单位）。数值越小，可检测的目标越小，但可能会增加误报。数值越大，则需要更大、更清晰的目标区域才能进行检测。
* **何时调整**：
  * 若因图像中的微小伪影导致误检，请增加该值
  * 若校准目标在图像中显得较小且无法被检测到，请减少该值

### 最小目标聚类阈值 (0-100)

* **类型**：数字
* **范围**：0 至 100
* **默认值**：60
* **说明**：控制检测校准目标时，将颜色相似的区域分组所需的聚类阈值。数值越高，要求更多相似颜色才能被分组，从而导致目标检测结果更为保守；数值越低，则允许目标组内存在更多的颜色变化。
* **何时调整**：
  * 若校准目标被拆分为多个检测结果，请增加该值
  * 若存在颜色变化的校准目标未能被完全检测到，请减少该值

***

## 处理

这些设置控制Chloros 如何处理和校准您的图像。

### 暗角校正

* **类型**：复选框
* **默认**：启用（已勾选）
* **说明**：应用暗角校正以补偿图像边缘处的镜头变暗现象。暗角是一种常见的光学现象，由于镜头特性，图像的角落和边缘会比中心区域显得更暗。
* **副作用**：此开关还会决定运行时写入的 *未校准的备用产品* 类型（见下文）。

### 反射率校准 / 白平衡

* **类型**：复选框
* **默认**：启用（已选中）
* **描述**：启用反射率校准——根据相机类型及可用数据，通过检测到的画面内校准目标和/或 DAQ 光传感器下行光数据进行校准。这可对数据集中的反射率值进行标准化处理，并确保无论光照条件如何，测量结果均保持一致。
* **当禁用时**：将完全跳过目标检测，且**任何相机均无法生成反射率产品**——无论是由Survey3 目标驱动还是由LATTICE DAQ驱动。相关设置（*导出反射率*、*最小重新校准间隔*以及目标检测阈值）将显示为灰色不可用。

### 未校准的备用产品：导出传感器响应 / 导出暗角校正

* **类型**：两个复选框
* **默认设置**：均启用（已勾选）
* **说明**：当帧无法进行反射率校准时（未找到校准目标，或反射率校准已关闭），该帧将作为 *未校准的备用产品* 写入。**对于每种相机型号，每次运行仅存在其中一种备用产品**，由 *暗角校正* 开关决定：
  * 暗角校正 **开启**→ `Vignette_Corrected_Images/`（由**导出暗角校正** 控制）
  * 暗角校正 **关闭**→ `Sensor_Response_Images/`（由**导出传感器响应** 控制）
* 当前未启用的备用方案将显示为灰色。取消勾选当前启用的选项将完全停止该文件的写入。

### LATTICE 导出产品

对于包含 LATTICE 捕获数据的项目，每个导入的 LATTICE 帧将在单次处理过程中拆分生成所有已启用**且适用**的产品。 四个复选框用于控制扇出（默认均**开启**）：

| 设置 | 输出文件夹 | 导出内容 |
| --- | --- | --- |
| **导出去拜耳化图像** | `Debayered_Images/` | 线性去拜耳化图像： 已去除校准包中的暗场偏移，随后进行边缘感知去马赛克处理，不包含线性校正、平场校正或分色处理——与拍摄时在去马赛克层级保存的文件相同。保持传感器原生方向（记录在XMP中）。适用于RGB 和多光谱相机。 |
| **导出预览** | `Preview_Images/` | 屏幕预览。RGB = 白平衡（如有可用，则采用DAQ光源；否则采用灰度世界）+ 伽马校正；多光谱 = 伪彩色拉伸。 |
| **导出辐射度** | `Radiance_Images/` | 以 W/m²/sr/nm 为单位的 Float32 谱辐射度。仅限多光谱（M3C/M3M）——不适用于 RGB 主文件。 无论*校准图像格式*设置如何，始终以32位TIFF 格式写入。 |
| **导出反射率**| `Reflectance_Calibrated_Images/` | Uint16反射率，经过缩放使**32768 = 反射率 1.0** （标记为 XMP `Chloros:PixelScale`）。仅限多光谱，当有匹配的 `.daq` 下行记录（或通过质量保证的帧内目标）覆盖该帧时才会写入。 |

* RGB 主相机输出去拜耳化数据及预览图像；因其不适用，故不生成辐射度/反射率数据。
* 去拜耳化/预览数据的位深度遵循“*校准图像格式*”设置；辐射度数据始终为float32。
* Survey3 处理不受这四个开关选项的影响。
* 多光谱 M3C 相机的辐射度与反射率产品也可将每个像素的 **裁剪掩码**，该掩码存储在产品文件夹旁边的 `clamp_masks/` 文件夹中，用于记录波段解混过程中将负值裁剪为零的位置。参见 [输出图像格式](../output-image-formats.md#clamp-masks)。

这四个开关在无界面模式下以 `chloros-cli process --debayered / --preview / --radiance / --reflectance` 的形式存在。它们取代了旧的 CLI 标志 `--radiometric-output`，后者现已不存在。

{% hint style="warning" %}
**关闭所有适用的输出产品将导致处理任务失败。** 从 1.2.0 版本开始，如果处理任务被要求生成输出产品但未写入任何图像产品，则会报告失败，且 CLI 将以非零状态退出，而非默认报告静默成功。日志中会列出无法写入的产品及其原因。刻意仅生成元数据的处理任务 （未请求任何产品）仍会被视为成功。
{% endhint %}

### 反射率来源（项目设置，通过CLI

设置）

该项目还会存储 LATTICE 反射率产品所使用的 **反射率参考**。 设置面板中没有专门的控件；该值以 `Processing → "Target reflectance source"` 的形式存储在项目配置中，并通过 `chloros-cli process --reflectance-source {auto,target,daq}` 进行设置 （SDK 中的 `configure()` 没有相应的关键字）：

* **`auto`**（默认）：通过质量保证（QA）的帧内校准目标将成为绝对参考；若无目标或 QA 失败，则回退至 DAQ 下行分界值（ρ = πL/E）。 带有标记（如面板脱落或波段故障）但通过质量保证（QA）的目标不会被自动优先采用：系统将按其在保留面板上测得的得分顺序，依次尝试“目标”、“目标+DAQ”和“仅DAQ”这三种链路。
* **`target`**：严格的靶标驱动反射率——不进行DAQ替换。
* **`daq`**：DAQ权威反射率；帧内靶标不作为参考。

存储的值在匹配时不区分大小写，且接受几种拼写作为别名：`target`、`target_image`、 `empirical` 和 `empirical_line` 均表示 **目标**； `daq`、`dls`、`light_sensor` 和 `sensor` 均表示**daq**。其他情况——包括缺失的键——均解析为**auto**。

按单位进行的 **测量** 目标扫描将根据目标单位的序列号/QR（如 `<serial>.csv`）在以下三个位置进行查找： 由 `--target-reflectance-dir` 指定的目录（存储为 `Processing → "Target reflectance dir"`）、项目自身的 `target_reflectance/` 文件夹， 以及 `CHLOROS_TARGET_REFLECTANCE_DIR` 环境变量中的路径。如果该单元不存在实测扫描数据，则改用目标模型的标称发布曲线。

### 去拜耳化方法

* **类型**：下拉菜单选择
* **选项**：
  * 标准（快速，中等质量）
  * 纹理感知（慢速，最高质量）\[Chloros+\]
* **默认**：标准（快速，中等质量）
* **描述**：选择用于将原始拜耳阵列传感器数据转换为全彩图像的去马赛克算法。“标准（快速，中等质量）”方法在处理速度和图像质量之间实现了最佳平衡。“纹理感知（慢速，最高质量）” \[Chloros+] 采用高质量的边缘感知去马赛克算法，并结合 AI/ML 降噪模型，可消除几乎所有的去马赛克噪声。纹理感知模型运行时需要 GPU 内存（VRAM）。建议在可用 VRAM 超过 4GB 时使用该模式，以获得更快的处理速度。
* **当该行是下拉菜单时**：只有当**两个条件**均成立时才会显示——即您已使用符合条件的Chloros+订阅登录，**且**项目中不包含任何LATTICE拍摄素材。否则，该行将显示为纯文本“`Standard (Fast, Medium Quality)`”，且无选项可选。
* **LATTICE注意事项**： 目前尚无针对 LATTICE 训练的“纹理感知”模型，且无论存储值为何，管道都会强制对 LATTICE 帧应用标准去马赛克处理。如果您将 LATTICE 文件夹添加到已选中“纹理感知”的项目中，Chloros 会将该设置重写为“标准”，而非在 `project.json` 中保留过时的值。

### 最小重新校准间隔

* **类型**：数字
* **范围**：0 至 3,600 秒
* **默认值**：0 秒
* **描述**：设置使用校准目标之间的最小时间间隔（以秒为单位）。当设置为 0 时，Chloros 将使用所有检测到的校准目标。当设置为较大数值时，Chloros 仅会使用间隔至少为该数值秒数的校准目标，从而减少校准目标捕获频繁的数据集的处理时间。
* **何时调整**：
  * 在光照条件多变时，将该值设为 0 以获得最高的校准精度
  * 当光照条件稳定且频繁捕获校准目标图像时，可增加该值（例如设为 60-300 秒）以加快处理速度

### 光传感器时区偏移

* **类型**：数字
* **范围**：-12 至 +12 小时
* **默认值**：0 小时
* **说明**：指定光传感器数据时间戳的时区偏移量（以小时为单位，相对于协调世界时），用于将光传感器日志与图像捕获时间进行匹配。较新的 `.daq` 记录自带时区来源信息，因此此设置主要适用于以当地时间记录的旧日志。

### 应用 PPK 校正

* **类型**：复选框
* **默认**：禁用 （未勾选）
* **说明**：启用来自配备 GPS（GNSS）的 MAPIR DAQ 记录仪的后处理运动学（PPK）校正。启用后，Chloros 将使用项目目录中包含曝光引脚数据的任何 .daq 日志文件，并对图像应用精确的地理位置校正。
* **要求**：项目目录中必须存在包含曝光引脚条目的 .daq 日志文件
* **何时启用**：如果您的 .daq 日志文件中包含曝光反馈条目，建议始终启用 PPK 校正。

### 曝光引脚 1

* **类型**：下拉菜单
* **可见性**：仅在以下条件同时满足时可见 “应用 PPK 校正”已启用且第 1 引脚有可用曝光数据时才可见
* **选项**：
  * 项目中检测到的相机型号名称
  * “不使用” - 忽略此曝光引脚
* **默认值**：根据项目配置自动选择
* **说明**：为曝光引脚 1 分配特定的相机，用于 PPK 时间同步。曝光引脚记录相机快门触发的精确时间，这对准确的 PPK 地理定位至关重要。
* **自动选择行为**：
  * 单台相机 + 单个引脚：自动选择相机
  * 单台相机 + 两个引脚：引脚 1 自动分配给该相机
  * 多台相机：需要手动选择

### 曝光引脚 2

* **类型**：下拉菜单选择
* **可见性**：仅在“应用 PPK 校正”已启用且引脚 2 存在曝光数据时可见
* **选项**：
  * 项目中检测到的相机型号
  * “不使用” - 忽略此曝光引脚
* **默认值**：根据项目配置自动选择
* **说明**：在使用双相机设置时，将特定相机分配给曝光引脚 2，以实现 PPK 时间同步。
* **自动选择行为**：
  * 单相机 + 单引脚：引脚 2 自动设置为“不使用”
  * 单相机 + 双引脚：引脚 2 自动设置为“不使用”
  * 多相机：需要手动选择
* **注意**：同一台相机不能同时分配给引脚 1 和引脚 2。***

## DAQ 光传感器

此部分显示在“项目设置”中，列出了项目中的所有 DAQ 下行文件——`.daq` 记录文件以及 DAQ-M `.csv`向下辐射日志。在“光传感器”选项卡中创建的记录会自动添加到当前打开的项目中。

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->



每行显示文件、传感器型号以及该文件实际生效的漫射盖校正。行上方有一个适用于整个项目的控制选项：

### 漫射盖覆盖（所有文件）

* **类型**：下拉菜单
* **选项**：`Auto` 加上项目中现有传感器类型适用的散射帽校正曲线
* **默认值**：自动
* **存储为**：`Processing → "DAQ cap id"` （默认值为 `auto`）
* **说明**：`Auto` 使用每个文件中记录的顶盖参数（若未记录，则默认采用 Sunshine 顶盖；但 DAQ-U、DAQ-M 和 DAQ-E 出厂时未配备顶盖，且 Chloros 无法识别已安装的顶盖， 因此处理时会针对假定盖子发出警告：请选择实际安装的盖子）。选择特定盖子将覆盖项目中**所有**下行文件，但 DAQ-E-S 除外，因其扩散器是其校准的一部分；仅包含 DAQ-E-S 的项目不会显示下拉菜单：原始记录将使用该盖子进行校正， 而已包含校正值的记录将重新校准（撤销原记录的校正并应用所选校正）。
* **重要提示**：所选校正值必须与记录过程中实际安装的校正值一致。无论是传感器还是软件均无法检测物理校正值——校正值 ID 不匹配会导致光谱校正错误。

此处特意设置了**一个**项目级控制选项，而非按文件设置的下拉菜单：该设置将作用于项目中的所有下行源。***

## 阵列对齐

本节**仅**在项目中至少有一张图像携带 LATTICE 阵列在拍摄时标记的模块间对齐变换（XMP `Chloros:Alignment*` 标签）时才会显示。该部分显示了带有对齐标签的图像数量、参考相机（`REF` 标识），以及按相机分类的图像数量表。

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### 应用阵列对齐

* **类型**：复选框
* **默认**：启用（已选中）
* **存储格式**：`Processing → "Array alignment"`
* **描述**：使用拍摄时记录的变换参数，将每个处理后的产品（去拜耳化/预览/辐射度/反射率/指数）变换到数组的共享参考几何中。关闭时，将以原生逐传感器几何格式导出。

### 裁剪至公共重叠区域

* **类型**：复选框（仅在 *应用阵列对齐* 启用时有效）
* **默认**：启用（已选中）
* **存储为**：`Processing → "Array alignment crop"`
* **描述**：将对齐后的导出结果裁剪至所有相机模块共有的区域，从而使每个波段具有相同的覆盖范围。关闭时保留完整的传感器画布（源图像外部填充为黑色）。

### 重采样

* **类型**：下拉菜单（仅在 *应用阵列对齐* 启用时有效）
* **选项**：`Bilinear (smooth, default)`、`Nearest (preserve exact values)`、`Cubic (sharpest)`
* **默认**：双线性
* **存储为**：`Processing → "Array alignment interpolation"`
* **说明**：对齐变形所使用的插值方法。“最近邻”可保留源图像的精确值（无像素间混合），适用于严格的辐射测量分析；“双线性”最适合映射和可视化用途。

这三个选项在无头模式下也存在，分别为 `chloros-cli process --array-alignment`、`--array-alignment-crop` 和 `--array-alignment-interp {bilinear,nearest,cubic}`。

***

## 指数

这些设置允许您配置用于分析和可视化的多光谱指数。

### 添加指数

* **类型**：特殊指数配置面板
* **描述**：打开一个交互式面板，您可以在其中选择并配置多光谱植被指数（NDVI、NDRE、EVI 等），以便在图像处理过程中进行计算。您可以添加多个指数，每个指数都有其独立的可视化设置。
* **可用指数**：图形用户界面（GUI）的下拉菜单包含**27** 个预定义的多光谱指数公式（完整列表请参见 [多光谱指数公式](multispectral-index-formulas.md)，其中包含 CLI/SDK `--indices`选项）所支持的名称。
* **功能**：
  * 从预定义的指数公式中进行选择
  * 将相机的滤波通道拖拽到公式的波段槽位中
  * 配置可视化颜色渐变（LUT - 查找表）
  * 设置阈值和剪切模式
  * 创建自定义指数公式
* **注意**：对于单波段的 LATTICE M3M 单色相机，不会计算指数——在单波段情况下，多波段指数未定义。Survey3 和 LATTICE M3C 不受此影响。



<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->

您添加的每个指数都会将其公式以数学表达式形式呈现，每个波段插槽对应一个彩色圆圈：红色 = 红色（Red），绿色 = 绿色（Green），蓝色 = 蓝色（Blue），橙色 = 橙色（Orange），青色 = 青色（Cyan）， 紫色 = NIR，品红 = RE。将公式上方行中的圆圈拖动到插槽上即可绑定；双击已绑定的插槽可清除绑定。只有当公式中使用的每个插槽都有对应通道时，该指数才会进行计算。

### 自定义公式（Chloros+ 功能）

* **类型**：自定义公式定义数组
* **可用性**：需要使用符合条件的 Chloros+ 订阅登录。
* **描述**：允许您使用波段运算创建并保存自定义多光谱指数公式。自定义公式将与您的项目设置一同保存，并可像内置指数一样使用。
* **创建方法**：
  1. 在“指数配置”面板中，打开自定义公式计算器
  2. 使用 **波段-插槽符号** 编写公式，而非波段名称
  3. 为公式保存一个描述性名称——该公式随后会出现在公式下拉菜单的底部，您可以像使用内置预设一样，将相机的通道圆圈精确拖放到其槽位上
* **公式语法**：
  * 波段槽位： `x`, `y`, `z`, `a`, `b`, `c` — 六个位置，可通过拖拽映射到实际通道
  * 运算符：`+`、`-`、`*`、`/`、 `^` 以及 `()` 用于分组
  * 函数：`sqrt()`、`log()`、`ln()`、 `abs()`、`sign()`、`log1p()`、`log2()`
* **为何使用符号而非乐队名称**：写成 `(y-x)/(y+x)` 的公式在任何相机上都适用，因为拖放映射会决定 `y` 是 RGN 滤光片的 850 nm NIR 波段，还是 OCN 滤光片的 808 nm NIR 波段。 内置预设也是以同样方式存储的——请参阅 [多光谱指数公式](multispectral-index-formulas.md)，了解全部 27 个公式的确切符号形式。
* **适用范围**：自定义公式将随项目设置一同保存，既可在[指数/LUT沙盒](../image-viewer-gui/index-lut-sandbox.md)中使用，也可在图像处理过程中使用。 CLI/SDK 的`--indices`名称列表**不**支持这些公式，该列表仅扩展了22个内置预设名称。***

## 导出

这些设置控制导出处理后图像的格式和质量。

### 校准图像格式

* **类型**：下拉菜单
* **选项**：
  * **TIFF（16 位）** - 未压缩的 16 位 TIFF 格式
  * **TIFF（8位）** - 未压缩的8位TIFF 格式
  * **TIFF（32位，百分比）** - 32位浮点TIFF，反射率值以百分比表示
  * **PNG（8位）** - 压缩的8位PNG 格式
  * **JPG（8位）** - 压缩的8位JPEG 格式
* **默认**：TIFF（16位）；全8位项目显示为TIFF（8位），且仅提供8位格式
* **说明**：选择用于保存已处理和已校准图像的文件格式。导出的图像将保存在各相机文件夹内的、按格式划分的子文件夹中 (`tiff16`、`tiff32`、`png8`、`jpg8`)，每个产品对应一个 `<Product>_Images/` 文件夹。导出文件保留源文件名——通过文件夹（而非文件名后缀）来标识产品。
* **格式建议**：
  * **TIFF（16位）**：推荐用于科学分析和专业工作流程。可最大程度保留数据质量，且无压缩伪影。最适合多光谱分析以及在 GIS 软件中的进一步处理。
  * **TIFF（32 位，百分比）**：最适合需要以百分比（0-100%）表示反射率值的工作流程。为辐射测量提供最高精度。
  * **PNG（8位）**：适用于网页浏览和常规可视化。采用无损压缩，文件大小较小，但动态范围受限。
  * **JPG（8位）**：文件大小最小，仅适用于预览和网页显示。采用有损压缩，不适合科学分析。
* **注意**：无论此设置如何，LATTICE辐射度数据始终以32位浮点TIFF 格式导出。

***

## 保存项目模板

此功能允许您将当前的项目设置保存为可重复使用的模板。

* **类型**：文本输入框 + “保存”按钮
* **说明**：为您的设置模板输入一个描述性名称，然后点击保存图标。该模板将存储您当前的所有项目设置（目标检测、处理选项、索引、 及导出格式），以便在未来的项目中轻松复用。模板存储在项目保存文件夹内的 `Project Templates/` 文件夹中，也可通过主菜单（*选择模板* / *保存模板* / *导出模板*）进行选择或导出。
* **应用场景**：
  * 为不同的相机系统（RGB、多光谱、NIR）创建模板
  * 保存特定作物类型或分析工作流的标准配置
  * 在团队内共享统一的设置
* **使用方法**：
  1. 配置所有所需的项目设置
  2. 输入模板名称（例如，“RedEdge Survey3 NDVI 标准”）
  3. 点击保存图标
  4. 创建新项目时即可加载该模板

***

## 项目保存文件夹

此设置指定新项目的默认保存位置。

* **类型**：目录路径显示 + 编辑按钮
* **默认（Windows）**：位于“文档”文件夹中的 `Documents\MAPIR\Chloros Projects` —— 通常为 `C:\Users\[Username]\Documents\MAPIR\Chloros Projects`，或 OneDrive 及漫游配置文件将“文档”移动到的任何位置
* **默认（Linux）**：`~/Documents/MAPIR/Chloros Projects`
* **说明**：显示当前用于创建新 Chloros 项目的默认目录。点击编辑图标可选择其他目录。该覆盖设置将以单行文本形式存储在 `~/.chloros/working_directory.txt` 中——在 Windows 系统中即为 `C:\Users\<Username>\.chloros\working_directory.txt`。 如果该文件缺失，或指定的路径已不存在，Chloros 将回退到上述默认设置。CLI 和 GUI 读写的是同一个文件，因此 `chloros-cli` 和 GUI 始终对项目位置保持一致。
* **项目模板** 位于该目录下的 `Project Templates/` 子文件夹中。
* **何时更改**：
  * 设置为网络驱动器以实现团队协作
  * 对于大型数据集，更改为存储空间更大的驱动器
  * 按年份、客户或项目类型将项目整理到不同的文件夹中
* **注意**：更改此设置仅影响新项目。现有项目仍保留在原始位置。
* **Windows 路径长度**：除非启用了 Windows 长路径（安装程序中的 **启用 Windows 长路径** 选项），否则在处理前，路径长度超过约 130 个字符的项目文件夹将被拒绝，因为其生成的文件会超过 Windows 260 个字符的路径限制。 请参阅 [启动处理](../processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer)。

***

## 设置的持久性

Chloros 项目即为一个**文件夹**。所有项目设置均保存在其中的`project.json`文件中； 已连接的硬件信息则与项目一同保存在 `cameras.json` 和 `sensors.json` 中，因此重新打开项目时，其摄像头和光传感器也会自动重新连接。当您重新打开项目时，所有设置都会完全恢复为您离开时的状态。 保存的项目还可以通过 `chloros-cli project` 或 SDK 的 `open_project` 进行无界面控制。

### 设置层次结构

设置按以下顺序应用：

1. **系统默认值** - 由 Chloros 定义的内置默认值2. **模板设置** - 若在创建项目时加载了模板
3. **已保存的项目设置** - 随项目文件保存的设置
4. **手动调整** — 您在当前会话中进行的任何更改

### 设置与图像处理

处理设置会在处理运行开始时被读取。更改设置不会追溯性地修改已存储在磁盘上的处理结果——请重新运行处理以应用新设置。以下几项设置完全不会影响处理：

* 图像缩略图分辨率（仅用于显示）
* 保存项目模板
* 保存项目文件夹

***

## 配置键参考

对于自动化操作（CLI `--config`、SDK `configure` 或直接读取 `project.json`），以下是 `Project Settings` 下的确切键：

| 键路径 | 类型 | 默认值 |
| --- | --- | --- |
| `Display → Image Thumbnail Resolution` | `"512" \| "1024" \| "2048" \| "full"` | `"512"` |
| `Target Detection → Minimum calibration sample area (px)` | 数字 0-10000 | `25` |
| `Target Detection → Minimum Target Clustering (0-100)` | 0-100 之间的数值 | `60` |
| `Processing → Vignette correction` | 布尔型 | `true` |
| `Processing → Reflectance calibration / white balance` | 布尔型 | `true` |
| `Processing → Export sensor response` | 布尔型 | `true` |
| `Processing → Export vignette corrected` | 布尔型 | `true` |
| `Processing → Export debayered` | bool | `true` |
| `Processing → Export preview` | bool | `true` |
| `Processing → Export radiance` | bool | `true` |
| `Processing → Export reflectance` | bool | `true` |
| `Processing → Array alignment` | bool | `true` |
| `Processing → Array alignment crop` | bool | `true` |
| `Processing → Array alignment interpolation` | `"Bilinear" \| "Nearest" \| "Cubic"` | `"Bilinear"` |
| `Processing → Debayer method` | `"Standard (Fast, Medium Quality)" \| "Texture Aware (Slow, Highest Quality)"` | 标准 |
| `Processing → Minimum recalibration interval` | 数值 0-3600 | `0` |
| `Processing → Light sensor timezone offset` | 数值 -12..12 | `0` |
| `Processing → Apply PPK corrections` | 布尔值 | `false` |
| `Processing → DAQ cap id` | 限流配置文件 ID 或 `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | 索引配置列表 | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (8-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

`Array alignment` 键在“数组对齐”部分首次渲染时，或通过自动化调用设置时会被写入。当这些键不存在时，管道将使用上文所示的相同值 （`true`、`true`、双线性）的相同值，因此未包含这些键的项目.json的行为与包含这些键的项目完全一致。

### 存储在 `project.json` 中且在设置面板中无法控制的键

这些键位于同一 `Project Settings` 树下，并在处理过程中被读取，但侧边栏中不会显示相应的控件：

| 键路径 | 类型 | 默认值 | 设置者 |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK `input_level=`。覆盖 LATTICE 输入 TIFF 文件的解析方式； `auto` 根据每个文件的 `Chloros:ProcessingLevel` XMP 标签及通道数进行推断。对于 Survey3 `.raw` 捕获数据，此设置将被忽略。 刻意不作为GUI设置——在所有正常情况下，“auto”都是正确的。 |
| `Processing → Target reflectance dir` | 路径字符串 | `""` | `chloros-cli process --target-reflectance-dir`，或项目目标API |
| `Processing → Target reflectance config` | 以相机序列号为键的字典 | `{}` | 注册帧内目标 （模式 `fixed_block` / `fixed_strip` / `aruco`） |
| `Processing → DAQ-U log path` | 路径字符串 | `""` | SDK `process_folder(daq_log_path=…)`。指向一个 `.daq` 录制文件或其文件夹 |
| `Target Detection → Minimum calibration target squares` | 数字 | `4` | 旧版默认值；无控制项且无CLI 标志 |
| `UI → Grid thumbnail size` | 数字 | `160` | 图像网格自身的缩略图缩放滑块 |

有两项查看器首选项存储在 **`project.json` 的顶级位置**，完全位于 `Project Settings` 之外，因为它们属于显示状态而非处理设置：

| 键路径 | 类型 | 默认值 | 设置者 |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | 整数 1–256 | `1` | 图像选项卡的 GSD（像素）控件 — 参见 [全屏打开图像](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## 最佳实践

1. **从默认设置开始**：默认设置适用于大多数MAPIR 相机系统和典型工作流程。
2. **创建模板**：针对特定工作流程或相机优化设置后，请将其保存为模板，以确保不同项目间的一致性。
3. **全面处理前先测试**：尝试新设置时，请先对一小部分图像进行测试，再处理整个数据集。
4. **记录设置**：使用描述性模板名称，注明相机系统、处理类型和预期用途（例如，“Survey3\_RGB\_NDVI\_Agriculture”）。
5. **导出格式选择**：根据最终用途选择导出格式：
   * 科学分析 → TIFF（16 位或 32 位）
   * GIS 处理 → TIFF（16 位）
   * 快速可视化 → PNG（8 位）
   * 网络共享 → JPG（8 位）

***

有关 Chloros 中多光谱指数的更多信息，请参阅 [多光谱指数公式](multispectral-index-formulas.md) 页面。
