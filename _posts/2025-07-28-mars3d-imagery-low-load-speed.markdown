---
layout: post
title:  "Mars3d 影像图加载速度优化"
date:   2025-07-28 11:11:00 +0800
---

* 目录
{:toc #markdown-toc}

### 1 基本情况

#### 1.1 问题描述

在项目中加载指定区域影像图的速度缓慢。指定区域影像图加载缓慢的感受源于：影像底图的加载速度明显快于指定区域影像图。

#### 1.2 技术实现

项目中的地图组件基于mars3d实现。

```
import * as mars3d from 'mars3d'

class MapWrapper {

  private map!: mars3d.Map | null

  public initMap(id: string, options?: any) {
    this.map = new mars3d.Map(id, { ...defaultOptions, ...options })
  }

  public getMap(): mars3d.Map {
    return this.map as mars3d.Map
  }
}

export default new MapWrapper()
```

影像底图的配置：

```
{
  type: 'xyz',
  id: 'basic_img',
  name: '底图',
  thumb: 'images/layer_thumb/basemap.jpg',
  url: `https://tiles{s}.abc.com/base/v1/img/{z}/{x}/{y}?format=webp&tmsIds=w&token=${MapToken}`,
  minimumLevel: 1,
  maximumLevel: 18,
  subdomains: ['1', '2', '3'],
  zIndex: 2,
  coverLayerId: 'daas_dxannotation',
  filterColor: '#e6eaf2',
  gamma: 1.4,
  layerType: BaseLayerTypeEnum.GENERAL,
  active: true
}
```

指定区域影像图的加载基于mars3d.layer.WmsLayer实现。

```
import mapWrapper from ...

let tileLayer: any = null;

onMounted(async () => {
  if (!tileLayer) {
    tileLayer = new mars3d.layer.WmsLayer({
      name: "指定区域WMS",
      url: "https://abc.com/geoserver/xxx/wms",
      layers: "...",
      parameters: {
        version: "1.1.1",
        styles: "",
        service: "WMS",
        format: "image/png",
        transparent: true,
        request: 'GetMap'
      },
      crs: 'EPSG:4490',
      zIndex: 20,
      bbox: [116.79290771484375, 31.70482635498047, 117.32025146484375, 31.98566436767578],
      tileWidth: 768,
      tileHeight: 409
    })
    mapWrapper.getMap().addLayer(tileLayer)
  }
```

WmsLayer构造器的几个关键参数：

// TABLE-1

| 参数名称       | 类型                        | 描述信息                                       |
| ---------- | ------------------------- | ------------------------------------------ |
| url        | Cesium.Resource \| string | WMS服务的URL。                                 |
| layers     | string                    | 要包含的图层，用逗号分隔。                              |
| crs        | string \| CRS             | 瓦片数据的坐标系信息，默认为墨卡托投影，CRS规范，用于WMS规范>= 1.3.0。 |
| bbox       | Array.<number>            | bbox规范的瓦片数据的矩形区域范围,与rectangle二选一即可。        |
| tileWidth  | number                    | 图像图块的像素宽度。默认值256。                          |
| tileHeight | number                    | 图像图块的像素高度。默认值256。                          |
| parameters | object                    | 要在URL中 传递给WMS服务GetMap请求的其他参数。              |

#### 1.3 排查思路

首先，查看各个地图层级下，影像底图的加载情况。在开发者工具的 Network 中，通过特定链接片段过滤出影像底图的加载请求，记录请求的数据体积的耗时。

![img](../../../assets/mars3d-imagery-low-load-speed/network-filter.png)

影像底图的请求连接，遵循以下格式，其各自的 tile-coord 查询参数的值不同。

```
https://tiles{1-3}.abc.com/base/v1/img/{tile-coord-params}?format=webp&tmsIds=w&token=1f7474901a8c57d1eed040b623cad983612b68e579f3c99e8952e9f5f349856b
```

请求结果是宽高为256*256的webp图片。

![img](../../../assets/mars3d-imagery-low-load-speed/base-layer-tile.png)

然后，查看各个地图层级下，指定影像图的加载情况。在开发者工具的Network中，过滤出指定区域影像图的加载请求，记录请求的数据体积的耗时。

![img](../../../assets/mars3d-imagery-low-load-speed/network-filter2.png)

指定区域影像图的请求连接，遵循以下格式，其各自的bbox查询参数的值不同。

```
https://abc/geoserver/.../wms?version=1.1.1&styles=&service=WMS&format=image%2Fpng&transparent=true&request=GetMap&layers=...%3Agaoxin&bbox={bbox-params}&width=768&height=409&srs=EPSG%3A4326
```

请求结果是指定宽高为768*409的png图片。

![img](../../../assets/mars3d-imagery-low-load-speed/area-layer-tile.png)

最后，将两组结果进行对比，找到影响加载性能的决定性因素。

### 2 影像底图的加载情况

关闭浏览器缓存：

![img](../../../assets/mars3d-imagery-low-load-speed/disable-cache.png)

从10级逐级放大至17级，统计影像地图的加载情况：

| 批次     | 瓦片数量(个) | 瓦片平均尺寸(kb) | 瓦片最大尺寸(kb) | 加载平均耗时(ms) | 加载最大耗时(ms) |
| ------ | ------- | ---------- | ---------- | ---------- | ---------- |
| 放大至11级 | 42      | 15.8       | 26.7       | 35.8       | 97         |
| 放大至12级 | 35      | 15.8       | 31.5       | 106.9      | 157        |
| 放大至13级 | 35      | 14.9       | 32.5       | 70.9       | 175        |
| 放大至14级 | 24      | 12         | 22.3       | 103.4      | 203        |
| 放大至15级 | 38      | 10.3       | 22.6       | 66.2       | 287        |
| 放大至16级 | 26      | 7.8        | 17.3       | 68.2       | 196        |
| 放大至17级 | 35      | 10.4       | 22.1       | 78         | 142        |
| 总计     | 235     | 12.7       | 32.5       | 73.3       | 287        |

启用浏览器缓存，从16级放大至17级，影像底图加载全命中缓存的情况下，耗时指标如下：

| 指标     | 数值    |
| ------ | ----- |
| 加载平均耗时 | 1.8ms |
| 加载最大耗时 | 4ms   |

### 3 指定区域影像图加载情况

从10级逐级放大至17级：

| 编号  | 数据体积(kb) | 耗时(ms) |
| --- | -------- | ------ |
| 1   | 61.6     | 60     |
| 2   | 60.1     | 93     |
| 3   | 252      | 80     |
| 4   | 155      | 74     |
| 5   | 101      | 65     |
| 6   | 679      | 104    |
| 7   | 273      | 85     |
| 8   | 135      | 86     |
| 9   | 332      | 78     |
| 10  | 219      | 60     |
| 11  | 96.4     | 49     |
| 12  | 818      | 128    |
| 13  | 761      | 132    |
| 14  | 588      | 154    |
| 15  | 442      | 128    |
| 16  | 784      | 303    |
| 17  | 791      | 346    |
| 18  | 757      | 336    |
| 19  | 794      | 357    |
| 20  | 785      | 362    |
| 21  | 734      | 136    |
| 22  | 741      | 142    |
| 23  | 660      | 115    |
| 24  | 670      | 210    |
| 25  | 663      | 135    |
| 26  | 668      | 139    |
| 27  | 675      | 156    |
| 28  | 709      | 160    |

总体情况：

| 指标     | 数值    |
| ------ | ----- |
| 瓦片平均尺寸 | 514kb |
| 瓦片最大尺寸 | 794kb |
| 加载平均耗时 | 152ms |
| 加载最大耗时 | 362ms |

### 4 对比结果分析

将指定区域影像图与影像底图加载情况的几个指标对比如下：

| 分组         | 瓦片平均尺寸(kb) | 瓦片最大尺寸(kb) | 加载平均耗时(ms) | 加载最大耗时(ms) |
| ---------- | ---------- | ---------- | ---------- | ---------- |
| 影像底图       | 12.7       | 32.5       | 73.3       | 287        |
| 影像底图(启用缓存) | /          | /          | 1.8        | 4          |
| 指定区域影像图    | 514        | 794        | 152        | 362        |

几点结论： 

（1）指定区域影像图的请求结果未能缓存，开发者工具中Disable Cache是否勾选对其加载速度无影响。

（2）在未启用缓存的情况下，指定区域影像图的平均加载耗时约为影像底图的2倍。制约其加载速度的因素可能有：

- 瓦片尺寸较大：指定区域影像图的瓦片尺寸是768*409，而影像底图是256*256；

- 图片格式的不同：指定区域影像图的格式是png，而影像底图的格式是webp；

（3）在未启用缓存的情况下，影像底图的加载速度也称不上“快”。“影像底图相对于指定区域影像图，加载速度很快”的直观感受，需要缓存机制的支持。

### 5 请求头和响应头比对

底图请求的完整信息：

![img](../../../assets/mars3d-imagery-low-load-speed/headers-compare-1.png)

指定区域影像图请求的完整信息：

![img](../../../assets/mars3d-imagery-low-load-speed/headers-compare-2.png)

比对两个请求的请求头和响应头：

（1）两个请求的请求头字段基本一致，其中，部分字段值有差异。 

（2）两个请求的响应头差异很大，底图请求的响应头包含了控制浏览器缓存的cache-control字段，而指定区域影像图请求的响应头则不包含此字段。

```
cache-control: public,max-age=86400
```
cache-control字段的相关规范见于[RFC 7234 5.2节](https://www.rfc-editor.org/rfc/rfc7234#section-5.2)。

![img](../../../assets/mars3d-imagery-low-load-speed/cache-control.png)

### 6 在GeoServer中配置cache-control

在发布指定区域的GeoServer中找到对应的图层。

![img](../../../assets/mars3d-imagery-low-load-speed/geo-server-1.png)

进入图层编辑页面。切换到【发布】标签页。在【Caching Settings】项目下，勾选【缓存响应头】，设置【缓存时间（秒）】为86400，即1天。

![img](../../../assets/mars3d-imagery-low-load-speed/geo-server-2.png)

在浏览器中访问指定区域影像图，验证配置效果。

![img](../../../assets/mars3d-imagery-low-load-speed/geo-server-config-result.png)

响应头中包含了cache-control字段。

![img](../../../assets/mars3d-imagery-low-load-speed/must-revalidate.png)

### 7 调整瓦片的尺寸

在代码中将指定区域影像图的瓦片尺寸调整为256×256。

```
new mars3d.layer.WmsLayer({
  ...
  tileWidth: 256, // 原值为768
  tileHeight: 256, // 原值为409
})
```

验证缓存配置对加载效率的增强效果。多次访问并缩放、移动地图，瓦片数据可命中缓存。

- 指定区域影像图
![img](../../../assets/mars3d-imagery-low-load-speed/result-compare-1.png)

- 影像底图
![img](../../../assets/mars3d-imagery-low-load-speed/result-compare-2.png)

粗略地将两个影像图的加载做比较。相形之下：

（1）在瓦片尺寸相同的情况下（均为256×256），影像底图瓦片的体积小得多，基本上差了一个数量级。

（2）在命中缓存时，影像底图的耗时也较小，也可能与瓦片体积有关。