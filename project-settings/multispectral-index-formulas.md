---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# 多光谱指数公式

以下指数公式综合采用了Survey3滤光片平均透射率范围：

<table><thead><tr><th align="center">Survey3滤波器颜色</th><th width="196.199951171875" align="center">Survey3滤光片名称</th><th width="159.800048828125" align="center">透射率范围（FWHM）</th><th align="center">平均透射率</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483nm</td><td align="center">475纳米</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476-512纳米</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558纳米</td><td align="center">547纳米</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640纳米</td><td align="center">619纳米</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668纳米</td><td align="center">661纳米</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735纳米</td><td align="center">724纳米</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848纳米</td><td align="center">823纳米</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865纳米</td><td align="center">850 nm</td></tr></tbody></table>

使用这些公式时，名称可能以“\_1”或“\_2”结尾，这对应于所使用的NIR滤光片类型，即NIR1或NIR2。

对于 LATTICE M3C（拜耳三波段通）相机，同一索引引擎采用 M3C 滤光片波段：

| M3C 滤光片 | 波段 1（中心/FWHM） | 波段 2（中心/FWHM） | 波段 3（中心/FWHM） |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

LATTICE M3M 相机为单波段（每台相机配备一个窄带滤光片），因此无法针对单张 M3M 图像计算多波段指数。 若要使用 M3M 计算指数，请将两台或更多相机图像组合成对齐的多波段堆栈，并使用 LATTICE 指数引擎（`chloros-cli lattice index` 或 GUI 中的实时指数计算器）。

***

## 各指数名称的适用场景

Chloros拥有**三个**指数表面，且它们的预设列表并不完全相同。请利用本节内容，确认某个名称在您计划使用的场景中是否有效。

| 当前位置 | 适用列表 | 数量 |
| --- | --- | --- |
| 项目设置 → 指数 → 添加指数 (GUI) | 表面 1 | 27 |
| 图像查看器 [索引/LUT 沙盒](../image-viewer-gui/index-lut-sandbox.md) (GUI) | 表面 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | 表面 2 | 27 |
| SDK `process_folder(indices=[...])` | 表面 2 | 27 |
| `chloros-cli lattice index --preset` | 表面 3 | 22（另一个 22） |
| “相机”选项卡实时索引计算器 | 表面 3 | 22（另一个 22） |

Surface 1 和 2 处理的是**来自单个摄像头的单张图像**， 使用与该相机滤光片通道绑定的符号槽 `x`/`y`/`z`(/`a`)。 Surface 3 处理的是**对齐的多波段堆栈**——即多台 LATTICE 相机配准后合并为一个立方体——并通过小写名称引用通道。

### 1. GUI 项目设置 / 图像查看器沙盒下拉菜单 — 27 个公式

下拉菜单按以下顺序列出这些公式 （按插入顺序排列，而非按字母顺序）：

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

在 GUI 中，您可以将相机的滤光片通道拖放到公式的波段插槽中，因此任何公式均可与相机支持的任意波段分配组合使用。您已保存的自定义公式将附加在此列表下方。

**仅限 GUI 使用的五个**公式——即 CLI/SDK 的 `--indices` 列表不支持的那些——其实现方式如下：

| 仅限 GUI 的预设 | 公式（实现形式） | 插槽 |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a（四个槽位） |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

每种公式的预期映射关系在本页后面的独立章节中给出（例如，GARI 期望 x=Green、y=NIR、z=Blue、a=Red）。GARI 是 Chloros 中唯一使用第四个槽位的公式。

### 2. CLI/SDK `--indices` 名称扩展 — 22 个预设

`chloros-cli process --indices` 选项（以及 SDK 中的 `indices` 参数）支持以下预设名称：

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**未识别的索引名称将被静默跳过。** 此列表之外的名称（包括仅限 GUI 使用的五个公式 `FCI1`、`FCI2`、`GARI`、`GEMI`、 `LCI`，以及您在图形用户界面中保存的任何自定义公式）将被忽略，仅生成日志提示——运行将跳过该索引继续进行，且运行结果仍报告成功。提示信息显示为：

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

名称在去除空格后进行匹配，且不区分大小写，因此 `ndvi`、`NDVI` 和 ` NDVI ` 属于同一个预设。 如果某个预设需要相机滤镜未提供的波段，该预设也会被跳过。
{% endhint %}

实际实现的精确公式（符号 `x`/`y`/`z` 代表波段槽位；每个预设的默认映射如下所示）：

| 预设 | 公式（实际实现） | 默认滤光片 | 波段槽位 (x, y, z) |
| --- | --- | --- | --- |
| NDVI | `(y-x)/(y+x)` | RGN | Red, NIR |
| GNDVI | `(y-x)/(y+x)` | RGN | Green, NIR |
| NDRE | `(y-x)/(y+x)` | RE | RE, NIR |
| OSAVI | `(y-x)/(y+x+0.16)` | RGN | Red, NIR |
| SAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Red, NIR |
| MSAVI2 | `(2*y+1-sqrt((2*y+1)*(2*y+1)-8*(y-x)))/2` | RGN | Red, NIR |
| EVI | `2.5*(y-x)/(y+6*x-7.5*z+1)` | RGN | Red, NIR, Blue |
| MSR | `((y/x)-1)/(sqrt(y/x)+1)` | RGN | Red, NIR |
| TDVI | `1.5*(y-x)/sqrt(y*y+x+0.5)` | RGN | Red, NIR |
| LAI | `3.618*(2.5*(y-x)/(y+6*x-7.5*z+1))-0.118` | RGN | Red, NIR, Blue |
| GCI | `(y/x)-1` | RGN | Green, NIR |
| GRVI | `y/x` | RGN | Green, NIR |
| GSAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Green, NIR |
| GOSAVI | `(y-x)/(y+x+0.16)` | RGN | Green, NIR |
| NLI | `((y*y)-x)/((y*y)+x)` | RGN | Red, NIR |
| MNLI | `((y*y-x)*(1+0.5))/((y*y)+x+0.5)` | RGN | Red, NIR |
| RDVI | `(y-x)/sqrt(y+x)` | RGN | Red, NIR |
| WDRVI | `(0.2*y-x)/(0.2*y+x)` | RGN | Red, NIR |
| CVI | `(z/y)*(x/y)` | RGN | Red, Green, NIR |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | NGB | NIR, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red, Green, Blue |

#### 预设名称如何转换为波段位置

当您传入一个纯名称（如 `NDVI`）时，Chloros 必须确定每个符号读取的是哪个文件的哪个通道。它使用下表，该表将滤波器代码映射到每个通道的数组位置：

| 滤波器代码 | 通道 → 数组索引 |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2（`Red` 被视为 Orange 的别名，同样为 0） |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

当项目中包含带有该滤镜的图像时，将使用预设的**默认滤镜**（即上表中的“默认滤镜”列）。 如果不包含，Chloros 将按 `RGN, OCN, NGB, RGB, RE, NIR` 的顺序扫描项目中实际存在的滤光片，并选择第一个能够提供预设所需所有通道的滤光片。如果没有，则该预设在本次运行中将被弃用。 这就是为什么在仅包含OCN的数据集上请求`NDVI`仍能产生合理结果的原因——它会绑定到OCN的Orange和NIR位置。

LATTICE M3C模型字符串中包含以`F`为前缀的滤光片 (`LATT-M3C-L41-FRGN`)，但在从图像中读取滤光片代码时，该前缀会被省略，因此 FRGN 相机通过上方的 `RGN` 行进行解析，无需特殊处理。

### 3. LATTICE 索引引擎（`lattice index --preset`，实时索引计算器）—— 22 种预设

LATTICE 引擎处理对齐的多波段堆栈（实时数组或导出的多波段 TIFF 文件），并使用小写通道名称（`red`、`green`、 `blue`、`red_edge`、`nir`）。其预设列表与上述两种不同：

| 预设 | 公式 | 通道 |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | 红光、近红外 |
| GNDVI | `(nir - green) / (nir + green)` | 红光、近红外 |
| BNDVI | `(nir - blue) / (nir + blue)` | 蓝光、近红外 |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | 红光边缘、近红外 |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | 蓝光、绿光、近红外 |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | 红光、近红外 |
| OSAVI | `(nir - red) / (nir + red + 0.16)` | 红光, 近红外 |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | 红光, 近红外 |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | 蓝光, 红光, 近红外 |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | 红光、近红外 |
| CVI | `nir * red / green**2` | 红光、绿光、近红外 |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | 红光、近红外 |
| TDVI | `1.5 * (nir - red) / sqrt(nir**2 + red + 0.5)` | 红光、近红外 |
| LAI | `3.618 * (2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)) - 0.118` | 蓝色、红色、近红外 |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | 红色、绿色、蓝色 |
| NGRDI | `(green - red) / (green + red)` | 红、绿 |
| VARI | `(green - red) / (green + red - blue)` | 红、绿、蓝 |
| TGI | `green - 0.39*red - 0.61*blue` | 红、绿、蓝 |
| EXG | `2*green - red - blue` | 红、绿、蓝 |
| CIRE | `(nir / red_edge) - 1` | 红_边缘, 近红外 |
| CIGREEN | `(nir / green) - 1` | 绿, 近红外 |
| NDWI | `(green - nir) / (green + nir)` | 绿色、近红外 |

运行 `chloros-cli lattice index --list-presets` 可从已安装的版本中打印此表格， 并运行 `--list-gradients` 以获取可用的颜色渐变。通道符号区分大小写，必须与预设的小写名称完全一致（例如 `--channel red=Red_660 --channel nir=NIR_850`）。

***

## CVI - 叶绿素植被指数

该指数对不同冠层密度下的叶片叶绿素含量较为敏感。它使用NIR、红光和绿光波段。

$$
CVI = NIR * {Red \over Green^{2}}
$$

在图形用户界面（GUI）以及命令行界面（CLI）/软件开发工具包（SDK）的预设列表中，其符号公式为 `(z/y)*(x/y)`，其中 x=Red、y=Green、z=NIR （即RGN相机的波段）；LATTICE指数引擎的预设为`nir * red / green**2`。每个地表均计算相同的指数。

_参考文献：Vincini, M., E. Frazzi, and P. D&#x27;Alessio. “冠层尺度的宽带叶绿素植被指数。”《精准农业》9 (2008): 303-319._

***

## ENDVI - 增强型归一化差值植被指数

该指数除NIR和绿色通道外还使用蓝色通道，在采用NGB滤光片的相机中广受欢迎，此类相机中蓝色波段取代了红色波段。

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

其实现采用符号公式 `((x+y)-(2*z))/((x+y)+(2*z))` —— 将相机的NIR和Green通道分配到x/y槽位，Blue通道分配到z槽位（对于NGB相机：x=NIR，y=Green，z=Blue）。

***

## EVI - 增强植被指数

该指数最初是为配合MODIS数据而开发的，旨在通过优化叶面积指数（LAI）较高的区域中的植被信号，来改进NDVI。它在LAI可能出现饱和的高LAI区域最为有用。它利用蓝色反射率区域来校正土壤背景信号，并减少大气影响，包括气溶胶散射。

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

对于植被像素，EVI值应在0到1之间。云层和白色建筑物等高亮特征，以及水体等暗色特征，都可能导致EVI图像中出现异常像素值。在生成EVI图像之前， 应从反射率图像中抠除云层和高亮特征，并可选地将像素值阈值化为0到1。

_参考文献：Huete, A. 等。《MODIS植被指数的辐射测量与生物物理性能概述》。《遥感环境》第 83 卷（2002）：195–213。_

***

## FCI1 - 森林覆盖指数 1

_仅限GUI——CLI/SDK中不提供`--indices`预设._

该指数利用包含红边波段的多光谱反射率图像，将森林冠层与其他类型的植被区分开来。

$$
FCI1 = Red * RedEdge
$$

由于树木反射率较低且树冠内存在阴影，林地区域的FCI1值会较低。

_参考文献：Becker, Sarah J.、Craig S.T. Daughtry 和 Andrew L. Russ. 《多光谱图像的鲁棒森林覆盖指数》。《摄影测量与遥感》第 84 卷第 8 期（2018）：505-512。_

***

## FCI2 - 森林覆盖指数 2

_仅限GUI——不作为CLI/SDK预设提供 `--indices`._

该指数利用不包含红边波段的多光谱反射率影像，将森林树冠与其他类型的植被区分开来。

$$
FCI2 = Red * NIR
$$

由于树木反射率较低且树冠内存在阴影，林地区域的FCI2值会较低。

_参考文献：Becker, Sarah J., Craig S.T. Daughtry, and Andrew L. Russ. “多光谱图像的鲁棒森林覆盖指数”。《摄影测量与遥感》84.8 (2018): 505-512。_

***

## GEMI——全球环境监测指数

_仅限GUI——不提供CLI/SDK版本的`--indices`预设._

该非线性植被指数用于基于卫星图像进行全球环境监测，并试图校正大气效应。 它与NDVI类似，但对大气效应的敏感度较低。该指数会受到裸土的影响；因此，不建议在植被稀疏或中等密度的区域使用。

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

其中：

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_参考文献：Pinty, B. 和 M. Verstraete。《GEMI：一种用于通过卫星监测全球植被的非线性指数》。《植被》101 (1992): 15-20._

***

## GARI - 抗大气干扰绿色指数（Green）

_仅限GUI使用——不作为CLI/SDK的`--indices`预设提供。_

与NDVI相比，该指数对广泛的叶绿素浓度范围更为敏感，且受大气影响较小。

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

伽马常数是一种加权函数，其值取决于大气中的气溶胶状况。ENVI采用的值为1.7，这是Gitelson、Kaufman和Merzylak （1996年，第296页）推荐的数值。

_参考文献：Gitelson, A., Y. Kaufman 和 M. Merzylak. “利用绿色通道通过EOS-MODIS对全球植被进行遥感。”《环境遥感》58 (1996): 289-298._

***

## GCI - 绿色叶绿素指数（Green Chlorophyll Index）

该指数用于估算多种植物物种的叶片叶绿素含量。

$$
GCI = {NIR \over Green} - 1
$$

利用宽范围的NIR和绿色波长，不仅能更好地预测叶片叶绿素含量，还能提高灵敏度并获得更高的信噪比。

_参考文献：Gitelson, A., Y. Gritz, 和 M. Merzlyak. “叶片叶绿素含量与光谱反射率之间的关系，以及高等植物叶片无损叶绿素评估算法。”《植物生理学杂志》160 (2003): 271-282._

***

## GLI - 绿叶指数（Green Leaf Index）

该指数最初是为配合数字RGB相机测量小麦覆盖率而设计的，其中红、绿、蓝三色的数字值（DN）范围均为0至255。

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLI值范围为-1至+1。负值代表土壤和非生物特征，而正值代表绿叶和茎秆。

_参考文献：Louhaichi, M.、M. Borman 和 D. Johnson. “利用定位平台和航空摄影记录放牧对小麦的影响”。《Geocarto International》第16卷第1期（2001）：65-70。_

***

## GNDVI——Green归一化差值植被指数

该指数与NDVI类似，区别在于它测量的是540至570纳米的绿色光谱，而非红色光谱。与NDVI相比，该指数对叶绿素浓度的敏感度更高。

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_参考文献：Gitelson, A. 和 M. Merzlyak. “高等植物叶片叶绿素浓度的遥感测定”。《空间研究进展》第22卷（1998）：689-692页。_

***

## GOSAVI——绿色优化土壤校正植被指数（Green Optimized Soil Adjusted Vegetation Index）

该指数最初是为利用彩色红外摄影预测玉米氮需求而设计的。它与OSAVI相似，但用绿色波段代替了红色波段。

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_参考文献：Sripada, R. 等。《利用航空彩色红外摄影确定玉米生长季氮需求》。博士学位论文，北卡罗来纳州立大学，2005年。_

***

## GRVI——绿色比值植被指数（Green Ratio Vegetation Index）

该指数对森林冠层的光合作用速率较为敏感，因为绿色和红色的反射率会受到叶片色素变化的显著影响。

$$
GRVI = {NIR \over Green }
$$

_参考文献：Sripada, R. 等。《利用航空彩色红外摄影确定玉米生长初期氮需求》。《农学杂志》98 (2006): 968-977._

***

## GSAVI——Green土壤校正植被指数

该指数最初是为利用彩色红外摄影预测玉米氮需求而设计的。它与SAVI相似，但用绿色波段代替了红色波段。

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_参考文献：Sripada, R. 等。《利用航空彩色红外摄影确定玉米生长季氮素需求》。博士学位论文，北卡罗来纳州立大学，2005年。_

***

## LAI - 叶面积指数

该指数用于估算叶片覆盖度，并预测作物的生长和产量。ENVI 采用 Boegh 等（2002）提出的以下经验公式计算绿色 LAI：

$$
LAI = 3.618 * EVI - 0.118
$$

其中EVI定义为：

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

LAI的高值通常在0到3.5之间。然而，当场景中存在云层及其他会产生饱和像素的明亮特征时，LAI值可能会超过3.5。在生成LAI图像之前，最好先对场景中的云层和明亮特征进行遮罩处理。

_参考文献：Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde 和 A. Thomsen. “用于量化农业中叶面积指数、氮浓度和光合作用效率的机载多光谱数据》。《环境遥感》第81卷，第2-3期（2002）：179-193。_

***

## 叶绿素指数（LCI）

_仅限GUI——不提供CLI/SDK预设`--indices`._

该指数用于估算高等植物中的叶绿素含量，对叶绿素吸收引起的反射率变化较为敏感。

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_参考文献：Datt, B. “桉树叶片含水量的遥感测定”。《植物生理学杂志》154卷，第1期（1999）：30-36._

***

## MNLI - 改良非线性指数

该指数是对非线性指数（NLI）的改进，它整合了土壤校正植被指数（SAVI）以消除土壤背景的影响。ENVI采用的冠层背景校正因子（_L_）值为0.5。

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_参考文献：Yang, Z., P. Willis 和 R. Mueller。《带比增强型 AWIFS 图像对作物分类精度的影响》。《佩科拉第 17 届遥感研讨会论文集》（2008），科罗拉多州丹佛市。_

***

## MSAVI2 - 改良土壤校正植被指数 2

该指数是Qi等人提出的MSAVI指数 （1994）提出的MSAVI指数的简化版本，该指数是对土壤校正植被指数（SAVI）的改进。它能降低土壤噪声并提高植被信号的动态范围。MSAVI2基于一种归纳法，不使用恒定的_L_值（如SAVI所做）来突出健康的植被。

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_参考文献：Qi, J., A. Chehbouni, A. Huete, Y. Kerr 和 S. Sorooshian。《改进的土壤调整植被指数》。《遥感环境》第 48 卷 (1994): 119-126。_

***

## MSR - 改良简单比值

该指数是对简单NIR/Red比值的改良，旨在使其与生物物理参数的关系线性化，且在植被密度较高时比NDVI更敏感。

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_参考文献：Chen, J. “植被指数评估及适用于北方林区的改良简单比值”。《加拿大遥感杂志》22 (1996): 229-242._

***

## NDRE——归一化差值RedEdge

该指数与NDVI类似，但将NIR与RedEdgeEdge而非Red进行对比，通常能更早地检测到植被胁迫。

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI——归一化差值植被指数

该指数是衡量健康绿色植被的指标。其归一化差值计算公式与叶绿素最高吸收和反射波段的结合，使其在各种条件下均表现稳健。然而， 在植被密集且叶面积指数（LAI）较高时，该指数可能出现饱和。

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

该指数的取值范围为-1至1。绿色植被的常见范围为0.2至0.8。

_参考文献：Rouse, J., R. Haas, J. Schell 和 D. Deering。《利用 ERTS 监测大平原地区的植被系统》。第三届 ERTS 研讨会，美国宇航局 (1973)：309-317。_

***

## NLI - 非线性指数

该指数假设许多植被指数与地表生物物理参数之间的关系是非线性的。它将那些往往呈非线性关系的地表参数关系线性化。

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_参考文献：Goel, N. 和 W. Qin. “冠层结构对各种植被指数与LAI及Fpar之间关系的影响：一项计算机模拟。”《遥感评论》10 (1994): 309-347._

***

## OSAVI——优化土壤校正植被指数

该指数基于土壤校正植被指数（SAVI）。它采用0.16作为冠层背景校正因子的标准值。 Rondeaux（1996）确定，在植被覆盖率较低的情况下，该值比SAVI能更好地反映土壤变异性，同时在植被覆盖率超过50%时表现出更高的敏感性。该指数最适用于植被相对稀疏、土壤可透过冠层可见的区域。

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_参考文献：Rondeaux, G., M. Steven, and F. Baret. “土壤校正植被指数的优化。”《遥感环境》55 (1996): 95-107._

***

## RDVI - 再归一化差值植被指数

该指数利用近红外波段与红光波段之间的差值，结合NDVI，以突出显示健康的植被。它对土壤和太阳观测几何关系的影响不敏感。

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_参考文献：Roujean, J. 和 F. Breon。《基于双向反射率测量估算植被吸收的有效光合辐射》。《环境遥感》第51卷（1995年）：375-384页。_

***

## SAVI——土壤校正植被指数

该指数与NDVI类似，但能抑制土壤像元的影响。它使用冠层背景校正因子_L_，该因子是植被密度的函数，通常需要事先了解植被量。 Huete（1988）建议将_L_设为0.5这一最优值，以考虑一阶土壤背景变化。该指数最适用于植被相对稀疏、土壤可透过冠层观察到的区域。

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_参考文献：Huete, A. “一种经土壤校正的植被指数（SAVI）。”《遥感环境》25 (1988): 295-309._

***

## TDVI - 转换差值植被指数

该指数适用于监测城市环境中的植被覆盖情况。它不会像NDVI和SAVI那样出现饱和现象。

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_参考文献：Bannari, A., H. Asalhi 和 P. Teillet。《用于植被覆盖制图的变换差值植被指数（TDVI）》，载于《地球科学与遥感研讨会论文集》（IGARSS &#x27;02），IEEE 国际，第 5 卷（2002）。_

***

## VARI——可见光抗大气干扰指数

该指数基于ARVI，用于估算场景中的植被占比，且对大气影响的敏感度较低。

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_参考文献：Gitelson, A., 等。《可见光谱空间中的植被与土壤分界线：一种遥感估算植被比例的概念与技术》。《国际遥感杂志》第23卷（2002年）：2537−2562。_

***

## WDRVI - 宽动态范围植被指数

该指数与NDVI类似，但采用权重系数（_a_）来减小近红外和红光信号对NDVI贡献之间的差异。当NDVI值超过0.6时，WDRVI在中等至高植被密度的场景中表现尤为有效。 随着植被覆盖率和叶面积指数（LAI）的增加，NDVI往往趋于平稳，而WDRVI则对更广泛的植被覆盖率范围以及LAI的变化更为敏感。

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

权重系数（_a_）的取值范围为0.1至0.2。Henebry、Viña和Gitelson（2004）建议采用0.2这一数值。

_参考文献_

_Gitelson, A. “用于遥感定量分析植被生物物理特性的宽动态范围植被指数。”《植物生理学杂志》161卷第2期（2004）：165-173。_

_Henebry, G., A. Viña 和 A. Gitelson。《宽动态范围植被指数及其在缺口分析中的潜在应用》。《缺口分析公报》第 12 期：50-56。_
