---
layout: post
title:  "React Native 核心三方库"
date:   2025-11-10 18:42:00 +0800
---

* 目录
{:toc #markdown-toc}

### react-native-safe-area-context

A library with a flexible API for accessing the device's safe area inset information.

`react-native-safe-area-context` provides a flexible API for accessing device safe area inset information. This allows you to position your content appropriately around notches, status bars, home indicators, and other such device and operating system interface elements. It also provides a `SafeAreaView` component that you can use in place of `View` to automatically inset your views to account for safe areas.

- github: https://github.com/AppAndFlow/react-native-safe-area-context

- document: https://appandflow.github.io/react-native-safe-area-context/

- blog: https://www.jianshu.com/p/ae55813d4ce7

#### SafeAreaProvider

You should add SafeAreaProvider in your app root component.

```
import { SafeAreaProvider } from 'react-native-safe-area-context';

function App() {
  return <SafeAreaProvider>{/*...*/}</SafeAreaProvider>;
}
```

#### SafeAreaView

SafeAreaView is a regular View component with the safe area insets applied as padding or margin.

```
import { SafeAreaView } from 'react-native-safe-area-context';

function SomeComponent() {
  return (
    <SafeAreaView style={{ flex: 1, backgroundColor: 'red' }}>
      <View style={{ flex: 1, backgroundColor: 'blue' }} />
    </SafeAreaView>
  );
}
```

#### useSafeAreaInsets

Returns the safe area insets of the nearest provider. This allows manipulating the inset values from JavaScript. Note that insets are not updated synchronously so it might cause a slight delay for example when rotating the screen.

```
import { useSafeAreaInsets } from 'react-native-safe-area-context';

function HookComponent() {
  const insets = useSafeAreaInsets();

  return <View style={{ paddingBottom: Math.max(insets.bottom, 16) }} />;
}
```

#### useSafeAreaFrame

Returns the frame of the nearest provider. This can be used as an alternative to the Dimensions module.

### react-native-screens

A library that provides native primitives to represent screens for better operating system behavior and screen optimizations.

`react-native-screens` provides native primitives to represent screens instead of plain `<View>` components To better take advantage of operating system behavior and optimizations around screens. This capability is used by library authors and **is unlikely to be used directly by most app developers**. It also provides the native components needed for React Navigation's `createNativeStackNavigator`.

- github: https://github.com/software-mansion/react-native-screens

### react-native-config

Module to expose config variables to your javascript code in React Native, supporting iOS, Android, macOS and Windows.

Create a new file .env in the root of your React Native app:

```
API_URL=https://myapi.com
GOOGLE_MAPS_API_KEY=abcdefgh
```

Then access variables defined there from your app:

```
import Config from "react-native-config";

Config.API_URL; // 'https://myapi.com'
Config.GOOGLE_MAPS_API_KEY; // 'abcdefgh'
```

- github: https://github.com/lugg/react-native-config

- blog: https://juejin.cn/post/6999295392139444232

#### Different environments

Save config for different environments in different files: `.env.staging`, `.env.production`, etc.

By default react-native-config will read from `.env`, but you can change it when building or releasing your app.

The simplest approach is to tell it what file to read with an environment variable, like:

```
$ ENVFILE=.env.staging react-native run-ios           # bash
$ SET ENVFILE=.env.staging && react-native run-ios    # windows
$ env:ENVFILE=".env.staging"; react-native run-ios    # powershell
```

This also works for `run-android`. 

### react-native-image-picker

A React Native module that allows you to select a photo/video from the device library or camera.

For Android, No permissions required (saveToPhotos requires permission check). This library does not require Manifest.permission.CAMERA, if your app declares as using this permission in manifest then you have to obtain the permission before using launchCamera.

```
import {launchCamera, launchImageLibrary} from 'react-native-image-picker';
```

#### launchCamera()

Launch camera to take photo or video.

```
launchCamera(options?, callback);

// You can also use as a promise without 'callback':
const result = await launchCamera(options?);
```

#### launchImageLibrary

Launch gallery to pick image or video.

```
launchImageLibrary(options?, callback)

// You can also use as a promise without 'callback':
const result = await launchImageLibrary(options?);
```

- github: https://github.com/react-native-image-picker/react-native-image-picker

- implement: https://android-docs.cn/training/data-storage/shared/photopicker

### react-native-image-resizer

A React Native module that can create scaled versions of local images (also supports the assets library on iOS).

```
import ImageResizer from 'react-native-image-resizer';

ImageResizer.createResizedImage(
  path,
  maxWidth,
  maxHeight,
  compressFormat,
  quality,
  rotation,
  outputPath
)
  .then((response) => {
    // response.uri is the URI of the new image that can now be displayed, uploaded...
    // response.path is the path of the new image
    // response.name is the name of the new image with the extension
    // response.size is the size of the new image
  })
  .catch((err) => {
    // Oops, something went wrong. Check that the filename is correct and
    // inspect err to get more details.
  });
```

- github: https://github.com/bamlab/react-native-image-resizer

- blog: https://blog.csdn.net/gitblog_00970/article/details/141078638

### react-native-webview-invoke

Invoke functions between React Native and WebView directly.

- github: https://github.com/pinqy520/react-native-webview-invoke?tab=readme-ov-file#readme

#### React Native Side

```
import createInvoke from 'react-native-webview-invoke/native'


class SomePage extends React.Component {
    webview: WebView
    invoke = createInvoke(() => this.webview)
    
    function whatIsTheNameOfA() {
        return 'A'
    }
    
    function tellAYouArea(someone: string, prefix: string) {
        return 'Hi, ' + prefix + someone + '!'
    }
    
    invoke
        .define('whatIsTheNameOfA', whatIsTheNameOfA)
        .define('tellAYouArea', tellAYouArea)
    
    render() {
        // Note: add 'useWebKit' property for rn > 0.59
        return <Webview useWebKit
            ref={webview => this.webview = webview}
            onMessage={this.invoke.listener}
            source={require('./index.html')}
            />
    }
}
```

#### Web Side

```
import invoke from 'react-native-webview-invoke/browser'

const whatIsTheNameOfA = invoke.bind('whatIsTheNameOfA')
const tellAYouArea = invoke.bind('tellAYouArea')

await whatIsTheNameOfA()
// 'A'
await tellAYouArea('B', 'Mr.')
// 'Hi, Mr.B'
```

### react-native-toast-message

Animated toast message component for React Native.

Render the Toast component in your app's entry file, as the LAST CHILD in the View hierarchy (along with any other components that might be rendered there):

```
import Toast from 'react-native-toast-message';

export function App(props) {
  return (
    <>
      {/* ... */}
      <Toast />
    </>
  );
}
```

Then use it anywhere in your app (even outside React components), by calling any Toast method directly:

```
import Toast from 'react-native-toast-message';
import { Button } from 'react-native'

export function Foo(props) {
  const showToast = () => {
    Toast.show({
      type: 'success',
      text1: 'Hello',
      text2: 'This is some something 👋'
    });
  }

  return (
    <Button
      title='Show toast'
      onPress={showToast}
    />
  )
}
```

- github: https://github.com/calintamas/react-native-toast-message

### react-native-device-info

Device Information for React Native.

```
import DeviceInfo from 'react-native-device-info';
 
console.log("Device Unique ID", DeviceInfo.getUniqueID());  // e.g. FCDBD8EF-62FC-4ECB-B2F5-92C9E79AC7F9
// * note this is IDFV on iOS so it will change if all apps from the current apps vendor have been previously uninstalled
console.log("Device Manufacturer", DeviceInfo.getManufacturer());  // e.g. Apple
console.log("Device Model", DeviceInfo.getModel());  // e.g. iPhone 6
console.log("Device ID", DeviceInfo.getDeviceId());  // e.g. iPhone7,2 / or the board on Android e.g. goldfish
console.log("Device Name", DeviceInfo.getSystemName());  // e.g. iPhone OS
console.log("Device Version", DeviceInfo.getSystemVersion());  // e.g. 9.0
console.log("Bundle Id", DeviceInfo.getBundleId());  // e.g. com.learnium.mobile
console.log("Build Number", DeviceInfo.getBuildNumber());  // e.g. 89
console.log("App Version", DeviceInfo.getVersion());  // e.g. 1.1.0
console.log("App Version (Readable)", DeviceInfo.getReadableVersion());  // e.g. 1.1.0.89
console.log("Device Name", DeviceInfo.getDeviceName());  // e.g. Becca's iPhone 6
console.log("User Agent", DeviceInfo.getUserAgent()); // e.g. Dalvik/2.1.0 (Linux; U; Android 5.1; Google Nexus 4 - 5.1.0 - API 22 - 768x1280 Build/LMY47D)
console.log("Device Locale", DeviceInfo.getDeviceLocale()); // e.g en-US
console.log("Device Country", DeviceInfo.getDeviceCountry()); // e.g US
```

- github: https://github.com/react-native-device-info/react-native-device-info

### react-native-exit-app

Exit / Close / Kill / shutdown your react native app. Does not invoke a crash notification.

```
import RNExitApp from 'react-native-exit-app';

RNExitApp.exitApp();
```

- github: https://github.com/wumke/react-native-exit-app

### react-native-fast-image

`FastImage` is a drop-in replacement for React Native’s `Image` component, offering solutions for common image loading challenges like: Flickering during loading,  Cache inconsistencies, Slow loading from cache, Overall suboptimal performance, FastImage leverages SDWebImage (iOS) and Glide (Android) for native caching and high efficiency.

```
import FastImage from "@d11/react-native-fast-image";
import * as React from "react";

const YourImage = () => (
  <FastImage
    style={{ width: 200, height: 200 }}
    source={{
      uri: "https://unsplash.it/400/400?image=1",
      headers: { Authorization: "someAuthToken" },
      priority: FastImage.priority.normal,
    }}
    resizeMode={FastImage.resizeMode.contain}
  />
);
```

- github: https://github.com/ds-horizon/react-native-fast-image

### react-native-gesture-flip-card

A pure javascript implementation of a flip card animation using gesture for React Native.

```
import GestureFlipView from 'react-native-gesture-flip-card';

<View style={styles.container}>
  <GestureFlipView
    width={300}
    height={500}
    renderBack={renderBack}
    renderFront={renderFront}
    onFaceChanged={(face) => {
      // trigger when card face changed
      console.log('face changed:', face);
    }}
    onFlipEnd={(face) => {
      // trigger when flip animation ended
      console.log('on flip end:', face);
    }}
  />
</View>

const renderFront = () => {
  return (
    <View style={styles.frontStyle}>
      <Text style={{fontSize: 25, color: '#fff'}}>{'Front'}</Text>
    </View>
  );
};

const renderBack = () => {
  return (
    <View style={styles.backStyle}>
      <Text style={{fontSize: 25, color: '#fff'}}>{'Back'}</Text>
    </View>
  );
};
```

- github: https://github.com/JungHsuan/react-native-gesture-flip-card

### react-native-image-crop-picker

iOS/Android image picker with support for multiple images and cropping

NOTE: This library is result of one-night hacking, so please use it with caution. Don't assume there are no bugs. It is tested just on simple cases.

```
import ImagePicker from 'react-native-image-crop-picker';

ImagePicker.openPicker({
  width: 300,
  height: 400,
  cropping: true
}).then(image => {
  console.log(image);
});

ImagePicker.openPicker({
  multiple: true
}).then(images => {
  console.log(images);
});
```

- github: https://github.com/ivpusic/react-native-image-crop-picker

### react-native-linear-gradient

A `<LinearGradient>` element for React Native.

```
import LinearGradient from 'react-native-linear-gradient';

<LinearGradient
  start={{x: 0.0, y: 0.25}} end={{x: 0.5, y: 1.0}}
  locations={[0,0.5,0.6]}
  colors={['#4c669f', '#3b5998', '#192f6a']}
  style={styles.linearGradient}>
  <Text style={styles.buttonText}>
    Sign in with Facebook
  </Text>
</LinearGradient>
```

- github: https://github.com/react-native-linear-gradient/react-native-linear-gradient

### react-native-network-info

React Native library for getting information about the devices network.

```
import {NetworkInfo} from 'react-native-network-info';

// Get Local IP
NetworkInfo.getIPAddress().then(ipAddress => {
  console.log(ipAddress);
});

// Get IPv4 IP (priority: WiFi first, cellular second)
NetworkInfo.getIPV4Address().then(ipv4Address => {
  console.log(ipv4Address);
});

...
```

- github: https://github.com/pusherman/react-native-network-info

### react-native-outside-press

airbnb/react-outside-click-handler but for React Native.

Wrap your app with EventProvider.

```
import { EventProvider } from 'react-native-outside-press';

export default function App() {
  return (
    <EventProvider>
      <RestOfYourApp />
    </EventProvider>
  );
}
```

Wrap every component you want to detect outside press with OutsidePressHandler.

```
import { View } from 'react-native';
import OutsidePressHandler from 'react-native-outside-press';

export default function MyComponent() {
  return (
    <OutsidePressHandler
      onOutsidePress={() => {
        console.log('Pressed outside the box!');
      }}
    >
      <View style={{ height: 200, width: 200, backgroundColor: 'black' }} />
    </OutsidePressHandler>
  );
}
```

- github: https://github.com/dcangulo/react-native-outside-press

### react-native-progress

Progress indicators and spinners for React Native using React Native SVG.

```
import * as Progress from 'react-native-progress';

<Progress.Bar progress={0.3} width={200} />
<Progress.Pie progress={0.4} size={50} />
<Progress.Circle size={30} indeterminate={true} />
<Progress.CircleSnail color={['red', 'green', 'blue']} />
```

- github: https://github.com/oblador/react-native-progress

### react-native-ratings

An interactive rating component for React Native, which can display ratings using stars, hearts, emojis, or custom symbols of your choice. The component leverages the PanResponder and Animated APIs to create high-performing animations. It is written in TypeScript and has zero dependencies.

```
import React, { useCallback, useState } from 'react';
import { StyleSheet, Text, View } from 'react-native';
import { Rating } from '@kolking/react-native-rating';

const App = () => {
  const [rating, setRating] = useState(0);

  const handleChange = useCallback(
    (value: number) => setRating(Math.round((rating + value) * 5) / 10),
    [rating],
  );

  return (
    <View style={styles.root}>
      <Rating size={40} rating={rating} onChange={handleChange} />
      <Text style={styles.text}>Rated {rating} out of 5</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  root: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
  text: {
    fontSize: 17,
    marginTop: 20,
  },
});

export default App;
```

- github: https://github.com/kolking/react-native-rating

### react-native-render-html

An iOS/Android pure javascript react-native component that renders your HTML into 100% native views.

```
import React from 'react';
import { useWindowDimensions } from 'react-native';
import RenderHtml from 'react-native-render-html';

const source = {
  html: `
<p style='text-align:center;'>
  Hello World!
</p>`
};

export default function App() {
  const { width } = useWindowDimensions();
  return (
    <RenderHtml
      contentWidth={width}
      source={source}
    />
  );
}
```

- github: https://github.com/meliorence/react-native-render-html

### react-native-sound

React Native module for playing sound clips on iOS, Android, and Windows. Be warned, this software is alpha quality and may have bugs. Test on your own and use at your own risk!

```
// Import the react-native-sound module
var Sound = require('react-native-sound');

// Enable playback in silence mode
Sound.setCategory('Playback');

// Load the sound file 'whoosh.mp3' from the app bundle
// See notes below about preloading sounds within initialization code below.
var whoosh = new Sound('whoosh.mp3', Sound.MAIN_BUNDLE, (error) => {
  if (error) {
    console.log('failed to load the sound', error);
    return;
  }
  // loaded successfully
  console.log('duration in seconds: ' + whoosh.getDuration() + 'number of channels: ' + whoosh.getNumberOfChannels());

  // Play the sound with an onEnd callback
  whoosh.play((success) => {
    if (success) {
      console.log('successfully finished playing');
    } else {
      console.log('playback failed due to audio decoding errors');
    }
  });
});
```

- github: https://github.com/zmxv/react-native-sound

### react-native-swiper

The best Swiper component for React Native.

```
import React, { Component } from 'react'
import { AppRegistry, StyleSheet, Text, View } from 'react-native'

import Swiper from 'react-native-swiper'

const styles = StyleSheet.create({
  wrapper: {},
  slide1: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#9DD6EB'
  },
  slide2: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#97CAE5'
  },
  slide3: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#92BBD9'
  },
  text: {
    color: '#fff',
    fontSize: 30,
    fontWeight: 'bold'
  }
})

export default class SwiperComponent extends Component {
  render() {
    return (
      <Swiper style={styles.wrapper} showsButtons={true}>
        <View style={styles.slide1}>
          <Text style={styles.text}>Hello Swiper</Text>
        </View>
        <View style={styles.slide2}>
          <Text style={styles.text}>Beautiful</Text>
        </View>
        <View style={styles.slide3}>
          <Text style={styles.text}>And simple</Text>
        </View>
      </Swiper>
    )
  }
}

AppRegistry.registerComponent('myproject', () => SwiperComponent)
```

- github: https://github.com/leecade/react-native-swiper

### react-native-system-navigation-bar

React Native lets you customize the navigation bar for Android.

```
import SystemNavigationBar from 'react-native-system-navigation-bar';

SystemNavigationBar.navigationHide();
SystemNavigationBar.navigationShow();
SystemNavigationBar.leanBack();
SystemNavigationBar.immersive();
...
```

- github: https://github.com/kadiraydinli/react-native-system-navigation-bar

### react-native-view-shot

Capture a React Native view to an image.

```
import ViewShot from "react-native-view-shot";

function ExampleCaptureOnMountManually {
  const ref = useRef();

  useEffect(() => {
    // on mount
    ref.current.capture().then(uri => {
      console.log("do something with ", uri);
    });
  }, []);

  return (
    <ViewShot ref={ref} options={{ fileName: "Your-File-Name", format: "jpg", quality: 0.9 }}>
      <Text>...Something to rasterize...</Text>
    </ViewShot>
  );
}
```

- github: https://github.com/gre/react-native-view-shot