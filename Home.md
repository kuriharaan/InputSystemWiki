This is an overview of the architecture of the system.

The system breaks down into a "passive" and an "active" part.

# Passive

The "passive" part is concerning with capturing state using minimal per-frame processing. It is largely set up automatically but can be configured/adjusted by the user. It has a zero-setup API that will be familiar to users of Unity's existing input system.

## Controls

A control captures a (possibly structured) value.

There is a range of pre-defined control types: [ButtonControl](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Controls/ButtonControl.cs), [AxisControl](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Controls/AxisControl.cs), [StickControl](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Controls/StickControl.cs), [PoseControl](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Controls/PoseControl.cs), etc.

Controls are named and form hierarchies. Controls in a hierarchy can be looked up by path (e.g. "/gamepad/leftStick" but can also include patterns like in "/*/leftStick").

### Paths

### Usages

### Processors

## Templates

## Devices

## State

Values of controls are stored in state buffers. There are multiple buffers serving different purposes but all devices in the system share the same buffers and all buffers share the same single allocation.

Every control hierarchy gets its own layout which is determined by [InputControlSetup](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Controls/InputControlSetup.cs). The layouts can be a combination of automatic layouting as well as fixed offset layouts supplied by templates.

State is bit-addressable. [ButtonControl](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Controls/ButtonControl.cs) uses this to store its state as a single bit and allows several buttons to be packed into a single int (see [GamepadState](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Devices/Gamepad.cs) for an example).

## Events

Events change the state of the system.

# Active

The "active" part of the system requires explicit set up by the user and incurs processing overhead proportional to the amount of enabled functionality. Unlike the "passive" part, it is concerned with state *change* rather than with state itself.

## Actions

An action monitors for state change and invokes user code in response to them.

An action has four phases:

1. Waiting
2. Started
3. Performed
4. Cancelled

## Modifiers

A modifier controls an action's progression through its phases. An example is a "hold" modifier that will only go to the "Performed" phase after a specified amount of time has elapsed and will go to the "Cancelled" phase if the monitored state goes back to its default before that (in other words, in the case of a button that would mean releasing the button).

## Bindings

A binding correlates an action with one or more sources. While sources can be specified directly on actions, bindings are a way to override and externally supply bindings to actions.

## Action Sets

An action set groups a set of actions and allows them to be enabled and disabled in bulk. Action sets also allow applying binding sets to the entire group of actions as well as getting binding sets from the currently used sources of the actions in a set.

## Binding Sets
