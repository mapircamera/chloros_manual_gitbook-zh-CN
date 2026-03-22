# 图形用户界面：导航

首次启动 Chloros 和 Chloros（浏览器）时，系统将启动其后端。准备就绪后，左上角的主菜单图标将显示出来 <img src=".gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

顶部标题栏从左至右包含：

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> 主菜单

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

通过主菜单，您可以：

* **新建项目** — 创建新项目
* **打开项目** — 打开现有项目
* **打开项目文件夹** — 在文件资源管理器中打开项目文件夹
* **添加文件** — 将单个图像文件添加到当前项目中 _(项目打开后可见)_
* **添加文件夹** — 将图像文件夹添加到当前项目中 _(项目打开后可见)_
* **开始处理 / 停止处理** — 启动或停止图像处理流程 _(添加文件后启用)_

{% hint style="info" %}
**仅限 Windows**：Chloros 桌面图形界面可在 Windows 上使用。Linux 用户应查看 [CLI](CLI.md) 及 [Python SDK](api-python-sdk.md) 文档以了解无头处理。
{% endhint %}

### <img src=".gitbook/assets/image (2) (1).png" alt="" data-size="line"> 播放/开始按钮

启用后，开始处理按钮将启动图像处理管道。

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> 进度条 <img src=".gitbook/assets/image (5).png" alt="" data-size="line">在免费的 Chloros 模式下（该模式按顺序处理所有文件），进度条将显示两个阶段：目标检测和处理。

在付费的 Chloros+ 授权模式下（该模式同时处理所有文件），进度条将显示四个阶段：检测、分析、校准、导出。 若将鼠标光标悬停在 Chloros+ 进度条上，将展开扩展的 4 阶段进度条面板以便您实时跟踪进度。点击顶部的进度条可锁定下拉面板，再次点击则解除锁定。

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## 侧边菜单

左侧边栏菜单包含多个交互图标：

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [项目设置](project-settings/project-settings.md)

“项目设置”选项卡允许您调整项目全局设置和项目处理设置。请在开始处理文件前调整这些设置。

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> 文件浏览器

向项目中添加文件/文件夹或移除文件。系统将忽略重复文件。勾选目标图像对应的“目标”列复选框后，处理过程仅会针对已勾选的图像进行目标检索，从而大幅加快处理速度。使用“图像/元数据”切换按钮，可在查看所选图像的缩略图网格与详细元数据表格之间切换。

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [图像查看器](image-viewer-gui/opening-an-image-full-screen.md)

在主图像查看器中点击某张图像时，该图像将在“图像查看器”选项卡中全屏打开。

#### <img src=".gitbook/assets/image (7).png" alt="" data-size="line"> [地图](image-viewer-gui/map-markers.md)

基于 GPS 坐标，在交互式 2D 地图上查看您的图像。支持 Google 地图和 ESRI 瓦片提供商，并会根据您的位置自动选择最佳服务。将鼠标悬停在标记上即可查看图像缩略图预览。

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> 调试日志

发生问题时，请查看日志中的调试输出。复制/下载日志并发送至 [MAPIR 支持中心](https://www.mapir.camera/community/contact) 以获取帮助。

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [用户登录](chloros+-login.md)

通过用户登录侧边栏，您可以登录您的 Chloros+ 账户以解锁高级功能。您还可以查看当前应用程序版本，并调整 Chloros 图形用户界面 (GUI) 及 CLI 中显示文本的语言。
