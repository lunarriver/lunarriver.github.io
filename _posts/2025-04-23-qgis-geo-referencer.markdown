---
layout: post
title:  "QGIS配置工具使用"
date:   2025-04-23 15:43:08 +0800
---

文中针对于QGIS界面及操作的描述，可能仅适用于以下QGIS版本：

![img](../../../assets/qgis-geo-referencer/1.png)

## 1 前置条件

进行配准操作前，需要安装【HCMGIS】插件，以取得底图。

![img](../../../assets/qgis-geo-referencer/2.png)

安装完成后，菜单项上，将出现【HCMGIS】，点击打开下拉框，从中选择【Google Maps】等作为底图。

![img](../../../assets/qgis-geo-referencer/3.png)

每选择一项时，【图层】中将出现一个相对应的项目，同时地图视图将渲染出对应的底图。

![img](../../../assets/qgis-geo-referencer/4.png)

在进行配准操作时，可根据原图的内容类型，选择与之对应的底图。一般而言，【Google Maps】能够满足大部分的需求，但由于其注记信息丰富，偶尔会出现注记信息遮盖关键位置的情形，此时使用其他底图可能更为方便。

## 2 操作过程

以下两个链接是配准操作的使用手册及其中文版（中文版部分翻译生硬，可与原版手册对照使用）。

- [georeferencer](https://docs.qgis.org/3.40/en/docs/user_manual/working_with_raster/georeferencer.html)

- [地理参照器](https://www.osgeo.cn/qgisdoc/docs/user_manual/working_with_raster/georeferencer.html)

### 2.1 控制点操作

（1）从图层下拉框中打开【配准工具】。

![img](../../../assets/qgis-geo-referencer/5.png)

（2）在【配准工具】中【打开栅格】，选取目标图片文件。

![img](../../../assets/qgis-geo-referencer/6.png)

选取成功后，【配准工具】将展示出所选图片。

![img](../../../assets/qgis-geo-referencer/7.png)

（3）通过逐步【添加控制点】将【图片上的位置】与【底图上的位置】对应起来。

![img](../../../assets/qgis-geo-referencer/8.png)

【添加控制点】先在图片上选取一个位置，然后【输入底图坐标】将被打开。点击【地图画布】，在地图上选取对应的位置。（点击【地图画布】后，【输入地图坐标】将最小化至屏幕的左下角）

![img](../../../assets/qgis-geo-referencer/9.png)

![img](../../../assets/qgis-geo-referencer/10.png)

地图上选取的位置将展示在【输入地图坐标】的【横(纵)坐标】输入框中。点击【OK】完成一个控制点的录入。

（4）已经添加的【控制点】可以进行【删除】和【移动】。

![img](../../../assets/qgis-geo-referencer/11.png)

![img](../../../assets/qgis-geo-referencer/12.png)

其中，【移动】既可以移动【图片上的位置】，也可以移动【底图上的位置】。

（5）在添加控制点的过程中，【地图视图】和【配准工具】均可通过【鼠标滚轮】进行缩放，均可通过【移动】（或键盘上的【上】、【下】、【左】、【右】）移动。

![img](../../../assets/qgis-geo-referencer/13.png)

### 2.2 变换设置

在录入足够的控制点后，点击【变换设置】对配准操作进行配置。

![img](../../../assets/qgis-geo-referencer/14.png)

【变换设置】需要配置的项包括：

- 【变换类型】：通常选择【薄板样条法】。

- 【目标CRS】：通常选择【EPSG:4326 - WGS 84】。

- 【输出文件】。

点击【OK】以确认配置。

![img](../../../assets/qgis-geo-referencer/15.png)

### 2.3 执行配准

点击【开始配准】以执行配准。

![img](../../../assets/qgis-geo-referencer/16.png)

配准完成后，输入的tif文件将默认被加载到【图层】中，并渲染在地图视图上。

![img](../../../assets/qgis-geo-referencer/17.png)

通常，一次配准的效果不足以令人满意，如图中右下角的位置偏移，这就意味着需要在发生位置偏移处添加更多的【控制点】，并进行再次配准。

![img](../../../assets/qgis-geo-referencer/18.png)

进行多次配准时，其结果总是输出到同一个tif文件，但每次配准都会在【图层】中新增一个项目。通过图层的【删除】和【取消勾选】可以让地图视图上只展示最新的配准结果。

![img](../../../assets/qgis-geo-referencer/19.png)

## 3 最佳实践

### 3.1 控制点保存与加载

控制点可以进行保存和加载。

![img](../../../assets/qgis-geo-referencer/20.png)

![img](../../../assets/qgis-geo-referencer/21.png)

当遇到原图有变更的情况，而又在之前保存了控制点，只需要选择新的图片，加载既有的控制点，执行配准即可。严格来说，如此做的前提是：图片的变更仅限于内容有增减，但图的尺寸以及图上内容的相对位置无变化。

另外，如果有多个图片需要配准，而它们的尺寸以及内容的相对位置是一致的（或至少的相差不大的），那么可以在配准第一张图片时保其存控制点，在配准第二张及后续图片时使用第一张图片的控制点。这么做的好处是：第一张图片的控制点在地图视图中的位置是已经对准到相应位置的，此时只需要在第二张图片上对控制点的位置做相应调整即可。

![img](../../../assets/qgis-geo-referencer/22.png)

### 3.2 使用自定义投影

在高纬度地区进行配准操作时，可以使用自定义投影以使得图片与地图视图更为贴合，这将更有利于控制点的选取与对准。

首先，在[projectionwizard](https://projectionwizard.org/)中生成自定义投影：

- 【Distortion Property】选择【Equal-area】

- 对照原图，操作右侧的矩形框，使其四个边界尽量与原图相贴合。

- 点击【WKT】拷贝投影参数。

![img](../../../assets/qgis-geo-referencer/23.png)

然后，在QGIS的【设置】下拉框中选择【自定义投影】。

![img](../../../assets/qgis-geo-referencer/24.png)

在【选项-用户自定义CRS】中，点击右侧【+】添加投影，在【名称】中指定投影名称，在【参数】中拷贝[projectionwizard](https://projectionwizard.org/)中拷贝投影参数。点击【OK】以完成添加。

![img](../../../assets/qgis-geo-referencer/25.png)

点击QGIS下方状态栏右侧的投影，打开【工程属性-CRS】，在【预定义坐标参照系】中的【用户自定义】中找到并选中添加的投影，点击【OK】以确认。（此处可以看到两个用户自定义投影【阿拉斯加】，它们对于配准操作过程的效力是相同的，可任选其一）

![img](../../../assets/qgis-geo-referencer/26.png)

此时，地图视图将以自定义投影进行渲染，进而与原图在形状上更为相仿，配准将会更易于操作。

![img](../../../assets/qgis-geo-referencer/27.png)

以下是投影为【EPSG: 4326 WGS24】时地图视图的渲染效果。相比较之下，选取和对准控制点是更为更困难的。

![img](../../../assets/qgis-geo-referencer/28.png)

在使用自定义投影时，需要格外注意在【配准工具】的【变换配置】中，将【目标CRS】设置为：【EPSG:4326 - WGS 84】。