---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# 输出图像格式

Chloros 以四种文件格式导出处理后的产品。您可以在“项目设置”（GUI）中选择格式，或通过 `--format`（CLI）或 `export_format`（SDK）进行设置。CLI 和 SDK 接受以下精确的字符串。

| 格式字符串 | 扩展名 | 像素类型 | 像素范围 | 备注 |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *（默认）* | `.tif` | uint16 数字 | 0 – 65535 | 推荐用于摄影测量/GIS。 |
| `TIFF (8-bit)` | `.tif` | uint8 数字 | 0 – 255 | 适用于所有 8 位捕获，此类捕获仅提供 8 位格式：Chloros 不会提升位深度。 |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0.0 – 1.0 | 1.0 = 100% 反射率。部分应用程序无法读取浮点 TIFF 文件；文件体积较大。 |
| `PNG (8-bit)` | `.png` | uint8 数字 | 0 – 255 | 无损压缩，适用于网页浏览和可视化。 |
| `JPG (8-bit)` | `.jpg` | uint8 数字 | 0 – 255 | 有损压缩，文件体积最小。 |

## 输出文件的存储位置

生成文件保存在项目文件夹下，按相机分组，再按文件格式分类：

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera (model+lens+filter)
    ├── tiff16/                          # follows --format: tiff16, tiff8, png8, jpg8, or tiff32
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one <INDEX>_Index_Images/ folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

LATTICE 的相机文件夹为 `LATT-<sensor>-<lens>-F<filter>`，Survey3 的相机文件夹为 `<model>_<filter>`（例如 `Survey3N_RGN`）。 **每个导出的产品都保留源文件的名称——通过文件夹而非文件名后缀来识别产品。** 有关完整规则，请参阅 CLI 参考手册中的 [输出文件存放位置](reference/cli-reference.md)。

## LATTICE 产品（捕获和导出级别）

单次处理中，一个 LATTICE 原始帧会拆分为所有请求的产品。每种产品类型都有其专属的开关（GUI 复选框，或 CLI 命令 `--debayered` / `--preview` / `--radiance` / `--reflectance`，默认均开启)：

| 级别 | 内容 | 数据类型 |
| --- | --- | --- |
| `raw` | 直接来自传感器的拜耳数据（单色相机：单波段）。处理始终从原始数据开始。 | 原始采集数据 |
| `debayered` | 线性去马赛克——M3C 为 3 通道，M3M 为 1 通道灰度：先去除校准包中的暗偏移，然后进行边缘感知去马赛克，不进行线性校正、平场校正或解混。 在任何处理模式下，`process`导出都会生成与该级别拍摄保存的文件完全相同的文件。 | 线性去马赛克 |
| `radiance` | 来自完整辐射测量链的绝对光谱辐射度，单位为 **W/m²/sr/nm**。 无论选择何种导出格式，始终以 32 位 TIFF 格式写入（`tiff32/Radiance_Images/`）。 | float32 |
| `reflectance` | 反射率 ρ，其中 **DN 32768 = ρ 1.0 (100%)**，且留有扩展余量至 ρ 2.0。支持 Pix4D 格式。 | uint16 |
| `preview` | 显示就绪渲染：RGB = 白平衡 + 伽马校正；多光谱 = 伪彩色拉伸。 | 8 位显示 |

### 方向

`raw` 和 `debayered` 文件始终保持传感器的**原生**方向；相机的翻转/旋转设置仅应用于其他产品（预览、辐射度、反射率、指数）。 数组捕获会保留每个成员文件的原始方向，除非您指定方向（CLI 中为 `--orient`，SDK 中为 `orient=True`）。 每个文件都会在 XMP 中记录其像素所属的方位——`ChlorosRotation`、`ChlorosFlipHorizontal`、 `ChlorosFlipVertical`——而 `ChlorosOrientationApplied` 则表示其像素是否携带该信息（原始文件和去拜耳化文件中为 `false`）。 Chloros图像查看器仅根据该记录对原始文件和去拜耳化文件进行显示调整；文件本身绝不会被重写。没有该记录的文件（在1.3.0之前生成，或由其他工具生成）将按原始存储状态显示。

## 裁剪掩码

多光谱 M3C 相机的辐射度和反射率数据会进行 3×3 波段解混。当解混结果在某个波段产生负值时，该值会被裁剪为零（实时预览的裁剪方式完全相同）。 1.3.0 版本新增功能：若某产品中至少有一个像素被截断，则会生成一个配套的 **截断掩膜**，该掩膜会精确标明哪些像素和波段被截断：

* **位置：**位于 `clamp_masks/` 文件夹中，**紧邻**产品文件夹，绝不会位于产品文件夹内部： `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`，以及 `tiff32/Radiance_Images/x.tif` → `tiff32/clamp_masks/Radiance_Images/x_clampmask.tif`。
* **内容：** 基于产品自身像素网格（包含方向、ROI、像素合并和数组对齐）的单波段 `uint8` 格式TIFF文件。 当产品 R 通道中的波段被裁剪时，位 0 (`mask & 1`) 被设置；位 1 （`& 2`）表示 G 通道，第 2 位（`& 4`）表示 B 通道。 各波段被剪切像素的比例也以 `Chloros:BandRUnmixClampFraction`、`Chloros:BandGUnmixClampFraction` 和 `Chloros:BandBUnmixClampFraction` 的形式标记在产品中。
* **产品如何表示这一点：** XMP 标签 `Chloros:UnmixClampMask` 包含相对于产品文件夹的蒙版路径（`../clamp_masks/<product folder>/<name>_clampmask.tif`， 始终以`/`分隔），当没有像素被裁剪时（未写入蒙版）为`none`，或当无法写入蒙版时为`unavailable`。 没有标签的产品从未运行过解混操作（RGB和单色相机，原始和去拜耳化级别）。

由于没有任何产品文件夹包含遮罩，您可以将产品文件夹（例如 `Reflectance_Calibrated_Images/`）原样（包括子文件夹）直接提供给 Pix4D、Metashape 或其他摄影测量软件——请提供产品文件夹，而非其上层的格式文件夹。 Chloros 自身的导入功能会跳过钳位遮罩。CLI 会将其单独计数：`Image products written: N (plus M clamp masks)`。

## 索引产品

每个索引都会被放置在各自的 `<INDEX>_Index_Images/` 文件夹中（例如 `NDVI_Index_Images/`）。

* **整数格式**（`TIFF (16-bit)`、`TIFF (8-bit)`、`PNG (8-bit)`、 `JPG (8-bit)`）存储的是**每帧线性拉伸**，而非−1…1映射：DN 0 代表该帧的最低索引值，全尺度则代表最高值。 该文件包含 `Chloros:IndexScaling=linear_minmax`、`Chloros:IndexMin` 和 `Chloros:IndexMax`，因此还包含 `value = IndexMin + DN / full_scale × (IndexMax − IndexMin)`。 由于每个帧都有自己的端点，因此应按数值而非DN值来比较帧。
* **`TIFF (32-bit, Percent)`** 将索引值本身以float32格式存储，且未进行重新缩放——这是进行整趟飞行定量分析的最佳选择。

1.3.0 版本新增功能：索引产品描述的是**自身**，而非重复源原始数据的标签。其 GPS 和相机标签会从计算该索引产品的源产品（如有，即反射率产品）中复制而来，然后：

| 标签 | 值 |
| --- | --- |
| `Chloros:ProcessingLevel` | `index`（因此该文件绝不会作为采集数据被重新导入） |
| `Chloros:IndexApplied` / `Chloros:IndexExpression` | `True` / 所用的公式 |
| `Chloros:IndexLevel` | `reflectance` 或 `radiance` —— 该指数的计算依据（若两者均不适用则不显示） |
| `Chloros:IlluminationInvariant` | 基于反射率计算的指数使用 `True`，基于辐射度或传感器值计算的指数使用 `False` |
| `Chloros:ChlorosPipelineVersion` | 生成该指数的Chloros版本 |

`Chloros:PixelScale` 已从索引产品中移除 — 请使用上方的 `IndexMin` / `IndexMax`。 完整的回读配方详见 [CLI 参考](reference/cli-reference.md) 中的“整数索引栅格是按图像进行的对比度拉伸”。

## 读取反射率像素值

反射率以整数数字值（DN）形式存储，而**表示 ρ = 1.0（100% 反射率）的 DN 值取决于源相机**：

| 源相机 | ρ = 1.0 对应的 DN | 如何判断 |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768`（动态范围上限为 ρ 2.0） | 文件中带有 XMP 标签 `Chloros:PixelScale=32768`。 |
| Survey3 | `65535`（在 ρ 1.0 处被裁剪） | 没有 `Chloros:PixelScale` 标签——该标签的缺失即为特征。 （自 1.3.0 版本起，Survey3 产品在所有处理模式下均包含其他 `Chloros:*` 处理标签，例如 `Chloros:CalibrationApplied`。） |

**读取 `Chloros:PixelScale` XMP 标签并以此除以**，而不是假设一个常数。 该标签定义在 uint16 域中，因此在进行缩放的输出格式中仍保持为 `32768`——请先将存储的数据类型归一化回 uint16（8 位数据乘以 257，float32 数据乘以 65535）。

{% hint style="warning" %}
**8 位捕获数据。** 8 位捕获数据的导出值为 8 位。其反射率经过重新缩放并标记为 `Chloros:PixelScale=32768`，因此比例关系得以保持； 8位DN产品以及1.3.0版本之前生成的反射率文件，其数值会被裁剪至0–255范围，且不包含比例标签。如果LATTICE反射率文件中缺少该标签，请勿假设存在比例；应改用16位或32位重新导出。
{% endhint %}

有关完整规则（包括与 MicaSense 兼容的标签），请参阅 [CLI 参考](reference/cli-reference.md) 中的 **“读取反射率像素”**。
