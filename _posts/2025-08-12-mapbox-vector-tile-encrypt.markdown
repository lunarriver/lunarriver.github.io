---
layout: post
title:  "Mapbox Vector Tile的一种简单加解密机制"
date:   2025-08-12 19:26:00 +0800
---

* 目录
{:toc #markdown-toc}

### 1 mapbox-gl-js-encrypt项目的加密实现

#### 1.1 基本逻辑流程

**mapbox-gl-js-encrypt**项目中， `src\source\vector_title_worker_source.js` 的`loadVectorTile`函数用于加载矢量瓦片数据。

![img](../../../assets/mapbox-vector-tile-encrypt/source.png)

当矢量瓦片数据的请求链接包含`isEncrypt`参数时，表示瓦片数据是加密的，通过`EncryptedVectorTile`进行解析。若不包含，则表示瓦片数据是常规的，通过`VectorTile`进行解析。

加密数据的请求链接形如：

```
https://tiles1.abc.com/base/v1/vector/{z}/{x}/{y}?format=pbf&isEncrypted=true
```

其中，`VectorTile`由`@mapbox/vector-tile`实现，其Github主页是：[mapbox/vector-tile-js](https://github.com/mapbox/vector-tile-js)，它基于[Mapbox Vector Tiles](https://github.com/mapbox/vector-tile-spec)的[vector_tile.proto](https://github.com/mapbox/vector-tile-spec/blob/master/2.1/vector_tile.proto)指定的数据结构对矢量瓦片数据进行编解码。

mapbox/vector-tile-js的项目结构：

```
vector-title
    |- index.js
    |- lib
        |- vectortile.js
        |- vectorfeature.js
        |- vectortilelayer.js
```

vector_tile的结构：

```
message tile {
    message feature {
        optional uint64 id = 1;
        repeated uint32 tags = 2 [ packed = true ];
        optional GeomType type = 3 [ default = Unknown ];
        repeated uint32 geometry = 4 [ packed = true ];
    }

    message layer {
        required uint32 version = 15 [ default = 1 ];
        required string name = 1;
        repeated feature features = 2;
        repeated string keys = 3;
        repeated value values = 4;
        optional uint32 extent = 5 [ default = 4096 ];
    }

    repeated layer layers = 3;
}
```

两者是一一对应的：

- tile表示瓦片，它包括若干个图层layer。layer则包括若干个feature（点、线、面(以及其他属性数据)）。

- vectortile.js用于解析瓦片，vectortilelayer.js用于解析瓦片中的图层，vectorfeature.js用于解析图层中的点、线、面（以及其他属性数据）。

`EncryptedVectorTile`是**mapbox-gl-js-encrypt**项目自定义的，其代码文件目录与mapbox/vector-tile-js保持一致。并且，其整体代码逻辑流程也是与标准流程保持一致的，只是对照瓦片数据的加密机制做了针对性的处理。

#### 1.2 瓦片数据的加密机制

`EncryptedVectorTile`与`VectorTile`的相异之处如下：

（1）vectortilelayer.js中，readLayer()函数读取图层属性时，部分字段的tag存在调换：

| 字段  | EncryptedVectorTile中的tag | VectorTile中的tag |
| --- | --- | --- |
| version | 15  | 15  |
| name | 3   | 1   |
| extent | 5   | 5   |
| _features | 2   | 2   |
| _keys | 4   | 3   |
| _vaues | 1   | 4   |

（2）vectortilelayer.js中，readValueMessage()读取value时，部分字段的tag存在调换：

| 字段  | EncryptedVectorTile中的tag | VectorTile中的tag |
| --- | --- | --- |
| string_value | 1   | 1   |
| float_value | 3   | 2   |
| double_value | 2   | 3   |
| int_value | 4   | 4   |
| uint_value | 6   | 5   |
| sint_value | 5   | 6   |
| bool_value | 7   | 7   |

（3）vectortilefeature.js中，readFeature()读取属性时，部分字段的tag存在调换：

| 字段  | EncryptedVectorTile中的tag | VectorTile中的tag |
| --- | --- | --- |
| id  | 4   | 1   |
| tags | 3   | 2   |
| type | 1   | 3   |
| geometry | 2   | 4   |

（4）vectortilefeature.js中，readFeature()读取type属性时，对属性值做了调换（重新定义了点、线、面的类型编码）：

- `EncryptedVectorTile`将加密数据中的 type-1 视作 type-3 (Polygon)。

- `EncryptedVectorTile`将加密数据中的 type-2 视作 type-1 (Point)。

- `EncryptedVectorTile`将加密数据中的 type-3 视作 type-2 (LineString)。


（5）vectortilefeature.js中，loadGeometry()和bbox()函数对vector-tile-spec中Geometry编码部分做了重定义，具体地，对CommandInteger中Command的数值做了重定义：

| Command | EncryptedVectorTile中的数值 | VectorTile中的数值 |
| --- | --- | --- |
| MoveTo | 4   | 1   |
| LineTo | 3   | 2   |
| ClosePath | 7   | 7   |

### 2 加密数据的解密处理（Android实现）

对加密的矢量瓦片数据做解密处理，有两种可取的模式：

- 自定义能够接受加密数据的自定义Vector Layer进行渲染。(mapbox-gl-js-encrypt项目采取此种方式)

- 拦截瓦片数据的请求过程，将加密的数据复原到标准的符合vector-tile-spec要求的格式，由内置的Vector Layer进行渲染。

相比较而言，后一种实现方式侵入性更低，难度更小。

#### 2.1 拦截瓦片数据请求

Mapbox Android Sdk 可以通过 `HttpServiceFactory` 对地图加载过程中的http请求进行拦截处理：

```
HttpServiceFactory.setHttpServiceInterceptor(object : HttpServiceInterceptorInterface {
    override fun onRequest(
        request: HttpRequest,
        continuation: HttpServiceInterceptorRequestContinuation
    ) {
        continuation.run(HttpRequestOrResponse(request))
    }

    override fun onResponse(
        response: HttpResponse,
        continuation: HttpServiceInterceptorResponseContinuation
    ) {
        continuation.run(
            if (response.request.url.contains("isEncrypted")) {
                // 对矢量pbf做解密处理
            } else {
                response
            }
        )
    }
})
```

### 2.2 pbf数据的编解码

瓦片数据的加密机制中，（1）、（2）、（3）均属于对字段tag的调换。应对此机制的处理方式是：

- 根据tag调换的规则，基于标准的 vector_tile.proto 进行相应修改，形成与加密数据相对应的自定义的 custom_vector_tile.proto 文件。

- 基于 custom_vector_tile.proto 对请求返回的加密瓦片数据进行解码，得到瓦片对象实例。

- 对瓦片对象实例进行其他加密机制的处理（见于2.3节）后，再基于 vector_tile.proto 进行编码，作为请求的响应数据返回。

基于Android项目(Java语言)的protobuf编解码处理具体包括：

- 获取[protocbuf工具](https://github.com/protocolbuffers/protobuf/releases/tag/v3.19.2https://github.com/protocolbuffers/protobuf/releases/tag/v3.19.2)，配置到环境变量。

- 通过protoc命令行生成 vector_tile.proto、custom_vector_tile.proto 对应的Java类。

```
protoc --java_out=${"要生成的 Java 文件目录"} ${".proto 文件位置"}
```

- 添加生成Java类所依赖的类库（库的版本需要与protocbuf工具的版本相对应）。

```
implementation("com.google.protobuf:protobuf-java:3.19.2")
```

- 基于生成的Java类进行编解码：

```
// 解码加密数据
val customVectorTile = CustomVectorTile.tile.parseFrom(data)

// 编码成标准格式数据
val vectorTile = VectorTile.tile.newBuilder()
    .addAllLayers(customVectorTile.layersList.map { transformLayer(it) })
    .build()

return vectorTile.toByteArray()
```

其中，`transformLayer()`将 CustomVectorTile 中的 layer 转换成 VectorTile 中的 layer。

```
private fun transformLayer(customLayer: CustomVectorTile.tile.layer): VectorTile.tile.layer {
    val layerBuilder = VectorTile.tile.layer.newBuilder()

    layerBuilder.setName(customLayer.name)
    layerBuilder.setExtent(customLayer.extent)
    layerBuilder.setVersion(customLayer.version)
    layerBuilder.addAllKeys(customLayer.keysList)
    layerBuilder.addAllValues(customLayer.valuesList.map {
        transformValue(it)
    })
    layerBuilder.addAllFeatures(customLayer.featuresList.map {
        transformFeature(it)
    })

    return layerBuilder.build()
}
```

`tranformValue()`和`transformFeature()`的作用与`transformLayer()`类似。

#### 2.3 其他加密机制的处理

加密机制（4）对feature的type属性值做了调换，语义相应的还原：

```
featureBuilder.setType(
    when (customFeature.type!!) {
        CustomVectorTile.tile.GeomType.LineString -> {
            VectorTile.tile.GeomType.Point
        }
        CustomVectorTile.tile.GeomType.Polygon -> {
            VectorTile.tile.GeomType.LineString
        }
        CustomVectorTile.tile.GeomType.Point -> {
            VectorTile.tile.GeomType.Polygon
        }
        CustomVectorTile.tile.GeomType.Unknown -> {
            VectorTile.tile.GeomType.Unknown
        }
    }
)
```

加密机制（5）CommandInteger中Command做了重定义，对其进行复原：

```
// 指示在 geometry integer 序列的遍历过程中，剩余参数个数
var remainParamsIntegerCount = 0

customFeature.geometryList.forEach { customGeometry ->
    var geometry = customGeometry

    if (remainParamsIntegerCount == 0) {
        // CommandInteger
        // 命令类型
        val cmdId = when (geometry and 0x7) {
            3 -> 2 // LineTo
            4 -> 1 // MoveTo
            7 -> 7 // ClosePath
            else -> throw RuntimeException()
        }

        // 命令参数数量
        val cmdCount = geometry ushr 3

        // A ClosePath command MUST have a command count of 1 and no parameters.
        if (cmdId != 7) {
            // 每个参数由2个integer组成
            remainParamsIntegerCount = cmdCount * 2
        }

        // 组成新的CommandInteger
        geometry = (cmdCount shl 3) or cmdId
    } else {
        remainParamsIntegerCount--
    }

    featureBuilder.addGeometry(geometry)
}
```

#### 2.4 实现过程中遇到的问题

（1）Mapbox Android Sdk会对http请求的返回数据做缓存，并非每次地图加载都会触发http请求，这可能导致：即便修改了代码逻辑，但程序运行的结果未产生变化。

（2）加密瓦片数据有返回`referer is disallowed`的情况，它不符合protocbuf的结构定义，需要进行屏蔽。

（3）vector_tile.proto中定义的value结构是一个复合类型，它包括了7种可能的类型，每个value只会取其中的某一个值。在处理过程中，需要判断原数据是否具有某个类型的值，否则会得到不符合标准结构的结果，进而导致渲染失败。

```
// 原数据
values {
    string_value: "\346\275\256\347\231\275\346\262\263"
}

// 处理不当的结果
values {
    string_value: "\346\275\256\347\231\275\346\262\263"
    float_value: 0
    double_value: 0
    int_value: 0
    uint_value: 0
    sint_value: 0
    bool_value: false
}
```
