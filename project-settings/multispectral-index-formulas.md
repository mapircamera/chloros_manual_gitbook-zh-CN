---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# 多光谱指数公式

以下指数公式综合运用了 Survey3 滤光片各波段的平均透射率范围：

<table><thead><tr><th align="center">Survey3 滤光片颜色</th><th width="196.199951171875" align="center">Survey3 滤光片名称</th><th width="159.800048828125" align="center">透射率范围 (FWHM)</th><th align="center">平均透射率</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483nm</td><td align="center">475nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476-512nm</td><td align="center">494nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558nm</td><td align="center">547nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640nm</td><td align="center">619nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668nm</td><td align="center">661nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735nm</td><td align="center">724nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848nm</td><td align="center">823nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865nm</td><td align="center">850nm</td></tr></tbody></table>使用这些公式时，名称可能以“\_1”或“\_2”结尾，这对应于所使用的 NIR 滤波器，即 NIR1 或 NIR2。

***

## EVI - 增强植被指数

该指数最初是为配合 MODIS 数据而开发的，旨在通过优化叶面积指数（LAI）较高的区域中的植被信号，改进 NDVI。 该指数在叶面积指数（LAI）较高的区域最为有效，因为在这些区域，NDVI 可能会出现饱和。它利用蓝色反射率区域来校正土壤背景信号，并减少大气影响，包括气溶胶散射。

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

对于植被像素，EVI的值应在0到1之间。云层和白色建筑物等高亮特征，以及水体等暗色特征，都可能导致EVI图像中出现异常像素值。 在生成 EVI 图像前，应从反射率图像中抠除云层和亮色特征，并可选地将像素值阈值化为 0 到 1。

_参考文献：Huete, A. 等。《MODIS 植被指数的辐射测量与生物物理性能概述》。 《遥感环境》83 (2002):195–213。_

***

## FCI1 - 森林覆盖指数 1

该指数利用包含红边波段的多光谱反射率图像，将森林冠层与其他类型的植被区分开来。

$$
FCI1 = Red * RedEdge
$$

由于树木反射率较低且树冠内存在阴影，林地区域的FCI1值会较低。

_参考文献：Becker, Sarah J., Craig S.T. Daughtry, and Andrew L. Russ. &quot;多光谱图像的鲁棒森林覆盖指数。&quot;《摄影测量与遥感》84.8 (2018): 505-512._

***

## FCI2 - 森林覆盖指数 2

该指数利用不含红边波段的多光谱反射率影像，将森林冠层与其他类型的植被区分开来。

$$
FCI2 = Red * NIR
$$

由于树木反射率较低且树冠内存在阴影，林地区域的FCI2值会较低。

_参考文献：Becker, Sarah J., Craig S.T. Daughtry, and Andrew L. Russ. &quot;适用于多光谱图像的鲁棒森林覆盖指数。&quot;《摄影测量与遥感》84.8 (2018): 505-512._

***

## GEMI - 全球环境监测指数

该非线性植被指数用于基于卫星图像的全球环境监测，并试图校正大气效应。它与NDVI相似，但对大气效应的敏感度较低。该指数受裸露土壤影响，因此不建议在植被稀疏或中等密度的区域使用。

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

其中：

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_参考文献：Pinty, B. 和 M. Verstraete. GEMI：一种基于卫星监测全球植被的非线性指数。《植被》101 (1992): 15-20._

***

## GARI - Green 大气抗干扰指数

与 NDVI 相比，该指数对广泛的叶绿素浓度范围更为敏感，且受大气影响较小。

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

伽马常数是一种取决于大气中气溶胶状况的加权函数。ENVI采用1.7这一数值，该值是Gitelson、Kaufman和Merzylak（1996年，第296页）推荐的数值。

_参考文献：Gitelson, A., Y. Kaufman, and M. Merzylak. &quot;利用Green波段进行EOS-MODIS全球植被遥感.&quot; 《遥感环境》58 (1996): 289-298._

***

## GCI - Green 叶绿素指数

该指数用于估算多种植物物种的叶片叶绿素含量。

$$
GCI = {NIR \over Green} - 1
$$

采用宽波段的NIR和绿色波长，既能提高叶绿素含量的预测精度，又能增强灵敏度并提升信噪比。

_参考文献：Gitelson, A., Y. Gritz, and M. Merzlyak. &quot;叶片叶绿素含量与光谱反射率之间的关系及高等植物叶片无损叶绿素评估算法。&quot;《植物生理学杂志》160 (2003): 271-282._

***

## GLI - Green 叶片指数

该指数最初是为配合数字 RGB 相机测量小麦覆盖率而设计的，其中红、绿、蓝三色的数字值（DN）范围为 0 至 255。

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLI 的取值范围为 -1 至 +1。负值代表土壤和非生物特征，而正值代表绿叶和茎秆。

_参考文献：Louhaichi, M., M. Borman, and D. Johnson. 《基于空间定位平台与航空摄影记录放牧对小麦的影响》。《Geocarto International》第16卷第1期（2001）：65-70。_

***

## GNDVI - Green 归一化植被指数

该指数与NDVI相似，区别在于其测量的是540至570纳米的绿色光谱，而非红色光谱。该指数对叶绿素浓度的敏感度高于NDVI。

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_参考文献：Gitelson, A. 和 M. Merzlyak. “高等植物叶片叶绿素浓度的遥感测定。”《空间研究进展》22 (1998): 689-692._

***

## GOSAVI - Green 优化土壤校正植被指数

该指数最初是为利用彩色红外摄影预测玉米氮素需求而设计的。它与OSAVI相似，但用红色波段代替了绿色波段。

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_参考文献：Sripada, R. 等。《利用航空彩色红外摄影确定玉米生长季氮需求》。博士学位论文，北卡罗来纳州立大学，2005年。_

***

## GRVI - Green 比值植被指数

该指数对森林冠层的光合作用速率较为敏感，因为绿色和红色的反射率会受到叶片色素变化的显著影响。

$$
GRVI = {NIR \over Green }
$$

_参考文献：Sripada, R. 等. &quot;利用航空彩色红外摄影确定玉米生长季初期的氮素需求.&quot; 《农学杂志》98 (2006): 968-977._

***

## GSAVI - Green 土壤校正植被指数

该指数最初是利用彩色红外摄影技术设计，用于预测玉米的氮素需求。它与SAVI相似，但用红色波段代替了绿色波段。

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_参考文献：Sripada, R. 等。《利用航空彩色红外摄影确定玉米生长季氮素需求》。博士学位论文，北卡罗来纳州立大学，2005年。_

***

## LAI - 叶面积指数

该指数用于估算叶片覆盖度，并预测作物生长和产量。 ENVI 采用 Boegh 等（2002）提出的以下经验公式计算绿色 LAI：

$$
LAI = 3.618 * EVI - 0.118
$$

其中 EVI 为：

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

LAI 的典型值范围约为 0 至 3.5。然而，当场景中包含云层及其他产生饱和像素的亮部特征时，LAI 的值可能会超过 3.5。在生成 LAI 图像前，建议先对场景中的云层和亮部特征进行遮罩处理。

_参考文献：Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde, and A. Thomsen. &quot;用于量化农业中叶面积指数、氮浓度和光合效率的机载多光谱数据。&quot; 《环境遥感》81卷，第2-3期（2002）：179-193。_

***

## LCI - 叶片叶绿素指数

该指数用于估算高等植物中的叶绿素含量，对叶绿素吸收引起的反射率变化较为敏感。

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_参考文献：Datt, B. “桉树叶片含水量的遥感测定。”《植物生理学杂志》154卷，第1期（1999）：30-36._

***

## MNLI - 改良非线性指数

该指数是对非线性指数（NLI）的改进，其整合了土壤校正植被指数（SAVI）以消除土壤背景干扰。ENVI采用0.5作为冠层背景校正因子（_L_）值。

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_参考文献：Yang, Z., P. Willis, and R. Mueller. &quot;带比增强AWIFS图像对作物分类精度的影响。&quot;《Pecora 17遥感研讨会论文集》（2008），科罗拉多州丹佛市。_

***

## MSAVI2 - 改良土壤校正植被指数 2

该指数是 Qi 等（1994）提出的 MSAVI 指数的简化版本，该指数是对土壤校正植被指数（SAVI）的改进。它减少了土壤噪声，并提高了植被信号的动态范围。 MSAVI2 基于一种归纳法，不使用常数 _L_ 值（如 SAVI 所做）来突出健康的植被。

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_参考文献：Qi, J., A. Chehbouni, A. Huete, Y. Kerr, and S. Sorooshian. 《一种改良的土壤校正植被指数》。《遥感环境》48 (1994): 119-126。_

***

## NDRE- 归一化差值 RedEdge

该指数与NDVI类似，但比较的是NIR与RedEdge之间的对比度，而非Red，这通常能更早地检测到植被胁迫。

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - 归一化植被指数

该指数是衡量健康绿色植被的指标。其归一化差值公式与叶绿素最高吸收和反射波段的结合，使其在各种条件下均表现稳健。然而，当 LAI 值较高时，在植被密集的环境中该指数可能会出现饱和。

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

该指数的取值范围为-1至1。绿色植被的常见范围为0.2至0.8。

_参考文献：Rouse, J., R. Haas, J. Schell, and D. Deering. 《利用ERTS监测大平原植被系统》。第三届ERTS研讨会，NASA（1973）：309-317。_

***

## NLI - 非线性指数

该指数假设许多植被指数与地表生物物理参数之间的关系是非线性的。它将那些往往呈现非线性特征的地表参数关系线性化。

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_参考文献：Goel, N. 和 W. Qin. “冠层结构对各种植被指数与 LAI 及 Fpar 之间关系的影响：一项计算机模拟。”《遥感评论》10 (1994): 309-347._

***

## OSAVI - 优化土壤校正植被指数

该指数基于土壤校正植被指数（SAVI）。其采用0.16作为冠层背景校正因子的标准值。 Rondeaux（1996）指出，在植被覆盖率较低的情况下，该值比SAVI更能体现土壤差异，同时对植被覆盖率超过50%的情况表现出更高的敏感度。该指数最适用于植被相对稀疏、土壤可透过冠层观察到的区域。

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_参考文献：Rondeaux, G., M. Steven, and F. Baret. &quot;Optimization of Soil-Adjusted Vegetation Indices.&quot; Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI - 再归一化植被指数

该指数利用近红外与红光波段之间的差值，结合NDVI，以突出显示健康的植被。它对土壤的影响以及太阳观测几何角不敏感。

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_参考文献：Roujean, J. 和 F. Breon. “基于双向反射率测量估算植被吸收的有效光合辐射（PAR）。”《环境遥感》51 (1995): 375-384._

***

## SAVI - 土壤校正植被指数

该指数与NDVI类似，但能抑制土壤像素的影响。它采用冠层背景校正因子_L_，该因子是植被密度的函数，通常需要预先了解植被量。Huete (1988) 建议将_L_设为0.5，以考虑一阶土壤背景变化。 该指数最适用于植被相对稀疏、土壤可透过冠层观察到的区域。

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_参考文献：Huete, A. &quot;A Soil-Adjusted Vegetation Index (SAVI).&quot; 《环境遥感》25 (1988): 295-309。_

***

## TDVI - 转换差值植被指数

该指数适用于监测城市环境中的植被覆盖。它不会像NDVI和SAVI那样出现饱和现象。

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_参考文献：Bannari, A., H. Asalhi, and P. Teillet. &quot;用于植被覆盖制图的转换差值植被指数（TDVI）&quot; 载于《地球科学与遥感研讨会论文集》，IGARSS &#x27;02，IEEE国际，第5卷（2002）。_

***

## VARI - 可见光抗大气干扰指数

该指数基于 ARVI，用于估算场景中植被所占比例，且对大气效应的敏感度较低。

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_参考文献：Gitelson, A. 等. “可见光谱空间中的植被与土壤分界线：一种遥感估算植被覆盖率的概念与技术”。《国际遥感杂志》23 (2002): 2537−2562._

***

## WDRVI - 宽动态范围植被指数

该指数与 NDVI 类似，但采用权重系数 (_a_) 来减小近红外与红光信号对 NDVI 贡献值之间的差异。 当 NDVI 超过 0.6 时，WDRVI 在植被密度中等至较高的场景中表现尤为出色。 当植被比例和叶面积指数 (LAI) 增加时，NDVI 往往趋于平稳，而 WDRVI 对更广泛的植被比例范围和 LAI 的变化更为敏感。

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

权重系数（_a_）的取值范围为 0.1 至 0.2。Henebry、Viña 和 Gitelson（2004）建议采用 0.2 的值。

_参考文献_

_Gitelson, A. &quot;用于遥感量化植被生物物理特性的宽动态范围植被指数。&quot;《植物生理学杂志》161, 第2期 (2004): 165-173._

_Henebry, G., A. Viña, and A. Gitelson. &quot;宽动态范围植被指数及其在缺口分析中的潜在应用。&quot;《缺口分析公报》12: 50-56._
