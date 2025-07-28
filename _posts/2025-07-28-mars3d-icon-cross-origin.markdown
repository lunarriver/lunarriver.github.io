---
layout: post
title:  "Mars3d BillboardIcon 图片访问跨域问题"
date:   2025-07-28 09:44:00 +0800
---

使用 mars3d 中的 [BillboardEntity](http://mars3d.cn/api.html#BillboardEntity) 添加图标至球组件：

![img](../../../assets/mars3d-icon-cross-origin/sample_code.png)

图标加载不出，开发者工具中报出跨域问题：

![img](../../../assets/mars3d-icon-cross-origin/cross-origin-problem.png)

在开发者工具的 Network 中查看对应请求的 Headers：

![img](../../../assets/mars3d-icon-cross-origin/request-headers.png)

在响应头上，可以看到图片是允许跨域的。

尝试将图标链接放到 mars3d 的示例页面中运行，图标正常加载，没有跨域问题。

排查是否由浏览器缓存造成，尝试以下操作：

- 清空浏览器缓存
- 在无痕摸下运行
- 在开发者工具的 Network 中勾选 Disable Cache

图标正常加载，没有跨域问题。经过详细区分，起决定性作用的是：勾选 Disable Cache。

根据开发者工具 Console 中的报错信息，定位到 Cesium 源码：engine/Source/Core/Resource.js

```
Resource._Implementations.loadImageElement = function (
  url,
  crossOrigin,
  deferred,
) {
  const image = new Image();
  
  if (crossOrigin) {
    if (TrustedServers.contains(url)) {
      image.crossOrigin = "use-credentials";
    } else {
      image.crossOrigin = "";
    }
  }

  image.src = url;
}
```

在 loadImageElement() 函数中，创建了一个 HTMLImageElement 实例，然后指定其 crossOrigin 和 src 属性。

关于 HTMLImageElement 的 [crossOrigin 属性](https://developer.mozilla.org/en-US/docs/Web/API/HTMLImageElement/crossOrigin)的作用：

![img](../../../assets/mars3d-icon-cross-origin/cross-origin-property.png)

简单的理解是：

- 指定 crossorigin 属性，则图片必定会按照 CORS 执行请求。
- 该属性可设置的值：anonymous、use-credentials，2个值都将导致图片按照 CORS 执行请求，其中，use-credentials 是使用证书的 CORS。
- 如果不指定该属性，则不会按照 CORS 执行请求。

同时，注意到同一张图片发生了多次请求。除了请求失败的，也有请求成功的。因为，项目中该功能提供了多个图标给用户选择：

```
<div class="attr-item">
  <span>选择图标样式</span>
  <a-select v-model:value="imageRef" class="white-arrow-select" @change="props.onIconChange">
    <a-select-option v-for="icon in iconList" :key="icon.id" :value="icon.dropDownUrl">
      <div class="billboard-icon-select-item">
        <img class="billboard-icon" :src="icon.dropDownUrl"/>
        <span style="padding-left: 4px;">{{ icon.name }}</span>
      </div>
    </a-select-option>
  </a-select>
</div>
```

至此，问题的缘由有了大致的脉络：

- 选择图标样式组件通过 img 标签加载的图片，默认情况下，浏览器会对图片资源进行缓存。
- 在loadImageElement() 函数中的 HTMLImageElement 实例加载图片时，浏览器不会发起新的请求，而是直接访问缓存的图片资源。但由于缓存的图片资源是非 CROS 模式下请求，因此，浏览器拒绝使用该图片资源，并且没有发起请求。
- 在开发者工具的 Network 中勾选 Disable Cache后，浏览器不再访问图片资源的缓存，而总是发起请求。是以，此种模式下，不会产生跨域问题。

最终的解决办法，在选择图标样式组件中，指定 img 标签的 crossOrigin 属性：

```
<img class="billboard-icon" :src="icon.dropDownUrl" crossorigin="anonymous"/>
```




