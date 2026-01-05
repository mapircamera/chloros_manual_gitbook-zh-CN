# 图形用户界面：导航

首次启动 Chloros 和 Chloros（浏览器）时，其后端将开始启动。准备就绪后，左上角的主菜单图标将显示出来 <img src=".gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

顶部标题栏从左至右包含：

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> 主菜单

通过主菜单可新建项目、打开现有项目或访问项目文件夹。

### <img src=".gitbook/assets/image (2) (1).png" alt="" data-size="line"> 播放/启动按钮

启用后，此按钮将启动图像处理流水线。

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> 进度条 <img src=".gitbook/assets/image (5).png" alt="" data-size="line">在免费Chloros模式（顺序处理所有文件）下，进度条显示两个阶段：目标检测与处理。

在付费Chloros+授权模式（并行处理所有文件）下，进度条显示四个阶段：检测、分析、校准、导出。 将鼠标悬停在进度条上将展开扩展面板以便实时跟踪。点击顶部进度条可冻结下拉面板，再次点击则解除冻结。

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## 侧边菜单

左侧边栏菜单包含多种交互图标：

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [项目设置](project-settings/project-settings.md)

项目设置选项卡可调整项目全局参数及处理设置。请在开始文件处理前完成这些配置。

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> 文件浏览器

添加/移除项目中的文件/文件夹。重复文件将被忽略。勾选目标列中的图像，处理程序仅针对已选目标图像执行操作，可大幅提升处理效率。通过图像/元数据切换按钮，可在选定图像的缩略图网格与详细元数据表格视图间切换。

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [图像查看器](image-viewer-gui/opening-an-image-full-screen.md)

在主图像查看器中点击图像时，该图像将在图像查看器标签页中全屏打开。

#### <img src=".gitbook/assets/image (7).png" alt="" data-size="line"> [地图](image-viewer-gui/map-markers.md)

基于GPS坐标在交互式2D地图上查看图像。支持Google Maps与ESRI图块服务商，自动选择最优地图服务。悬停标记可查看图像缩略图预览。

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> 调试日志

问题发生时可查阅调试输出日志。复制/下载日志并发送至[MAPIR支持](https://www.mapir.camera/community/contact)获取协助。

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [用户登录](chloros+-login.md)

用户登录侧边栏可用于登录您的Chloros+账户以解锁高级功能。您还可在此查看当前应用程序版本，并调整Chloros图形界面及CLI中显示文本的语言设置。
