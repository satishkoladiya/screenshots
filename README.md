
[![pub package](https://img.shields.io/pub/v/screenshots.svg)](https://pub.dartlang.org/packages/screenshots) 
[![Build Status](https://travis-ci.com/mmcc007/screenshots.svg?branch=master)](https://travis-ci.com/mmcc007/screenshots)

![alt text][demo]

[demo]: https://i.imgur.com/gkIEQ5y.gif "Screenshot with overlaid 
status bar placed in frame"  
A screenshot image with overlaid status bar placed in a device frame.  

For an example of images generated with _Screenshots_ on a live app in both stores see:  
[![GitErDone](https://play.google.com/intl/en_us/badges/images/badge_new.png)](https://play.google.com/store/apps/details?id=com.orbsoft.todo)
[![GitErDone](https://linkmaker.itunes.apple.com/en-us/badge-lrg.svg?releaseDate=2019-02-15&kind=iossoftware)](https://itunes.apple.com/us/app/giterdone/id1450240301)


See a demo of _Screenshots_ in action:
[![Screenshots demo](https://i.imgur.com/V9VFSYb.png)](https://vimeo.com/317112577 "Screenshots demo - Click to Watch!")

For introduction to _Screenshots_ see https://medium.com/@nocnoc/automated-screenshots-for-flutter-f78be70cd5fd.

# _Screenshots_

_Screenshots_ is a standalone command line utility and package for capturing screenshot images for Flutter.   

_Screenshots_ will start the required android emulators and iOS simulators (or find attached devices), run your screen capture tests on each emulator/simulator (or device), process the images, and drop them off to Fastlane for delivery to both stores.

It is inspired by three tools from Fastlane:  
1. [Snapshots](https://docs.fastlane.tools/getting-started/ios/screenshots/)  
This is used to capture screenshots on iOS using iOS UI Tests.
1. [Screengrab](https://docs.fastlane.tools/actions/screengrab/)  
This captures screenshots on android using Android Espresso tests.
1. [FrameIt](https://docs.fastlane.tools/actions/frameit/)  
This is used to place captured iOS screenshots in a device frame.

Since all three of these Fastlane tools do not work with Flutter, _Screenshots_ combines key features of these Fastlane tools into one tool. 

# Features
Since Flutter integration testing is designed to work transparently across iOS and Android, capturing images using _Screenshots_ is easy.

Features include: 
1. Works with your existing tests  
Add a single line for each screenshot.
1. Run your tests on any device  
Select the devices you want to run on, using a convenient config file.
1. One run for both platforms  
_Screenshots_ runs your tests on both iOS and Android in one run.  
(as opposed to making separate Snapshots and Screengrab runs)
1. One run for multiple locales  
If your app supports multiple locales, _Screenshots_ will optionally set the locales listed in the config file before running each test (see Limitations below).
1. One run for frames  
Optionally places images in device frames in same run.  
(as opposed to making separate FrameIt runs... which supports iOS only)
1. One run for clean status bars  
Every image that _Screenshots_ generates has a clean status bar.  
(no need to run a separate stage to clean-up status bars)
1. Works with Fastlane  
_Screenshots_ drops-off images where Fastlane expects to find them. Fastlane's [deliver](https://docs.fastlane.tools/actions/deliver/) and [supply](https://docs.fastlane.tools/actions/supply/) can then be used to upload to respective stores.  

Additional automation features:  
1. _Screenshots_ runs in the cloud.  
For live demo of _Screenshots_ running with the internationalized [example](example) app on macOS in cloud, see [below](#sample-run-on-travis)    
1. _Screenshots_ works with any CI/CD tool.  
For live demo of uploading images, generated by _Screenshots_, to both store consoles, see demo of Fledge at https://github.com/mmcc007/fledge#demo

# Installation
On macOS:
````bash
$ brew update && brew install imagemagick
$ pub global activate screenshots
````
Note:  
If `pub` is not found, add to PATH using:  
```
export PATH="$PATH:<path to flutter installation directory>/bin/cache/dart-sdk/bin"
```
# Usage

````
$ screenshots
````
Or, if using a config file other than the default 'screenshots.yaml':
````
$ screenshots -c <path to config file>
````

# Modifying your tests for _Screenshots_
A special function is provided in the _Screenshots_ package that is called by the test each time you want to capture a screenshot. 
_Screenshots_ will then process the images appropriately during a _Screenshots_ run.

To capture screenshots in your tests:
1. Include the _Screenshots_ package in your pubspec.yaml's dev_dependencies section  
   ````yaml
     screenshots: ^<current version>
   ````
2. In your tests
    1. Import the dependencies  
       ````dart
       import 'package:screenshots/config.dart';
       import 'package:screenshots/capture_screen.dart';
       ````
    2. Create the config map at start of test  
       ````dart
            final Map config = Config().config;
       ````  
    3. Throughout the test make calls to capture screenshots  
       ````dart
           await screenshot(driver, config, 'myscreenshot1');
       ````
       
Note: make sure your screenshot names are unique across all your tests.

Note: to turn off the debug banner on your screens, in your integration test's main(), call:
````dart
  WidgetsApp.debugAllowBannerOverride = false; // remove debug banner for screenshots
````

# Configuration
_Screenshots_ uses a configuration file to configure a run.  
 The default config filename is `screenshots.yaml`:
````yaml
# A list of screen capture tests
tests:
  - test_driver/main1.dart
  - test_driver/main2.dart

# Note: flutter driver expects a pair of files for testing
# For example:
#   main1.dart is the test app (that calls your app)
#   main1_test.dart is the matching test that flutter driver 
#   expects to find.

# Interim location of screenshots from tests
staging: /tmp/screenshots

# A list of locales supported by the app
locales:
  - de-DE
  - en-US

# A list of devices to emulate
devices:
  ios:
    iPhone X:
      frame: false
    iPad Pro (12.9-inch) (2nd generation):
  android:
    Nexus 6P:

# Frame screenshots
frame: true
````
Note: emulators and simulators corresponding to the devices in your config file must be installed on your test machine.

# Integration with Fastlane
Since _Screenshots_ is intended to be used with Fastlane, after _Screenshots_ completes, the images can be found in your project at:
````
android/fastlane/metadata/android
ios/fastlane/screenshots
````
Images are in a format suitable for upload via [deliver](https://docs.fastlane.tools/actions/deliver/) 
and [supply](https://docs.fastlane.tools/actions/supply/).

Tip: The easiest way to use _Screenshots_ with Fastlane is to call _Screenshots_ before calling Fastlane. Calling Fastlane (for either iOS or Android) will then find the images in the appropriate place.  

(For a live demo of using Fastlane to upload screenshot images to both store consoles, see demo of Fledge at https://github.com/mmcc007/fledge#demo)

## Changing devices

To change the devices to run your tests on, just change the list of devices in screenshots.yaml.

Make sure the devices you select have supported screens and corresponding emulators/simulators.

Note: _In practice, multiple devices share the same screen size.
Devices are therefore organized by supported screen size in a file called `screens.yaml`._

For each selected device:
1. Confirm device is present in [screens.yaml](https://github.com/mmcc007/screenshots/blob/master/lib/resources/screens.yaml).  
2. Add device to the list of devices in screenshots.yaml.  
3. Install an emulator or simulator for device.   
 
_Screenshots_ will check your configuration before running.

# Upgrading
To upgrade, simply re-issue the install command
````bash
$ pub global activate screenshots
````
Note: the _Screenshots_ version should be the same for both the command line and package:
1. If upgrading the command line version of _Screenshots_, also upgrade
 the version of _Screenshots_ in your pubspec.yaml.    
2. If upgrading the version of _Screenshots_ in your pubspec.yaml, also upgrade the command line version.    

# Sample run on Travis
To view _Screenshots_ running with the internationalized [example](example) app on macOS in the cloud see:  
https://travis-ci.com/mmcc007/screenshots

To view the images generated by _Screenshots_ during run on travis see:  
https://github.com/mmcc007/screenshots/releases/

* Running _Screenshots_ in the cloud is  useful for automating the generation of your screenshots in a CI/CD environment.  
* Running _Screenshots_ on macOS in the cloud can be used to generate your screenshots when developing on Linux and/or Windows (if not using locally attached iOS devices).

# Limitations

Due to a Flutter issue ([flutter/issues/27785](https://github.com/flutter/flutter/issues/27785)), running _Screenshots_ in multiple locales has limitations. 

To raise priority of this Flutter issue, so it will be fixed sooner rather than later, please give a thumbs-up on [flutter/issues/27785](https://github.com/flutter/flutter/issues/27785).

Priority of this limitation in Flutter project:

| Date | `flutter driver` | `internationalization` | `test` |
| --- | --- | --- | ---  |
| 4/26/2019 | [#1](https://github.com/flutter/flutter/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aopen+sort%3Areactions-%2B1-desc+label%3A%22t%3A+flutter+driver%22+) | [#5](https://github.com/flutter/flutter/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aopen+sort%3Areactions-%2B1-desc+label%3A%22a%3A+internationalization%22+) | [#7](https://github.com/flutter/flutter/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aopen+sort%3Areactions-%2B1-desc+label%3A%22a%3A+tests%22+) |

(This limitation is being tracked by _screenshots_ in [screenshots/issues/20](https://github.com/mmcc007/screenshots/issues/20)).

# Issues and Pull Requests
[Issues](https://github.com/mmcc007/screenshots/issues) and 
[pull requests](https://github.com/mmcc007/screenshots/pulls) are welcome.

Your feedback is welcome and is used to guide where development effort is focused. So feel free to create as many issues and pull requests as you want. You should expect a timely and considered response.