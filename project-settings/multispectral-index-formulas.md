---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# 多光谱指数公式

以下指数公式采用Survey3滤光片平均透射范围组合：

<table><thead><tr><th align="center">Survey3 滤光片颜色</th><th width="196.199951171875" align="center">Survey3 滤光片名称</th><th width="159.800048828125" align="center">透射范围 (FWHM)</th><th align="center">平均透射率</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483nm</td><td align="center">475nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476-512nm</td><td align="center">494nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558纳米</td><td align="center">547nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640纳米</td><td align="center">619nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668nm</td><td align="center">661nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735nm</td><td align="center">724nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848nm</td><td align="center">823nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865纳米</td><td align="center">850nm</td></tr></tbody></table>使用这些公式时，名称可能以&quot;\_1&quot;或&quot;\_2&quot;结尾，分别对应使用了NIR滤波器中的NIR1或NIR2。

***

## EVI - 增强植被指数

该指数最初为配合MODIS数据开发，通过优化高叶面积指数区域（LAI）的植被信号，对NDVI进行了改进。 在高叶面积指数区域（LAI）中，该指数对可能饱和的NDVI数据具有显著增强作用。其利用蓝色反射波段校正土壤背景信号，并减少大气影响（包括气溶胶散射）。

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

植被像素的EVI值应在0至1之间。云层、白色建筑等高亮特征以及水体等暗色特征，可能导致EVI图像中出现异常像素值。 生成EVI图像前，需对反射率图像进行云层与高亮特征遮罩处理，并可选地将像素值阈值化为0至1区间。

_参考文献：Huete, A. 等. &quot;MODIS植被指数辐射计量与生物物理性能综述.&quot; 《环境遥感》83卷(2002):195–213。

***

## FCI1 - 森林覆盖指数1

该指数利用包含红边波段的多光谱反射率影像，将森林冠层与其他植被类型区分开来。

$$
FCI1 = Red * RedEdge
$$

林地因树木反射率较低且冠层存在阴影，其FCI1值将更低。

_参考文献：Becker, Sarah J., Craig S.T. Daughtry, and Andrew L. Russ. &quot;Robust forest cover indices for multispectral images.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 - 森林覆盖指数 2

该指数通过不含红边波段的多光谱反射率影像，将森林冠层与其他植被类型区分开来。

$$
FCI2 = Red * NIR
$$

森林区域因树木反射率较低且冠层内存在阴影，其FCI2值将更低。

_参考文献：Becker, Sarah J., Craig S.T. Daughtry, and Andrew L. Russ. &quot;Robust forest cover indices for multispectral images.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI - 全球环境监测指数

该非线性植被指数用于基于卫星影像的全球环境监测，旨在修正大气效应。其原理与NDVI相似，但对大气效应的敏感度较低。该指数易受裸露土壤影响，故不建议在植被稀疏或中等密度的区域使用。

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

定义：

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_参考文献：Pinty, B. 与 M. Verstraete. GEMI：基于卫星监测全球植被的非线性指数。《植被》101 (1992): 15-20._

***

## GARI - Green 大气抗干扰指数

该指数相较于NDVI，对更广泛的叶绿素浓度范围更敏感，且对大气影响的敏感度较低。

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

伽马常数是取决于大气气溶胶状况的权重函数。ENVI采用1.7的数值，该值源自Gitelson、Kaufman与Merzylak（1996年，第296页）的推荐值。

_参考文献：Gitelson, A., Y. Kaufman, and M. Merzylak. &quot;利用Green通道进行EOS-MODIS全球植被遥感研究.&quot;《环境遥感》58卷（1996年）：289-298页._

***

## GCI - Green 叶绿素指数

该指数用于估算多种植物的叶片叶绿素含量。

$$
GCI = {NIR \over Green} - 1
$$

采用宽带红外波段与绿色波段可更精准预测叶绿素含量，同时提升灵敏度与信噪比。

_参考文献：Gitelson, A., Y. Gritz, and M. Merzlyak. &quot;叶片叶绿素含量与光谱反射率的关系及高等植物叶片无损叶绿素测定算法.&quot;《植物生理学杂志》160 (2003): 271-282._

***

## GLI - Green 叶片指数

该指数最初设计用于配合数字RGB相机测量小麦覆盖度，其中红、绿、蓝三色数字数值(DNs)范围为0至255。

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLI数值范围为-1至+1。负值代表土壤及非生物特征，正值则代表绿色叶片与茎秆。

_参考文献：Louhaichi, M., M. Borman, and D. Johnson. 《基于空间定位的平台与航空摄影记录放牧对小麦的影响》。《国际地理制图》第16卷第1期（2001年）：65-70页。

***

## GNDVI - Green 归一化差值植被指数

该指数与NDVI相似，但测量波段为540至570纳米的绿色光谱而非红色光谱。相较于NDVI，该指数对叶绿素浓度更敏感。

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_参考文献：Gitelson, A. 与 M. Merzlyak. &quot;高等植物叶片叶绿素浓度的遥感测定.&quot; 《空间研究进展》第22卷（1998年）：689-692页._

***

## GOSAVI - Green 优化土壤校正植被指数

该指数最初基于彩色红外摄影技术设计，用于预测玉米氮需求量。其原理与OSAVI相似，但以绿色波段替代红色波段。

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_参考文献：Sripada, R. 等. &quot;利用航空彩色红外摄影技术确定玉米生长季氮需求量.&quot; 博士学位论文，北卡罗来纳州立大学，2005年。

***

## GRVI - Green 比值植被指数

该指数对森林冠层光合速率敏感，因绿波段与红波段反射率受叶绿素变化显著影响。

$$
GRVI = {NIR \over Green }
$$

_参考文献：Sripada, R. 等。《基于航空彩色红外摄影技术确定玉米生长初期氮需求》。《农学杂志》第98卷（2006年）：968-977页。_

***

## GSAVI - Green 土壤校正植被指数

该指数最初基于彩色红外摄影技术设计，用于预测玉米氮需求。其原理与SAVI相似，但以绿色波段替代红色波段。

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_参考文献：Sripada, R. 等.《利用航空彩色红外摄影确定玉米生长季氮需求》。博士论文，北卡罗来纳州立大学，2005年._

***

## LAI - 叶面积指数

该指数用于估算植被覆盖度，并预测作物生长与产量。 ENVI采用Boegh等人（2002）提出的经验公式计算绿色LAI：

$$
LAI = 3.618 * EVI - 0.118
$$

其中EVI为：

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

高LAI值通常在0至3.5之间。但当场景包含云层及其他产生饱和像素的亮区时，LAI值可能超过3.5。创建LAI图像前，建议先对场景进行云层与亮区遮罩处理。

_参考文献：Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde, and A. Thomsen. &quot;航空多光谱数据在农业领域量化叶面积指数、氮浓度及光合效率的应用&quot; 《环境遥感》第81卷第2-3期（2002年）：179-193页。

***

## LCI - 叶绿素指数

该指数用于估算高等植物的叶绿素含量，对叶绿素吸收引起的反射率变化敏感。

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_参考文献：Datt, B.《桉树叶片含水量的遥感测量》。《植物生理学杂志》第154卷第1期（1999年）：30-36页。_

***

## MNLI - 改良非线性指数

该指数是对非线性指数（NLI）的改进版本，整合了土壤校正植被指数（SAVI）以考虑土壤背景影响。ENVI软件采用0.5的冠层背景校正因子（_L_）值。

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_参考文献：Yang, Z., P. Willis, and R. Mueller. &quot;Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy.&quot; Proceedings of the Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 - 改良土壤校正植被指数2

该指数是Qi等人（1994）提出的MSAVI指数的简化版本，在土壤校正植被指数（SAVI）基础上进行了改进。它能降低土壤噪声并提升植被信号的动态范围。 MSAVI2基于感应法构建，不同于SAVI采用固定_L_值来突出健康植被。

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_参考文献：Qi, J., A. Chehbouni, A. Huete, Y. Kerr, and S. Sorooshian. 《改良土壤校正植被指数》。《环境遥感》48卷（1994）：119-126页。

***

## NDRE- 归一化差值RedEdge

该指数与NDVI类似，但将NIR与RedEdge进行对比，而非Red，通常能更早检测到植被胁迫。

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - 归一化差值植被指数

该指数衡量健康绿色植被状况。其归一化差值公式结合叶绿素吸收反射峰值区域的特性，使其在广泛环境条件下表现稳定。但在高植被密度区域，当LAI值升高时，该指数可能出现饱和现象。

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

该指数取值范围为-1至1，绿色植被的典型值域为0.2至0.8。

_参考文献：Rouse, J., R. Haas, J. Schell, and D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. 第三届ERTS研讨会，NASA (1973): 309-317页。

***

## NLI - 非线性指数

该指数基于多数植被指数与地表生物物理参数间存在非线性关系的假设，通过线性化处理将非线性关系转化为线性关系。

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_参考文献：Goel, N., and W. Qin. 《冠层结构对各类植被指数与LAI及Fpar关系的影响：计算机模拟研究》。《遥感评论》10卷（1994）：309-347页。_

***

## OSAVI - 优化土壤校正植被指数

该指数基于土壤校正植被指数（SAVI），采用0.16作为冠层背景校正因子的标准值。Rondeaux（1996）研究表明，该值在低植被覆盖度条件下能提供比SAVI更显著的土壤变化，同时对大于50%的植被覆盖度表现出更高的敏感性。 该指数最适用于植被相对稀疏、土壤可透过冠层可见的区域。

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_参考文献：Rondeaux, G., M. Steven, and F. Baret. &quot;土壤调整植被指数的优化.&quot; 《环境遥感》55卷（1996年）：95-107页。

***

## RDVI - 重归一化差异植被指数

该指数利用近红外与红光波段的差异，结合NDVI，突出显示健康植被。其对土壤影响及太阳观测几何条件不敏感。

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_参考文献：Roujean, J. 与 F. Breon。《基于双向反射率测量估算植被吸收的光合有效辐射》。《环境遥感》第51卷（1995年）：375-384页。_

***

## SAVI - 土壤校正植被指数

该指数与NDVI类似，但可抑制土壤像素的影响。其采用冠层背景校正因子_L_，该因子取决于植被密度，通常需要预先了解植被量。 Huete（1988）建议采用_L_=0.5的优化值以应对一级土壤背景变化。该指数最适用于植被相对稀疏、土壤可透过冠层可见的区域。

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_参考文献：Huete, A. &quot;土壤校正植被指数（SAVI）.&quot; 《环境遥感》25卷（1988）：295-309页._

***

## TDVI - 转换差值植被指数

该指数适用于监测城市环境中的植被覆盖，不会像NDVI和SAVI那样出现饱和现象。

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_参考文献：Bannari, A., H. Asalhi, and P. Teillet. &quot;Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping&quot; In Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, Volume 5 (2002)._

***

## VARI - 可见光大气抗干扰指数

该指数基于ARVI构建，用于估算场景中植被占比，对大气效应具有较低敏感度。

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_参考文献：Gitelson, A. 等. &quot;可见光谱空间中的植被与土壤线：植被比例遥感估算的概念与技术.&quot; 《国际遥感杂志》23 (2002): 2537−2562._

***

## WDRVI - 宽动态范围植被指数

该指数与NDVI类似，但采用权重系数(_a_)来缩小近红外与红光信号对NDVI的贡献差异。 当植被密度中等至较高且NDVI值超过0.6时，WDRVI在该场景中表现尤为显著。 当植被覆盖率和叶面积指数（LAI）增加时，NDVI趋于趋于平稳；而WDRVI对更广泛的植被覆盖率范围以及LAI的变化更为敏感。

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

权重系数(_a_)取值范围为0.1至0.2。Henebry、Viña与Gitelson（2004）建议采用0.2的系数值。

_参考文献_

_Gitelson, A. &quot;宽动态范围植被指数在植被生物物理特性遥感量化中的应用.&quot; 《植物生理学杂志》161卷第2期（2004）：165-173页._

_Henebry, G., A. Viña, and A. Gitelson. &quot;宽动态范围植被指数及其在缺口分析中的潜在应用.&quot; 《缺口分析公报》12: 50-56._
