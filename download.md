# 下载

下载最新版本的Chloros，开始进行多光谱图像处理。

### 系统要求

| 要求                        | 最低要求                        | 推荐要求                     |
| -------------------- | ------------------------------- | ------------------------------- |
| **操作系统** | Windows 10 (64位)             | Windows 11 (64位)             |
| **处理器**        | 英特尔酷睿i5或同等性能     | 英特尔酷睿i7或更高性能         |
| **内存 (RAM)**     | 8GB                             | 16GB 或更高                    |
| **显卡**    | DirectX 11 兼容           | NVIDIA GPU 配备 4GB+ 显存       |
| **存储空间**          | 6GB 可用空间                  | SSD 配备 10GB+ 可用空间       |
| **显示器**          | 1920x1080                       | 2560x1440 或更高分辨率             |
| **网络连接**         | 许可证激活必需 | 许可证激活必需 |

{% 提示 style=&quot;info&quot; %}
**GPU加速**：Chloros+用户若配备NVIDIA显卡（4GB+显存），可通过CUDA加速实现显著提升的处理速度。Chloros+用户还可获得多线程处理能力以实现最高运行速度。
{% endhint %}

***

## 下载 Chloros

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">点击此处下载Chloros</a>

### 最新稳定版

**Chloros安装程序适用于Windows*** **版本**：1.0.4
* **发布日期**：2026年1月5日
* **文件大小（下载）**：1.8GB
* **文件大小（安装后）**：5.7GB
* **文件类型**：.exe（Windows安装程序）

#### **安装步骤：**

1. 下载 `CHLOROS INSTALLER - CURRENT VERSION.exe` 文件
2. 双击安装程序启动安装
3. 按照安装向导提示操作
4. 选择安装目录（默认：`C:\Program Files\[USER]\Chloros\`）
5. 安装完成后启动 Chloros、Chloros（浏览器）或 Chloros CLI
6. 使用您的[MAPIR云Chloros+账户](https://cloud.mapir.camera/pricing)登录（或继续使用免费版）

提示 style=&quot;success&quot; %}
安装程序会自动将`chloros-cli`添加至系统PATH环境变量，便于命令行访问。
结束提示 %}

***

## 扩展资源

### Python SDK

面向开发者及自动化工作流，请安装 Chloros Python SDK：

```bash
pip install chloros-sdk
```

**文档**：[API: Python SDK](api-python-sdk.md)**要求**：需安装Chloros桌面版，需Chloros+许可证登录***

## 包含内容

Chloros安装包包含：

* ✅ **Chloros** - 全功能图形界面
* ✅ **Chloros (浏览器版)** - 适用于低配置系统的网页界面
* ✅ **Chloros CLI** - 命令行界面（需Chloros+许可证）
* ✅ **Chloros SDK** - Python API（需Chloros+许可证）
* ✅ **相机配置文件** - 预配置MAPIR相机模板***

## 升级至Chloros+

通过Chloros+订阅解锁高级功能：

* 🚀 **多线程处理** - 并行处理图像
* ⚡ **GPU（CUDA）加速** - 发挥NVIDIA GPU性能
* 💻 **CLI 访问** - 通过命令行工具实现自动化
* 🐍 **Python SDK** - 程序化 API 访问
* 📱 **多设备支持** - 适用于2-10+台设备（取决于套餐）
* 🧮 **自定义公式** - 创建多光谱自定义指数

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">查看 Chloros+ 方案与定价</a></p>***

## 安装帮助

### 故障排除

**安装失败并显示错误信息：**

* 确保拥有管理员权限
* 暂时禁用杀毒软件
* 确认满足最低系统要求

**应用程序无法启动：**

* 尝试使用Chloros（浏览器）版本
* 确认已安装Windows 10/11（64位）
* 更新显卡驱动程序
* 通过Windows事件查看器检查错误详情
* 附上错误日志联系技术支持

**许可证激活问题：**

* 确保网络连接正常
* 在[https://cloud.mapir.camera](https://cloud.mapir.camera)验证凭证
* 检查防火墙是否阻断Chloros
* 详见[Chloros+登录](chloros+-login.md)操作指南

### 获取支持

需要安装或配置帮助？

* 📧 **邮件**：info@mapir.camera
* 🌐 **官网**：[https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **文档**：[入门指南](./)
* ❓ **常见问题**：[常见问题解答](faq.md)***

## 变更日志

<details>

<summary>版本 1.0.4</summary>

#### **发布日期**：2026年1月5日**新增功能*** **图像/元数据切换**：文件浏览器新增切换功能，可将选定图像的元数据以表格形式显示（替代图像网格）
* **图像网格缩放滑块**：新增UI滑块调节缩略图尺寸 （同时支持Ctrl+滚轮操作）
* **图像网格导出按钮**：顶部工具栏新增按钮，可切换缩略图格式为JPG或处理后导出格式（目标值、反射率、指数、LUT）
* **地图标签页**：新增交互式2D地图，显示图像GPS定位标记
  * 支持Google地图与ESRI地图瓦片（根据缩放级别自动选择最佳瓦片服务）
  * 鼠标悬停地图标记可预览缩略图

**错误修复*** 改进非英语系统安装Chloros的支持

</details>

<details>

<summary>版本 1.0.3</summary>

#### **发布日期**：2025年12月20日**新增功能*** 初始版本

**优化改进*** 初始版本

**错误修复*** 初始版本

**已知问题*** 初始版本

</details>***

## 许可协议**专有软件** - 版权所有 (c) 2025 MAPIR 公司

禁止未经授权的使用、分发或修改。

**免费版**：可用于个人及商业用途，功能有限制**Chloros+**：订阅制许可，提供高级功能及商业部署支持
