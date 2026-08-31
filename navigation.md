# 图形用户界面：导航

首次启动 Chloros 时，程序会启动其处理后端。后端准备就绪后，左上角的主菜单图标 <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> 将会显示出来，且左侧边栏中的“摄像头”和“光传感器”选项卡将解锁（在此之前，这些选项卡处于灰色不可用状态）。

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

顶部标题栏从左至右包含：

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> 主菜单

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

通过主菜单，您可以：

* **新建项目**— 创建新项目。如果您已保存项目模板，将出现一个**选择模板** 下拉菜单，以便新项目基于模板的设置开始。
* **打开项目**— 打开现有项目。 该列表中包含一个**打开项目文件夹** 按钮，点击后将在文件资源管理器中打开项目文件夹。
* **复制项目** — 将当前打开的项目复制为新名称（建议使用“MyProject (2)”等自由命名的名称），并打开该副本。 _(项目打开后可见)_
* **添加文件** — 将单个图像文件添加到当前项目中 _(项目打开后可见)_
* **添加文件夹** — 将一个或多个包含图像的文件夹添加到当前项目中 _(项目打开后可见)_
* **开始处理 / 停止处理** — 启动或停止图像处理流程 _(在添加文件后可用)_
* **连接摄像头** — 跳转至 [“摄像头”选项卡](lattice/) 以连接 LATTICE 摄像头或阵列。无需打开项目即可使用。
* **连接光传感器** — 跳转至 [光传感器选项卡](daq/) 以连接 DAQ 光传感器。无需打开项目即可使用。

{% hint style="info" %}
**仅限 Windows**：Chloros 桌面图形用户界面可在 Windows 上使用。 Linux 用户请参阅 [CLI](CLI.md) 以及 [Python SDK](api-python-sdk.md) 文档以了解无头处理的相关信息。
{% endhint %}

###<img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">

播放/开始按钮

启用后，开始处理按钮将启动图像处理管道。

###<img src=".gitbook/assets/image (4).png" alt="" data-size="line">

进度条<img src=".gitbook/assets/image (5).png" alt="" data-size="line">

在免费模式 Chloros 中，系统会依次处理所有文件，进度条将显示两个阶段：目标检测和处理。

在付费的 Chloros+ 授权模式下，该模式会同时处理所有文件，进度条将显示 4 个阶段：检测、分析、校准、导出。 将鼠标光标悬停在 Chloros+ 进度条上，会下拉显示扩展的 4 个进度条面板，以便您跟踪进度。点击顶部的进度条将冻结下拉面板，再次点击则解除冻结。

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## 侧边菜单

左侧边栏菜单包含多个交互图标，从上到下的顺序如下：

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [项目设置](project-settings/project-settings.md)

“项目设置”选项卡允许您调整项目的全局设置和处理设置。 请在开始处理文件之前调整这些设置。

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> 文件浏览器

向项目中添加文件/文件夹，或从项目中移除文件。系统会忽略重复文件。勾选目标列中任意目标图像的复选框后，处理过程将仅针对已勾选的图像进行目标匹配，从而大幅加快处理速度。 使用“图像/元数据”切换按钮，可在查看所选图像的缩略图网格和详细的元数据表格之间切换。

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [图像查看器](image-viewer-gui/opening-an-image-full-screen.md)

在主图像查看器中点击某张图像后，该图像将在“图像查看器”选项卡中以全屏模式打开。

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [地图查看器](image-viewer-gui/map-markers.md)

根据图像的 GPS 坐标，在交互式 2D 地图上查看您的图像。 支持 Google Maps 和 ESRI 图块提供商，会根据您的位置自动选择最佳服务。将鼠标悬停在标记上即可查看图像缩略图预览。

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [摄像头](lattice/)

实时连接并控制 LATTICE 摄像头——既可单个控制，也可作为同步的多摄像头阵列进行控制。 该选项卡显示带有叠加层和直方图的实时预览图块、单摄像头和阵列设置，以及“捕获设置”——用于选择“捕获全部”功能将处理哪些摄像头和导出类型。后端准备就绪后即可使用；请参阅 [LATTICE 部分](lattice/) 以获取完整操作指南。

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [光传感器](daq/)

连接 DAQ 光传感器——DAQ-U（USB）、DAQ-M（蓝牙）和 DAQ-E（以太网）——并在 W/m²/nm 单位下查看其实时校准光谱图。 在此您可以将 `.daq` 文件记录到打开的项目中、重命名传感器、选择盖帽校正配置文件，以及更新 DAQ-E 固件。待后端准备就绪后即可使用；请参阅 [DAQ 部分](daq/) 以获取完整操作指南。

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> 调试日志

出现问题时，请查看日志中的调试输出。复制/下载日志并发送至 [MAPIR 支持中心](https://www.mapir.camera/community/contact) 以获取帮助。

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [用户登录](chloros+-login.md)

通过用户登录侧边栏，您可以登录您的 Chloros+ 账户以解锁高级功能。 您还可以查看当前应用程序版本，以及调整 Chloros 图形用户界面和 CLI 中显示文本的语言。
