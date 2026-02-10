---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# 下载

下载最新版本的Chloros，开始进行多光谱图像处理。

### 系统要求

| 要求          | 最低                                                         | 推荐                                                         |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **操作系统** | Windows 10 (64位)                                  | Windows 11 (64位)                                  |
| **处理器**        | Intel Core i5 或同等性能                          | Intel Core i7 或更高性能                              |
| **内存（RAM）**     | 8GB                                                  | 16GB 或更高                                         |
| **显卡**    | DirectX 11 兼容                                | 配备 4GB+ 显存的 NVIDIA GPU                            |
| **存储空间**          | 6GB 可用空间                                       | 配备 10GB+ 可用空间的 SSD                            |
| **显示器**          | 1920x1080                                            | 2560x1440 或更高分辨率                                  |
| **网络连接**         | 需联网激活\[可选]Chloros+许可证 | 需联网激活\[可选]Chloros+许可证 |

{% hint style="info" %}
**GPU加速**：配备NVIDIA显卡的Chloros+用户可启用CUDA加速实现显著提速。Chloros+用户还可获得多线程处理能力以达到最高速度。
{% endhint %}

***

## 下载 Chloros

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">在此下载 Chloros</a>

### 最新稳定版

**Chloros 安装程序适用于 Windows*** **版本**：1.0.5
* **发布日期**：2026年2月10日
* **文件大小（下载）**：1.6GB
* **安装后文件大小**：5.7GB
* **文件类型**：.exe（Windows安装程序）

#### **安装步骤：**

1. 下载`CHLOROS INSTALLER - CURRENT VERSION.exe`文件
2. 双击安装程序启动安装
3. 跟随安装向导提示操作
4. 选择安装目录（默认：`C:\Program Files\[USER]\Chloros\`）
5. 安装完成后启动Chloros或Chloros CLI
6. 使用您的[MAPIR Cloud Chloros+账户](https://cloud.mapir.camera/pricing)登录（或继续使用免费版）

{% hint style="success" %}
安装程序会自动将`chloros-cli`添加至系统PATH环境变量，以便命令行访问。
{% endhint %}

***

## 扩展资源

### 面向开发者与自动化工作流：安装Chloros Python SDK

面向开发者及自动化工作流，请安装 Chloros Python SDK：

```bash
pip install chloros-sdk
```

**文档**：[API: Python SDK](api-python-sdk.md)**要求**：需安装Chloros桌面版，需Chloros+许可证登录***

## 包含内容

Chloros安装包包含：

* ✅ **Chloros** - 全功能图形界面（GUI）
* ✅ **Chloros CLI** - 命令行界面（需Chloros+许可证）
* ✅ **Chloros SDK** - Python API（需Chloros+许可证）
* ✅ **相机配置文件** - 预配置的MAPIR相机模板***

## 升级至Chloros+

通过Chloros+订阅解锁高级功能：

* 🚀 **多线程处理** - 并行处理图像
* ⚡ **GPU（CUDA）加速** - 释放NVIDIA GPU性能
* 💻 **CLI访问** - 通过命令行工具实现自动化
* 🐍 **Python SDK** - 程序化API访问
* 📱 **多设备支持** - 可在2-10+台设备上使用 (取决于套餐)
* **🐻 高级纹理感知去拜耳化算法** - 高质量边缘感知去拜耳化技术结合AI/ML降噪模型，可消除几乎所有去拜耳化噪声
* 🧮 **自定义公式** - 创建多光谱自定义指数

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">查看 Chloros+ 方案与定价</a></p>***

## 安装帮助

### 故障排除

**安装失败并显示错误信息：**

* 确保拥有管理员权限
* 临时禁用杀毒软件
* 确认满足最低系统要求

**应用程序无法启动：**

* 确认已安装Windows 10/11 (64位)
* 更新显卡驱动程序
* 通过Windows事件查看器检查错误详情
* 附带错误日志联系技术支持

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

<summary>版本 1.0.5</summary>

#### **发布日期**：2026年2月10日**新增功能*** **纹理感知去拜耳化算法 \[仅限Chloros+版本] -** 该算法结合高精度边缘感知去拜耳化技术与AI/ML降噪模型，可消除几乎所有去拜耳化噪声。
* **支持T4P校准靶标*** **Chloros+ GPU处理加速，内存管理优化**

**错误修复*** 全新前端界面（GUI），现已适配所有Windows计算机。

</details>

<details>

<summary>版本 1.0.4</summary>

#### **发布日期**：2026年1月5日**新增功能*** **图像/元数据切换**：文件浏览器新增切换按钮，可将选定图像的元数据以表格形式替代网格图展示
* **图像网格缩放滑块**：新增UI滑块调节缩略图尺寸 （同时支持Ctrl+滚轮操作）
* **图像网格导出按钮**：顶部工具栏新增按钮，可将缩略图格式切换为处理后导出格式（目标值、反射率、指数、LUT）
* **地图标签页**：新增交互式2D地图，显示图像GPS定位标记
  * 支持Google地图与ESRI地图瓦片（根据缩放级别自动选择最佳瓦片服务）
  * 鼠标悬停地图标记可预览缩略图

**错误修复*** 改进非英语系统安装Chloros的支持

</details>

<details>

<summary>版本 1.0.3</summary>

#### **发布日期**：2025年12月20日**新增功能*** 初始版本

**改进*** 初始版本

**错误修复*** 初始版本

**已知问题*** 初始版本

</details>***

## 许可协议**专有软件** - 版权所有 (c) 2026 MAPIR 公司

禁止未经授权的使用、分发或修改。

**免费版**：支持个人及商业用途，功能有限制**Chloros+**：订阅制许可，提供高级功能及商业部署支持
