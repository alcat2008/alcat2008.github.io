---
layout: post
title: React Native 项目打包
date: '2016-05-04 15:37:01 +0800'
categories: react-native
---

# Steps to release ipa file for react native

## 1\. Clear caches of react.

```shell
rm -rf $TMPDIR/react-*
```

## 2\. Delete existed bundle file.

```shell
rm ios/main.jsbundle
```

## 3\. Bundle main.jsbundle.

```shell
react-native bundle --platform ios --dev false --entry-file index.ios.js --bundle-output ios/main.jsbundle --assets-dest ios/
```

## 4\. Release ipa file in xcode.

a) Open AppDelegate.m

b) Comment jsCodeLocation = [NSURL URLWithString ]..., Uncomment jsCodeLocation = [[NSBundle mainBundle] ...

c) The JS bundle will be built for dev or prod depending on your app's scheme (Debug = development build with warnings, Release = minified prod build with perf optimizations). To change the scheme navigate to Product > Scheme > Edit Scheme... in xcode and change Build Configuration between Debug and Release.

d) Confirm the 'Code Signing' settings.

e) Run Product > Archive.

# Steps to bundle android files for react native

```shell
react-native bundle --platform android --dev false --entry-file index.android.js --bundle-output android/app/src/main/assets/index.android.bundle --assets-dest android/app/src/main/res/
```

# react-native 命令参考

```javascript
Options:
  --entry-file        Path to the root JS file, either absolute or relative to JS root                                   [required]
  --platform          Either "ios" or "android"                                                                        
  --transformer       Specify a custom transformer to be used (absolute path)                                            [default: "./node_modules/react-native/packager/transformer.js"]
  --dev               If false, warnings are disabled and the bundle is minified                                         [default: true]
  --prepack           If true, the output bundle will use the Prepack format.                                            [default: false]
  --bridge-config     File name of a a JSON export of __fbBatchedBridgeConfig. Used by Prepack. Ex. ./bridgeconfig.json
  --bundle-output     File name where to store the resulting bundle, ex. /tmp/groups.bundle                              [required]
  --bundle-encoding   Encoding the bundle should be written in (https://nodejs.org/api/buffer.html#buffer_buffer).       [default: "utf8"]
  --sourcemap-output  File name where to store the sourcemap file for resulting bundle, ex. /tmp/groups.map            
  --assets-dest       Directory name where to store assets referenced in the bundle                                    
  --verbose           Enables logging                                                                                    [default: false]
```
