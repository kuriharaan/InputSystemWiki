This is an overview of the architecture of the system.

The system breaks down into a "passive" and an "active" part.

# Passive

The "passive" part is concerning with capturing state using minimal per-frame processing. It is largely set up automatically but can be configured/adjusted by the user. It has a zero-setup API that will be familiar to users of Unity's existing input system.

## Controls

A control captures a (possibly structured) value.

There is a range of pre-defined control types: [ButtonControl](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Controls/ButtonControl.cs), [AxisControl](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Controls/AxisControl.cs), [StickControl](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Controls/StickControl.cs), [PoseControl](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Controls/PoseControl.cs), etc.

Controls are named and form hierarchies. Controls in a hierarchy can be looked up by path (e.g. "/gamepad/leftStick" but can also include patterns like in "/*/leftStick").

### Paths

## Templates

## Devices

## State

## Events

# Active

## Actions

## Modifiers

## Bindings

## Action Sets

## Binding Sets
