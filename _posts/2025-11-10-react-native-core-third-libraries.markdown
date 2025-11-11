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
- npm: https://www.npmjs.com/package/react-native-safe-area-context?activeTab=versions
- document: https://appandflow.github.io/react-native-safe-area-context/
- blog: https://www.jianshu.com/p/ae55813d4ce7

**SafeAreaProvider**

You should add SafeAreaProvider in your app root component.

```
import { SafeAreaProvider } from 'react-native-safe-area-context';

function App() {
  return <SafeAreaProvider>{/*...*/}</SafeAreaProvider>;
}
```

**SafeAreaView**

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

**useSafeAreaInsets**

Returns the safe area insets of the nearest provider. This allows manipulating the inset values from JavaScript. Note that insets are not updated synchronously so it might cause a slight delay for example when rotating the screen.

```
import { useSafeAreaInsets } from 'react-native-safe-area-context';

function HookComponent() {
  const insets = useSafeAreaInsets();

  return <View style={{ paddingBottom: Math.max(insets.bottom, 16) }} />;
}
```

**useSafeAreaFrame**

Returns the frame of the nearest provider. This can be used as an alternative to the Dimensions module.

### react-native-screens

- github: https://github.com/software-mansion/react-native-screens
- npm: https://www.npmjs.com/package/react-native-screens?activeTab=versions

A library that provides native primitives to represent screens for better operating system behavior and screen optimizations.

`react-native-screens` provides native primitives to represent screens instead of plain `<View>` components To better take advantage of operating system behavior and optimizations around screens. This capability is used by library authors and **is unlikely to be used directly by most app developers**. It also provides the native components needed for React Navigation's `createNativeStackNavigator`.

### react-native-config

- github: https://github.com/lugg/react-native-config
- npm: https://www.npmjs.com/package/react-native-config?activeTab=versions
- blog: https://juejin.cn/post/6999295392139444232

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

**Different environments**

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

- github: https://github.com/react-native-image-picker/react-native-image-picker
- npm: https://www.npmjs.com/package/react-native-image-picker?activeTab=versions
- android implement: https://android-docs.cn/training/data-storage/shared/photopicker

A React Native module that allows you to select a photo/video from the device library or camera.

For Android, No permissions required (saveToPhotos requires permission check). This library does not require Manifest.permission.CAMERA, if your app declares as using this permission in manifest then you have to obtain the permission before using launchCamera.

```
import {launchCamera, launchImageLibrary} from 'react-native-image-picker';
```

**launchCamera()**

Launch camera to take photo or video.

```
launchCamera(options?, callback);

// You can also use as a promise without 'callback':
const result = await launchCamera(options?);
```

**launchImageLibrary**

Launch gallery to pick image or video.

```
launchImageLibrary(options?, callback)

// You can also use as a promise without 'callback':
const result = await launchImageLibrary(options?);
```

### react-native-image-resizer

- github: https://github.com/bamlab/react-native-image-resizer
- npm: https://www.npmjs.com/package/@bam.tech/react-native-image-resizer?activeTab=versions
- blog: https://blog.csdn.net/gitblog_00970/article/details/141078638

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

### react-native-webview-invoke

Invoke functions between React Native and WebView directly.

- github: https://github.com/pinqy520/react-native-webview-invoke?tab=readme-ov-file#readme
- npm: https://www.npmjs.com/package/react-native-webview-invoke?activeTab=versions

**React Native Side**

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

**Web Side**

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

- github: https://github.com/calintamas/react-native-toast-message
- npm: https://www.npmjs.com/package/react-native-toast-message?activeTab=versions

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

### react-native-device-info

- github: https://github.com/react-native-device-info/react-native-device-info
- npm: https://www.npmjs.com/package/react-native-device-info?activeTab=versions

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

### react-native-exit-app

- github: https://github.com/wumke/react-native-exit-app
- npm: https://www.npmjs.com/package/react-native-exit-app?activeTab=versions

Exit / Close / Kill / shutdown your react native app. Does not invoke a crash notification.

```
import RNExitApp from 'react-native-exit-app';

RNExitApp.exitApp();
```

### react-native-fast-image

- github: https://github.com/ds-horizon/react-native-fast-image
- npm: https://www.npmjs.com/package/react-native-fast-image?activeTab=versions

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

### react-native-gesture-flip-card

- github: https://github.com/JungHsuan/react-native-gesture-flip-card
- npm: https://www.npmjs.com/package/react-native-gesture-flip-card?activeTab=versions

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

### react-native-image-crop-picker

- github: https://github.com/ivpusic/react-native-image-crop-picker
- npm: https://www.npmjs.com/package/react-native-image-crop-picker?activeTab=versions

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

### react-native-linear-gradient

- github: https://github.com/react-native-linear-gradient/react-native-linear-gradient
- npm: https://www.npmjs.com/package/react-native-linear-gradient?activeTab=versions

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

### react-native-network-info

- github: https://github.com/pusherman/react-native-network-info
- npm: https://www.npmjs.com/package/react-native-network-info?activeTab=versions

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

### react-native-outside-press

- github: https://github.com/dcangulo/react-native-outside-press
- npm: https://www.npmjs.com/package/react-native-outside-press?activeTab=versions

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

### react-native-progress

- github: https://github.com/oblador/react-native-progress
- npm: https://www.npmjs.com/package/react-native-progress?activeTab=versions

Progress indicators and spinners for React Native using React Native SVG.

```
import * as Progress from 'react-native-progress';

<Progress.Bar progress={0.3} width={200} />
<Progress.Pie progress={0.4} size={50} />
<Progress.Circle size={30} indeterminate={true} />
<Progress.CircleSnail color={['red', 'green', 'blue']} />
```

### react-native-ratings

- github: https://github.com/kolking/react-native-rating
- npm: https://www.npmjs.com/package/react-native-rating?activeTab=versions

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

### react-native-render-html

- github: https://github.com/meliorence/react-native-render-html
- npm: https://www.npmjs.com/package/react-native-render-html?activeTab=versions

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

### react-native-sound

- github: https://github.com/zmxv/react-native-sound
- npm: https://www.npmjs.com/package/react-native-sound?activeTab=versions

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

### react-native-swiper

- github: https://github.com/leecade/react-native-swiper
- npm: https://www.npmjs.com/package/react-native-swiper?activeTab=versions

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

### react-native-system-navigation-bar

- github: https://github.com/kadiraydinli/react-native-system-navigation-bar
- npm: https://www.npmjs.com/package/react-native-system-navigation-bar?activeTab=versions

React Native lets you customize the navigation bar for Android.

```
import SystemNavigationBar from 'react-native-system-navigation-bar';

SystemNavigationBar.navigationHide();
SystemNavigationBar.navigationShow();
SystemNavigationBar.leanBack();
SystemNavigationBar.immersive();
...
```

### react-native-view-shot

- github: https://github.com/gre/react-native-view-shot
- npm: https://www.npmjs.com/package/react-native-view-shot?activeTab=versions

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

### react-native-modal

- github: https://github.com/react-native-modal/react-native-modal
- npm: https://www.npmjs.com/package/react-native-modal?activeTab=versions

An enhanced, animated, customizable React Native modal.

The goal of react-native-modal is expanding the original React Native <Modal> component by adding animations, style customization options, and new features, while still providing a simple API.

```
import React, {useState} from 'react';
import {Button, Text, View} from 'react-native';
import Modal from 'react-native-modal';

function ModalTester() {
  const [isModalVisible, setModalVisible] = useState(false);

  const toggleModal = () => {
    setModalVisible(!isModalVisible);
  };

  return (
    <View style={{flex: 1}}>
      <Button title="Show modal" onPress={toggleModal} />

      <Modal isVisible={isModalVisible}>
        <View style={{flex: 1}}>
          <Text>Hello!</Text>

          <Button title="Hide modal" onPress={toggleModal} />
        </View>
      </Modal>
    </View>
  );
}

export default ModalTester;
```

### react-native-gesture-handler

- github: https://github.com/software-mansion/react-native-gesture-handler
- document: https://docs.swmansion.com/react-native-gesture-handler/docs/

Gesture Handler provides a declarative API exposing the native platform's touch and gesture system to React Native. It's designed to be a replacement of React Native's built in touch system called Gesture Responder System. Using native touch handling allows to address the performance limitations of React Native's Gesture Responder System. It also provides more control over the platform's native components that can handle gestures on their own.

### react-native-animatable

- github: https://github.com/oblador/react-native-animatable
- npm: https://www.npmjs.com/package/react-native-animatable?activeTab=versions

Declarative transitions and animations for React Native.

```
import * as Animatable from 'react-native-animatable';

class ExampleView extends Component {
  handleViewRef = ref => this.view = ref;
  
  bounce = () => this.view.bounce(800).then(endState => console.log(endState.finished ? 'bounce finished' : 'bounce cancelled'));
  
  render() {
    return (
      <TouchableWithoutFeedback onPress={this.bounce}>
        <Animatable.View ref={this.handleViewRef}>
          <Text>Bounce me!</Text>
        </Animatable.View>
      </TouchableWithoutFeedback>
    );
  }
}
```

### react-native-reanimated

- github: https://github.com/software-mansion/react-native-reanimated/
- npm: https://www.npmjs.com/package/react-native-reanimated?activeTab=versions
- document: https://docs.swmansion.com/react-native-reanimated/docs/fundamentals/getting-started/

React Native Reanimated is a powerful animation library built by Software Mansion. With Reanimated, you can easily create smooth animations and interactions that run on the UI thread.

```
import { Button, View } from 'react-native';
import Animated, { useSharedValue, withSpring } from 'react-native-reanimated';

export default function App() {
  const width = useSharedValue(100);

  const handlePress = () => {
    width.value = withSpring(width.value + 50);
  };

  return (
    <View style={{ flex: 1, alignItems: 'center' }}>
      <Animated.View
        style={{
          width,
          height: 100,
          backgroundColor: 'violet',
        }}
      />
      <Button onPress={handlePress} title="Click me" />
    </View>
  );
}
```

### react-native-permissions

- github: https://github.com/zoontek/react-native-permissions
- npm: https://www.npmjs.com/package/react-native-permissions?activeTab=versions

An unified permissions API for React Native on iOS, Android and Windows.

**check**

```
import {check, PERMISSIONS, RESULTS} from 'react-native-permissions';

check(PERMISSIONS.IOS.CAMERA).then((status) => {
  switch (status) {
    case RESULTS.UNAVAILABLE:
      return console.log('This feature is not available (on this device / in this context)');
    case RESULTS.DENIED:
      return console.log('The permission has not been requested / is denied but requestable');
    case RESULTS.BLOCKED:
      return console.log('The permission is denied and not requestable');
    case RESULTS.GRANTED:
      return console.log('The permission is granted');
    case RESULTS.LIMITED:
      return console.log('The permission is granted but with limitations');
  }
});
```

**request**

```
import {request, PERMISSIONS} from 'react-native-permissions';

request(PERMISSIONS.IOS.CAMERA).then((status) => {
  // …
});
```

**checkMultiple**

```
import {checkMultiple, PERMISSIONS} from 'react-native-permissions';

checkMultiple([PERMISSIONS.IOS.CAMERA, PERMISSIONS.IOS.FACE_ID]).then((statuses) => {
  console.log('Camera', statuses[PERMISSIONS.IOS.CAMERA]);
  console.log('FaceID', statuses[PERMISSIONS.IOS.FACE_ID]);
});
```

**requestMultiple**

```
import {requestMultiple, PERMISSIONS} from 'react-native-permissions';

requestMultiple([PERMISSIONS.IOS.CAMERA, PERMISSIONS.IOS.FACE_ID]).then((statuses) => {
  console.log('Camera', statuses[PERMISSIONS.IOS.CAMERA]);
  console.log('FaceID', statuses[PERMISSIONS.IOS.FACE_ID]);
});
```

**openSettings**

```
import {openSettings} from 'react-native-permissions';

openSettings('application').catch(() => console.warn('Cannot open app settings'));
```

### react-native-storage

- github: https://github.com/sunnylqm/react-native-storage
- npm: https://www.npmjs.com/package/react-native-storage?activeTab=versions

This is a local storage wrapper for both react native apps (using AsyncStorage) and web apps (using localStorage). ES6 syntax, promise for async load, fully tested with jest.

**Init**

```
import Storage from 'react-native-storage';
import AsyncStorage from '@react-native-async-storage/async-storage';

const storage = new Storage({
  size: 1000,
  storageBackend: AsyncStorage,
  defaultExpires: 1000 * 3600 * 24,
  enableCache: true,
});

export default storage;
```

**Save & Load & Remove**

```
var userA = {
  name: 'A',
  age: 20,
  tags: ['geek', 'nerd', 'otaku']
};

storage.save({
  key: 'user', // Note: Do not use underscore("_") in key!
  id: '1001', // Note: Do not use underscore("_") in id!
  data: userA,
  expires: 1000 * 60
});

// load
storage
  .load({
    key: 'user',
    id: '1001'
  })
  .then(ret => {
    // found data goes to then()
    console.log(ret.userid);
  })
  .catch(err => {
    // any exception including data not found
    // goes to catch()
    console.warn(err.message);
    switch (err.name) {
      case 'NotFoundError':
        // TODO;
        break;
      case 'ExpiredError':
        // TODO
        break;
    }
  });

// --------------------------------------------------

// get all ids for "key-id" data under a key,
// note: does not include "key-only" information (which has no ids)
storage.getIdsForKey('user').then(ids => {
  console.log(ids);
});

// get all the "key-id" data under a key
// !! important: this does not include "key-only" data
storage.getAllDataForKey('user').then(users => {
  console.log(users);
});

// clear all "key-id" data under a key
// !! important: "key-only" data is not cleared by this function
storage.clearMapForKey('user');

// --------------------------------------------------

// remove a single record
storage.remove({
  key: 'lastPage'
});
storage.remove({
  key: 'user',
  id: '1001'
});

// clear map and remove all "key-id" data
// !! important: "key-only" data is not cleared, and is left intact
storage.clearMap();
```

### react-native-fs

- github: https://github.com/itinance/react-native-fs
- npm: https://www.npmjs.com/package/react-native-fs/v/2.9.7?activeTab=versions

Native filesystem access for react-native.

```
// require the module
var RNFS = require('react-native-fs');

// get a list of files and directories in the main bundle
RNFS.readDir(RNFS.MainBundlePath) // On Android, use "RNFS.DocumentDirectoryPath" (MainBundlePath is not defined)
  .then((result) => {
    console.log('GOT RESULT', result);

    // stat the first file
    return Promise.all([RNFS.stat(result[0].path), result[0].path]);
  })
  .then((statResult) => {
    if (statResult[0].isFile()) {
      // if we have a file, read it
      return RNFS.readFile(statResult[1], 'utf8');
    }

    return 'no file';
  })
  .then((contents) => {
    // log the file contents
    console.log(contents);
  })
  .catch((err) => {
    console.log(err.message, err.code);
  });
```

### react-native-webview

- github: https://github.com/react-native-webview/react-native-webview
- npm: https://www.npmjs.com/package//react-native-webview?activeTab=versions
- document: https://github.com/react-native-webview/react-native-webview/blob/master/docs/Guide.md

React Native WebView is a community-maintained WebView component for React Native. It is intended to be a replacement for the built-in WebView (which was removed from core).

```
import React, { Component } from 'react';
import { WebView } from 'react-native-webview';

class MyWeb extends Component {
  render() {
    return <WebView source={{ uri: 'https://reactnative.dev/' }} />;
  }
}
```

### react-native-paper

- github: https://github.com/callstack/react-native-paper
- npm: https://www.npmjs.com/package/react-native-paper?activeTab=versions
- document: https://callstack.github.io/react-native-paper/docs/guides/getting-started/
- components: https://callstack.github.io/react-native-paper/docs/components/ActivityIndicator

React Native Paper is the cross-platform UI kit library containing a collection of customizable and production-ready components, which by default are following and respecting the Google’s Material Design guidelines.

Wrap your root component in PaperProvider from react-native-paper(if you are using versions prior to 5.8.0 you need to use Provider). 

```
import * as React from 'react';
import {MD3LightTheme as DefaultTheme, PaperProvider} from 'react-native-paper';
import App from './src/App';

const theme = {
  ...DefaultTheme,
  colors: {
    ...DefaultTheme.colors,
    primary: 'tomato',
    secondary: 'yellow',
  },
};

export default function Main() {
  return (
    <PaperProvider theme={theme}>
      <App />
    </PaperProvider>
  );
}
```

**ActivityIndicator**

Activity indicator is used to present progress of some activity in the app. It can be used as a drop-in replacement for the ActivityIndicator shipped with React Native.

```
import * as React from 'react';
import { ActivityIndicator, MD2Colors } from 'react-native-paper';

const MyComponent = () => (
  <ActivityIndicator animating={true} color={MD2Colors.red800} />
);

export default MyComponent;
```

### react-native-pager-view

- github: https://github.com/callstack/react-native-pager-view
- npm: https://www.npmjs.com/package/react-native-pager-view/v/5.1.5?activeTab=versions

This component allows the user to swipe left and right through pages of data. Under the hood it is using the native Android ViewPager and the iOS UIPageViewController implementations.

```
import React from 'react';
import { StyleSheet, View, Text } from 'react-native';
import PagerView from 'react-native-pager-view';

const MyPager = () => {
  return (
    <PagerView style={styles.pagerView} initialPage={0}>
      <View key="1">
        <Text>First page</Text>
      </View>
      <View key="2">
        <Text>Second page</Text>
      </View>
    </PagerView>
  );
};

const styles = StyleSheet.create({
  pagerView: {
    flex: 1,
  },
});
```

### react-native-keyboard-controller

- github: https://github.com/kirillzyusko/react-native-keyboard-controller
- npm: https://www.npmjs.com/package/react-native-keyboard-controller?activeTab=versions

A universal keyboard handling solution for React Native — lightweight, fully customizable, and built for real-world apps. Smooth animations, consistent behavior on both iOS and Android, with a developer-oriented design.

```
import { TextInput, View, StyleSheet } from 'react-native';
import { KeyboardAwareScrollView, KeyboardToolbar } from 'react-native-keyboard-controller';

export default function FormScreen() {
  return (
    <>
      <KeyboardAwareScrollView bottomOffset={62} contentContainerStyle={styles.container}>
        <View>
          <TextInput placeholder="Type a message..." style={styles.textInput} />
          <TextInput placeholder="Type a message..." style={styles.textInput} />
        </View>
        <TextInput placeholder="Type a message..." style={styles.textInput} />
        <View>
          <TextInput placeholder="Type a message..." style={styles.textInput} />
          <TextInput placeholder="Type a message..." style={styles.textInput} />
          <TextInput placeholder="Type a message..." style={styles.textInput} />
        </View>
        <TextInput placeholder="Type a message..." style={styles.textInput} />
      </KeyboardAwareScrollView>
      <KeyboardToolbar />
    </>
  );
}
```

### @react-native-community/hooks

- github: https://github.com/react-native-community/hooks
- npm: https://www.npmjs.com/package/@react-native-community/hooks?activeTab=versions

React Native APIs turned into React Hooks allowing you to access asynchronous APIs directly in your functional components.

**useBackHandler**

```
import {useBackHandler} from '@react-native-community/hooks'

useBackHandler(() => {
  if (shouldBeHandledHere) {
    // handle it
    return true
  }
  // let the default thing happen
  return false
},[shouldBeHandledHere])
```

**useImageDimensions**

```
import {useImageDimensions} from '@react-native-community/hooks'

const source = require('./assets/yourImage.png')
// or
const source = {uri: 'https://your.image.URI'}

const {dimensions, loading, error} = useImageDimensions(source)

if (loading || error || !dimensions) {
  return null
}
const {width, height, aspectRatio} = dimensions
```

### @react-native-camera-roll/camera-roll

- github: https://github.com/react-native-cameraroll/react-native-cameraroll
- npm: https://www.npmjs.com/package/@react-native-camera-roll/camera-roll?activeTab=versions

`CameraRoll` provides access to the local camera roll or photo library.

```
import { CameraRoll } from "@react-native-camera-roll/camera-roll";
```

**save()**

```
CameraRoll.save(tag, { type, album })
```

Saves the photo or video to the photo library, and returns the URI of the newly created asset. The tag must be a local image or video URI, such as "file:///sdcard/img.png". Returns a Promise which will resolve with the new URI.

**getAlbums()**

```
CameraRoll.getAlbums(params);
```

Returns a Promise with a list of albums.

**useCameraRoll()**

`useCameraRoll` is a utility hooks for the CameraRoll module.

```
import React, {useEffect} from 'react';
import {Button} from 'react-native';
import {useCameraRoll} from "@react-native-camera-roll/camera-roll";

function Example() {
  const [photos, getPhotos, save] = useCameraRoll();

  return <>
    <Button title='Get Photos' onPress={() => getPhotos()}>Get Photos</Button>
    {
      photos.map((photo, index) => /* render photos */)
    }
  </>;
};
```