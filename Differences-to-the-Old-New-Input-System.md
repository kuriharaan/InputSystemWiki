This page highlights a number of ways in how [InputSystemX](https://github.com/Unity-Technologies/InputSystemX) differs from [InputSystem](https://github.com/Unity-Technologies/InputSystem). It also tries to highlight similarities.

## Events

* Native and managed use the same event representation
* The structure of native events is still the same
* Events still come in as a chunk of raw memory passed on directly from native
* There is only one event queue and it sits in native
* There are no C# event objects anymore
* There is no routing of events; InputManager processes events in a tight loop
* There are no events anymore that require interpretation

    What this means is that before you had something like PointerEvent, for example, which encapsulated multiple state changes on a pointer device. The Pointer class in its ProcessEventIntoState() method would then figure out how those state changes translated into value changes on controls.

    What you have now is only StateEvents and DeltaEvents. Both memcpy data directly into the state buffer of a device. There can be no interpretation of which data has to go where -- the event needs to come in in the right format.

## State

* State storage is managed centrally; all devices share the same chunk of memory
* State cannot accumulate anymore (-> pointer deltas)
* State cannot automatically reset anymore (-> pointer deltas)

## Devices

* Device profiles are replaced by templates which can both describe as well as alter control setups
* Devices/control setups can be described using JSON

## Controls

* Controls still represent values (possibly structured)
* Controls form hierarchies now
* Devices are the root entry points into control hierarchies
* Controls and devices can be looked up using flexible path strings that can match an arbitrary number of controls
* The concept of "common controls" is replaced by the concept of "usage" which gives meaning to a control

## Actions

* Actions can be lose
* Actions can be created in code without assets to back them
* Actions now deal with state change rather than state values