
# Old vs New Input System APIs

This page provides a listing of the APIs in `UnityEngine.Input` (and related APIs in `UnityEngine`) and their corresponding APIs in the new input system.

The new APIs are currently in the `UnityEngine.Experimental.Input` namespace. The namespace is omitted here for brevity. `UnityEngine.Input` is referenced in full for easy disambiguation.

## [`UnityEngine.Input`](https://docs.unity3d.com/ScriptReference/Input.html)

### [`UnityEngine.Input.acceleration`](https://docs.unity3d.com/ScriptReference/Input-acceleration.html)

`Accelerometer.current.acceleration.ReadValue()`

### [`UnityEngine.Input.accelerationEventCount`](https://docs.unity3d.com/ScriptReference/Input-accelerationEventCount.html)

See next section.

### [`UnityEngine.Input.accelerationEvents`](https://docs.unity3d.com/ScriptReference/Input-accelerationEvents.html)

Acceleration events are not made available separately from other input events. The following code will listen to all input events and filter out state changes on `Accelerometer` devices.

```
InputSystem.onEvent +=
    eventPtr =>
    {
        if (eventPtr.IsA<StateEvent>() || eventPtr.IsA<DeltaStateEvent>())
        {
            var accelerometer = InputSystem.TryGetDeviceById(eventPtr.deviceId) as Accelerometer;
            if (accelerometer != null)
            {
                var acceleration = accelerometer.acceleration.ReadValueFrom(eventPtr);
                //...
            }
        }
    }
```

>////TODO: wrap in nicer APIs

### [`UnityEngine.Input.anyKey`](https://docs.unity3d.com/ScriptReference/Input-anyKey.html)

ATM this only exists for the keyboard, not yet in a way where it also covers mouse buttons.

`Keyboard.current.anyKey.isPressed`

### [`UnityEngine.Input.anyKeyDown`](https://docs.unity3d.com/ScriptReference/Input-anyKeyDown.html)

No corresponding API yet.

### [`UnityEngine.Input.backButtonLeavesApp`](https://docs.unity3d.com/ScriptReference/Input-backButtonLeavesApp.html)

No corresponding API yet.

### [`UnityEngine.Input.compass`](https://docs.unity3d.com/ScriptReference/Input-compass.html)

`Compass.current`

### [`UnityEngine.Input.compass.enabled`](https://docs.unity3d.com/ScriptReference/Compass-enabled.html)

```
// Get.
Compass.current.enabled

// Set.
InputSystem.EnableDevice(Compass.current);
InputSystem.DisableDevice(Compass.current);
```

### [`UnityEngine.Input.compass.headingAccuracy`](https://docs.unity3d.com/ScriptReference/Compass-headingAccuracy.html)

No corresponding API yet.

### [`UnityEngine.Input.compass.magneticHeading`](https://docs.unity3d.com/ScriptReference/Compass-magneticHeading.html)

No corresponding API yet.

### [`UnityEngine.Input.compass.rawVector`](https://docs.unity3d.com/ScriptReference/Compass-rawVector.html)

No corresponding API yet.

### [`UnityEngine.Input.compass.timestamp`](https://docs.unity3d.com/ScriptReference/Compass-timestamp.html)

`Compass.current.lastUpdateTime`

>////TODO: time units don't match

### [`UnityEngine.Input.compass.trueHeading`](https://docs.unity3d.com/ScriptReference/Compass-trueHeading.html)

No corresponding API yet.

### [`UnityEngine.Input.compensateSensors`](https://docs.unity3d.com/ScriptReference/Input-compensateSensors.html)

No corresponding API yet.

### [`UnityEngine.Input.compositionCursorPos`](https://docs.unity3d.com/ScriptReference/Input-compositionCursorPos.html)

No corresponding API yet.

### [`UnityEngine.Input.compositionString`](https://docs.unity3d.com/ScriptReference/Input-compositionString.html)

No corresponding API yet.

### [`UnityEngine.Input.deviceOrientation`](https://docs.unity3d.com/ScriptReference/Input-deviceOrientation.html)

No corresponding API yet.

### [`UnityEngine.Input.gyro`](https://docs.unity3d.com/ScriptReference/Input-gyro.html)

`Gyro.current`

### [`UnityEngine.Input.gyro.attitude`](https://docs.unity3d.com/ScriptReference/Gyroscope-attitude.html)

`Gyro.current.orientation.ReadValue()`

### [`UnityEngine.Input.gyro.enabled`](https://docs.unity3d.com/ScriptReference/Gyroscope-enabled.html)

```
// Get.
Gyro.current.enabled

// Set.
InputSystem.EnableDevice(Gyro.current);
InputSystem.DisableDevice(Gyro.current);
```

### [`UnityEngine.Input.gyro.gravity`](https://docs.unity3d.com/ScriptReference/Gyroscope-gravity.html)

`Gyro.current.gravity.ReadValue()`

### [`UnityEngine.Input.gyro.rotationRate`](https://docs.unity3d.com/ScriptReference/Gyroscope-rotationRate.html)

`Gyro.current.angularVelocity.ReadValue()`

### [`UnityEngine.Input.gyro.rotationRateUnbiased`](https://docs.unity3d.com/ScriptReference/Gyroscope-rotationRateUnbiased.html)

No corresponding API yet.

### [`UnityEngine.Input.gyro.updateInterval`](https://docs.unity3d.com/ScriptReference/Gyroscope-updateInterval.html)

`Gyro.current.samplingFrequency = 30.0f; // 30 Hz.`

>////FIXME: time units don't match

### [`UnityEngine.Input.gyro.userAcceleration`](https://docs.unity3d.com/ScriptReference/Gyroscope-userAcceleration.html)

`Gyro.current.acceleration.ReadValue()`

### [`UnityEngine.Input.imeCompositionMode`](https://docs.unity3d.com/ScriptReference/Input-imeCompositionMode.html)

No corresponding API yet.

### [`UnityEngine.Input.imeIsSelected`](https://docs.unity3d.com/ScriptReference/Input-imeIsSelected.html)

No corresponding API yet.

### [`UnityEngine.Input.inputString`](https://docs.unity3d.com/ScriptReference/Input-inputString.html)

```
Keyboard.current.onText +=
    character => /* ... */;
```

### [`UnityEngine.Input.location`](https://docs.unity3d.com/ScriptReference/Input-location.html)

No corresponding API yet.

### [`UnityEngine.Input.location.isEnabledByUser`](https://docs.unity3d.com/ScriptReference/LocationService-isEnabledByUser.html)

No corresponding API yet.

### [`UnityEngine.Input.location.lastData`](https://docs.unity3d.com/ScriptReference/LocationService-lastData.html)

No corresponding API yet.

### [`UnityEngine.Input.location.lastData.altitude`](https://docs.unity3d.com/ScriptReference/LocationInfo-altitude.html)

No corresponding API yet.

### [`UnityEngine.Input.location.lastData.horizontalAccuracy`](https://docs.unity3d.com/ScriptReference/LocationInfo-altitude.html)

No corresponding API yet.

### [`UnityEngine.Input.location.lastData.latitude`](https://docs.unity3d.com/ScriptReference/LocationInfo-latitude.html)

No corresponding API yet.

### [`UnityEngine.Input.location.lastData.timestamp`](https://docs.unity3d.com/ScriptReference/LocationInfo-timestamp.html)

No corresponding API yet.

### [`UnityEngine.Input.location.lastData.verticalAccuracy`](https://docs.unity3d.com/ScriptReference/LocationInfo-verticalAccuracy.html)

No corresponding API yet.

### [`UnityEngine.Input.location.lastData.status`](https://docs.unity3d.com/ScriptReference/LocationService-status.html)

No corresponding API yet.

### [`UnityEngine.Input.location.Start`](https://docs.unity3d.com/ScriptReference/LocationService.Start.html)

No corresponding API yet.

### [`UnityEngine.Input.location.Stop`](https://docs.unity3d.com/ScriptReference/LocationService.Stop.html)

No corresponding API yet.

### [`UnityEngine.Input.mousePosition`](https://docs.unity3d.com/ScriptReference/Input-mousePosition.html)

`Mouse.current.position.ReadValue()`

>NOTE: Mouse simulation from touch is not implemented yet.

### [`UnityEngine.Input.mousePresent`](https://docs.unity3d.com/ScriptReference/Input-mousePresent.html)

`Mouse.current != null`

### [`UnityEngine.Input.multiTouchEnabled`](https://docs.unity3d.com/ScriptReference/Input-multiTouchEnabled.html)

No corresponding API yet.

### [`UnityEngine.Input.simulateMouseWithTouches`](https://docs.unity3d.com/ScriptReference/Input-multiTouchEnabled.html)

No corresponding API yet.

### [`UnityEngine.Input.stylusTouchSupported`](https://docs.unity3d.com/ScriptReference/Input-stylusTouchSupported.html)

No corresponding API yet.

### [`UnityEngine.Input.touchCount`](https://docs.unity3d.com/ScriptReference/Input-touchCount.html)

`Touchscreen.current.activeTouches.Count`

### [`UnityEngine.Input.touches`](https://docs.unity3d.com/ScriptReference/Input-touches.html)

`Touchscreen.current.activeTouches`

### [`UnityEngine.Input.touchPressureSupported`](https://docs.unity3d.com/ScriptReference/Input-touchPressureSupported.html)

No corresponding API yet.

### [`UnityEngine.Input.touchSupported`](https://docs.unity3d.com/ScriptReference/Input-touchSupported.html)

`Touchscreen.current != null`

### [`UnityEngine.Input.GetAccelerationEvent`](https://docs.unity3d.com/ScriptReference/Input.GetAccelerationEvent.html)

See `UnityEngine.Input.accelerationEvents`.

## [`UnityEngine.Handheld`](https://docs.unity3d.com/ScriptReference/Handheld.html)

