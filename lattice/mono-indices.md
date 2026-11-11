# 单色相机与植被指数

## 一台相机 = 一个波段

**M3M**相机是拜耳**M3C** 的单色版本：其后方配备了一个窄带干涉滤光片，搭载 IMX265 单色传感器。 型号字符串即代表波段名称——例如 `M3M-<lens>-F<wavelength>`、`M3M-L87-F685`（在 Chloros 中显示为 `LATT-M3M-L87-F685`）。 该传感器提供**单一灰度波段**，不采用拜耳马赛克结构：无需进行去马赛克处理，无需分离通道间的串扰，也无需设置白平衡。

在规划单色系统前，需了解以下要点：

* **每个波段的辐射度和反射率均已完全定义。**这些是按波段划分的辐射度图，因此一台 M3M 相机生成的校准后 float32 辐射度（W/m²/sr/nm）和 uint16 反射率（`32768` = ρ 1.0）与 M3C 波段完全一致。 单波段帧携带一个**单位**传感器响应矩阵——无需进行3×3解混操作。
* **单个单波段相机无法生成植被指数。** NDVI、NDRE 等指数至少需要两个波段。若要利用单波段硬件计算指数，需组合多台M3M相机——详见下文。
* M3M 相机以 **Mono12** 格式进行数据流传输 （12 位：传输时为 2 字节/像素，若数组采用 [压缩 12 位传输](arrays.md#packed-12-bit-transport)，则为 1.5 字节/像素），这对 [数组带宽规划](arrays.md#bandwidth-how-many-cameras-fit) 至关重要。

## Chloros 在单色模式下会跳过哪些步骤——以及如何通知用户

色彩处理管道的各个阶段对于单波段传感器而言根本不适用。 Chloros **会通过一行提示信息跳过这些阶段**，而非报错，并且在同一会话中针对任何 M3C（拜耳）相机仍会正常执行这些阶段：

| 阶段 | 单色 (M3M) 行为 | M3C 行为 |
| --- | --- | --- |
| 去马赛克 / 拜耳去马赛克 | 跳过 — `debayered` 导出级别为 1 通道灰度图像。 | 3 通道去马赛克。 |
| 白平衡 (`lattice white-balance`) | 跳过，并显示一行提示信息。 | 正常运行。 |
| 色彩配置文件 (`lattice color-profile`) | 跳过，并显示一行提示信息。 | 正常运行。 |
| 饱和度/对比度 (`lattice color`) | 跳过，并显示一行提示信息。 | 运行正常。 |
| 光谱串扰解混 | 单位矩阵（无 3×3 矩阵）。 | 应用了每台摄像头的 3×3 矩阵。 |
| 辐射度/反射率 | **运行** — 按波段，完全校准。 | 按波段运行。 |

图形用户界面（GUI）应用了相同的门控机制：对于单色相机，每台相机的设置面板会隐藏仅RGB 的行（白平衡、伽马、色彩配置文件、饱和度、对比度、通道拆分），且实时直方图被锁定为单条**MONO**曲线。 整个堆栈过程中的区分标志是模型字符串中的 `M3M` 令牌，该令牌在 GUI/SDK 中显示为 `is_mono`。

## 索引需要 ≥ 2 个波段：对齐 → 堆栈 → 索引

单波段索引的工作流程始终包含以下三个步骤：

1. **对齐** — 将多台 M3M 相机对准不同波长（例如一台 F650 “Red” 和一台 F850 “NIR”），将其连接成一个 [多相机阵列](arrays.md)，并让Chloros 计算相机间的共注册变形量。
2. **堆叠** — 对齐后的帧将合并为一张多波段图像（每台相机贡献一个命名波段）。
3. **索引** — 对堆栈中的各波段计算索引公式，可选通过 LUT 进行渲染。

在图形用户界面中，整个处理链即为 **组合相机**阵列显示模式：实时合成图像已处于对齐状态，而阵列的索引计算器（如下所示）则定义了其渲染所用的公式。 使用**对齐** 捕获选项，可将导出的素材变换为相同的对齐状态。

## 索引计算器

索引计算器用于编写实时预览和各摄像机索引导出所使用的索引表达式。这是一个共享界面，可通过“摄像机”选项卡侧边栏的两个位置打开：

* **按相机**— 实时预览 →**索引** 齿轮图标（仅限 RGN/OCN/NGB 拜耳相机；单个单色相机没有索引控制，因为单个波段无法生成索引）。
* **按阵列**— 阵列设置 → 实时预览 →**索引**齿轮图标。这是单色通道的路径：波段列表涵盖**所有成员相机**，因此单色相机对会在此处贡献其两个波段。

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

其控件从上到下依次为：

* **波段芯片**（“波段 — 点击添加到表达式”）——每个可用波段对应一个按钮，标签为颜色名称 + 波长（单位：nm）（重复的颜色名称会通过添加波长进行区分，例如“Color 850”）。点击后会在光标位置插入波段标记。 无法生成单波段辐射度的相机（RGB/FRGB）所对应的波段会被过滤掉。
* **运算符和函数按钮** — `+ - * / ( ) ^ ,` 以及 `abs() sqrt() log() log10() exp() min() max() pow()`。
* **表达式文本区** — 自由输入公式；占位符显示经典的NDVI 形式 `(NIR - Red) / (NIR + Red)`。其上方的只读分词预览会将波段图标、数字和标志渲染为未知标记。
* **有效性提示条**— 灰色“为空 — 不会应用任何指数”；绿色“有效表达式”；红色并显示具体的解析错误（未知波段、由多台相机导致的波段歧义、缺失括号等）；或当表达式有效但包含**常量**时显示琥珀色 （例如 `X/X`，或将 NDVI 分母输入为 `−` 而非 `+`）——常量会将整个帧映射为单一颜色。
* 如果应用的表达式没有问题，但**实时帧颜色均匀**（平淡或饱和的场景），则会显示一个单独的琥珀色警告——系统会自动检测到直方图坍塌。
* **应用 LUT**（默认开启；关闭 = 灰度拉伸）、**级别**2/3/5/7 档（默认 7 档），以及位于渐变条两侧的**最小值 / 最大值**输入框。 Min 默认值为**

0.2**—— 它将色彩渐变缩放至与植被相关的范围，而低于该值的区域将作为灰度通过；将 Min 设为 −1 可获得完整的指数范围（**重置** 按钮可恢复 −1…+1 的范围）。Max 默认值为 1。
* 指数分布的 **实时直方图** —— 采用平方根缩放的柱状图、琥珀色的第 2/98 百分位线、白色中位数线，以及超出范围的尾部读数（“◀ N% < lo” / “hi < N% ▶”），当数值超过 1% 时会变为琥珀色，提示应扩大最小/最大范围。
* **应用** 将表达式应用到实时流中；LUT 调整无需点击“应用”即可实时生效。若项目已打开，表达式和 LUT 设置将随项目保存，并在重新打开项目时恢复；若无项目，这些设置仅在当前会话中有效。

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->



## CLI 路径

相同的对齐 → 堆叠 → 索引链，可编写脚本实现端到端控制：

```bash
chloros-cli lattice align-calibrate --serials SN_RED,SN_NIR --save align.json
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`align-calibrate`使用一个连接数组来存储其摄像头，或为单次运行自行打开摄像头。 `index --live` 始终自行打开相机（一个 [临时数组](../reference/cli-reference.md#multi-camera-transient-sessions)），且不包含 `--serials`：它会打开网络上的每台相机。 因此，在Chloros 释放其持有的所有摄像头之前——包括 GUI、`cam-connect`、`array-connect`（`chloros-cli lattice array-disconnect --all`）： 第二次打开已被占用的摄像头可能会中断其持有者的流，或导致后端崩溃。无法传输帧的摄像头会导致运行失败，且不会写入堆栈或索引。

`--channel` 将预设的符号映射到堆栈的波段名称。 以下两条规则可避免运行失败：

* **符号区分大小写**，且必须与预设的通道名称完全匹配——预设使用小写（NDVI 的预设为 `red`、`nir`； 请参见 `--list-presets`）。`--channel red=Red_660` 有效；`--channel RED=660` 会因 `channel_map missing entries` 错误而失败。
* 波段侧必须指定对齐堆栈中的某个波段（`lattice align-info --profile align.json`列出了这些波段）。离线模式也接受以0为起点的波段索引，例如 `--channel red=0 --channel nir=1`。

`lattice index` 也可在离线状态下针对已保存的对齐多波段 TIFF 文件运行。 所有形式均在后端运行，因此后端必须正在运行并登录到付费的 Chloros+ 层级；相对路径 `--input` 或 `--output` 将根据您执行命令的文件夹进行解析：

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### 索引预设

`lattice index --preset`（以及“图像”选项卡中的 [索引/LUT 沙盒](../image-viewer-gui/index-lut-sandbox.md)，该功能使用相同的引擎）提供了以下 **22 个预设**：

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

运行 `chloros-cli lattice index --list-presets` 可查看每个预设的公式和通道符号，运行 `--list-gradients` 可查看可用的颜色渐变。自定义公式需使用 `--formula EXPR`，其语法与“指数计算器”相同。 请注意，此预设列表专用于 LATTICE 指数引擎——导入影像时的“项目设置”处理下拉菜单中的列表则不同（参见 [多光谱指数公式](../project-settings/multispectral-index-formulas.md)）。

完整的标志集（`--output-format`、`--vmin/--vmax/--percentile`、 `--bg-mode`、用于 `--live` 的对齐变形旋钮等）已在 [CLI 参考 § 指数/植被数学](../reference/cli-reference.md#index--vegetation-maths) 中详细说明； SDK 中的对应功能详见 [SDK 参考](../reference/sdk-reference.md)。

## 从单色数组捕获索引产品

连接数组并应用索引表达式后，`array-capture`（或 GUI 中的 **捕获全部**) 可保存每台摄像机的导出层级 *以及* 索引渲染结果——`--index`/`--no-index` 可在CLI 上切换此功能，且默认捕获包含所有适用的层级。 单色摄像机对每个捕获组的贡献包括其单个波段在原始/去拜耳化（灰度）/辐射度/反射率层级的数据，以及当阵列运行在组合模式下时生成的共享组合索引合成图像。 参见 [多摄像头阵列 § 采集](arrays.md#capturing-monitoring-vs-analysis)。
