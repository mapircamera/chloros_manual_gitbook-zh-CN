---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Multispectral Index Formulas

The below index formulas use a combination of Survey3 filter average transmission ranges:

<table><thead><tr><th align="center">Survey3 Filter Color</th><th width="196.199951171875" align="center">Survey3 Filter Name</th><th width="159.800048828125" align="center">Transmission Range (FWHM)</th><th align="center">Average Transmission</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483nm</td><td align="center">475nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476-512nm</td><td align="center">494nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558nm</td><td align="center">547nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640nm</td><td align="center">619nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668nm</td><td align="center">661nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735nm</td><td align="center">724nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848nm</td><td align="center">823nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865nm</td><td align="center">850nm</td></tr></tbody></table>

When these formulas are used the name may end in "\_1" or "\_2", which corresponds to which NIR filter, either NIR1 or NIR2 was used.

***

## EVI - Enhanced Vegetation Index

This index was originally developed for use with MODIS data as an improvement over NDVI by optimizing the vegetation signal in areas of high leaf area index (LAI). It is most useful in high LAI regions where NDVI may saturate. It uses the blue reflectance region to correct for soil background signals and to reduce atmospheric influences, including aerosol scattering.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

EVI values should range from 0 to 1 for vegetation pixels. Bright features such as clouds and white buildings, along with dark features such as water, can result in anomalous pixel values in an EVI image. Before creating an EVI image, you should mask out clouds and bright features from the reflectance image, and optionally threshold the pixel values from 0 to 1.

_Reference: Huete, A., et al. "Overview of the Radiometric and Biophysical Performance of the MODIS Vegetation Indices." Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 - Forest Cover Index 1

This index distinguishes forest canopies from other types of vegetation using multispectral reflectance imagery that includes a red edge band.

$$
FCI1 = Red * RedEdge
$$

Forested areas will have lower FCI1 values due to the lower reflectance of trees and the presence of shadows within the canopy.

_Reference: Becker, Sarah J., Craig S.T. Daughtry, and Andrew L. Russ. "Robust forest cover indices for multispectral images." Photogrammetric Engineering & Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 - Forest Cover Index 2

This index distinguishes forest canopies from other types of vegetation using multispectral reflectance imagery that does not include a red edge band.

$$
FCI2 = Red * NIR
$$

Forested areas will have lower FCI2 values due to the lower reflectance of trees and the presence of shadows within the canopy.

_Reference: Becker, Sarah J., Craig S.T. Daughtry, and Andrew L. Russ. "Robust forest cover indices for multispectral images." Photogrammetric Engineering & Remote Sensing 84.8 (2018): 505-512._

***

## GEMI - Global Environmental Monitoring Index

This non-linear vegetation index is used for global environmental monitoring from satellite imagery and attempts to correct for atmospheric effects. It is similar to NDVI but is less sensitive to atmospheric effects. It is affected by bare soil; therefore, it is not recommended for use in areas of sparse or moderately dense vegetation.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Where:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Reference: Pinty, B., and M. Verstraete. GEMI: a Non-Linear Index to Monitor Global Vegetation From Satellites. Vegetation 101 (1992): 15-20._

***

## GARI - Green Atmospherically Resistant Index

This index is more sensitive to a wide range of chlorophyll concentrations and less sensitive to atmospheric effects than NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

The gamma constant is a weighting function that depends on aerosol conditions in the atmosphere. ENVI uses a value of 1.7, which is the recommended value from Gitelson, Kaufman, and Merzylak (1996, page 296).

_Reference: Gitelson, A., Y. Kaufman, and M. Merzylak. "Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS." Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI - Green Chlorophyll Index

This index is used to estimate leaf chlorophyll content across a wide range of plant species.

$$
GCI = {NIR \over Green} - 1
$$

Having broad NIR and green wavelengths provides a better prediction of chlorophyll content while allowing for more sensitivity and a higher signal-to-noise ratio.

_Reference: Gitelson, A., Y. Gritz, and M. Merzlyak. "Relationships Between Leaf Chlorophyll Content and Spectral Reflectance and Algorithms for Non-Destructive Chlorophyll Assessment in Higher Plant Leaves." Journal of Plant Physiology 160 (2003): 271-282._

***

## GLI - Green Leaf Index

This index was originally designed for use with a digital RGB camera to measure wheat cover, where the red, green, and blue digital numbers (DNs) range from 0 to 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLI values range from -1 to +1. Negative values represent soil and non-living features, while positive values represent green leaves and stems.

_Reference: Louhaichi, M., M. Borman, and D. Johnson. "Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat." Geocarto International 16, No. 1 (2001): 65-70._

***

## GNDVI - Green Normalized Difference Vegetation Index

This index is similar to NDVI except that it measures the green spectrum from 540 to 570 nm instead of the red spectrum. This index is more sensitive to chlorophyll concentration than NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Reference: Gitelson, A., and M. Merzlyak. "Remote Sensing of Chlorophyll Concentration in Higher Plant Leaves." Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green Optimized Soil Adjusted Vegetation Index

This index was originally designed with color-infrared photography to predict nitrogen requirements for corn. It is similar to OSAVI, but it substitutes the green band for red.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Reference: Sripada, R., et al. "Determining In-Season Nitrogen Requirements for Corn Using Aerial Color-Infrared Photography." Ph.D. dissertation, North Carolina State University, 2005._

***

## GRVI - Green Ratio Vegetation Index

This index is sensitive to photosynthetic rates in forest canopies, as green and red reflectances are strongly influenced by changes in leaf pigments.

$$
GRVI = {NIR \over Green }
$$

_Reference: Sripada, R., et al. "Aerial Color Infrared Photography for Determining Early In-season Nitrogen Requirements in Corn." Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Green Soil Adjusted Vegetation Index

This index was originally designed with color-infrared photography to predict nitrogren requirements for corn. It is similar to SAVI, but it substitutes the green band for red.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Reference: Sripada, R., et al. "Determining In-Season Nitrogen Requirements for Corn Using Aerial Color-Infrared Photography." Ph.D. dissertation, North Carolina State University, 2005._

***

## LAI - Leaf Area Index

This index is used to estimate foliage cover and to forecast crop growth and yield. ENVI computes green LAI using the following empirical formula from Boegh et al (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Where EVI is:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

High LAI values typically range from approximately 0 to 3.5. However, when the scene contains clouds and other bright features that produce saturated pixels, the LAI values can exceed 3.5. You should ideally mask out clouds and bright features from your scene before creating an LAI image.

_Reference: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde, and A. Thomsen. "Airborne Multi-spectral Data for Quantifying Leaf Area Index, Nitrogen Concentration and Photosynthetic Efficiency in Agriculture." Remote Sensing of Environment 81, no. 2-3 (2002): 179-193._

***

## LCI - Leaf Chlorophyll Index

This index is used to estimate chlorophyll content in higher plants, sensitive to variation in reflectance caused by chlorophyll absorption.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Reference: Datt, B. "Remote Sensing of Water Content in Eucalyptus Leaves." Journal of Plant Physiology 154, no. 1 (1999): 30-36._

***

## MNLI - Modified Non-Linear Index

This index is an enhancement to the Non-Linear Index (NLI) that incorporates the Soil Adjusted Vegetation Index (SAVI) to account for the soil background. ENVI uses a canopy background adjustment factor (_L_) value of 0.5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Reference: Yang, Z., P. Willis, and R. Mueller. "Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy." Proceedings of the Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 - Modified Soil Adjusted Vegetation Index 2

This index is a simpler version of the MSAVI index proposed by Qi, et al (1994), which improves upon the Soil Adjusted Vegetation Index (SAVI). It reduces soil noise and increases the dynamic range of the vegetation signal. MSAVI2 is based on an inductive method that does not use a constant _L_ value (as with SAVI) to highlight healthy vegetation.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Reference: Qi, J., A. Chehbouni, A. Huete, Y. Kerr, and S. Sorooshian. "A Modified Soil Adjusted Vegetation Index." Remote Sensing of Environment 48 (1994): 119-126._

***

## NDRE- Normalized Difference RedEdge

This index is similar to NDVI but compares the contrast between NIR with RedEdge instead of Red, which often detects vegetation stress sooner.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - Normalized Difference Vegetation Index

This index is a measure of healthy, green vegetation. The combination of its normalized difference formulation and use of the highest absorption and reflectance regions of chlorophyll make it robust over a wide range of conditions. It can, however, saturate in dense vegetation conditions when LAI becomes high.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

The value of this index ranges from -1 to 1. The common range for green vegetation is 0.2 to 0.8.

_Reference: Rouse, J., R. Haas, J. Schell, and D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. Third ERTS Symposium, NASA (1973): 309-317._

***

## NLI - Non-Linear Index

This index assumes that the relationship between many vegetation indices and surface biophysical parameters is non-linear. It linearizes relationships with surface parameters that tend to be non-linear.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Reference: Goel, N., and W. Qin. "Influences of Canopy Architecture on Relationships Between Various Vegetation Indices and LAI and Fpar: A Computer Simulation." Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI - Optimized Soil Adjusted Vegetation Index

This index is based on the Soil Adjusted Vegetation Index (SAVI). It uses a standard value of 0.16 for the canopy background adjustment factor. Rondeaux (1996) determined that this value provides greater soil variation than SAVI for low vegetation cover, while demonstrating increased sensitivity to vegetation cover greater than 50%. This index is best used in areas with relatively sparse vegetation where soil is visible through the canopy.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Reference: Rondeaux, G., M. Steven, and F. Baret. "Optimization of Soil-Adjusted Vegetation Indices." Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI - Renormalized Difference Vegetation Index

This index uses the difference between near-infrared and red wavelengths, along with the NDVI, to highlight healthy vegetation. It is insensitive to the effects of soil and sun viewing geometry.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Reference: Roujean, J., and F. Breon. "Estimating PAR Absorbed by Vegetation from Bidirectional Reflectance Measurements." Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI - Soil Adjusted Vegetation Index

This index is similar to NDVI, but it suppresses the effects of soil pixels. It uses a canopy background adjustment factor, _L_, which is a function of vegetation density and often requires prior knowledge of vegetation amounts. Huete (1988) suggests an optimal value of _L_=0.5 to account for first-order soil background variations. This index is best used in areas with relatively sparse vegetation where soil is visible through the canopy.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Reference: Huete, A. "A Soil-Adjusted Vegetation Index (SAVI)." Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI - Transformed Difference Vegetation Index

This index is useful for monitoring vegetation cover in urban environments. It does not saturate like NDVI and SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Reference: Bannari, A., H. Asalhi, and P. Teillet. "Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping" In Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS '02, IEEE International, Volume 5 (2002)._

***

## VARI - Visible Atmospherically Resistant Index

This index is based on the ARVI and is used to estimate the fraction of vegetation in a scene with low sensitivity to atmospheric effects.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Reference: Gitelson, A., et al. "Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI - Wide Dynamic Range Vegetation Index

This index is similar to NDVI, but it uses a weighting coefficient (_a_) to reduce the disparity between the contributions of the near-infrared and red signals to the NDVI. The WDRVI is particularly effective in scenes that have moderate-to-high vegetation density when NDVI exceeds 0.6. NDVI tends to level off when vegetation fraction and leaf area index (LAI) increase, whereas the WDRVI is more sensitive to a wider range of vegetation fractions and to changes in LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

The weighting coefficient (_a_) can range from 0.1 to 0.2. A value of 0.2 is recommended by Henebry, Viña, and Gitelson (2004).

_References_

_Gitelson, A. "Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation." Journal of Plant Physiology 161, No. 2 (2004): 165-173._

_Henebry, G., A. Viña, and A. Gitelson. "The Wide Dynamic Range Vegetation Index and its Potential Utility for Gap Analysis." Gap Analysis Bulletin 12: 50-56._
