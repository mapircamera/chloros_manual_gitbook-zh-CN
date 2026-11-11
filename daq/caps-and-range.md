# 探头轮廓与校准范围

> 关于盖板本身——即哪款盖板与哪种传感器配套、安装方式及其光学特性——已在**[DAQ用户手册](https://mapir.gitbook.io/daq)**中详细说明。本页介绍如何向Chloros *声明*所安装的盖板，这正是确保校正准确的关键。

每台 DAQ 光传感器的出厂辐射校准数据描述的都是*裸露*的传感器。安装在扩散板上的物理盖子会改变传感器收集的光线，因此 Chloros 会在校准数据包的基础上应用出厂测得的 **盖子校正曲线**。 声明正确的盖子是获得校准数据的一部分——本页面介绍了各型号对应的盖子种类、声明方法，以及传感器实际的校准光谱范围。

## 各型号的盖板可用性

| 盖板配置文件 (`cap_id`) | 物理盖板 | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | 阳光余弦校正盖（**所有型号默认配备**） | 是 | 是 | 是 |
| `fov_15` / `fov_45` / `fov_90` | 视场限制锥（15° / 45° / 90°） | 是 | — | 是 |
| `fov_30` / `fov_60` | 视场限制锥（30° / 60°） | 是 | — | — |
| `none` | 未安装盖帽 | — | — | 是 |

型号特定说明：

* **DAQ-M 仅有一种盖帽轮廓：`sunshine_cosine`。** 其产品定义为“裸机+Sunshine盖”，而裸机版DAQ-M无需几何轮廓。
* **裸机版DAQ-U是真正的“裸机”** ——它完全不需要几何轮廓，因此不存在针对它的`none`轮廓。
* **DAQ-E 上的 `none` 并非无操作。** DAQ-E 的内嵌式玻璃覆盖扩散器本身具有实际的几何校正，因此“无顶盖”状态本身就是该型号的一种测量配置。
* **未加装遮光罩的 DAQ-E 无法测量任何仰角的直射阳光** —— 阳光遮光罩是现场配置的必要组件。请勿计划使用未加装遮光罩的 DAQ-E 进行户外工作。

在图形用户界面（GUI）的单传感器设置中（“光传感器”选项卡中的齿轮图标），**盖帽**下拉菜单在 DAQ-U 和 DAQ-M 上也提供“无（裸传感器）”选项——根据上述说明，在这两款型号上，“裸”仅表示不应用盖帽校正。 仅当物理上拆下遮光罩时才应选择此选项。

## 声明遮光罩——及其重要性

**声明的 `cap_id` 必须与传感器上实际安装的遮光罩相匹配。** 无论是传感器还是软件，均无法检测到已安装的盖子。该声明决定以下两点：

1. 应用于每个光谱的**实时校正**。
2. **写入每个 `.daq` 记录中的盖子标记**，后续反射率处理会以此为依据。

Sunshine 盖的设计衰减系数约为 **12×**，因此若声明的盖型号错误，记录的光谱会按此倍数出现比例失真。请立即声明盖的变更。

### 设置盖

GUI：光传感器选项卡 → 传感器行上的齿轮图标 → **盖** 下拉菜单。 所有型号的默认值均为 `sunshine_cosine`，这是一种默认假设（DAQ-U、DAQ-M 和 DAQ-E 出厂时未配置盖帽，且记录中会显示“默认使用盖帽”直至您选择具体型号），该选择将随项目保留。



CLI<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->
（后端必须正在运行）：

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

CLI 在语法上接受完整的 `cap_id` 列表（例如 `{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`）； 每个配置文件在连接时都会根据传感器模型进行验证，因此如果 cap ID 不可用（例如在 DAQ-U 上使用仅限 E 型的 ID），系统会明确报错而非进行错误校正。若未传入参数，后端的默认值为 `sunshine_cosine`。

Python SDK：在连接时使用 `connect_daq_sensor(cap_id="sunshine_cosine")` 为 cap 命名，或在运行时使用 `DAQSensorSession.set_cap("fov_45")` 进行替换；请参阅 [SDK 参考](../reference/sdk-reference.md#daq-sensor-sessions)。

高级：配置文件随Chloros 安装包内置于 `daq/cap_profiles/<u|m|e>/<cap_id>.json` 中，用户可通过 `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json` 进行覆盖。

除了上限限制外，从未重新校准过的传感器会自动获得基于设备群的小幅暗偏移优化——无需用户操作。

## DAQ-E-S：固定扩散板和倾斜角

**DAQ-E-S** 是一款 DAQ-E，但有两处对 Chloros 至关重要的差异。其余所有功能（发现、连接、PTP、记录、设备设置）均与 DAQ-E 完全一致。

* **其漫射器为固定式，因此无需进行漫射器校正。** 在进行出厂校准测量时，该扩散器已安装在设备上，因此该校准值已准确反映真实的向下辐射通量。无论系统如何要求，Chloros 均不会对其应用任何上限曲线：以该设备命名的上限曲线会被忽略并显示提示信息，其记录数据会显示该上限曲线来自模型，且针对该设备的“假设上限”警告绝不会触发。 在“项目设置”→“DAQ 光传感器”中，该设备的行显示无上限值，且项目范围内的上限值覆盖设置对其无效。请勿手动对其数据应用上限值：否则，叠加的日照校正值将高出约 11 倍。
* **该设备配备了一个三轴加速度计。** 该扩散器的余弦响应仅在水平状态下有效，因此倾斜的传感器会报告错误的辐照度，且无其他异常症状。因此，每个 DAQ-E-S 读数都包含其俯仰角、横滚角和偏航角，而从该传感器记录的 `.daq` 文件会将这些数据存储在每个光谱旁边 (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, …) 旁边，因此您可以区分是云层还是支架晃动造成的。

有关每个姿态字段以及姿态数据流中故障的报告方式，请参阅 [CLI 参考中的 DAQ-E-S](../reference/cli-reference.md#daq-e-s--fixed-diffuser-and-tilt-sensor)。

## Sunshine cap 性能（户外配置）

可用于制定操作流程的参数：

| 属性 | 值 |
| --- | --- |
| 视场 | 180° 半球形 |
| 余弦响应误差 | 入射角在 60° 以内时 ≤ ±4%；入射角在 70° 以内时 ≤ ±4.5% |
| 低日照限值 | 不建议在太阳高度角低于 ~15° 时使用 |
| 衰减 | ~12×（按设计） |
| 遮阳罩重新安装重复性 | ≈ 1.5 % |
| 定量辐照度 | 取**≥ 15 秒**的平均读数（仪器特性，非缺陷） |

对于任何定量辐照度数值（包括反射率参考值），请使用至少 15 秒读数的平均值，而非单帧数据。

## 校准光谱范围

| 属性 | 值 |
| --- | --- |
| 光谱采样 | 340–1010 nm，步长 5 nm（135 个点） |
| 辐射校准范围 | **~374–974 nm**（由软件强制执行） |

传感器报告完整的 340–1010 nm 网格数据，但可追溯至 NIST 的辐射增益范围为 ~374–974 nm。 对于任何光谱权重不足一半位于该范围内的相机波段，Chloros **将拒绝采用绝对反射率划分**，并报告跳过原因 `dls-uncalibrated-band-<nm>`，而非生成未经校准的产品。 在已上市的相机型号中，仅 F988 滤光片超出此范围；该型号改用反射率面板工作流——详见 [反射率工作流](reflectance.md)。

有关传感器型号、传输方式和传感器 ID，请参阅 [DAQ 概述](README.md)。关于处理过程中 cap 标记的使用方式，请参阅 [记录与 .daq 格式](recording.md)。
