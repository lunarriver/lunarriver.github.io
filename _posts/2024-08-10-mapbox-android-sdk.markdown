---
layout: post
title:  "Mapbox Android Sdk (v11)"
date:   2024-08-10 22:48:00 +0800
---

* 目录
{:toc #markdown-toc}

Simplify from: [Maps SDK for Android](https://docs.mapbox.com/android/maps/guides/)

## 1 Get Started

The SDK requires two pieces of sensitive information from your Mapbox account (or sign up to create one):
- A public access token
- A secret access token with the Downloads:Read scope.

There are two ways to provide your public token to Mapbox SDK:
- adding it as an Android string resource (app/src/main/res/values/mapbox_access_token.xml).
- setup MapboxOptions#accessToken at runtime.

To add the Mapbox Maps SDK as a dependency, you will need to configure your build to download the Maps SDK from Mapbox maven repository directly.

```
maven {
	url = uri("https://api.mapbox.com/downloads/v2/releases/maven")
	// Do not change the username below. It should always be "mapbox" (not your username).
	credentials.username = "mapbox"
	// Use the secret token stored in gradle.properties as the password
	credentials.password = providers.gradleProperty("MAPBOX_DOWNLOADS_TOKEN").get()
	authentication { basic(BasicAuthentication) }
}
```

You can add a map view to your application using Jetpack Compose or layouts in views (using XML or instantiate at runtime).

## 2 Markers and annotations

### Annotations

Add annotations to the map including point, circle, polyline, and polygon shapes using the Mapbox Maps SDK's Annotations API.

```
Inefficient for adding many features (> 250) to the map.
```

#### Default markers

```
// Create an instance of the Annotation API and get the PointAnnotationManager.
val annotationApi = mapView?.annotations
val pointAnnotationManager = annotationApi?.createPointAnnotationManager(mapView)
// Set options for the resulting symbol layer.
val pointAnnotationOptions: PointAnnotationOptions = PointAnnotationOptions()
	// Define a geographic coordinate.
	.withPoint(Point.fromLngLat(18.06, 59.31))
	// Specify the bitmap you assigned to the point annotation
	// The bitmap will be added to map style automatically.
	.withIconImage(YOUR_ICON_BITMAP)
// Add the resulting pointAnnotation to the map.
pointAnnotationManager?.create(pointAnnotationOptions)
```

#### Other shapes

The Annotation API also supports putting other shapes on the map including circles using CircleAnnotationManager, polylines using PolylineAnnotationManager, and polygons using PolygonAnnotationManager.

#### Interactivity

Make annotations draggable:

```
val pointAnnotationOptions: PointAnnotationOptions = PointAnnotationOptions()
    .withPoint(Point.fromLngLat(18.06, 59.31))
    .withIconImage(YOUR_ICON_BITMAP)
    // Make the annotation draggable.
    .withDraggable(true)
// Add the draggable pointAnnotation to the map.
pointAnnotationManager?.create(pointAnnotationOptions)
```

Update an annotation that's already been added:

```
// When a user clicks on a draggable menu item defined in the layout,
// then toggle whether all point annotations are draggable.
R.id.menu_action_draggable -> {
	pointAnnotationManager?.annotations?.forEach {
		it.isDraggable = !it.isDraggable
	}
}
```

### View annotations

Add a regular Android View bound to some Geometry on top of the Mapbox MapView using the View Annotations API.

```
// Define the view annotation
val viewAnnotation = viewAnnotationManager.addViewAnnotation(
	// Specify the layout resource id
	resId = R.layout.annotation_view,
	// Set any view annotation options
	options = viewAnnotationOptions {
		// View annotation is placed at the feature with `FEATURE_ID` 
                // from layer with `LAYER_ID`
		annotatedLayerFeature(LAYER_ID) {
			featureId(FEATURE_ID)
		}
	}
)
```

#### Customize appearance

The z-index of view annotations is based on the order in which they are added. Bring a view annotation on top of others regardless of the order:

```
viewAnnotationManager.updateViewAnnotation(
	existingViewAnnotation,
	viewAnnotationOptions {
		selected(true)
	}
)
```

To update the anchor position and the offset of the view annotation you can specify single or multiple anchors.

```
// set single anchor
viewAnnotationManager.updateViewAnnotation(
	existingViewAnnotation,
	viewAnnotationOptions {
		annotationAnchor {
		  // bottom of the view annotation is placed on the geometry
		  anchor(ViewAnnotationAnchor.BOTTOM)
		  offsetX(-10.0)
		  offsetY(20.0)
		}
	}
)
```

#### Attach to the layer feature

Specify the Feature to attach view annotation with the feature id and the layer id. The visibility of the feature will then determine the visibility of the view annotation.


## 3 Set a style

Across the Mapbox ecosystem, the appearance of the map is determined by the map style.

**A Mapbox style is a JSON object** that defines exactly how to draw a map. 

### Load a style

To render a map in the MapView, you can:
- rely on the default Mapbox Standard loaded by default.
- initialize the map view with constructor by styleUri property of MapInitOptions.
- load a style at any time after the map has been initialized using one of MapboxMap’s style loading methods.

Instantiate a MapView without specifying any style means your map will use Mapbox Standard.

You can set null to the styleUri of MapInitOptions or pass an empty string to loadStyle methods, then Mapbox Standard style loading will be prevent.

Mapbox-designed style's convenience variables: `Style.STANDARD`, `Style.SATELLITE`, ...

You can also change the style any time after initializing the map using any of loadStyle methods.

### Configure a style

Mapbox Standard and Mapbox Standard Satellite offer serveral configuration properties that can be used to change the appearance of the basemap using setStyleImportConfigProperty.

```
mapboxMap.loadStyle(Style.STANDARD) { style ->
	style.setStyleImportConfigProperty(styleImportId, "lightPreset", Value.valueOf("dusk"))
}
```

Mapbox Style APIs allow you to import other styles into the main style you display to your users. To import a style, you need to add an imports property to your Style JSON.

## 4 Work with layers

### Add and update layers

You can use the Maps SDK to add more styled data to the map at runtime.

There are two key concepts:
- Sources contain geographic data, determine the shape of the features.
- Layers contain styling information, determine how the data in a source should look on the map.

#### Rendering order

Typically, layers are displayed in the order of their declaration, with later layers appearing on top of earlier ones.

#### Style domain-specific language

With the Style DSL, authoring or updating map styles is like writing style JSON directly.

```
mapView.mapboxMap.loadStyle(
	style(style = Style.TRAFFIC_DAY) {
		+geoJsonSource(id = "earthquakes") {
			data(GEOJSON_URL)
			cluster(false)
		}
		+circleLayer(layerId = "earthquakeCircle", sourceId = "earthquakes") {
			circleRadius(get { literal("mag") })
			circleColor(Color.RED)
			circleOpacity(0.3)
			circleStrokeColor(Color.WHITE)
		}
	}
)
```

#### Add a layer at runtime

To add a new layer to the map at runtime, start by adding a source using the Style’s addSource method. To add a new layer to the map at runtime, start by adding a source using the Style’s addSource method.

```
mapView.mapboxMap.getStyle { style ->
	// Specify a unique string as the source ID (SOURCE_ID)
	// and reference the location of source data
	style.addSource(
		geoJsonSource(SOURCE_ID) {
			data("asset://from_crema_to_council_crest.geojson")
		}
	)
	// Specify a unique string as the layer ID (LAYER_ID)
	// and reference the source ID (SOURCE_ID) added above.
	style.addLayer(
		lineLayer(LAYER_ID, SOURCE_ID) {
			lineColor(ContextCompat.getColor(context, R.color.black))
			lineWidth(3.0)
		}
	)
}
```

#### Update a layer at runtime

You can also update the style of any layer at runtime using the layer's unique layer ID.

```
// Get the style
mapView.mapboxMap.getStyle { style ->
	// Get an existing layer by referencing its
	// unique layer ID (LAYER_ID)
	val layer = style.getLayerAs<FillLayer>(LAYER_ID)
	// Update layer properties
	layer?.fillOpacity(0.7)
}
```

#### Specify order of a layer at runtime

You can change the position of a layer at runtime using the moveStyleLayer API.

```
// Get the style
mapView.mapboxMap.getStyle { style ->
	// Move position of the population layer
	// below the state-labels layer
	style.moveStyleLayer("population", LayerPosition(null, "state-labels", null))
}
```

#### Remove a layer at runtime

You can remove a layer from a style using Style's removeStyleLayer.

```
// Where LAYER_ID is a valid id of a layer that already
// exists in the style
mapView.mapboxMap.getStyle { style ->
	// If a layer with a given layer ID exists in the
	// style, remove the layer
	if (style.styleLayerExists(LAYER_ID)) {
		style.removeStyleLayer(LAYER_ID)
	}
}
```

### Source types

#### Vector

A vector source is a vector tileset that conforms to the [Mapbox Vector Tile](https://docs.mapbox.com/data/tilesets/guides/vector-tiles-standards/) format.

A vector source contains geographic features (and their data properties) that have already been tiled.

#### GeoJSON

A GeoJSON source is data in the form of a JSON object that conforms to the [GeoJSON specification](https://geojson.org/). 

A GeoJSON source is a collection of one or more geographic features, which may be points, lines, and polygons.

#### Raster

A raster source is a raster tileset. 

#### Image

An image source is an image that you supply along with geographic coordinates.

### Layer types

#### Fill layer

A fill style layer renders one or more filled (and optionally stroked) polygons on a map.

You need to first add a vector or GeoJSON source that contains polygon data.

#### Line layer

A line style layer renders one or more stroked polylines on the map.

You need to first add a vector or GeoJSON source that contains line data.

#### Symbol layer

A symbol style layer renders icon and text labels at points or along lines on a map.

You need to first add a vector or GeoJSON source that contains point data.

If you want to use icons in this layer, you also need to add images to the style before adding the layer.

#### Circle layer

A circle style layer renders one or more filled circles on a map.

You need to first add a vector or GeoJSON source that contains point data.

#### Fill extrusion layer

A fill-extrusion style layer renders one or more filled (and optionally stroked) extruded (3D) polygons on a map.

You need to first add a vector or GeoJSON source that contains polygon data.

Often you will want the data to contain a data property that you will use to determine the height of extrusion of each feature.

#### Heatmap layer

A heatmap style layer renders a range of colors to represent the density of points in an area.

You need to first add a vector or GeoJSON source that contains point data. 

#### Raster layer

A raster style layer renders raster tiles on a map.

You need to first add a raster source. 

#### Sky layer

A sky style layer renders a stylized spherical dome that encompasses the entire map and is automatically rendered behind all layers.

This can be used to fill the area above the horizon with a simulated sky that illustrates a particular time-of-day, or stylized custom gradients.

#### Background layer

The background style layer covers the entire map.

This can be used to configure a color or pattern to show below all other map content.

## 5 Styling your layers

### Work with expressions

You can specify the value for any layout property, paint property, or filter as an expression.
- A property expression is any expression defined using a reference to feature property data.
- A camera expression is any expression defined using ['zoom'].

#### Syntax

Expressions follow this format:

```
expression_name {
	argument_0
	argument_1
}
```

The expression_name is the expression operator.

The arguments are either literal (numbers, strings, or boolean values) or else themselves expressions. The number of arguments varies based on the expression.

Here's a (π * 3^2) expression:

```
product {
	pi()
	pow {
		literal(3)
		literal(2)
	}
}
```

#### Expression types

- **Mathematical operators** for performing arithmetic and other operations on numeric values
- **Logical operators** for manipulating boolean values and making conditional decisions
- **String operators** for manipulating strings
- **Data operators** for providing access to the properties of source features
- **Camera operators** for providing access to the parameters defining the current map view

### Data-driven styling at runtime

You can use expressions to determine the style of features in a layer based on data properties in the source data.

To read the values of the data property, you can use the **get** expression.

### Zoom-driven styling at runtime

You can use expressions to determine the style of features in a layer based on the camera position, including the zoom level.

The **zoom** expression will return the value of the current zoom level of the map.

## 6 Camera position

The camera's location and behavior is defined by its properties:
- **center**: The longitude and latitude at which the camera is pointed.
- **bearing**: The visual rotation of the map. The bearing value is the compass direction the camera points to show the user which way is "up". For example, a bearing of 90° orients the map so that east is up.
- **pitch**: The visual tilt of the map. A pitch of 0° is perpendicular to the surface, looking straight down at the map, while a greater value like 60° looks ahead towards the horizon.
- **zoom**: The zoom level specifies how close the camera is to the features being viewed. 
  - At zoom level **0**, the viewport shows continents and oceans. 
  - A middle value of **11** shows city-level details, 
  - and at a higher zoom level the map begins to show buildings and points of interest.
- **padding**: Insets from each edge of the map. The padding value impacts the location at which the center point is rendered.
- **anchor**: The point in the map's coordinate system around which zoom and bearing are applied. Mutually exclusive with center.

### Set camera position

#### Set camera on map initialization

```
val initialCameraOptions = CameraOptions.Builder()
	.center(Point.fromLngLat(-74.0066, 40.7135))
	.pitch(45.0)
	.zoom(15.5)
	.bearing(-17.6)
	.build()

val mapInitOptions = MapInitOptions(
	context = this,
	mapOptions = mapOptions,
	plugins = plugins,
	cameraOptions = initialCameraOptions,
	textureView = true
)
mapView = MapView(this, mapInitOptions)
```

If these properties are not defined in the style JSON, the map will be centered on the coordinates 0,0 with a bearing and pitch of 0 at zoom level 0.

#### Set after map initialization

```
val cameraPosition = CameraOptions.Builder()
	.zoom(14.0)
	.center(it.point)
	.build()
// set camera position
mapView.mapboxMap.setCamera(cameraPosition)
```

#### Fit the camera to a given shape

MapboxMap includes a few convenience methods to generate a CameraOptions based on given coordinates or geometries:
- Use `cameraForCoordinateBounds` to fit the camera to a set of rectangular coordinate bounds (in other words, a bounding box).
- Use `cameraForCoordinates` to fit a collection of coordinates.
- Use `cameraForGeometry` to fit a given geometry.

### Listen for camera changes

```
val callback = CameraChangedCallback { cameraChanged ->
	// Do something when the camera position changes
}
// val cancelable = mapboxMap.subscribeCameraChanged(callback)
val cancelable = mapboxMap.subscribeCameraChanged(callback)

...

// To cancel the subscription
cancelable.cancel()
```

### Get camera position

```
// Get the `cameraState.center`
val center = mapView.mapboxMap.cameraState.center
```

### Restrict camera

Use MapboxMap's setBounds function to restrict a user's panning to limit the map camera to a chosen area.

```
val cameraBoundsOptions = CameraBoundsOptions.Builder()
	.bounds(
		CoordinateBounds(
			Point.fromLngLat(-122.66336, 37.492987),
			Point.fromLngLat(-122.250481, 37.87165),
			false
		)
	)
  .minZoom(10.0)
  .build()
// Fit camera to the bounding box
mapboxMap.setBounds(cameraBoundsOptions)
```

## 7 Animations

You can animate the camera to move to a new center location and to update the bearing, pitch, zoom, padding, and anchor. 

### High-level animation APIs

The Maps SDK has a set of ready-to-use functions for animating map camera transitions.

The built-in animation types are:
- **flyTo** changes the camera's position from the old values to the new values using a combination of zooming and panning in an animated transition that evokes flight.
- **easeTo** gradually changes the camera's position with an animated transition from the old values to the new values.
- **rotateBy** rotates the map to a different bearing with an animated transition.
- **scaleBy** animates a change in the size of the images on the map.
- **pitchBy** animates a change in the pitch of the map.
- **moveBy** instantaneously re-positions the camera with a quick animated transition from the old values to the new values.

You can call these functions with the MapboxMap object.

It's important to note that only one high-level animation can run at a time.

### Mid-level animation APIs

Mid-level animation APIs offer more flexibility, but require writing more custom code.

Any number of animators extending CameraAnimator can be created and started either sequentially or in parallel.

```
import com.mapbox.maps.plugin.animation.CameraAnimatorOptions.Companion.cameraAnimatorOptions

mapView.camera.apply {
    val bearing = createBearingAnimator(cameraAnimatorOptions(18.0, 20.0) { startValue(15.0) }) {
		duration = 8500
		interpolator = AnticipateOvershootInterpolator()
    }
    val pitch = createPitchAnimator(cameraAnimatorOptions(30.0) { startValue(15.0) }) {
		duration = 2000
    }
    playAnimatorsTogether(bearing, pitch)
}
```

### Low-level animation APIs

With the low-level animation APIs, any number of animators extending CameraAnimator can be created, but you must also register them in the CameraAnimationsPlugin.

To start camera animations created using the low-level animation APIs, use the CameraAnimator.start method. For the animator to start, you need to register it first using CameraAnimationsPlugin.registerAnimators.

```
import com.mapbox.maps.plugin.animation.CameraAnimatorOptions.Companion.cameraAnimatorOptions

val plugin = mapView.camera
val bearing = plugin.createBearingAnimator(cameraAnimatorOptions(160.0) { startValue = 0.0 }) {
	duration = 8500
	interpolator = AnticipateOvershootInterpolator()
}
plugin.registerAnimators(bearing)

bearing.apply {
	addListener(
		object : Animator.AnimatorListener {
			override fun onAnimationStart(animation: Animator) {}

			override fun onAnimationEnd(animation: Animator) {
				(animation as? CameraAnimator<*>)?.let {
					plugin.unregisterAnimators(it)
				}
			}

			override fun onAnimationCancel(animation: Animator) {
				(animation as? CameraAnimator<*>)?.let {
					plugin.unregisterAnimators(it)
				}
			}

			override fun onAnimationRepeat(animation: Animator) {}
		}
	)
	start()
}
```

### Listen for camera animation events

#### Listen for high-level animation API events

```
mapboxMap.flyTo(
	cameraOptions = cameraOptions,
	animatorListener = object : Animator.AnimatorListener {
		override fun onAnimationStart(animation: Animator) {}

		override fun onAnimationEnd(animation: Animator) {}

		override fun onAnimationCancel(animation: Animator) {}

		override fun onAnimationRepeat(animation: Animator) {}
	}
)
```

#### Listen for lower-level animation API events

You can add the regular Android SDK listener as CameraAnimator.addListener(listener: AnimatorListener) to any CameraAnimator, which extends Android's ValueAnimator with all existing functionality.

```
lowOrMidLevelAnimator.addListener(
	object : Animator.AnimatorListener {
		override fun onAnimationStart(animation: Animator) {}

		override fun onAnimationEnd(animation: Animator) {}

		override fun onAnimationCancel(animation: Animator) {}

		override fun onAnimationRepeat(animation: Animator) {}
	}
)
```

#### Listen for all animation events

If you want to listen to all events happening to the camera animation system, use `CameraAnimationsPlugin`'s `addCameraAnimationsLifecycleListener()` method.

#### Listen for specific animation value change events

The Maps SDK `CameraAnimationsPlugin` also provides methods to listen to specific value change events.

## 8 User interaction

Users interacting with the Mapbox map in your application can explore the map by performing standard Android gestures on the touchscreen.

The Mapbox Gestures for Android library wraps GestureDetectorCompat and ScaleGestureDetector. It also introduces implementations of rotate, move, shove, and tap gesture detectors.

### Default map gestures

By default the following standard Android gestures will allow the user to explore the map:
- **Scroll around**: Hold one finger down on the screen and move it in any direction.
- **Adjust pitch**: Hold two fingers down on the screen and move them vertically across the screen.
- **Gradually zoom in/out**: Pinch with two fingers to adjust the zoom level. Move fingers apart to zoom in, move fingers closer together to zoom out.
- **Rotate**: Hold two fingers down on the screen and move them in a circular motion to rotate the map (adjust the bearing).
- **Zoom in one zoom level**: Double tap on the screen with one finger to zoom in on the map's anchor point.
- **Zoom out one zoom level**: Double tap on the screen with two fingers to zoom out with the map's anchor point centered.
- **Quick zoom**: Double tap and drag up on the screen to zoom out, or double tap and drag down to zoom in.

### Enable or disable default gestures

You can enabled or disabled default gestures by the GestureSettings. For example,

```
mapView.gestures.pitchEnabled = false
```

There are additional configuration options for scroll gestures:
- Use scrollMode to configure the directions in which the map is allowed to move during a scroll gesture. By default, the map will move both horizontally and vertically.
- Use scrollDecelerationEnabled to enable or disable the fling behavior after the scroll ends.

You can also configure pinch behavior by the GestureSettings.

### Listen for gesture events

The GesturesPlugin provides many gesture listeners. For example,

```
val rotateListener = object : OnRotateListener {
	override fun onRotateBegin(@NonNull detector: RotateGestureDetector) {}

	override fun onRotate(@NonNull detector: RotateGestureDetector) {}

	override fun onRotateEnd(@NonNull detector: RotateGestureDetector) {}
}
gesturesPlugin.addOnRotateListener(rotateListener)

// don't forget to remove
gesturesPlugin.removeOnRotateListener(rotateListener)
```
## 9 Offline

skip...

## 10 Cache management

When a user loads and interacts with a map on their device, any visible tiles and style resources (style JSON, fonts, sprites, etc.) are placed in the device's disk cache.

The disk cache is located in the maps data directory defined in ResourceOptions.dataPath.

The disk cache observes when these resources are used and makes intelligent guesses about which resources may be needed again.

### Cache behavior

![img](../../../assets/mapbox-android-sdk/cache-tile-fetching-scheme.png)

The response for the "update network tile" request is stored in the disk cache when received.

The ETag is used to determine if the data on the client is still valid. 

If the ETag in the request matches the ETag on the server, a "Not Modified" response is sent along with the new expiration timestamp. In this case, the data payload is not sent, saving network traffic.

#### Storage & volatile tiles

The SDK provides an option to change this behavior so that tile responses are not saved to the disk cache, but still get cached in memory.

This property is defined by setting the source to volatile - either through SDK methods or by adding "volatile": true to the tile JSON. This makes it so volatile resources are not saved between application runs.

![img](../../../assets/mapbox-android-sdk/cache-volatile-tile-fetching-scheme.png)

### Default cache location

The disk cache (map_data.db) is located at ./files/.mapbox/map_data/map_data.db.

### Clearing the cache

The clearData method can be used to clear the temporary map data from the data path defined in the given resource options.

### Tile loading

You can use the subscribeResourceRequest event to observe all resource requests the map makes.

For modifying the default tile requesting behavior for vector, raster and raster-dem sources, you may use the following methods:
- **minimumTileUpdateInterval**: Overrides the server expiration time on the client in cases where it may be expiring too often or too rarely.
- **tileRequestsDelay**: Sets a delay for tile requests to any local storage (both disk cache and tile store), represented by a floating point value in milliseconds.
- **tileNetworkRequestsDelay**: Sets a delay for tile requests from the network, represented by a floating point value in milliseconds.