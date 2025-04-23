---
layout: post
title:  "QGIS矢量数据操作"
date:   2025-04-23 16:03:08 +0800
---

文中针对于QGIS界面及操作的描述，可能仅适用于以下QGIS版本：

![img](../../../assets/qgis-vector-data/1.png)

### 1 跟随操作的前置条件

若需要进行跟随操作，需要完成以下准备工作：

（1）安装【HCMGIS】插件，以取得底图。

![img](../../../assets/qgis-vector-data/2.png)

安装完成后，菜单项上，将出现【HCMGIS】，点击打开下拉框，从中选择【Google Maps】等作为底图。

（2）取得世界国家的geojson数据：[world-geo-json-zh](https://github.com/Surbowl/world-geo-json-zh/blob/main/world.zh.json)。

![img](../../../assets/qgis-vector-data/3.png)

将其加载到QGIS中，置于【Google Maps】底图之上。

![img](../../../assets/qgis-vector-data/4.png)

（3）QGIS关于矢量数据操作的使用手册及其中文版（中文版部分翻译生硬，可与原版手册对照使用）。

- [working with vector](https://docs.qgis.org/3.40/zh-Hans/docs/user_manual/working_with_vector/index.html)

- [使用矢量数据](https://www.osgeo.cn/qgisdoc/docs/user_manual/working_with_vector/index.html)

（4）关于GEOJSON数据格式的相关规定参看：[RFC 7946](https://www.rfc-editor.org/rfc/rfc7946)。

### 2 设置矢量数据属性

右击矢量图层，点击【属性】。

![img](../../../assets/qgis-vector-data/5.png)

#### 2.1 字段

【字段】即是 feature#properties 所包含的字段。它不是指某个 feature 的字段值，而是指每个 feature 都具有的字段。

对于[world-geo-json-zh](https://github.com/Surbowl/world-geo-json-zh/blob/main/world.zh.json) 而言，每个国家既是一个 feature，每个 feature的 properties 都具有 5 个字段：name、full_name、iso_a2、iso_a3、iso_a4。

![img](../../../assets/qgis-vector-data/6.png)

通过【切换编辑模式】，可以进入或退出【编辑模式】。在【编辑模式】种，可以针对于每个 feature 进行【新增字段】、【删除字段】的操作。

通过【新增字段】新增一个【abc】字段，【切换编辑模式】时【Save】。

![img](../../../assets/qgis-vector-data/7.png)

导出geojson数据后，可以看到每个 feature 的 properties 都新增了一个【abc】字段，值为 null。

![img](../../../assets/qgis-vector-data/8.png)

#### 2.2 符号化

【符号化】用于设置矢量数据在地图上基于何种模式进行展示。默认情况下，采用【单一符号】的模式进行展示，对于面数据（feature#gemotry#type为Polygon）而言，所有的区域将填充上同一种颜色。

![img](../../../assets/qgis-vector-data/9.png)

按照以下操作步骤，矢量数据将根据 feature#properties#name 的值（即国家名称）进行展示，【分类】操作将为每个具体的值生成对应的颜色。

![img](../../../assets/qgis-vector-data/10.png)

【Apply】后，地图上的展示效果如下：

![img](../../../assets/qgis-vector-data/11.png)

#### 2.3 标注

【标注】用于在地图上展示矢量数据的属性信息。默认情况下，无标注。

按照以下操作步骤，矢量数据 feature#properties#name 的值（即国家名称）将展示在地图上。

![img](../../../assets/qgis-vector-data/12.png)

【Apply】后，地图上的展示效果如下：

![img](../../../assets/qgis-vector-data/13.png)

对于某些不具直观意义的属性（比如数据的id），以标注形式展示在地图上后，进行该属性相关的操作时将会更为简便。

### 3 属性表

右击矢量图层，点击【打开属性表】。

![img](../../../assets/qgis-vector-data/14.png)

属性表的作用主要包括两个方面：

- 针对于属性，即 feature#properties 进行新增、删除、修改的操作；

- 基于属性，选择 feature。

#### 3.1 属性表窗口与属性编辑

![img](../../../assets/qgis-vector-data/15.png)

属性表窗口大致包括三部分：

（1）顶部操作栏（红色框中部分）：

- 进入和退出编辑模式，保存编辑；

- 新增、删除 feature，剪切、复制、粘贴选中的 feature；

- 通过表达式或手动筛选、全选、反选等进行 feature 的选择或取消选择；

- 将选中元素在鼠标表窗口种置顶，或在地图上定位；

- 字段的新增、删除，选择展示或隐藏某一属性（列）；

- 将属性表停靠在QGIS主窗口上，便于在地图上查看或协同操作；

（2）feature预览、点选区域（绿色框中部分）：

- 鼠标点击左侧列表项的勾选框，可以选中 feature，按住 ctrl 或 shift 可以多选或取消选择；

- 右侧展示有焦点 feature 的 properties，可以对某个属性值进行修改；

（3）【feature预览、点选区域】展示模式选择区域（右下角蓝色框中部分）：另一种展示模式如下：

![img](../../../assets/qgis-vector-data/16.png)

于该展示模式下，一行即是一个 feature。鼠标点击最左侧的索引列可以选中 feature，按住 ctrl 或 shift 可以多选或取消选择。点击某一行的某个属性，即可以直接更改其属性值。

![img](../../../assets/qgis-vector-data/17.png)

进入编辑模式后，红色框中部分可用于：修改全部 feature 或选中 feature 的某个属性的值。

以下展示的是选择两个feature，通过【更新所选】修改它们的 abc 字段值的效果：

![img](../../../assets/qgis-vector-data/18.png)

完成属性编辑操作后，需要点击【保存】以使其生效。

#### 3.2 在属性表中选择元素

未处于编辑模式的情况下，选中的feature将被赋予不同的颜色。

![img](../../../assets/qgis-vector-data/19.png)

![img](../../../assets/qgis-vector-data/20.png)

处于编辑模式的情况下，选中的feature的顶点也会被展示出来：

![img](../../../assets/qgis-vector-data/21.png)

下图中绿色框中部分的【将所选记录移到顶部】、【平移地图到所选记录】、【缩放地图到所选记录】用于在属性表或地图上快速定位选中的feature。

![img](../../../assets/qgis-vector-data/22.png)

### 4 顶点操作

顶点操作即是对 feature#geometry#coordinates中的每个坐标点进行操作。

![img](../../../assets/qgis-vector-data/23.png)

进行顶点操作，首先需要进入编辑模式，然后选中某个或某些 feature。CTRL+A 可选中所有 feature。

![img](../../../assets/qgis-vector-data/24.png)

进入编辑模式后，【数字化工具栏】和【高级数字化工具栏】将被激活。如果工具栏找不到它们，可以右击工具栏，在弹出窗中将它们勾选中。

当针对顶点进行操作时，不必每次都打开属性表窗口进行【全选】、【反选】、【取消选择】等操作，因为工具栏上也有这些操作的图标。

![img](../../../assets/qgis-vector-data/25.png)

#### 4.1 数字化工具栏

![img](../../../assets/qgis-vector-data/26.png)

（1）【切换编辑模式】：用于进入和退出编辑模式；

（2）【保存图层编辑】：用于保存对顶点的操作；

（3）【线段数字化】、【添加多边形要素】：需要说明的是，这两项操作与图层的几何图形类型有关。在新增临时图层时，QGIS提示我们进行选择。

![img](../../../assets/qgis-vector-data/27.png)

如果图层的几何图形类型是【点】或【线】，那么【添加多边形要素】将变成【添加点要素】：

![img](../../../assets/qgis-vector-data/28.png)

（4）【顶点工具】：针对【所有图层】或【当前图层】进行顶点操作。【当前图层】指的是在【图层】面板中处于选中状态的图层。

![img](../../../assets/qgis-vector-data/29.png)

顶点工具具体包含的操作如下：

![img](../../../assets/qgis-vector-data/30.png)

其中，【Alt键+单机-按多边形选择顶点】，并非点击某个 feature 以选中它的顶点，而是，通过鼠标绘制一个多边形，进而选中多边形内部的顶点。其效果如下：  

![img](../../../assets/qgis-vector-data/31.png)

![img](../../../assets/qgis-vector-data/32.png)

（5）【同时修改所有选中要素的属性】：可以通过它给新增的字段指定默认值。例如：

![img](../../../assets/qgis-vector-data/33.png)

（6）【删除所选要素】：以下是删除 feature-澳大利亚 的效果。

![img](../../../assets/qgis-vector-data/34.png)

（7）【剪切要素】、【复制要素】、【粘贴要素】：可用于将选择的 feature 复制到其他图层。将 feature-澳大利亚 复制到新建临时图层的效果如下：

![img](../../../assets/qgis-vector-data/35.png)

只需要呈现为点，是因为【新建临时图层】的几何图形类型为【点】。重新新建临时图层，将其几何图形类型指定为【多边形】，效果如下：

![img](../../../assets/qgis-vector-data/36.png)

在复制feature的过程中，feature#properties 同样会被复制，其前提在创建目标图层（即【新建临时图层】）时，需要为其指定相同的字段定义。

![img](../../../assets/qgis-vector-data/37.png)

将【新建临时图层】导出称geojson文件，可以看到 feature#properties#name 被复制过来了。

![img](../../../assets/qgis-vector-data/38.png)

（8）【撤销】、【重做】用于将已执行的操作进行取消和重新执行。

#### 4.2 高级数字化工具栏

![img](../../../assets/qgis-vector-data/39.png)

##### 4.2.1 移动要素

点击选中某个feature，再点击将其移动的目标位置。

![img](../../../assets/qgis-vector-data/40.png)

【旋转要素】、【缩放要素】同理。

##### 4.2.2 简化要素

点击需要进行操作的feature，地图右上角出现操作区域。

![img](../../../assets/qgis-vector-data/41.png)

简化操作的作用是：依据当前多边形（示例中的几何图形类型是多边形）轮廓，根据操作区域中指定的规则重新布置顶点。

它可能带来顶点数量的增加或减少。顶点数量减少意味着图形细节的减弱（如上图），同时geojson文件的体积将减少。顶点数量增加意为这图形细节的增强（如下图），同时geojson文件的体积将增加。

![img](../../../assets/qgis-vector-data/42.png)

##### 4.2.3 环操作

环操作包括【添加环】、【填充环】、【删除环】。

环的概念见于[RFC 7946](https://www.rfc-editor.org/rfc/rfc7946) 关于多边形（Polygon）的描述中：

![img](../../../assets/qgis-vector-data/43.png)

【添加环】用于在feature上挖洞：鼠标左键点击生成洞的各个顶点，确定全部的顶点后，点击鼠标右键以确认。

![img](../../../assets/qgis-vector-data/44.png)

【删除环】用于在将feature上的洞去除，点击洞的位置即可。

【填充环】用于在feature上挖洞，但这个洞本身会成为一个新的feature。其操作与【添加环】一致，只是在鼠标右键确认后，需要指定新feature的属性。

![img](../../../assets/qgis-vector-data/45.png)

导出geojson后可以看到新增的feature：

![img](../../../assets/qgis-vector-data/46.png)

由于【填充环】会生成一个新的feature，通过【删除环】无法将其删除，需要通过选中feature并删除的方式才可将其去除。

![img](../../../assets/qgis-vector-data/47.png)

将其删除后，相应的位置会呈现出一个洞，这说明【填充环】同样会对原feature的形状产生影响。这个洞就可以用【删除环】来进行去除了。

##### 4.2.4 部件操作

如果一个feature只包含一个几何形状，则称之为单部件，对应于[RFC 7946](https://www.rfc-editor.org/rfc/rfc7946)的Point、LineString、Polygon。如果一个feature包含多个几何形状，则称之为多部件，对应于[RFC 7946](https://www.rfc-editor.org/rfc/rfc7946)的MultiPoint、MultiLineString、MultiPolygon。

![img](../../../assets/qgis-vector-data/48.png)

对于[world-geo-json-zh](https://github.com/Surbowl/world-geo-json-zh/blob/main/world.zh.json)来说，如果一个国家由单个部分组成（没有岛屿、飞地），则其对应feature为单部件（Polygon）。

![img](../../../assets/qgis-vector-data/49.png)

如果一个国家由多个部分组成（有岛屿、飞地），则其对应feature为多部件（MultiPolygon）。

![img](../../../assets/qgis-vector-data/50.png)

【添加部件】用于为一个多部件feature新增一个几何图形。因此，需要首先选定一个feature。

![img](../../../assets/qgis-vector-data/51.png)

针对于[world-geo-json-zh](https://github.com/Surbowl/world-geo-json-zh/blob/main/world.zh.json)的原始图层进行该操作时，会提示【无法添加部件。此图层不支持多部件几何图形】。这是因为原始图层的几何图形类型是Polygon，即单部件。（需要注意的是，即便feature本身是MultiPolygon的，但由于存在Polygon的feature，整个图层仍被视作Polygon）

![img](../../../assets/qgis-vector-data/52.png)

可以通过新建一个MultiPolygon的图层，将原始图层的全部feature复制过去，再执行【添加部件】操作。

![img](../../../assets/qgis-vector-data/53.png)

![img](../../../assets/qgis-vector-data/54.png)

【删除部件】用于去除MultiPolygon类型feature某个几何图形。

![img](../../../assets/qgis-vector-data/55.png)

##### 4.2.5 重塑要素

【重塑要素】用于调整几何图形的边界（走向）。

![img](../../../assets/qgis-vector-data/56.png)

![img](../../../assets/qgis-vector-data/57.png)

##### 4.2.6 偏移曲线

【偏移曲线】用于对某个几何图形进行整体的放大、缩小操作。

![img](../../../assets/qgis-vector-data/58.png)

![img](../../../assets/qgis-vector-data/59.png)

可通过属性框设置放大、缩小的边界样式。

![img](../../../assets/qgis-vector-data/60.png)

##### 4.2.7 分割与合并

【分割要素】用于将一个feature分割成多个。

![img](../../../assets/qgis-vector-data/61.png)

分割要素操作前，【框选或单击选择要素】的效果是：

![img](../../../assets/qgis-vector-data/62.png)

分割要素操作后，【框选或单击选择要素】的效果是：（不仅大陆部分被一分为二，外围的小岛也成为了单独的feature）

![img](../../../assets/qgis-vector-data/63.png)

【合并所选要素】用于将两个feature合并成一个。

![img](../../../assets/qgis-vector-data/64.png)

合并要素时，需要对其properties进行编辑确认。

![img](../../../assets/qgis-vector-data/65.png)

合并的效果如下：

![img](../../../assets/qgis-vector-data/66.png)

【合并所选要素的属性】可以将多个feature的properties进行合并，但它们仍将作为单独的feature，只是（除id以外的其他）properties相同。

![img](../../../assets/qgis-vector-data/67.png)