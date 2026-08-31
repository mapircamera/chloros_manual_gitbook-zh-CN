# 单色相机与植被指数

## 一台相机 = 一个波段

**M3M**相机是拜耳**M3C**的单色版本：其后方配备了一个窄带干涉滤光片，搭载 IMX265 单色传感器。 型号字符串即代表波段名称——例如 `M3M-<lens>-F<wavelength>`、`M3M-L87-F685` （在 Chloros 中显示为 `LATT-M3M-L87-F685`）。 该传感器提供**单一灰度波段**，不采用拜耳马赛克结构：无需进行去马赛克处理，无需分离通道间的串扰，也无需设置白平衡。

在规划单色系统前，有几点后果值得了解：

* **每个波段的辐射度和反射率均已完全定义。**这些是按波段划分的辐射度图，因此一台 M3M 相机生成的校准后 float32 辐射度（W/m²/sr/nm）和 uint16 反射率（`32768` = ρ 1.0）与 M3C 波段完全一致。 单色帧携带的是**单位矩阵**（即单位响应矩阵）——无需进行3×3解混操作。
* **单个单色相机无法生成植被指数。** NDVI、NDRE 及其相关指数至少需要两个波段。若要利用单色硬件计算指数，需组合多台 M3M 相机——详见下文。
* M3M相机传输**Mono12**（12位，线速下每像素2字节），这对[阵列带宽规划](arrays.md#bandwidth-the-rules-of-thumb)至关重要。

## Chloros 在单色模式下会跳过哪些步骤——以及如何通知用户

色彩处理管道的各个阶段对于单波段传感器根本不适用。 Chloros **会通过一行提示信息跳过这些阶段** 而不是报错，并且在同一会话中对于任何 M3C（拜耳）相机仍会正常执行这些阶段：

| 阶段 | 单色 (M3M) 行为 | M3C 行为 |
| --- | --- | --- |
| 去马赛克 / 拜耳去马赛克 | 跳过 — `debayered` 的导出结果为 1 通道灰度图像。 | 3 通道去马赛克。 |
| 白平衡 (`lattice white-balance`) | 跳过，并显示一行提示信息。 | 正常运行。 |
| 色彩配置文件 (`lattice color-profile`) | 跳过，并显示一行提示信息。 | 正常运行。 |
| 饱和度/对比度 (`lattice color`) | 跳过，并显示一行提示信息。 | 运行正常。 |
| 光谱串扰解混 | 恒等矩阵（无 3×3 矩阵）。 | 应用了每台摄像机的 3×3 矩阵。 |
| 辐射度/反射率 | **运行** — 按波段，完全校准。 | 按波段运行。 |

图形用户界面（GUI）应用了相同的筛选机制：对于单色相机，每台相机的设置面板会隐藏仅适用于 RGB 的行（白平衡、伽马、色彩配置文件、饱和度、 对比度、通道分割），且实时直方图被锁定为单条**MONO**曲线。 整个堆栈过程中的区分依据是模型字符串中的 `M3M` 标记，该标记在 GUI/SDK 中显示为 `is_mono`。

## 索引需要 ≥ 2 个波段：对齐 → 堆栈 → 索引

单色索引工作流程始终遵循以下三个步骤：

1. **对准** — 将多台 M3M 相机对准不同波长（例如一台 F650 “Red” 和一台 F850 “NIR”），将其连接为 [多相机阵列](arrays.md)，并让 Chloros 计算相机间的共注册变形。
2. **堆叠** — 对齐后的帧将合并为一张多波段图像（每台摄像机贡献一个命名波段）。
3. **索引** — 对堆叠中的各波段计算索引公式，并可选通过 LUT 进行渲染。

在图形用户界面中，整个处理链即为 **组合摄像机**阵列显示模式：实时合成图像已对齐，且该阵列的索引计算器（如下所示）定义了其渲染所用的公式。通过**对齐** 捕获选项，可将捕获的导出文件变换为相同的对齐状态。

## 索引计算器

索引计算器用于编写实时预览和单相机索引导出所使用的索引表达式。它是一个共享界面，可通过“相机”选项卡侧边栏的两个位置打开：

* **单镜头**— 实时预览 →**索引** 齿轮图标（仅限 RGN/OCN/NGB 拜耳相机； 单通道相机不提供索引控制，因为单个波段无法生成索引）。
* **按阵列**— 阵列设置 → 实时预览 →**索引**齿轮图标。这是单通道相机的路径：波段列表涵盖**所有成员相机**，因此单通道相机对在此处贡献其两个波段。

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

其控制项，从上到下：

* **波段芯片**（“波段 — 点击添加到表达式”）——每个可用波段对应一个按钮，标签为颜色名称 + 波长（单位：nm）（重复的颜色名称会通过添加波长值进行区分，例如“颜色 850”）。 点击可在光标位置插入波段标记。无法生成按波段辐射度数据的相机（如 RGB/FRGB）所对应的波段将被过滤掉。
* **运算符和函数按钮** — `+ - * / ( ) ^ ,` 以及 `abs() sqrt() log() log10() exp() min() max() pow()`。
* **表达式文本区** — 自由输入公式；占位符显示经典的 NDVI 形式 `(NIR - Red) / (NIR + Red)`。其上方的只读分词预览会将波段芯片、数字和标志渲染为未知标记。
* **有效性横幅**— 灰色“为空 — 不会应用任何索引”； 绿色“有效表达式”；红色并显示具体的解析错误（未知频段、多台摄像头导致的频段歧义、缺失括号等）；或当表达式有效但为**常量**时显示琥珀色（例如 `X/X`， 或分母应为 `+` 却误输入为 `−` 的 NDVI）——常量会将整个帧映射为单一颜色。
* 如果应用的表达式没有问题，但**实时画面颜色均匀**（平淡或饱和的场景），则会显示单独的琥珀色警告——系统会自动检测到直方图坍塌。
* **应用 LUT**（默认开启；关闭 = 灰度拉伸）、**级别**2/3/5/7 档（默认 7 档），以及位于渐变条两侧的**最小值 / 最大值**输入框。 “Min”默认值为**

0.2**——该值会将色彩渐变缩放至与植被相关的范围，而低于该值的区域将作为灰度通过；将“Min”设为−1可获得完整的指数范围（**“重置”**按钮可恢复−1…+1的范围）。“Max”默认值为1。
* 指数分布的 **实时直方图** —— 采用平方根缩放的柱状图、琥珀色的第2/第98百分位线、白色中位数线，以及超出范围的尾部读数（“◀ N% &lt; lo” / “hi &lt; N% ▶”），当数值超过 1% 时会变为琥珀色，提示应扩大 Min/Max 范围。
* **应用**将表达式应用到实时流中；LUT 调整无需点击“应用”即可实时生效。表达式被设计为**仅限当前会话**——不同会话之间不会保留。

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->

## CLI 路径

相同的对齐 → 堆栈 → 索引链，可全程通过脚本控制：

```bash
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` 将预设的符号映射到堆栈的频段名称。以下两条规则可避免运行失败：

* **符号区分大小写**，且必须与预设的通道名称完全匹配——预设使用小写（NDVI 对应的通道名称应为 `red`，`nir`；请检查 `--list-presets`）。 `--channel red=Red_660` 有效；`--channel RED=660` 会因 `channel_map missing entries` 错误而失败。
* 频段一侧必须指定对齐堆栈中的一个频段（`lattice align-info --profile align.json` 列出了这些频段）。离线模式也接受以 0 为起点的频段索引，例如 `--channel red=0 --channel nir=1`。

`lattice index` 也可针对已保存的对齐多波段 TIFF 完全离线运行：

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### 索引预设

`lattice index --preset`（以及“图像”选项卡中的 [索引/LUT 沙盒](../image-viewer-gui/index-lut-sandbox.md)，其使用相同的引擎）提供了以下 **22 个预设**：

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

运行 `chloros-cli lattice index --list-presets` 可查看每个预设的公式和通道符号，运行 `--list-gradients` 可查看可用的颜色渐变。自定义公式请使用 `--formula EXPR`，其语法与“索引计算器”相同。 请注意，此预设列表专用于 LATTICE 指数引擎——导入影像时的“项目设置”处理下拉菜单中显示的是另一份列表（参见 [多光谱指数公式](../project-settings/multispectral-index-formulas.md)）。

完整的标志集（`--output-format`、`--vmin/--vmax/--percentile`、`--bg-mode`、`--live` 的对齐变形控件、 以及更多内容）在[CLI 参考手册 § 指数 / 植被数学](../reference/cli-reference.md#index--vegetation-maths)中进行了说明； SDK 的等效功能详见 [SDK 参考](../reference/sdk-reference.md)。

## 从单色数组捕获索引渲染结果

在连接数组并应用索引表达式后，`array-capture`（或 GUI 中的 **捕获全部**）会保存每台摄像机的导出级别 *以及* 索引渲染结果 ——`--index`/`--no-index`可在CLI上切换此功能，且默认捕获包含所有适用的级别。 单镜头相机对每个捕获组的贡献包括其在原始/去拜耳化（灰度）/辐射度/反射率层面的单个波段，以及当阵列以组合模式运行时共享的组合索引合成图像。 参见 [多摄像头阵列 § 采集](arrays.md#capturing-monitoring-vs-analysis)。
