
# react-native-rtmpview

## Getting started

`$ npm install react-native-rtmpview --save`

### iOS installation

Find or create an iOS podfile in the `./ios` directory, and add:

    pod 'Yoga', path: '../node_modules/react-native/ReactCommon/yoga/Yoga.podspec'
    pod 'React', path: '../node_modules/react-native'
    pod 'react-native-rtmpview', :path => '../node_modules/react-native-rtmpview'

Next, run

    pod install

Because react-native-rtmpview has cocoapod dependencies on third-party video playback libraries, it must be added through Cocoapods. (You cannot simply use `react-native link` for example, as you can with other libraries).

### Android installation

This library does not yet work with Android devices.


## Example

react-native-rtmpview includes an example project to help get you started. To build and run the example, download or clone the project from github, and then run the following from the root of the project:

```
    cd Example/
    npm install --save
    react-native run-ios
```

## Usage
```javascript

import { RtmpView } from 'react-native-rtmpview';

<RtmpView
  style={styles.player}
  shouldMute={true}
  ref={e => { this.player = e; }}
  onPlaybackState={(data) => {
    this.handlePlaybackState(data);
  }}
  onFirstVideoFrameRendered={(data) => {
    this.handleFirstVideoFrameRendered(data);
  }}
  url="rtmp://localhost:1935/live/stream"/>

```

### Events

By default this class will pass all events through using an event emitter,
which can be subscribed to this way:

```javascript
  const RNRtmpEventManager = new NativeEventEmitter(
    NativeModules.RNRtmpEventManager
  );

  RNRtmpEventManager.addListener(
    "RNRtmpEvent",
    (data) => this.handleRNRtmpEvent(data)
  );
```

However, there are two events that are especially useful: knowing playback
state changes, and knowing when the first video frame has been rendered.
This is because you can do things like remove a loading screen.

Thus, we have specially pulled out those events and made them actual
function properties of the view.


## Implementation details and alternatives

react-native-rtmpview is based on [KSYLive](https://github.com/ksvc/KSYLive_iOS), which is a popular iOS library for video and RTMP streaming. The complete list of options for RTMP streaming on iOS can be found [here on StackOverflow](https://stackoverflow.com/questions/43872012/ios-rtmp-streaming-library-lflivekit-vs-videocore-lib-vs-alternative), and includes:

* [HaishinKit (formerly lf)](https://github.com/shogo4405/HaishinKit.swift) - This library does not support RTMP playback (technically it does, but only as an '[experimental feature](https://github.com/shogo4405/HaishinKit.swift/issues/358)')
* [LaiFeng iOS Live Kit](https://github.com/LaiFengiOS/LFLiveKit) - Popular library in terms of stars (3k+) but not updated since 2016, so effectively abandoned.
* [VideoCore](https://github.com/jgh-/VideoCore-Inactive) - Popular (1k+ stars), but library abandoned in 2015.
* [react-native-nodemediaclient](https://github.com/NodeMedia/react-native-nodemediaclient) - The underlying native library is very limited and is still emerging in popularity (100+ stars). It does not surface as many playback events or provide as much configurabilty as other RTMP streaming libraries.
* [KSYLive_iOS](https://github.com/ksvc/KSYLive_iOS) - Growing in popularity (500+ stars) and updated very recently (in 2018)

As a result, we elected to base our implementation for RTMP in React Native on the actively-maintained KSYLive_iOS library, because it was both the most full-featured and still actively maintained.

We are actively investigating implementation options for Android.

## About the example configuration

You will note in the Example/package.json file that we have an explicit dependency on react-native-rtmpview:

```
  "react-native-rtmpview": "*"
```

We do not have a relative dependency (i.e., 

```
  "react-native-rtmpview": "file:.."
```

Relative dependencies are created by running `npm -i ../` and they create a symoblic link (`ln -s`) inside of `node_modules/<your_library>/`, and are preferable because changes you make at the root of your project in the actual source code of your library are immediately reflected within your Example project, thus making development of your library easier.

We cannot do this; instead we must have a complete and unconnected copy of the project within `Example/node_modules/`. This means if you use the Example project to test/debug react-native-rtmpview, you will have to make those changes to code buried within `Example/node_modules/react-native-rtmpview`, and manually apply or copy those changes up one level at the root of the project. 

We are required to use this architecture because if you create a relative link, every time you launch the app by running `react-native run-ios`, you will see a redbox stating:

```
"Unable to resolve module `react` from `<path>`: Module does not exist in the module map."
```

I have confirmed that this also plagues other apps with Example directories architected similiarly, including [react-native-twilio-video-webrtc](https://github.com/blackuy/react-native-twilio-video-webrtc)

There is [some chatter online](https://github.com/wix/wml/issues/14) about how to use wml to address this problem, but I was unable to get it working.

Thus, until React improves its module resolution process, we will be forced to manually copy over changes made while developing using the Example project.
