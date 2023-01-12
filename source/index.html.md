---
title: Pison SDK

language_tabs: # must be one of https://git.io/vQNgJ
  - kotlin--android 
  - kotlin--jvm

toc_footers:
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

search: true

code_clipboard: true
---

### Welcome to the Pison SDK Documentation! 

**This SDK was designed to interact with an instance of the Pison Server running on your local device or network*. The Pison Server is a piece of software that directly connects to and communicates with the Pison hardware. It handles the hard work of parsing, processing, and storing data sent from our Vulcan Control Band and exposes a simplified API for our development partners (like you!).**

> Note: networked usage of the Pison SDK is currently experimental!

**Please make sure you have a version of the Pison Server installed and running alongside your application before trying to use this SDK. Your application will not work without it! On Android, the Pison Server is wrapped in an application called Vulcan.**

# Pison Gesture Guide

## Understanding Gestures

A gesture is defined as a specific instance of a hand posture that can be mapped to a specific activity, such as activation/wake, gesture, and compound gesture. The gestures described in this section are dynamic gestures; that is, gestures in which the user moves at least one part of their arm or hand. The user making gestures must be wearing a properly worn and calibrated Vulcan Control Band also referred to as Control Band.

All gestures described in this section are supported in the SDK. All inference output from gestures is sent in the `frameTags` as an SDK output contained in `DeviceFrames` and can be monitored by client applications.

## What is the Vulcan Control Band?

The Vulcan Control Band is a wrist-worn wearable device equipped with an ENG sensor array, IMU, LED indicator, control button, and haptic feedback generator. It is used in conjunction with the Vulcan app to calibrate the system to the user and process ENG signals to create gesture inferences.

For more detailed information on the Control Band, including donning and calibrating it, please request a copy of the Vulcan User Manual.

## Waking the Vulcan Control Band

The Control Band continuously streams data by default (when connected to a compatible smartphone with the Vulcan app installed). As such, neither the Vulcan Control Band nor the Vulcan app require a specific action to wake the device and begin streaming data. The Pison team recognizes the importance of this functionality and have developed this concept within the SDK. You will see this referenced below as a `SHAKE` gesture.

Unlike other smart assistance platforms, `SHAKE` is not a typical wake word. While developers _can_ require a user to perform a specific gesture to prompt a listening window, Pison leaves this up to the developers of end user applications.

Through Pison's own product development efforts, we have determined that this makes sense for customers who are building companion applications to Vulcan, including plugins for Android Team Awareness Kit (ATAK). Within ATAK, a `SHAKE` gesture must precede any gesture.

A user performs a `SHAKE` by making a loosely closed fist with the thumb against the side of the hand and the wrist perpendicular to the ground, then turning the fist and returning it to the starting position. The IMU (device state that includes the latest readings from the Control Band's gyroscope, accelerometer, and magnetometer) detects a 60 degree wrist roll, either to the left or right, to register a `SHAKE`. You can configure the minimum and maximum time the Vulcan app will listen for a gesture to follow a `SHAKE`; see [Vulcan App Developer Settings](#vulcan-app-developer-settings) for more information.

SDK output: `SHAKE` or `SHAKE_N_<gesture>` when used to activate a specific gesture. For example, `SHAKE_N_FHEH` followed by `FHEH_SWIPE_UP`.

## Gestures

Pison uses a custom machine learning library that has models for three basic electroneurography (ENG) gestures. These gestures are inferred through processing of electromyographical data gathered from the user's wrist. When the Vulcan Control Band is properly worn and calibrated, it can be used as an input for technology-enabled experiences.

### Index Extension

Index Extension is a loosely closed fist with index finger fully extended.

SDK Output: `INEH`

<!--- Note: This section can be restored when click and hold are introduced in the SDK. These are currently possible when implemented by a client, but is not a function of the SDK. - CP 1/4/2023

### Index Click

Index Click is registered when a user makes an Index Extension (INEH) gesture and releases it within a total time that is less than n milliseconds. This gesture can be used for selection or confirmation.

### Index Hold

Index Hold is registered when a user makes an Index Extension (INEH) gesture and releases it within a total time that is greater than n milliseconds. This gesture can be used for shortcut selection.
--->

### Thumb Extension

Thumb Extension is a loosely closed fist with thumb fully extended.

SDK Output: `TEH`

### Full Hand Extension

Full Hand Extension is the entire hand open with the palm perpendicular to the ground.

SDK Output: `FHEH`

## Swipe Gestures

Swipe gestures are compound gestures, that is, additions to gesture primitives. A swipe requires that the user hold the gesture (for example, Index Extension) while moving in a specified direction (up, down, left, or right) at least 20 degrees. (See the section that follows, [Vulcan App Developer Settings](#vulcan-app-developer-settings), for details about customizing the swipe thresholds to be more or less than 20 degrees.)

The user cannot perform another swipe until they release the gesture (goes back to `REST`) and begin another gesture.

Once the user has initiated a swipe, the originating gesture inference (e.g. `INEH`) is maintained even if the inference switches to `FHEH`, `TEH`, or `NAC` (not a class), basically anything that indicates the user is still attempting to hold a gesture, until the inference determines `REST` or the user meets the minimum swipe threshold degrees of movement from start position.

The following table lists the supported swipe gestures and their corresponding SDK output.

| **Swipe Gesture**                      | **SDK Output**     |
|----------------------------------------|--------------------|
| Index Extension Hold + Swipe Up        | `INEH_SWIPE_UP `   |
| Index Extension Hold + Swipe Down      | `INEH_SWIPE_DOWN`  |
| Index Extension Hold + Swipe Left      | `INEH_SWIPE_LEFT`  |
| Index Extension Hold + Swipe Right     | `INEH_SWIPE_RIGHT` |
| Thumb Extension Hold + Swipe Up        | `TEH_SWIPE_UP`     |
| Thumb Extension Hold + Swipe Down      | `TEH_SWIPE_DOWN`   |
| Thumb Extension Hold + Swipe Left      | `TEH_SWIPE_LEFT`   |
| Thumb Extension Hold + Swipe Right     | `TEH_SWIPE_RIGHT`  |
| Full Hand Extension Hold + Swipe Up    | `FHEH_SWIPE_UP`    |
| Full Hand Extension Hold + Swipe Down  | `FHEH_SWIPE_DOWN`  |
| Full Hand Extension Hold + Swipe Left  | `FHEH_SWIPE_LEFT`  |
| Full Hand Extension Hold + Swipe Right | `FHEH_SWIPE_RIGHT` |

## Developer Settings

### Swipe Thresholds
| **Setting**     | **Description**     | **Impact of Decreasing**     | **Impact of Increasing**     | **Recommended Change**     |
|------------------------------------|-------------------------|--------------------------|------------------------------|------------------------|
| Swipe Left/Right/Up/Down Min Threshold | Minimum distance in degrees required to detect swipe | Swipe gestures may feel more natural, but false-positive rate may increase | Requires larger arm movement, may feel unnatural | Do not increase the default value of 20 degrees |

### Post-shake Wait Settings

Client applications are responsible for determining whether a `SHAKE` gesture is required to begin listening for a user to perform a gesture. When used with ATAK, A `SHAKE` gesture temporarily wakes the Vulcan app so that it knows that other gestures will likely follow. The `Post Shake Min Wait` and `Post Shake Max Wait` settings combine to control the timing (in milliseconds) for a lockout for when Vulcan will start and stop listening for a gesture following a `SHAKE`.

| **Setting**     | **Description**     | **Impact of Decreasing**     | **Impact of Increasing**     | **Recommended Change**     |
|------------------------------------|-------------------------|--------------------------|------------------------------|------------------------|
| Post Shake Min Wait                | Time in milliseconds between completing a `SHAKE` and registering another gesture | Shorter pause between completing `SHAKE` and registering another gesture | Longer pause between completing `SHAKE` and registering another gesture | Do not increase the default value of 313 milliseconds |
| Post Shake Max Wait                | Time in milliseconds that Vulcan will accept a gesture following a `SHAKE` | Shorter total time to register a gesture following a `SHAKE` | Longer total time to register a gesture following a `SHAKE` | Do not increase from default value of 1065 milliseconds |

### Full Shake Max Time

The Full Shake Max Time setting specifies the maximum amount of time (in milliseconds) a user has to perform a `SHAKE` - that is, to complete the motion of rotating the wrist and then rotating it back to the start position.

| **Setting**     | **Description**     | **Impact of Decreasing**     | **Impact of Increasing**     | **Recommended Change**     |
|------------------------------------|-------------------------|--------------------------|------------------------------|------------------------|
| Full Shake Max Time	| Maximum time in milliseconds to perform a `SHAKE` | User must make a faster `SHAKE` gesture from start to finish, may increase false negatives | User can make a `SHAKE` gesture slower | Do not increase the default value of 555 milliseconds |

### Roll Degree Threshold

The `Roll Degree Threshold` setting specifies the minimum roll offset (in degrees) that the Vulcan Control Band must cross to register the wrist rotation as a `SHAKE`. A `SHAKE` will not register until the user rotates their wrist back to the start position.

| **Setting**     | **Description**     | **Impact of Decreasing**     | **Impact of Increasing**     | **Recommended Change**     |
|------------------------------------|-------------------------|--------------------------|------------------------------|------------------------|
| Roll Degree Threshold	 | Minimum roll offset in degrees required to register a `SHAKE` | Requires a smaller range of motion to register a `SHAKE` gesture, may increase false positives | Requires a larger range of motion to register a `SHAKE` gesture, may feel unnatural and/or increase false negatives | Do not increase from default value of 60 degrees |

**Important: You should not change any of the other developer settings without first contacting the Pison SDK team to understand how changing them could impact the user experience.**

### Changing Values in Developer Settings

You can change each setting by moving its associated slider until the app displays the value you want. To return all these settings to system defaults, scroll to the bottom of the page and tap `Reset to Defaults`.

Keep in mind that the sliders you use to control the developer settings are not locked. When you scroll the page to view more settings, take care not to touch any sliders, as this could lead to accidentally changing a setting.


# SDK Library Installation

## Installation via Gradle/Maven (preferred)

```kotlin--android
// Add our maven repo in your root `build.gradle` file
repositories {
  mavenCentral()
  maven { url "https://dl.bintray.com/badoo/maven" }
  maven { url = "https://kotlin.bintray.com/kotlinx/" }
  maven {
    name = "GitHubPackages"
    url = uri("https://maven.pkg.github.com/pisontechnology/thanos")
    credentials {
      // store credentials in gradle properties
      username = github_username 
      password = github_token
    }
  }
  // ... any other maven repositories you need
}


// In your application module's `build.gradle` file
dependencies {
  implementation "com.pison.core:client-android:1.1.0-99af0c5fde"
}

// in your gradle.properties file

github_username=YourUsername
github_password="Your Token"

```

```kotlin--jvm
// Add our maven repo in your root `build.gradle` file
repositories {
  mavenCentral()
  maven { url "https://dl.bintray.com/badoo/maven" }
  maven { url = "https://kotlin.bintray.com/kotlinx/" }
  maven {
    name = "GitHubPackages"
    url = uri("https://maven.pkg.github.com/pisontechnology/thanos")
    credentials {
      // store credentials in gradle properties
      username = github_username 
      password = github_token
    }
  }
  // ... any other maven repositories you need
}


// In your application module's `build.gradle` file
dependencies {
  implementation "com.pison.core:client-jvm:1.1.0-99af0c5fde"
}

// in your gradle.properties file

github_username=YourUsername
github_password="Your Token"

```

All Pison artifacts are currently hosted on <a href="https://maven.pkg.github.com/pisontechnology/thanos">GitHub Packages</a>.

The repository is private, so you'll need someone from Pison to add you as a collaborator.

After that, you'll want to create a <a href ="https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token">personal access token</a> and use it in place of the `github_password` in your root `build.gradle` file. Make sure you give your token the necessary repository permissions.

Once you have a token, you'll need to add the repositories hosting the Pison SDK's dependencies to your gradle project (as shown on the right).

### Storing credentials in your local gradle properties file
If you want to store your github username and token in your local gradle properties, use your terminal to navigate to `[user root]/.gradle/gradle.properties`. If you do not have the `gradle.properties` file, create it, then add your github username and token. This will automatically pull your credentials from this file. 


## Alternate Installation via `.jar`/`.aar`

```kotlin--android
// in your module's build.gradle
dependencies {
  implementation fileTree(dir: 'libs', include: ['*.aar'], exclude: [])
}
```

In the past, we have distributed our SDK as pre-compiled Java archives.  This made it easy to bundle our code, as well as our dependencies, for developers working without Gradle (mostly Unity developers). These files should still work fine in a Gradle environement, though you may run into dependency conflicts if you are trying to pull in any of our bundled dependencies via a Maven repository as well.  If this happens, you can either remove the dependency declaration from your `build.gradle` (and rely on the one we bundle) or, you can manually unzip the `.aar`, remove the conflicting `.jar`, and re-zip the `.aar`.

To use, simply add our `.aar` file to the folder called `libs` inside your application module's root, then instruct Gradle to look for dependency artifacts inside this folder as demonstrated in the code sample.

# Reactive Programming

```kotlin--android
import com.badoo.reaktive.observable.*
import com.badoo.reaktive.scheduler.*

// Simple reactive example using an interval timer.
class MainActivity2 : AppCompatActivity() {
    
  private lateinit var intervalDisposable: Observable<Long>
  
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    // ... Activity setup ...
    val intervalPeriodMilliseconds = 2000L
    val backgroundScheduler: Scheduler = ioScheduler
    // Creates an observable that emits a Long every 2000 milliseconds. 
    // The scheduler tells the observable "where" to execute.
    val intervalObservable: Observable<Long> = observableInterval(intervalPeriodMilliseconds,backgroundScheduler)
    intervalDisposable = intervalObservable.subscribe(onNext = { intervalCount ->
      println("Interval $intervalCount has passed.")
    }, onError = { throwable ->
      println("An error has occurred: $throwable")
    })
  }

  override fun onDestroy() {
    super.onDestroy()
    // Always disconnect from the observable at the end of appropriate lifecycle!
    // The onNext and onError callbacks will not be called after this point.
    intervalDisposable.dispose()
  }
}
```

```kotlin--jvm
import com.badoo.reaktive.observable.*
import com.badoo.reaktive.scheduler.*

// Simple reactive example using an interval timer.
fun main() {
  val intervalPeriodMilliseconds = 2000L
  val backgroundScheduler: Scheduler = ioScheduler

  // Creates an observable that emits a Long every 2000 milliseconds. 
  // The scheduler tells the observable "where" to execute.
  val intervalObservable: Observable<Long> = observableInterval(intervalPeriodMilliseconds, backgroundScheduler)
  val intervalDisposable = intervalObservable.subscribe(onNext = { intervalCount ->
    // this callback is invoked whenever the observable has new data to consume
    println("Interval $intervalCount has passed.")
  }, onError = {throwable ->
    // this callback is invoked if an exception is thrown during the observable's internal execution.
    println("An error has occurred: $throwable")
  })

  while (true) {
    if (readLine() == "exit") {
      break
    }
  }

  // Disconnect from the observable. Neither callback can be invoked after this point.
  intervalDisposable.dispose()
}
```

The Pison SDK exposes `Reactive` endpoints to developers. If you are new to Reactive programming, fear not! You'll only need the bare essentials.

Many of our APIs return objects of the type `Observable<TypeParam>`.
An `Observable` is simply an asynchronous stream of data. The type parameter associated with the `Observable` indicates how the data will be presented. So we say an `Observable<TypeParam>` _emits_ instances of `TypeParam`.

To consume this data, you must provide the observable with: 

1. A callback function to be invoked whenever a new piece of data is available. This is referred to as the `onNext` function.
2. A callback function to be invoked in the case of an error. This is referred to as the `onError` function.

When you are ready to start consuming from an `Observable`, you must pass your callbacks as parameters to one of its `subscribe` methods. 

When you subscribe to an `Observable`, the Reactive framework will return a `Disposable`. When you are ready to _stop_ consuming data, you must call the `Disposable`'s `dispose()` method.

Check out the code examples for a simple, fully-functional demo.

If you'd like to learn more about Reactive programming in general, we recommend [this introduction.](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)

The Pison SDK is currently leveraging the [Reaktive](https://github.com/badoo/Reaktive) library under the hood. 

# Creating a 'PisonSdk' instance

```kotlin--android
import com.pison.core.client.newPisonSdkInstance

// ...
// Android factory method requires a Context instance
val pisonSdk = newPisonSdkInstance(applicationContext)
```

```kotlin--jvm
import com.pison.core.client.newPisonSdkInstance

// ...
val pisonSdk = newPisonSdkInstance()
```
All access to Pison data begins with the `PisonSdk` interface.

To produce an instance, use our factory function: 

`com.pison.core.client.newPisonSdkInstance`

# Connecting to the server

*Reminder: Please ensure that you have the Pison Server installed and running.*

The Pison Server will provide your application with all real-time, Pison-related data. There are 3 ways to connect to it:

## 1. Local Pison Server

```kotlin--android
import com.pison.core.client.*

val pisonSdk: PisonSdk = newPisonSdkInstance(applicationContext)
val remoteServer: PisonRemoteServer = pisonSdk.bindToLocalServer()
```

```kotlin--jvm
import com.pison.core.client.*

val pisonSdk: PisonSdk = newPisonSdkInstance()
val remoteServer: PisonRemoteServer = pisonSdk.bindToLocalServer()
```
If your application runs on the same hardware as the Pison Server, this is the quickest/simplest option.
Use the `PisonSdk`'s extension function:

`com.pison.core.client.bindToLocalServer`

This function returns an instance of `PisonRemoteServer`, which is documented in following sections.

Note that the actual "binding" happens lazily, so this call will not fail immediately if the server is not available.

## 2. Named Pison Server

```kotlin--android
import com.pison.core.client.*
import com.pison.core.shared.DEFAULT_PISON_PORT

val pisonSdk: PisonSdk = newPisonSdkInstance(applicationContext)
// This could also be an IP address encoded as a string, like "192.168.1.24"
val serverLocation = "Android.local"
val remoteServer: PisonRemoteServer = pisonSdk.bindToServer(hostAddress = serverLocation, port = DEFAULT_PISON_PORT)
```

```kotlin--jvm
import com.pison.core.client.*
import com.pison.core.shared.DEFAULT_PISON_PORT

val pisonSdk: PisonSdk = newPisonSdkInstance()
// This could also be an IP address encoded as a string, like "192.168.1.24"
val serverLocation = "JVM.local"
val remoteServer: PisonRemoteServer = pisonSdk.bindToServer(hostAddress = serverLocation, port = DEFAULT_PISON_PORT)
```

If your application runs on a different machine than the Pison Server, you can point directly to it by providing a hostname (or IP address) and a port number. Both machines _must_ be on the same network for this to work! 

For this approach, use the `PisonSdk`'s `bindToServer` method.

> NOTE: Most builds of the Pison Server are currently configured to run by default on the port number defined by `com.pison.core.shared.DEFAULT_PISON_PORT`. This may not always be the case!

> NOTE: The actual "binding" happens lazily, so this call will not fail immediately if the server is not available.

## 3. Automatic Service Discovery (Experimental) 

> Example demonstrating how to automatically bind the the first Pison Server discovered on a network.

```kotlin--android

// be sure to dispose of both of these during an appropriate Android lifecycle event
lateinit var browseDisposable: Disposable
lateinit var bindDisposable: Disposable
  
val pisonSdk = newPisonSdkInstance(applicationContext)
browseDisposable = pisonSdk
  .browseForPisonServers()
  .subscribe(onNext = { pisonAdvertisementList ->
    // bind to the first server we find
    pisonAdvertisementList.firstOrNull()?.let {
        bindToServer(it)
    }
  }, onError = { throwable ->
    println("Error browsing for servers: $throwable")
  })

fun bindToServer(advertisement: PisonServerAdvertisement) {
  // stop browsing as soon as we find a server to bind to
  browseDisposable.dispose()
  bindDisposable = advertisement.bind().subscribe(onSuccess = { server ->
    println("bound to server: ${server.name}")
  }, onError = { throwable ->
    println("error binding to server $throwable")
  })
}
```

```kotlin--jvm

lateinit var browseDisposable: Disposable
lateinit var bindDisposable: Disposable
  
val pisonSdk = newPisonSdkInstance()
browseDisposable = pisonSdk
  .browseForPisonServers()
  .subscribe(onNext = { pisonAdvertisementList ->
    // bind to the first server we find
    pisonAdvertisementList.firstOrNull()?.let {
        bindToServer(it)
    }
  }, onError = { throwable ->
    println("Error browsing for servers: $throwable")
  })

fun bindToServer(advertisement: PisonServerAdvertisement) {
  // stop browsing as soon as we find a server to bind to
  browseDisposable.dispose()
  bindDisposable = advertisement.bind().subscribe(onSuccess = { server ->
    println("bound to server: ${server.name}")
  }, onError = { throwable ->
    println("error binding to server $throwable")
  })
}
```

It is possible to automatically discover Pison Servers running on your network.

Use the `PisonSdk`'s `browseForPisonServers()` method to get an `Observable` of `PisonAdvertisement`s. 
When you susbscribe to this Observable, you will periodically receive a `List` of the currently "visible" Pison Servers on your network. The list provided is always as up-to-date as possible - if a server becomes available or disappears, your `onNext` callback will be invoked with an entirely new list.

Each `PisonAdvertisement` has a `transport` field of type `PisonServerTransport`. This determines _how_ a given server expects to be communicated with. In a large majority of cases, this should not matter to you as an application developer, so you can simply use the advertisement's `bind` extension function to access that server. Note that `PisonServerAdvertisement.bind()` returns an instance of `Single<PisonRemoteServer>`. A `Single` is a special type of Reactive `Observable` that promises to only emit a single item. So to access the `PisonRemoteServer`, you must subscribe to the `Single`.

> Here's a consolidated version of the same example, using built-in Reactive extensions.

```kotlin--android
val pisonSdk = newPisonSdkInstance(applicationContext)
val browseDisposable = pisonSdk
  .browseForPisonServers()
  // only forward non-empty lists
  .filter { it.isNotEmpty() }
  // only forward the first item from each list.
  .map { it.first() }
  // only forward the first emitted advertisement.
  .firstOrComplete()
  // automatically bind to that advertisement.
  .flatMapSingle { it.bind() }
  .subscribe(onSuccess = { server ->
    println("bound to server: ${server.name}")
    // implement this yourself!
    onServerBound(server)
  }, onError = {
    println("error binding to server $throwable")
  })
```

```kotlin--jvm
val pisonSdk = newPisonSdkInstance()
val browseDisposable = pisonSdk
  .browseForPisonServers()
  // only forward non-empty lists
  .filter { it.isNotEmpty() }
  // only forward the first item from each list.
  .map { it.first() }
  // only forward the first emitted advertisement.
  .firstOrComplete()
  // automatically bind to that advertisement.
  .flatMapSingle { it.bind() }
  .subscribe(onSuccess = { server ->
    println("bound to server: ${server.name}")
    // implement this yourself!
    onServerBound(server)
  }, onError = {
    println("error binding to server $throwable")
  })
```

# Monitoring Pison Devices

```kotlin--android
// ...
var connectionsDisposable: Disposable?

fun onServerBound(server: PisonRemoteServer) {
  // cancel any existing subscriptions
  connectionsDisposable?.dispose()
  connectionsDisposable = it.monitorConnections().subscribe(onNext = {
    when(it) {
      is ConnectedDeviceUpdate -> {
        println("Pison device connected: ${it.connectedDevice.deviceName}")
        // impement this yourself!
        onDeviceConnected(it.connectedDevice)
      }
      DisconnectedDeviceUpdate -> println("Pison device disconnected!")
      is ConnectedFailedUpdate -> println("Error connecting to Pison Device: ${it.reason}")
    }
  }, onError = {
    println("error while monitoring connections: $it")
  })
}
```

```kotlin--jvm
// ...
var connectionsDisposable: Disposable?

fun onServerBound(server: PisonRemoteServer) {
  // cancel any existing subscriptions
  connectionsDisposable?.dispose()
  connectionsDisposable = it.monitorConnections().subscribe(onNext = {
    when(it) {
      is ConnectedDeviceUpdate -> {
        println("Pison device connected: ${it.connectedDevice.deviceName}")
        // impement this yourself!
        onDeviceConnected(server, it.connectedDevice)
      }
      DisconnectedDeviceUpdate -> println("Pison device disconnected!")
      is ConnectedFailedUpdate -> println("Error connecting to Pison Device: ${it.reason}")
    }
  }, onError = {
    println("error while monitoring connections: $it")
  })
}
```

Once you are "bound" to a Pison Server, you can request to be updated about connections to Pison hardware. 

To do this, use the `PisonRemoveServer`interface's `monitorConnections()` method. This will return an `Observable<ConnectionUpdate>`, which emits each time a Pison Device connects to or disconnects from the server.

`ConnectionUpdate` is a sealed class, so you may need to check its type before getting all the data you need.

# Real-Time Device Data

```kotlin--android
// ...
var deviceMonitorDisposable: Disposable?
var dataDisposable: Disposable?
fun onDeviceConnected(server: PisonRemoteServer, connectedDevice: ConnectedDevice) {
  deviceMonitorDisposable?.dispose()
  deviceMonitorDisposable = server.monitorDevice(connectedDevice).subscribe(onNext = { remoteDevice ->
    println("monitoring device: ${remoteDevice.deviceId}")
    onMonitoringDevice(remoteDevice)
  }, onError = {
    println("device ${connectedDevice.deviceName} has disconnected!")
  })
}

fun onMonitoringDevice(remoteDevice: PisonRemoteClassifiedDevice) {
  dataDisposable?.dispose()
  // CompositeDisposable can group a bunch of disposables together
  dataDisposable = CompositeDisposable()
  remoteDevice.monitorGestures().subscribe { gesture ->
    println("user has performed gesture: $gesture")
  }.also { dataDisposable.add(it) }

  remoteDevice.monitorDeviceState().subscribe { deviceState ->
    println("device battery life: ${deviceState.battery}%")
  }.also { dataDisposable.add(it) }

  // remoteDevice.monitorEulerAngles()
  // remoteDevice.monitorActivationStates()
  // remoteDevice.monitorQuaternion()
  // remoteDevice.monitorLockState()
  // ... see full example below for usage of each of these.
}
```

```kotlin--jvm
// ...
var deviceMonitorDisposable: Disposable?
var dataDisposable: Disposable?
fun onDeviceConnected(server: PisonRemoteServer, connectedDevice: ConnectedDevice) {
  deviceMonitorDisposable?.dispose()
  deviceMonitorDisposable = server.monitorDevice(connectedDevice).subscribe(onNext = { remoteDevice ->
    println("monitoring device: ${remoteDevice.deviceId}")
    onMonitoringDevice(remoteDevice)
  }, onError = {
    println("device ${connectedDevice.deviceName} has disconnected!")
  })
}

fun onMonitoringDevice(remoteDevice: PisonRemoteClassifiedDevice) {
  dataDisposable?.dispose()
  // CompositeDisposable can group a bunch of disposables together
  dataDisposable = CompositeDisposable()
  remoteDevice.monitorGestures().subscribe { gesture ->
    println("user has performed gesture: $gesture")
  }.also { dataDisposable.add(it) }

  remoteDevice.monitorDeviceState().subscribe { deviceState ->
    println("device battery life: ${deviceState.battery}%")
  }.also { dataDisposable.add(it) }

  // remoteDevice.monitorEulerAngles()
  // remoteDevice.monitorActivationStates()
  // remoteDevice.monitorQuaternion()
  // remoteDevice.monitorLockState()
  // ... see full example below for usage of each of these.
}
```

Once you know a Pison Device has connected to the server, you can request real-time updates from that device. 

Use `PisonRemoteServer`'s `monitorDevice` method to request updates from a specific device. 

The `monitorDevice` method returns an `Observable<PisonRemoteClassifiedDevice>`. When you subscribe to this `Observable`, the server validates the `ConnectedDevice` that you've passed as a parameter and a `PisonRemoteClassifiedDevice` is emitted (assuming the Pison Device is still connected).

The `PisonRemoteClassifiedDevice` interface provides an `Observable` for each of the data streams available from the Pison device/server. The data streams fall into 1 of 3 categories:

### 1. User State

* `Activation States` - information about what the user is likely doing with their hand. These data points are all inferred by Pison's classification algorithms.

* `Euler Angles / Quaternion` - information about where the user has their arm positioned.

### 2. Device State

* `Imu` - Latest readings from the Vulcan Control Band's gyroscope, accelerometer, and magnetometer. These readings include accuracy information in the form of 2 variables: `Accuracy` and `Status`.
	* `Accuracy` is a float that indicates the estimated accuracy to a few degrees.
	* `Status` is an integer that will be from 0-3. 
		* `0` is an accuracy of over 60 degrees. 
		* `1` is an accuracy of 45 to 60 degrees. 
		* `2` is an accuracy of 16 to 45 degrees. 
		* `3` is most accurate with a reading less than 8 degrees. 

`NOTE:`Between 8 to 16 deg will be either 3 or 2 depending on the previous value.

The IMU reported status will degrade over time naturally. In order to improve accuracy, the user simply needs to move the Vulcan Control Band in 3-dimensional space. It is recommended to have the user move the Vulcan Control Band in a figure 8 motion for a few moments.     

* `Device Meta` - Information about the Vulcan Control Band itself and its connection to the server, battery life, RSSI, etc.

* `Lock State` -  Indicates whether or not the Vulcan Control Band is considered "locked". A "locked" device will not identify or issue gesture events but will still report IMU data. Applications may choose to ignore the real-time IMU data while this is true.


### 3. Events

* `Gesture` - Events emitted when Pison's classification algorithms have determined that user performed a specific gesture. 

# Haptics

Haptic technology is a type of technology that allows users to feel tactile feedback through their skin. The technology can be used to provide users with various types of feedback, such as notifications, alerts, and even virtual touches or textures. Haptic feedback can be an important aspect of the user experience in many different types of applications, as it allows users to better interact with and understand their environment.

## Haptic Attributes

Pison provides four (4) haptic attributes that can be controlled or modified to create different types of tactile sensations:

* **Frequency:** The frequency of a haptic buzz refers to the number of vibrations or oscillations per second. Higher frequencies can create a more rapid or "buzzy" sensation, while lower frequencies can produce a slower, deeper vibration.

* **Amplitude:** The amplitude of a haptic buzz refers to the magnitude or intensity of the vibration. Larger amplitudes can create stronger, more noticeable vibrations, while smaller amplitudes will produce weaker, less noticeable sensations.

* **Duration:** The duration of a haptic buzz is the length of time that the vibration is applied. Longer durations can create a sustained vibration, while shorter durations will produce a brief, transient sensation.

* **Waveform:** The waveform of a haptic buzz refers to the shape of the electrical input signal used to drive the haptic motor. Different waveforms can produce different types of tactile sensations, such as sinusoidal waves for smooth, continuous vibrations or square waves for sharp, abrupt vibrations.

## Haptic Examples

> Here's an example of turning haptics on with an intensity of 90

```kotlin--android
// ... 

        GlobalScope.launch {
            val hapticIntensity: Int = 90 
            
            val hapticOn = HapticOnCommand(hapticIntensity)
            
            pisonRemoteDevice?.sendHaptic(hapticON)
        }
```
```kotlin--jvm
// ... 

        GlobalScope.launch {
            val hapticIntensity: Int = 90 
            
            val hapticOn = HapticOnCommand(hapticIntensity)
            
            pisonRemoteDevice?.sendHaptic(hapticON)
        }
```
> Here's an example of turning haptics off

```kotlin--android
// ... 
        GlobalScope.launch {
            val hapticOff = HapticOffCommand
            
            pisonRemoteDevice?.sendHaptic(hapticOff)
        }
```
```kotlin--jvm
// ... 
        GlobalScope.launch {
            pisonRemoteDevice?.sendHaptic(HapticOffCommand)
        }
```
> Here is an example of sending 3 haptic bursts with an intensity of 60 and a duration of 1000 milliseconds per burst

```kotlin--android
// ... 
        GlobalScope.launch {
            val hapticIntensity: Int = 60
            val hapticDurationinMs = 1000
            val numberOfBursts = 3

            val hapticBurst = HapticBurstCommand(hapticIntensity, hapticDurationinMs, numberOfBursts)

            pisonRemoteDevice?.sendHaptic(hapticBurst)
        }
```

```kotlin--jvm
// ... 
        GlobalScope.launch {
            val hapticIntensity: Int = 60
            val hapticDurationinMs = 1000
            val numberOfBursts = 3

            val hapticBurst = HapticBurstCommand(hapticIntensity, hapticDurationinMs, numberOfBursts)

            pisonRemoteDevice?.sendHaptic(hapticBurst)
        }
```

> Here is an example of sending a custom haptic "waveform" with a low intensity pulse followed by a high intensity pulse

```kotlin--android
// ... 
        GlobalScope.launch {
            // A low intensity pulse for 200ms, followed by 200ms of silence, 
            // followed by a high intensity pulse for 200ms
            val hapticQueue = HapticQueueStep(listOf(
                HapticQueueStep(intensity = 20, timeOn = 200, timeOff = 200),
                HapticQueueStep(intensity = 90, timeOn = 200, timeOff = 200)
            ))

            pisonRemoteDevice?.sendHaptic(hapticQueue)
        }
```

```kotlin--jvm
// ... 
        GlobalScope.launch {
            // A low intensity pulse for 200ms, followed by 200ms of silence, 
            // followed by a high intensity pulse for 200ms
            val hapticQueue = HapticQueueStep(listOf(
                HapticQueueStep(intensity = 20, timeOn = 200, timeOff = 200),
                HapticQueueStep(intensity = 90, timeOn = 200, timeOff = 200)
            ))

            pisonRemoteDevice?.sendHaptic(hapticQueue)
        }
```

The SDK allows applications to send haptics to Pison devices using `Haptic Commands`. 

Before sending any haptics, make sure you have an actively monitored `pisonRemoteDevice` connected to the server.

Check out the `Connecting to a Server` and `Monitoring Pison Devices` sections to see how to bind to the pison server and monitor states in a `pisonRemoteDevice` 

* `sendHaptic()` - call this method and pass a haptic command of your choosing as a parameter. This is a suspend function, therefore you will need a coroutine to call this method. 

Keep in mind that you must turn haptics on before sending any bursts. 

If you are new to coroutines check out [this overview](https://kotlinlang.org/docs/reference/coroutines/basics.html) and [this codelab](https://developer.android.com/codelabs/kotlin-coroutines#0) to get familiar with them. 

### 1. HapticOnCommand

* The `HapticOnCommand(intensity)` turns haptics on. 

* It takes an integer between 0 and 100 for intensity and alerts the user that is on by sending a sending a prolonged haptic vibration

### 2. HapticOffCommand

The `HapticOffCommand()` turns haptics off and has no parameters. 

### 3. HapticBurstCommand

The `HapticBurstCommand(intensity, durationMS, numberBursts)` sends one or more haptic bursts vibration with of a configurable length, intensity and number of bursts.

It takes as parameters:

* `intensity` - an integer with a range of 1 - 100
* `duration` an integer of the vibration in milliseconds with a range of 50 - 2000
* `number of bursts` an integer for the number of bursts with a range of 1 - 100 

### 4. HapticQueueCommand

The `HapticQueueCommand(queueSteps)` sends a custom sequence of pulses defined by the caller.

It takes as parameters:

* `queueSteps` - a `List<HapticQueueStep>` (see below). The steps are executed back-to-back, in order.

Each `HapticQueueStep` takes the following parameters:

* `intensity` - an integer with a range of 1 - 100 for this pulse
* `timeOn` - an integer representing the amount of time in milliseconds that this pulse should last.
* `timeOff`- an integer representing the amount of time in milliseconds of "silence" (haptics off) between this pulse and the next.
* `override` - an optional boolean value, internal use only. Defaults to false.
