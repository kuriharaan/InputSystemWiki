This is an overview of the architecture of the system.

The system breaks down into a "passive" and an "active" part.

# Passive

The "passive" part is concerning with capturing state using minimal per-frame processing. It is largely set up automatically but can be configured/adjusted by the user. It has a zero-setup API that will be familiar to users of Unity's existing input system.

## Controls

A control captures a (possibly structured) value.

There is a range of pre-defined control types: [ButtonControl](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Controls/ButtonControl.cs), [AxisControl](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Controls/AxisControl.cs), [StickControl](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Controls/StickControl.cs), [PoseControl](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Controls/PoseControl.cs), etc.

Controls are named and form hierarchies. Controls in a hierarchy can be looked up by path (e.g. "/gamepad/leftStick" but can also include patterns like in "/*/leftStick").

### Hierarchies and Paths

### Usages

### Processors

## Templates

Templates describe control hierarchies. [InputControlSetup](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Controls/InputControlSetup.cs) turns them into a hierarchy of [InputControls](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Controls/InputControl.cs).

Templates can be constructed in three ways:

1. From JSON
2. Through reflection on control and state types
3. Manually through [InputTemplateBuilder](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Controls/InputTemplateBuilder.cs)

Templates have to be registered explicitly with the system. There is no automatic scanning. Any template can be replaced at any time by simply registering a template with an already registered name. Replacing templates will automatically take effect on all devices that are using the template.

Internally, templates are represented using [InputTemplate](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Controls/InputTemplate.cs). However, these objects are created on-demand only inside [InputControlSetup](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Controls/InputControlSetup.cs) and not kept in memory past device creation. For templates derived from types, only a reference the type and name are kept around. For templates created through JSON, the JSON data itself is kept around. For templates built in code through InputTemplateBuilder, the serialized JSON of the resulting template is kept around.

## Devices

Devices are controls that sit at the root of a control hierarchy. They have to be instances of [InputDevice](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Devices/InputDevice.cs).

## State

Values of controls are stored in state buffers. There are multiple buffers serving different purposes but all devices in the system share the same buffers and all buffers share the same single allocation.

Every control hierarchy gets its own layout which is determined by [InputControlSetup](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Controls/InputControlSetup.cs). The layouts can be a combination of automatic layouting as well as fixed offset layouts supplied by templates.

State is bit-addressable. [ButtonControl](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Controls/ButtonControl.cs) uses this to store its state as a single bit and allows several buttons to be packed into a single int (see [GamepadState](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Devices/Gamepad.cs) for an example).

## Events

Events change the state of the system.

There's two main kinds of events:

1. Events that push new state into devices ([StateEvent](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Events/StateEvent.cs) is a full-state update, [DeltaEvent](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Events/DeltaEvent.cs) is a partial-state update).
2. Events that relate other relevant information about devices ([disconnects](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Events/DisonnectEvent.cs) and [reconnects](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Events/ConnectEvent.cs), [text input](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Events/TextInput.cs), etc).

Events are accumulated on a queue which sits in the engine itself (NativeInputSystem). The event representation is shared with the code code (Modules/Input) and is flexible. An event basically is a FourCC type code, a size in bytes, a timestamps, and a flexible payload. We put some upper bound on the size of events but they can be large.

State events (both [StateEvents](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Events/StateEvent.cs) and [DeltaEvent](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Events/DeltaEvent.cs)) employ identical state layouts with the control hierarchy of the device they are targeting. Basic FourCC type checks are in place to catch blatant errors.

The data in state events is simply memcopied over the current state of the device. A device can choose to send a state snapshot every single frame (e.g. an XInput gamepad backend would query connected gamepads at regular intervals and push their full-state snapshots as single events into the system) or can choose to send events only when actual value changes are detected.

There is no class representation of events. The user can listen to the event stream but will get an [InputEventPtr](https://github.com/Unity-Technologies/InputSystemX/blob/master/Assets/InputSystem/Events/InputEventPtr.cs) that require unsafe code in order to work with the data. For the most part, the event stream is *not* meant for consumption at a user level. It is expected that users dealing with events directly will mostly be those authoring new device backends.

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
