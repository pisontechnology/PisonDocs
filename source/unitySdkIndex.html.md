---
title: Pison Unity SDK Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - csharp

toc_footers:
  - Contact Arthur Wollocko for Support 
  - on Pison Unity SDK
  - <a href='mailto:arthur@pison.com'>arthur@pison.com </a>

includes:
 
search: true

code_clipboard: true
---

# Introduction
> You will be able to view code examples or notes in this dark area.

Welcome to the ***Pison Unity SDK***! 

You can use our SDK to access information from the Pison Device within the Unity environment.
> Tested with Unity Versions 2019.x+

The Unity SDK is written in C#, but we support various other languages with other SDKs (Reach out to our contact email below).  



## Requirements

Utilization of the Pison Unity SDK requires:

* Unity Version 2019.2.x+, though native implementation so should work with previous versions as well
* Pison GAMBIT device, powered on
* Android device with Hubapp (TODO: LINK) application installed and connected to Pison device
* Android device on same network as the Unity application device (or the same device)


## Quick Start
> Follow these instructions to gain access to the SDK, along with integrate the Pison device to any Unity Project in < 2 minutes!


### Donwloading SDK

**Option 1:** Clone Repo

1. Click Code then copy HTTPS
2. Using git GUI of couse clone repository
3. From cloned repostory navigate to UnitySDK>Assets>Pison
4. Copy the Pison Folder
5. Paste it in desired unity project

**Option 2:** Download files

1. Click Code>Download ZIP
2. Export ZIPed file
3. In unzipped file navigate UnitySDK>Assets>Pison
4. Copy the Pison Folder
5. Paste it in desired unity project 


### Unity Set Up

Find the PisonManager Prefab. This is located at .../Pison/Prefabs

Place the prefab into the scene. 

>**Note: 'PisonManager.prefab' implements DontDestroyOnLoad() -- meaning it will persist between scenes!**

Configure the PisonManager. If building to an Android device, this is already configured for localhost. Otherwise, specify IP or Discovery.

The PisonManager should be configured, and you can access the PisonManager directly, or bind to PisonEvents


# Pison SDK Use
> Easy setup is enabled through use of the PisonManager prefab, which contains all relevant scripts needed to develop on the Pison Unity SDK.
 
> This contains all three main scripts
Use of the Pison device within the Unity environment is managed via the ***PisonManager***. 

> Details on each script's inspector variables can be found below in their respected section.

Core SDK Composition:

Class Name | Required | Description
--------- | ------- | -----------
PisonManager.cs | Required | Provides connection and data callbacks from the Pison Device
PisonEvents.cs | Optional | Provides event-based updates from the Pison Device
PisonEmulator.cs | Optional | Provides in-editor functionality to invoke updates to PisonManager and PisonEvents 



# PisonManager.cs

The PisonManager.cs class is the main class within the Pison Unity SDK. This contains critical connection capabilities (connection to the Pison Server/Hubapp), as well as callbacks from the SDK when device state changes.
The PisonManager provides real-time access to current device state, including current:
> Note: Orientation Euler Angles double as a compass on the device, with 0 degrees (euler.y) pointing to true-north

* Activation Status
* IMU Information (Gyro, Accel)
* Orientation (Euler Angles and Quaternion)
* Gesture callbacks
* Device State (battery, connection)



**Nice to know Inspector variables:**

* Use Auto Discorery Bind First (bool):
  * If true will automatically connect Hubapp address to computers address
* Hubapp Server Adress (string):
  * Used to mannually set Hubapp to computers address
* Pison Conection Type (enum)
  * Gestures and Connections: Get only gesture information from Hubapp
  * Raw Data nd Connections: Get only raw data of instraments in Pison Device

## Direct Connection


> To execute a direct connection, the PisonManager.cs invokes the following code in its Start() function:

```csharp
 string hubappServerAddress = "127.0.0.1";
 int port = 8002;
 pisonClient = sdk.newPisonClient(hubappServerAddress, "Hubapp Pison Server", port);
 pisonClient.BindToPisonServer(this);
```

> Note: 8002 is the default Hubapp connection port.

The Pison Unity SDK supports direct connection to an instance of the Pison Server, which runs inside "Hubapp" on the Android phone connected to the Pison Device. 
Direct connection occurs via an IP address and a port. If using the same device for Hubapp and the Unity application, localhost (127.0.0.1) is typically preferred.


## Discovery Connection


> To execute discovery connection, the PisonManager.cs (optionally) invokes the following code in its Start() function:

```csharp
PisonSdk sdk = new PisonSdk();
sdk.BindToFirstDiscoveredServer(client =>
{
    Debug.Log($"Discovered client! {client.GetIdentifier()}");
    pisonClient = client;
    pisonClient.BindToPisonServer(this);
    DeviceConnected();
});

```

> Note: 8002 is the default Hubapp connection port.

The Pison Unity SDK supports direct connection to an instance of the Pison Server, which runs inside "Hubapp" on the Android phone connected to the Pison Device.
Discovery connection does not require an IP address and a port, but instead searches the local network for PisonServer instances. Once found, a connection is established.

# PisonEvents.cs

The PisonEvents.cs class is an optional subclass that REQUIRES the PisonManager to be on the same GameObject. 
The Unity Prefab included (in /Prefabs) contains this structure. 
The PisonEvents provides event-based access to device adjustments. These include

> Note: Orientation Euler Angles double as a compass on the device, with 0 degrees (euler.y) pointing to true-north

* Activation changes
* IMU Information changes (Gyro, Accel)
* Orientation changes(Euler Angles and Quaternion)
* Gesture events
* Device State changes (battery, connection)

## Event Data

EventName | Paramters Passed | Description
--------- | ------- | -----------
OnGesture | OnGestureDelegate(ImuGesture gesture) | As a gesture is executed, the ImuGesture 'gesture' contains the new gesture
OnOrientation | OnOrientationChangedDelegate(Vector3 newEulers) | As orientation adjustments are made, the Vector3 newEulers contains the new euler Values. **Note: euler.y doubles as a compass, with 0 degrees pointing true north**
OnQuaternion | OnQuaternionChangedDelegate(Vector4 newQuat) | As rotation adjustments are made, the Vector4 newQuat contains the new quaternion rotation Values. 
OnImuAccelerationGyro | OnImuChangedDelegate(Vector3 newAcceleration, Vector3 newGyroscope) | As IMU adjustments are made, the Vector3s newAcceleration and newGyrsoscope contain IMU information with the new IMU values
OnActivation | OnActivationDelegate(ActivationStates activation) | As an activation (Index finger, watchcheck) is executed, the ActivationStates 'activation' contains the new state information (including if it is an ActivationState.HOLD or ActivationState.NONE)
OnDeviceState | OnDeviceStateDelegate(DeviceState deviceState) | As a state is changed for the device, this event is fired. Device states include connection and battery information
OnDeviceConnection | OnDeviceConnectionDelegate(bool connected) | As the device is connected and disconnected, this event is fired with the new connected value (boolean). ** Note: Currently activation events are not called by the SDK. **
OnDeviceError | OnDeviceErrorDelegate(string error) | As device errors are encountered, this event is fired and contains the error message



# PisonEmulator.cs
> For full functionality testing, please make sure to test against an actual Pison Device
The PisonEmulator is a developer tool that allows for rapid prototype and testing within the editor.

It provides numerous buttons and sliders (within the Unity editor) -- See custom editor window --  that enact changes on the PisonManager.cs and invoke events on the PisonEvents.cs classes to test functionality while running the application in editor.


# Included Examples
The core Pison Unity SDK provides many simple usage examples to guide developer interaction with the Pison device.


## Data Examples
> Use this example to help test connectivity between Pison Device and Unity Engine

Data scene will show if the Pison Device is connected properly, along with display time and the ADC raw data. 


## Simple Examples
> Use this example to view both real-time access of data from the device, as well as utilization of the Events and Haptic interfaces

The Simple example provides readouts of all of the values from the PisonManager in real time, as well as binding to several of the PisonEvent functions for gesture and connectivity/device state support.
Outputs are shown accessed at Unity's resolution via direct access to the PisonManager variables

Gestures made will stimulate the user with a haptic response, so this is a good place to look for Haptic stimulation


## SwipeGesture Examples
> Use this example to see event-driven interactions in the form of a simple game
Provides an event-based binding example to pison events for gesture, moving providing a "goal" state for the cube, and allowing 
the user to move the cube around the environment via gesture. 

Activation is also visualized, as the Pison logo on top of the cube.


## Menu Examples
> Use this example to see event-driven UI menu navigation that supports Unity's native UI event system
This is a simple example of menu-based navigation, which allows for user gesture to navigate the menu.
Gesture Roll Left/Right is used to focus on the next button, and Gesture IndexClick is used for selection.

This implementation utilizes unity's native event system, firing the OnClick behavior of the buttons when selected. 


## Vulcan SwipeGesture Examples
>  **Example Idea:** This example can show the new gestures Unity SDK supports with the Vulcan device. Specifically thumb swipe, and full hand extension

Moves a cube with thumb swipes in the direction of a provided goal, along with changing cube color with a full hand hold.


## Vulan Gesture Combination Examples
>  **Example Idea:** This example can show how two types of gestures can be stringed together to create a sequence of inputs to interact with different events.

Navigate a selection process between different cubes, yet can only select new objects after a wrist roll into thumb swipe in desired direction. 


## Wrist Gestures
> **Example Idea:** This example could have a UI panel on the top half of the screen and a cube on the bottom half of the screen. Similar to the Swipe gesture example, the cube can be on a track of some kind, where a goal particle effect will appear either on the right, left, or middle. If the goal is on the right the UI will update telling you to "right wrist flick", doing so will have the cube slide to the right before settling back to the middle. If the goal is on the left it will be the same goal as to the right but instead, the UI will update to say "left wrist flick", and doing the action will cause the cube to slide to the left before resettling in the middle. If the goal is in the middle the UI will update saying "Wrist shake" if done the cube will shake in place back and forth, for a couple of seconds once settled will activate the next goal. This will help demonstrate the gestures functionality for anyone interested.

***Work In Progress***


## Hand Zones
> **Example Idea:** This example could have a UI panel on the left half of the screen and a cube on the right half of the screen with a Pison logo on it. If the hand is in the NORMAL state the cube stays in the center of the screen. If the hand enters the HANDUP state the cube will slide to the top of the screen, if the hand is in the HANDDOWN state the cube will go to the bottom of the screen. if the hand is inverted the cube will flip sized showing a different type of Pison logo. The UI will display if the hand is inverted, the current hand state and the acceleration of the Pison Device. This will help demonstrate all the major factors of the modular for anyone looking into it. 

***Work In Progress***


## PisonExampleActivationEvents.cs
> Use this example to see how to bind Unity events (in-editor) to pison generated events easily. 
This is a very abstract and flexible implementation of binding a Unity event to the activation of the pison device.

It will provide a Unity event list for both Activation and WatchCheck, and when those occur, fire each of the methods in the event list provided by the developer within the unity editor.
This script is not currently deployed in any scene, but provided as a reference


# Permissions Required
> Android permissions enabled via (placed into the Android Manifest):

```csharp
<uses-permission android:name="android.permission.INTERNET" /> 
```

Communications with the pison device require local area network permissions.
Therefore, depending on the platform, please ensure your permissions are properly set. 

 Windows Standalone -- Please make sure your firewall is set to allow networking through for the selected application/Unity editor. 

 UWP (HoloLens) -- Please enable the capabilities (via the manifest, or via Unity's manipulation of the manifest in Player Settings > Publishing Settings

 Android -- Either directly place the permissions.INTERNET permission into the manifest, or utilize Unity's PlayerSettings to enable this from the editor. In the latter case, Unity will add the appropriate permissions into the manifest automatically

# Contact and Support
> ***TBD***

For support on the Unity SDK, reach out to ***TBD*** for assistance.
