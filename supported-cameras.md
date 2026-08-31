---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/supported-cameras
---

# 支持的摄像头

Chloros 可处理来自两个 MAPIR 摄像头系列的图像，适用于 **所有平台** （Windows、Linux amd64 以及 Linux arm64/Jetson）：

* **Survey3** — Survey3W（广角）和 Survey3N（窄角）相机。 输入：`RAW+JPG`。
* **LATTICE**— M3C 和 M3M 多光谱相机模块。输入：`.tif`/`.tiff` 拍摄的图像。 LATTICE 相机还可通过 Chloros**实时控制**——通过 GUI 的“相机”选项卡（Windows）或 `chloros-cli lattice` / Python SDK（Windows 和 Linux）——包括同步的多摄像头阵列。 请参阅 [LATTICE 指南](lattice/)。

该处理管道还支持 `.dng` 输入文件。

## Survey3

<table data-header-hidden><thead><tr><th width="156">制造商</th><th width="250">相机型号</th><th width="138">滤波器型号</th><th width="187">图像类型</th></tr></thead><tbody><tr><td><strong>制造商</strong></td><td><strong>相机型号</strong></td><td><strong>滤镜型号</strong></td><td><strong>图像类型</strong></td></tr><tr><td>MAPIR</td><td>Survey3W、Survey3N</td><td>RGB</td><td>RAW+JPG、JPG</td></tr><tr><td>MAPIR</td><td>Survey3W、Survey3N</td><td>RGN</td><td>RAW+JPG、JPG</td></tr><tr><td>MAPIR</td><td>Survey3W、Survey3N</td><td>OCN</td><td>RAW+JPG、JPG</td></tr><tr><td>MAPIR</td><td>Survey3W、Survey3N</td><td>NGB</td><td>RAW+JPG、JPG</td></tr><tr><td>MAPIR</td><td>Survey3W、Survey3N</td><td>RE</td><td>RAW+JPG、JPG</td></tr><tr><td>MAPIR</td><td>Survey3W、Survey3N</td><td>NIR</td><td>RAW+JPG、JPG</td></tr></tbody></table>## LATTICE

LATTICE 系列是一款基于索尼 IMX265 全局快门传感器（3.1 MP，3.45 µm 像素）构建的模块化多光谱相机系统。每台相机都会将其标识以模型字符串的形式存储：

```
<sensor>-<lens>-F<filter>        e.g.  M3C-L41-FRGN,  M3M-L87-F550
```

Chloros 会以 `LATT-` 为前缀显示该标识（例如 `LATT-M3M-L41-F550`），而该型号字符串将驱动后续所有环节——传感器配置文件、波段布局和校准均会自动完成； 无需针对每台相机进行单独配置。镜头编号代表**水平视场角（单位：度）**：`L41` = 窄视场 41°，`L87` = 广视场 87°。

共有两种传感器配置：

| 配置 | 传感器      | 滤光片类型                           | 每台摄像头的波段数                                                        |
| ------------- | ----------- | ------------------------------------- | ----------------------------------------------------------------------- |
| **M3C**       | 拜耳彩色 | 三波段通带                       | 单次曝光可获得 3 个光谱波段                                 |
| **M3M**       | 单色        | 单窄带干涉滤光片                           | 1 个校准波段 — 组合多台 M3M 相机以获取植被指数                                 |

### M3C（拜耳）滤光片选项

| 滤光片 | 波段（名称 @ 中心波长 nm / FWHM nm）       |
| ------ | ---------------------------------------- |
| `FRGB` | Blue 475/30 · Green 550/30 · Red 625/30  |
| `FRGN` | Red 660/21 · Green 550/30 · NIR 850/30   |
| `FOCN` | Orange 615/21 · Cyan 490/38 · NIR 808/14 |
| `FNGB` | Blue 475/30 · Green 550/30 · NIR 850/30  |

### M3M（单色）滤光片产品目录 — 23 个 SKU

F值即为SKU标签；测定波段（压印在每件经校准的出口产品上）是每批滤光片的扫描数据：

| SKU    | 中心波长（nm，测定值） | FWHM 边缘波长（nm） | 带宽（nm） |
| ------ | --------------------- | --------------- | ---------- |
| F385   | 379.4                 | 367–392         | 25         |
| F405   | 403.9                 | 390–417         | 27         |
| F450   | 443.7                 | 430–458         | 28         |
| F485   | 489.7                 | 478–502         | 24         |
| F520   | 519.9                 | 504–536         | 32         |
| F550   | 548.4                 | 531–566         | 35         |
| F590   | 589.0                 | 570–608         | 38         |
| F615   | 623.8                 | 614–634         | 20         |
| F632   | 633.4                 | 616–651         | 35         |
| F650   | 651.1                 | 636–666         | 30         |
| F685   | 686.2                 | 675–698         | 23         |
| F715   | —（名义）           | 706–724         | 18         |
| F725   | 725.2                 | 712–738         | 26         |
| F750   | 746.0                 | 729–763         | 34         |
| F780   | 775.1                 | 754–796         | 42         |
| F808   | 810.3                 | 789–832         | 43         |
| F832   | 826.1                 | 810–843         | 33         |
| F850   | 846.5                 | 828–865         | 37         |
| F880   | —（名义值）           | 867–893         | 26         |
| F905   | —（名义值）           | 892–920         | 28         |
| F940   | 940.6                 | 923–958         | 35         |
| F950   | 945.1                 | 929–961         | 32         |
| F988 † | 985.3                 | 968–1003        | 35         |

_“带边是根据MAPIR的每批次滤光片扫描测得的半高全宽值——与Chloros在每次校准导出中标注的数值相同。”_ “—（标称值）”表示尚未进行批次扫描；对于此类SKU，所列中心波长即为SKU编号，宽度则采用制造商提供的数值。

† “F988反射率使用场景内反射率面板进行校准：该波段超出了DAQ光传感器的校准范围，因此Chloros会应用您最近的面板捕获数据，并在两次面板观测之间保持该值。” 参见 [校准靶标](calibration-targets.md)。

有关实时摄像头控制、阵列、网络设置以及辐射测量处理链，请参阅 [LATTICE 指南](lattice/)。
