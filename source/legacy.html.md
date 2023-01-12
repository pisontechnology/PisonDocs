---
title: Pison SDK

language_tabs: # must be one of https://git.io/vQNgJ
  - kotlin--android

toc_footers:
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

search: true

code_clipboard: true
---

# Pison Android SDK (Legacy)

Welcome to the Pison SDK Documentation!

**Before using this SDK on an Android device, you must first install and run Pison's server application: HubApp.  This application is responsible for managing connections to Pison's hardware, and it streams data third-party applications leveraging our SDK (like yours!).**

# Installation

## Installation via Gradle/Maven (preferred)

> In your _root_ `build.gradle` file

```kotlin--android

maven {
  name = "GitHubPackages"
  url = uri("https://maven.pkg.github.com/pisontechnology/thanos")
  credentials {
    // store credentials in gradle properties
    username = github_username 
    password = github_password
  }
}
```

> In your _application_ `build.gradle` file


```kotlin--android
dependencies {
  implementation "com.pison.sdk:android:1.14.2-internal-beta"
}

```

All artifacts are currently hosted on <a href="https://maven.pkg.github.com/pisontechnology/thanos">GitHub Packages</a>.

The repository is private, so you'll need someone from Pison to add you as a collaborator.
After that, you'll want to create a <a href ="https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token">personal access token</a> and use it in place of the `github_password` in your root build.gradle file.

## Alternate Installation via AAR

```kotlin--android
// in your module's build.gradle
dependencies {
  implementation fileTree(dir: 'libs', include: ['*.aar'], exclude: [])
}
```

In the past, we have distributed our SDK as an `.aar` file.  This made it easy to bundle our code, as well as our dependencies, for developers working without Gradle (mainly Unity devs). These `.aar` files should still work fine in a Gradle environement, though you may run into dependency conflicts if you are trying to pull in any of our bundled dependencies via a Maven repository as well.  If this happens, you can either remove the dependency declaration from your build.gradle (and rely on the one we bundle) or, you can manually unzip the `.aar`, remove the conflicting `.jar`, and re-zip the `.aar`.

To use, simply add our `.aar` file to folder called `libs` inside your application module's root.
Then, instruct gradle to look for dependency artifacts inside this folder as demonstrated in the code sample.



# Usage

## Binding to the `PisonService`

> Example Android Activity

```kotlin--android

class MyActivity : AppCompatActivity() {

  val pisonEventListener = object : EventListener {
    //.. 
  }.

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    // ...

    // bindService takes an Android Context and an EventListener as args
    PisonService.instance.bindService(this, pisonEventListener)
  }

  override fun onDestroy() {
    super.onDestroy()
    // ...

    // unbindService takes an Android Context and an EventListener as args
    PisonService.instance.unbindService(this, pisonEventListener)
  }
}


```

To register for updates from a Pison device, you'll need to bind some component from your application to the `PisonService` instance provided by our SDK.  This component must implement Pison's `EventListener` interface (discussed below). This component should be unbound when it no longer needs to receive Pison device updates. We recommend linking these `bind`/`unbind` operations to a complementary pair of Android lifecycle events - `onCreate`/`onDestory` or `onStart`/`onStop`, for example.

**Note: as of SDK version `1.14.2`, it is possible to bind the same instance of `EventListener` more than once. If you find that you are receiving duplicate updates, make sure that you have not accidentally done this.**

## Implementing an `EventListener`
**Here is a summary of `EventListener`'s callback methods:**


```kotlin--android
val pisonEventListener = object : EventListener {

  override fun onDeviceConnected() {
    println("Connected to Pison device!")
  }

  override fun onDeviceDisconnected() {
    println("Pison device not available!")
  }

  override fun onDeviceFound(device: Device) {
    println("Scanned Pison device: $device")
  }

  override fun onError(code: Int, errorMessage: String) {
    println("Pison SDK error: $errorMessage")
  }

  override fun onDeviceFrame(frame: DeviceFrame) {
    println("Received Pison DeviceFrame: ${frame.timeStamp}")
  }
}
```

-
**`fun onDeviceConnected(): Unit`**

* Invoked when a Pison device is connected to the Pison server.
* Also invoked when this instance of `EventListener` is bound, _if_ a Pison device is connected at bind time.

-
**`fun onDeviceDisconnected(): Unit`**

* Invoked when a Pison device is no longer available.


-
**`fun onError(code: Int, errorMessage: String): Unit`**

* Invoked when an error has occurred. The full list of error codes is TBD, but the `errorMessage` will be consistent and descriptive.


-
**`fun onDeviceDisovered(device: Device): Unit`**

* **NOTE: MAY NOT WORK WITH EVERY VERSION OF THE PISON SERVER**
* Invoked when the Pison server is scanning, and has discovered a nearby Pison device.


-
**`fun onDeviceFrame(deviceFrame: DeviceFrame): Unit`**

* Invoked when the Pison server sends an update related to the Pison device.
* Full details on the `DeviceFrame` object in the following section.
* The frequency at which this this method is controlled by the Pison server. **By default it is very fast - upwards of once every 1.6ms**!
* Recent builds of `HubApp` (as of version 2.2.0) have a setting to let you throttle these frames.


# The `DeviceFrame` Object

A `DeviceFrame`s is passed to a bound `EventListener` whenever the Pison server has an update to share about a connected Pison device. It contains raw sensor information, as well as derived data and events classified by the server.

Here is a summary of the data packaged in a `DeviceFrame`

-
**`activation: Activation`**

* Enum with values: `NONE`, `START`, `HOLD`, `END`
* Indicates whether a user is currently attempting to "activate" their index finger
* `START` and `END` are equivalent to `HOLD`, but signal the respective start or end of a contiguous stream of `HOLD` events. 

-
**`batteryPercentage: Int`**

* The value of the currently connected Pison device's battery, between 0-100

-
**`detectedGesture: Gesture`**

* a one-time event indicating that the user has performed a gesture while wearing the connected Pison device.
* Enum with values:
  * (Note: all activations referring to user's index finger)
  * `NONE` - no gesture has been detected
  * `SWIPE_UP` - user has activated and swiped towards the ceiling
  * `SWIPE_DOWN` - user has activated and swiped towards the floor
  * `SWIPE_LEFT` - user has activated and swiped left
  * `SWIPE_RIGHT` - user has activated and swiped right
  * `ROLL_LEFT` - user has activated and rotated their wrist counter-clockwise
  * `ROLL_LEFT` - user has activated and rotated their wrist clockwise
  * `CLICK` - user has quickly activated and de-activated 
  * `HOLD` - user has activated and maintained their activation for a period of time

-
**`handPosition: HandPosition`**

* represents the estimated current orientation of a user's arm
* Enum with values:
  * `UNKNOWN`
  * `UP` - user has their arm pointed towards the ceiling
  * `DOWN` - user has their arm pointed towards the floor
  * `NEUTRAL` - user has their arm out in front of them

-
**`imuRaw: Imu`**

* Raw sensor data from a connected Pison device's IMU
* `acceleration` - Three dimensional vector representing accelerometer data 
* `gyro` - Three dimensional vector representing gyroscope data

-
**`quaternion: Quaternion`**

* contains Quaternion data (including derived `pitch`, `yaw`, and `roll`)
* Used to represent the connected Pison device's orientation in 3D space.
* learn more about Quaternions [here](http://www.chrobotics.com/library/understanding-quaternions) 

-
**`isLocked: Boolean`**

* represents whether or not the connected Pison device is locked (via sleep/wake gesture) or not
* depending on your version of the Pison server, you may more may not receive up-to-date device data _while_ the device is locked. It is up to you to check this value and adapt your application accordingly.

_
**`timeStamp: Double`**

* time, in fractional milliseconds, **as kept by the connected Pison device**.
* this value is **not** tied to real-world time, and is reset each time the device is power-cycled.

