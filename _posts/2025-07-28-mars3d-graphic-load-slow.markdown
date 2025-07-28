---
layout: post
title:  "Mars3d 项目矢量数据加载卡顿问题排查"
date:   2025-07-28 13:58:00 +0800
---

* 目录
{:toc #markdown-toc}

### 1 项目中的矢量对象

#### 1.1 基本情况

项目中的矢量对象通过GraphicLayer.addGraphic()函数添加到地图组件：

![img](../../../assets/mars3d-graphic-load-slow/1.png)

这两个函数都根据矢量对象的dataConfig的type字段来区分类型，部分type相同的矢量对象通过其style.materialType来区分：

| 矢量对象 | eleCode       | type          | style.materialType |
| ---- | ------------- | ------------- | ------------------ |
| 直线   | polyline      | polyline      | Color              |
| 曲线   | curve         | curve         |                    |
| 流动线  | flowLine      | polyline      | LineFlowColor      |
| 箭头线  | arrowLine     | polyline      | PolylineArrow      |
| 河流   | dynamicRiver  | dynamicRiver  |                    |
| 标签文字 | labelText     | label         |                    |
| 贴地文字 | clampText     | rectangle     | Text               |
| 直箭头  | arrow         | straightArrow |                    |
| 粗直箭头 | wideArrow     | fineArrow     |                    |
| 燕尾箭头 | dovetailArrow | attackArrowYW |                    |
| 多边形  | polygon       | polygon       |                    |
| 矩形   | rectangle     | rectangle     | PolyGradient       |
| 圆形   | circle        | circle        |                    |

各种矢量对象通过不同的函数所生成的BaseGraphic对应的类如下：

| 矢量对象 | eleCode        | BaseGraphic子类   |
| ---- | -------------- | --------------- |
| 直线   | polyline       | PolylineEntity  |
| 曲线   | curve          | CurveEntity     |
| 流动线  | flowLine       | PolylineEntity  |
| 箭头线  | arrowLine      | PolylineEntity  |
| 河流   | dynamicRiver   | DynamicRiver    |
| 标签文字 | labelText      | LabelEntity     |
| 贴地文字 | clampText      | RectangleEntity |
| 直箭头  | arrow          | StraightArrow   |
| 粗直箭头 | wideArrow      | FineArrow       |
| 燕尾箭头 | dovetailArrow  | AttackArrowYW   |
| 多边形  | polygon        | PolygonEntity   |
| 矩形   | rectangle      | RectangleEntity |
| 圆形   | circle         | CircleEntity    |
| 动态面板 | billBoardLabel | DivGraphic      |
| 图标   | billBoardIcon  | BillboardEntity |

各个类与BaseGraphic的继承关系如下：

![img](../../../assets/mars3d-graphic-load-slow/2.png)

#### 1.2 矢量对象的渲染方式

在mars3d的功能示例中，展示了矢量对象的几种渲染方式。以面状矢量为例：

![img](../../../assets/mars3d-graphic-load-slow/3.png)

其中，多边形、矩形、圆分别包含了3种渲染方式：

- 多边形面 / 矩形 / 圆 
- 多边形面(Primitive) / 矩形(Primitive) / 圆(Primitive)
- 大量多边形面(合并渲染Primitive) / 大量矩形(合并渲染Primitive) / 大量圆(Primitive)

以多边形为切入点，考察3种渲染方式的具体实现：

（1）[多边形面](http://mars3d.cn/editor-vue.html?key=ex_6_7_0&id=graphic/entity/polygon)

```
const graphic = new mars3d.graphic.PolygonEntity({
    positions: [pt1, pt2, pt3, pt4, pt5],
    style: {
        color: Cesium.Color.fromRandom({ alpha: 0.6 })
    },
    attr: { index }
})
```

（2）[多边形面(Primitive)](http://mars3d.cn/editor-vue.html?key=ex_6_7_1&id=graphic/primitive/polygon)

```
const graphic = new mars3d.graphic.PolygonPrimitive({
    positions: [pt1, pt2, pt3, pt4, pt5],
    style: {
        color: Cesium.Color.fromRandom({ alpha: 0.6 })
    },
    attr: { index }
})
```

（3）[大量多边形面(合并渲染Primitive)](http://mars3d.cn/editor-vue.html?key=ex_6_7_2&id=graphic/combine/polygon)

```
const graphic = new mars3d.graphic.PolygonCombine({
    instances: arrData,
    highlight: {
        type: mars3d.EventType.click,
        color: Cesium.Color.YELLOW.withAlpha(0.9)
    }
})
```

PolygonEntity、PolygonPrimitive、PolygonCombine在BaseGraphic继承体系中所处的位置如下：

![img](../../../assets/mars3d-graphic-load-slow/4.png)

mars3d关于[矢量数据](http://mars3d.cn/docs/basis/graphic/#_2-%E7%9F%A2%E9%87%8F%E6%95%B0%E6%8D%AE%E7%9A%84%E7%B1%BB%E5%88%AB)的渲染方式的比较：

![img](../../../assets/mars3d-graphic-load-slow/5.png)

#### 1.3 项目录入的矢量对象类型分布

基于项目dev环境数据进行分析（剔除测试数据）。各类型的矢量对象数量如下：

| 矢量对象 | eleCode        | BaseGraphic子类   | 数量  |
| ---- | -------------- | --------------- | --- |
| 直线   | polyline       | PolylineEntity  | 436 |
| 曲线   | curve          | CurveEntity     | 14  |
| 流动线  | flowLine       | PolylineEntity  | 9   |
| 箭头线  | arrowLine      | PolylineEntity  | 0   |
| 河流   | dynamicRiver   | DynamicRiver    | 2   |
| 标签文字 | labelText      | LabelEntity     | 648 |
| 贴地文字 | clampText      | RectangleEntity | 89  |
| 直箭头  | arrow          | StraightArrow   | 0   |
| 粗直箭头 | wideArrow      | FineArrow       | 0   |
| 燕尾箭头 | dovetailArrow  | AttackArrowYW   | 0   |
| 多边形  | polygon        | PolygonEntity   | 198 |
| 矩形   | rectangle      | RectangleEntity | 126 |
| 圆形   | circle         | CircleEntity    | 1   |
| 动态面板 | billBoardLabel | DivGraphic      | 12  |
| 图标   | billBoardIcon  | BillboardEntity | 0   |

共计1535个矢量对象。其中，直线、标签文本、贴地文字、多边形占据了大部分（1497个，占97.5%）。

| 实现类             | 数量  | 占比    |
| --------------- | --- | ----- |
| PolylineEntity  | 436 | 28.4% |
| LabelEntity     | 648 | 42.2% |
| RectangleEntity | 215 | 14%   |
| PolygonEntity   | 198 | 12.9% |

### 2 项目首页启动时的渲染过程分析

选取一次执行时间最长的渲染过程进行考察。

![img](../../../assets/mars3d-graphic-load-slow/6.png)

该次渲染过程中，函数执行耗时的总体情况如下：

- Self time排序

![img](../../../assets/mars3d-graphic-load-slow/7.png)

- Total time 排序

![img](../../../assets/mars3d-graphic-load-slow/8.png)

#### AT.render | CesiumWidget.prototype.render

该函数是Cesium 中每帧渲染的核心入口，其执行时间直接影响应用的帧率（FPS）和整体性能。其理想的执行时间需要控制在 **16.67ms** 以内，以保证60FPS。

```
// CesiumWidget.js
CesiumWidget.prototype.render = function () {
  if (this._canRender) {
    this._scene.initializeFrame();
    const currentTime = this._clock.tick();
    this._scene.render(currentTime);
  } else {
    this._clock.tick();
  }
};
```

在整个观测区间内，AT.render共执行了20次。记录历次执行时间如下：

```
69.99ms / 2.67ms / 4.06ms / 2.62ms / 41us / 0.61ms / 1.2ms / 2.77ms / 1.69s / 69.3ms / 29.35ms / 59.14ms / 18us / 35us / 0.37ms / 0.22ms / 0.33ms / 307.8ms / 0.19ms / 37.73ms
```

#### Of.update | GeometryVisualizer.prototype.update

![img](../../../assets/mars3d-graphic-load-slow/9.png)

该函数是Cesium的Entity系统与Primitive渲染系统之间的重要桥梁。该方法用于将当前时间下所有关联的Entity对象转换为Primitive图形对象并更新其状态。

```
// GeometryVisualizer.js
/**
 * Updates all of the primitives created by this visualizer to match their
 * Entity counterpart at the given time.
 *
 * @param {JulianDate} time The time to update to.
 * @returns {boolean} True if the visualizer successfully updated to the provided time,
 * false if the visualizer is waiting for asynchronous primitives to be created.
 */
GeometryVisualizer.prototype.update = function (time) {...}
```

#### zi.updateAndExecuteCommands | Scene.prototype.updateAndExecuteCommands

![img](../../../assets/mars3d-graphic-load-slow/10.png)

该函数负责更新并清除帧缓冲区（framebuffers），根据当前场景模式（3D、2D、WebVR）选择合适的绘制方式，执行绘制命令（draw commands）。

```
// Scene.js
/**
 * Update and clear framebuffers, and execute draw commands.
 *
 * @param {PassState} passState State specific to each render pass.
 * @param {Color} backgroundColor
 *
 * @private
 */
Scene.prototype.updateAndExecuteCommands = function(passState, backgroundColor) { ... }
```

##### s0 | executeCommandsInViewport

该函数负责执行所有已收集的绘制命令（draw commands），将三维场景渲染到当前视口（viewport）中。

```
// Scene.js
/**
 * Execute the draw commands to render the scene into the viewport.
 * If this is the first viewport rendered, the framebuffers will be cleared to the background color.
 *
 * @param {boolean} firstViewport <code>true</code> if this is the first viewport rendered.
 * @param {Scene} scene
 * @param {PassState} passState
 *
 * @private
 */
function executeCommandsInViewport(firstViewport, scene, passState) { ... }
```

##### V0e | updateAndRenderPrimitives

该函数负责更新并渲染所有 primitive 图形（如Point、Billboard、Label等）。

```
// Scene.js
function updateAndRenderPrimitives(scene) {
  const frameState = scene._frameState;

  scene._groundPrimitives.update(frameState);
  scene._primitives.update(frameState);

  updateDebugFrustumPlanes(scene);
  updateShadowMaps(scene);

  if (scene._globe) {
    scene._globe.render(frameState);
  }
}
```

##### ta.update | PrimitiveCollection.prototype.update

该函数对每个Primitive调用 其update函数。

```
// PrimitiveCollection.js 
/**
 * @private
 */
PrimitiveCollection.prototype.update = function (frameState) {
  if (!this.show) {
    return;
  }

  const primitives = this._primitives;
  // Using primitives.length in the loop is a temporary workaround
  // to allow quadtree updates to add and remove primitives in
  // update().  This will be changed to manage added and removed lists.
  for (let i = 0; i < primitives.length; ++i) {
    primitives[i].update(frameState);
  }
};
```

##### Zd.update | EntityCluster.prototype.update

该函数负责在渲染时更新和处理大量点状实体（如 billboard、point、label）的绘制命令： 

- 如果启用了 实体聚合（clustering）功能，该方法会生成用于渲染聚合图标（cluster icon）的绘图命令； 
- 如果未启用聚合功能，则正常为每个 Entity 的 billboard、point 或 label 生成绘图命令并排队等待渲染； 
- 它是渲染流程中关键的一环，决定了是否使用聚合图标来优化性能。

```
// EntityCluster.js
/**
 * Gets the draw commands for the clustered billboards/points/labels if enabled, otherwise,
 * queues the draw commands for billboards/points/labels created for entities.
 * @private
 */
EntityCluster.prototype.update = function (frameState) { ... }
```

##### sm.update | LabelCollection.prototype.update

该函数负责更新所有标签（Label）状态的核心逻辑： 

- 控制标签的显示/隐藏； 
- 更新每个标签的字符（glyph）； 
- 重新排版字符位置； 
- 管理 billboard 集合（字符和背景）； 
- 提升性能，避免重复渲染； 
- 与 Cesium 的渲染流程（frameState）同步。

```
// LabelCollection.js
/**
 * @private
 */
LabelCollection.prototype.update = function (frameState) { ... }
```

#### Uje | rebindAllGlyphs

![img](../../../assets/mars3d-graphic-load-slow/11.png)

该函数为一个 Label（标签）重新绑定所有字符（glyph），并更新其在 3D 场景中的显示效果： 

- 将文本拆分成字符； 
- 为每个字符生成或复用一个 Billboard（图标）； 
- 为字符生成或复用纹理（支持 SDF 渲染）； 
- 控制背景框的显示； 
- 将所有字符的 billboard 设置为与 label 一致的属性（如位置、缩放、颜色等）； 
- 支持性能优化（缓存纹理、复用 billboard、按需更新）； 
- 支持抗锯齿渲染（SDF：Signed Distance Field）； 
- 支持多种样式（填充、描边、仅描边等）；

```
// LabelCollection.js
function rebindAllGlyphs(labelCollection, label) { ... }
```

##### Fje | createGlyphCanvas

![img](../../../assets/mars3d-graphic-load-slow/12.png)

该函数根据给定的参数为一个字符创建一个带有填充和/或描边效果的画布（canvas）。

```
// LabelCollection.js
function createGlyphCanvas(
  character,
  font,
  fillColor,
  outlineColor,
  outlineWidth,
  style,
) {
  writeTextToCanvasParameters.font = font;
  writeTextToCanvasParameters.fillColor = fillColor;
  writeTextToCanvasParameters.strokeColor = outlineColor;
  writeTextToCanvasParameters.strokeWidth = outlineWidth;
  // Setting the padding to something bigger is necessary to get enough space for the outlining.
  writeTextToCanvasParameters.padding = SDFSettings.PADDING;

  writeTextToCanvasParameters.fill =
    style === LabelStyle.FILL || style === LabelStyle.FILL_AND_OUTLINE;
  writeTextToCanvasParameters.stroke =
    style === LabelStyle.OUTLINE || style === LabelStyle.FILL_AND_OUTLINE;
  writeTextToCanvasParameters.backgroundColor = Color.BLACK;

  return writeTextToCanvas(character, writeTextToCanvasParameters);
}
```

##### mOe | writeTextToCanvas

该函数将一段文本绘制到一个新的 HTML <canvas> 元素上，并返回这个 canvas。这个函数在 Cesium 的标签（Label）、广告牌（Billboard）等组件中广泛使用，用于动态生成带样式的文本图像： 

- 根据传入的文本和样式配置，创建一个 canvas； 
- 自动调整 canvas 尺寸以适配文本内容； 
- 支持填充（fill）、描边（stroke）、背景色、字体、字号、边距等样式； 
- 返回一个 canvas 元素，可用于后续图像处理或纹理上传； 
- 如果文本为空，返回 undefined；

```
// writeTextToCanvas.js
/**
 * Writes the given text into a new canvas.  The canvas will be sized to fit the text.
 * If text is blank, returns undefined.
 *
 * @param {string} text The text to write.
 * @param {object} [options] Object with the following properties:
 * @param {string} [options.font='10px sans-serif'] The CSS font to use.
 * @param {boolean} [options.fill=true] Whether to fill the text.
 * @param {boolean} [options.stroke=false] Whether to stroke the text.
 * @param {Color} [options.fillColor=Color.WHITE] The fill color.
 * @param {Color} [options.strokeColor=Color.BLACK] The stroke color.
 * @param {number} [options.strokeWidth=1] The stroke width.
 * @param {Color} [options.backgroundColor=Color.TRANSPARENT] The background color of the canvas.
 * @param {number} [options.padding=0] The pixel size of the padding to add around the text.
 * @returns {HTMLCanvasElement|undefined} A new canvas with the given text drawn into it.  The dimensions object
 *                   from measureText will also be added to the returned canvas. If text is
 *                   blank, returns undefined.
 * @function writeTextToCanvas
 */
function writeTextToCanvas(text, options) { ... }
```

##### hOe | measureText

该函数负责精确地测量文本的绘制区域（包括字体的 ascender、descender、起始偏移等），从而在生成文本 canvas 时能正确地设置大小和位置。它返回一个包含文本尺寸的对象： 

- width: 文本总宽度； 
- height: 文本总高度； 
- ascent: 文本上部高度（从基线到顶部）； 
- descent: 文本下部高度（从基线到底部）； 
- minx: 起始偏移（如字母 j 的点可能在左侧留白）；

```
// writeTextToCanvas.js
function measureText(context2D, textString, font, stroke, fill) { ... }
```

#### fm.update | OrderedGroundPrimitiveCollection.prototype.update

![img](../../../assets/mars3d-graphic-load-slow/13.png)

该函数用于管理贴地（ground）图元的绘制顺序，以确保它们在地形之上的正确渲染。

```
// OrderedGroundPrimitiveCollection.js
/**
 * @private
 */
OrderedGroundPrimitiveCollection.prototype.update = function (frameState) {
  if (!this.show) {
    return;
  }

  const collections = this._collectionsArray;
  for (let i = 0; i < collections.length; i++) {
    collections[i].update(frameState);
  }
};
```

##### pl.update | GroundPrimitive.prototype.update

该函数实现了将贴地图元（GroundPrimitive）转换为可渲染的图元（如 ClassificationPrimitive）并提交到渲染流程的核心逻辑，用于支持贴地图元（如多边形、折线）在地形上的正确渲染；

```
// GroundPrimitive.js
/**
 * Called when {@link Viewer} or {@link CesiumWidget} render the scene to
 * get the draw commands needed to render this primitive.
 * <p>
 * Do not call this function directly.  This is documented just to
 * list the exceptions that may be propagated when the scene is rendered:
 * </p>
 *
 * @exception {DeveloperError} For synchronous GroundPrimitive, you must call GroundPrimitive.initializeTerrainHeights() and wait for the returned promise to resolve.
 * @exception {DeveloperError} All instance geometries must have the same primitiveType.
 * @exception {DeveloperError} Appearance and material have a uniform with the same name.
 */
GroundPrimitive.prototype.update = function (frameState) { ... }
```

##### lp.update | GroundPolylinePrimitive.prototype.update

该函数包含了将贴地折线图元（GroundPolylinePrimitive）转换为可渲染的图元（Primitive）并提交到渲染流程的核心逻辑，用于支持贴地折线（如路径、边界线）在地形上的正确渲染。

```
// GroundPolylinePrimitive.js
/**
 * Called when {@link Viewer} or {@link CesiumWidget} render the scene to
 * get the draw commands needed to render this primitive.
 * <p>
 * Do not call this function directly.  This is documented just to
 * list the exceptions that may be propagated when the scene is rendered:
 * </p>
 *
 * @exception {DeveloperError} For synchronous GroundPolylinePrimitives, you must call GroundPolylinePrimitives.initializeTerrainHeights() and wait for the returned promise to resolve.
 * @exception {DeveloperError} All GeometryInstances must have color attributes to use PolylineColorAppearance with GroundPolylinePrimitive.
 */
GroundPolylinePrimitive.prototype.update = function (frameState)
```

### 3 结论

（1）项目中当前已录入的矢量对象，共计1535个。其中，直线、标签文本、贴地文字、多边形占据了大部分（1497个，占97.5%）。大部分的矢量对象所对应的BaseGraphic实现类是PolylineEntity、LabelEntity、RectangleEntity、PolygonEntity。即在Entity体系下进行渲染。mars3d对Entity体系的界定是性能较低的，矢量对象数据级在1000以内的应用场景。 

（2）在最长耗时的渲染过程中，将Entity对象转换为Primitive对象的函数执行耗时约40ms。将矢量对象的渲染从Entity体系迁移到Primitive体系，理论上可略过该过程。mars3d对Primitive体系的界定是性能中等的，矢量对象数据级在10000以内的应用场景。 

（3）mars3d所提供的矢量对象加载最高效是CombinePrimitive，其缺点是对于矢量对象的控制粒度不足。如项目中根据缩放层级显示、隐藏矢量对象的机制，在CombinePrimitive体系中不易于实现。 

（4）将矢量对象的渲染从Entity体系迁移到Primitive体系，可能会导致既有功能不再支持的情况，因为Entity.StyleOption与Primitive.StyleOption不完全一致。 

（5）在最长耗时的渲染过程(总耗时为1.69s)中，线、面等绘制耗时较低，约为20ms。而标签(Label)绘制耗时较长，约为1.53s，大部分的耗时集中在 writeTextToCanvas和measureText函数的调用中。对标签的渲染进行优化，理论上可显著降低该次渲染耗时。

（6）标签在Entity体系下对应的类是LabelEntity，其与BaseGraphic没有直接的继承关系。标签在Primitive体系下对应的类是LabelPrimitive。以LabelPrimitive替代LabelEntity所产生的优化效果，有待进一步考察和检验。
