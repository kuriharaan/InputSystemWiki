This page highlights a number of ways in how [InputSystemX](https://github.com/Unity-Technologies/InputSystemX) differs from [InputSystem](https://github.com/Unity-Technologies/InputSystem). It also tries to highlight similarities.

## Events

* Native and managed use the same event representation
* The structure of native events is still the same
* Events still come in as a chunk of raw memory passed on directly from native
* There is only one event queue and it sits in native
* There are no C# event objects anymore
* There is no routing of events; InputManager processes events in a tight loop
* There are no events anymore that require interpretation

>What this means is that before you had something like PointerEvent, for example, which encapsulated multiple state changes on a pointer device. The Pointer class in its ProcessEventIntoState() method would then figure out how those state changes translated into value changes on controls.
>
>What you have now is only StateEvents and DeltaEvents. Both memcpy data directly into the state buffer of a device. There can be no interpretation of which data has to go where -- the event needs to come in in the right format.

* Events are no longer time-sliced

>What this means is that the system will not try to parcel out events individually to fixed updates. Instead, all events will be processed every update (before render updates are still special, though).
>
>One consequence of this is that if you get multiple fixed updates in a single dynamic update, the second fixed update will not see any change in values.

## State

* State storage is managed centrally; all devices share the same chunk of (unmanaged) memory
* State cannot accumulate anymore (-> pointer deltas)
* State cannot automatically reset anymore (-> pointer deltas)

>What these two items mean is that before you were able to just send a bunch of pointer events each having a pointer motion delta. The system would then go and aggregate multiple deltas that happened in a single frame and combine them into a final value. It would also go and reset deltas automatically at the beginning of the next frame.
>
>To get the same effect now, accumulation and resetting has to be handled at the source. I.e. code that generates state that includes deltas has to accumulate samples itself and has to make sure it is sending events to reset deltas when necessary.

* Double buffering (previous and current) is handled centrally instead of per-control

## Devices

* Device profiles are replaced by templates which can both describe as well as alter control setups
* Devices/control setups can be described using JSON
* Metadata for discovered devices is still delivered through InputDeviceDescription (renamed from InputDeviceDescriptor)
* Every device has a numeric ID now which is managed by native

## Controls

* Controls still represent values (possibly structured)
* Controls form hierarchies now
* Devices are the root entry points into control hierarchies
* Devices are themselves controls now
* Controls and devices can be looked up using flexible path strings that can match an arbitrary number of controls
* The concept of "common controls" is replaced by the concept of "usage" which gives meaning to a control
* Control hierarchies are now created in a single place (InputControlSetup) from a single source of data (InputTemplate)

## Actions

* Actions can be lose
* Actions can be created in code without assets to back them
* Actions now deal with state change rather than state values
* Actions solely work through callbacks now (onStarted, onPerformed, onCancelled)
* Actions monitor for state value changes in bulk rather than on a per-action basis
* Bindings are a simple structs now containing a simple action name -> source path mapping
* Actions can be grouped in named sets
* Action sets can be read from JSON
* Bindings can get grouped in named sets
* Binding sets can be read from JSON
* Action sets no longer generate code
