---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# 常见问题

<details>

<summary>能否使用Chloros处理非MAPIR品牌的相机图像？</summary>

不行，Chloros仅支持处理MAPIR相机图像。 更多信息请参阅[支持的相机型号列表](supported-cameras.md)。我们确实提供MAPIR云端对其他相机的处理服务，完整列表请见[此处](https://mapir.gitbook.io/mapir-cloud/supported-cameras)。

</details>

<details>

<summary>能否在没有校准目标的情况下对图像进行反射率校准？</summary>

不行。若未在拍摄非校准图像时同步采集校准靶标图像，则无法将图像像素值与已知反射率百分比建立关联。若同时未包含MAPIR光传感器日志，则环境光谱将无法测得，反射率结果亦不准确。

</details>

<details>

<summary>能否在Chloros处理前编辑图像？</summary>

不可以。Chloros假定输入数据未经修改。请勿更改文件名。

</details>

<details>

<summary>能否将MAPIR相机设置为自动曝光后在Chloros中处理图像？</summary>

不可以。Survey3图像数据集必须采用固定/锁定曝光参数，禁止使用自动快门速度或自动ISO设置。同一型号相机的所有图像必须保持完全一致的快门速度和ISO（曝光）参数。

</details>

<details>

<summary>Chloros能否处理或分析正射镶嵌图像？</summary>

不支持。仅支持单张MAPIR相机图像，不支持正射镶嵌图等拼接图像。

</details>

<details>

<summary>如何加快Chloros的目标检测步骤？</summary>

在文件浏览器表格中，预先在右侧列中选择目标图像，Chloros 就会只从这些图像中寻找校准目标，大大加快处理速度。

</details>

<details>

<summary>若计划将图像上传至<a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">XPROTX云平台</a>，是否需要在XPROTX中预先处理？</summary>

若计划上传至我们的在线处理平台[MAPIR云](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription)，请勿在上传前编辑图像。云平台将执行全部相同处理流程并提供更多功能。

</details>

<details>

<summary>MAPIR是否会支持X功能？我非常希望MAPIR能提供X功能。</summary>

我们始终重视产品反馈。若您发现产品问题或有改进建议，请[联系我们](https://www.mapir.camera/community/contact)分享您的想法。我们的研发工作主要依据客户的核心需求进行指导。

</details>
