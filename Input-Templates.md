>Note that for most use cases of the input system, it is not necessary to be aware of templates and of what they do. Understanding templates is necessary for authoring support for new devices but is not necessary for using already existing support of devices. When authoring new templates, understanding the [state system](Input-State) can be helpful.

# What do they do?

They tell the system how to represent input devices. Specifically, they describe the arrangement of hierarchies of input controls and the correlation of those controls to input state stored in raw memory.

Example: the "Stick" template creates a hierarchy with a `StickControl` at the top, six `AxisControls` underneath ('left', 'right', 'top', 'down', 'x', and 'y'), and a default state of two floats with 'left', 'right' and 'x' reading from the first float and 'top', 'down', and 'y' reading from the second float.

![Stick Control Template](Images/StickControlTemplate.png)

When a device is plugged into the system, it looks for a matching template that can be used with the device. This is done by looking at the `InputDeviceDescription` provided for the device by the input runtime (`IInputRuntime`) and comparing it with the descriptions associated with the various templates that have been registered. Also, there is a callback `InputSystem.onFindMatchingTemplate` that can be used to programmatically alter the matching logic. If a matching template is found, it is "instantiated", i.e. turned into an actual `InputDevice` instance by `InputControlSetup`.

To look at all the templates that are registered in the system, open the Input Debugger (`Window >> Input Debugger`) and look under the "Templates" section.

![Input Debugger Templates View](Images/InputDebuggerTemplatesView.PNG)

Templates are also used for being able to create input devices that do not exist locally. A player connected to the editor will send all its templates to the editor and then instruct the editor which templates to instantiate devices from. This allows the editor to replicate devices with the exact same memory layouts they have in the player and thus consume input events directly from the player.

# How do I create them?

In one of three ways:

1. C# reflection
2. JSON
3. Manual C# construction code

All three ways produce the same format: an instance of `InputTemplate`. These are created on the fly and on demand during control creation and then thrown away.

Any `InputTemplate`, regardless of which way it was created, can be turned back into JSON. This is used for sending templates over the wire (e.g. when debugging player input in the editor).

Note that for templates to be seen by the system, they have to be explicitly registered.

1. To register C# types as templates, use `InputSystem.RegisterTemplate<T>(string)`.
2. To register a JSON template, use `InputSystem.RegisterTemplate(string, string).`
3. To register a manual template builder, use `InputSystem.RegisterTemplateConstructor`.

## C# Reflection

When registering a C# class as a template with `InputSystem.RegisterTemplate<T>`, reflection is used to automatically discover the control setup of the given class. Two attributes -- `InputTemplateAttribute` and `InputControlAttribute` -- can be used to guide and supplement this process.

>NOTE: If `T` is directly or indirectly derived from a type that is itself registered as a template, that template is taken as a base template for `T`. I.e. the template for `T` will inherit all the settings of the existing template and merge its own settings on top.

In this process the code goes looking through all properties and fields of the class (directly declared *and* inherited ones) and for each one that classifies as an `InputControl`, adds information about the control to the template. Every field or property that either has a type derived from `InputControl` or has an `InputControlAttribute` applied to it classifies.

![InputControlAttribute](Images/InputControlAttributes.png)

Defaults for the control added to the template are taken from the property or field. All defaults can be overridden with explicit settings in `InputControlAttribute`.

- `name` defaults to the name of the property or field.
- `template` defaults to what is inferred from the type of the field. If the type name ends in "Control", the suffix is removed and the resulting name taken as the name of the template ("ButtonControl" -> "Button"). Otherwise, if the type is not a primitive type, the name of the type is taken as the name of the template as is. If the the type is a primitive type, no template name is inferred. (`////REVIEW: The inference mechanism is questionable and likely to go away.`)
- `offset` defaults to the offset of the field (not applicable to properties).

## JSON

The JSON template format allows authoring new templates without writing C# code. They have the same capabilities as templates generated from C# classes.

To determine the C# control class to create when instantiating a JSON template, the system looks at the inheritance chain of the template. The first template along the chain that is generated from a C# class determines the class to instantiate. If there is no such template in the chain, `InputDevice` is assumed.

## Manual C# Construction Code

# What do those various settings do?

## `name`

Sets the name that the control is referred to by. This is the primary name that the control is looked up with. E.g. `Gamepad.current["leftStick"]` will look for the control called "leftStick" as a child of the given `Gamepad`.

Names are case-insensitive.

Names also become the default `displayName` of the control, if no display name is set explicitly.

`aliases` provide a set of alternate names for controls.

Names have to be unique within the context of the parent control. They are used to form paths (e.g. "leftStick/x").

## `displayName`

Name to display for the control in UIs. If not set, defaults to `name`.

## `resourceName`

## `aliases`

## `template`

The name of the template that is instantiated to represent the control. For example, the "leftStick" control on `Gamepad` uses the "Stick" template which automatically creates a child hiearchy with "left", "right", "up", "down", "x" and "y" child controls underneath the "leftStick" parent control.

## `offset`

Offset (in bytes) of where the control's state starts in memory. Offsets are relative to parents. For example, the following arrangement will offset `stick/y` 2 bytes from where `stick` starts (whose offset is inferred from the `stickX` field the attribute is applied to).

```
    [InputControl(name = "stick", template = "Stick")]
    [InputControl(name = "stick/x", offset = 0, format = "SHRT")]
    [InputControl(name = "stick/y", offset = 2, format = "SHRT")]
    public short stickX;
    public short stickY;
```

## `bit`

Offset in bits from the control's byte offset. This is only relevant for bit-addressed controls (like buttons using a bitfield) and should normally be set to 0.

```
    [InputControl(name = "buttonSouth", template = "Button", bit = (uint)Button.South)]
    [InputControl(name = "buttonWest", template = "Button", bit = (uint)Button.West)]
    [InputControl(name = "buttonNorth", template = "Button", bit = (uint)Button.North)]
    [InputControl(name = "buttonEast", template = "Button", bit = (uint)Button.East)]
    public uint buttons;
```

## `sizeInBits`

Width of the control's state in memory in bits. For example, for a button stored as a single bit this would be 1. For an axis stored as a float, this would be 32.

## `format`

State format code for the memory layout of the control. This is only relevant for controls that read their memory directly which usually is the case only with leaf controls and devices. Controls sitting in the middle of the control tree usually read their state by having child controls read theirs. However, there are exceptions (like `TouchControl`, for example).

The format code is always a FourCC code.

For devices the state format code is important as it has to correspond to the format code of incoming events. Events where the format code does not match are ignored. `////REVIEW: should we abandon this mechanism?`

The following table lists the built-in state formats as well as the controls that support them. Support for arbitray formats can be added by implementing custom controls that understand the formats.

|C# Type|Format Code|Data|Supported by Controls|
|-------|----------|----|--------------------|
|n/a|`"BIT"`|Single bit|`AxisControl` (and controls derived from it, like `ButtonControl`)|
|`byte`|`"BYTE"`|Unsigned 8-bit integer value|`AxisControl` (and controls derived from it, like `ButtonControl`)|
|`sbyte`|`"SBYT"`|Signed 8-bit integer value|`AxisControl` (and controls derived from it, like `ButtonControl`)|
|`short`|`"SHRT"`|Signed 16-bit integer value|`AxisControl` (and controls derived from it, like `ButtonControl`)|
|`ushort`|`"USHT"`|Unsigned 16-bit integer value|`AxisControl` (and controls derived from it, like `ButtonControl`)|
|`int`|`"INT"`|Signed 32-bit integer value|`AxisControl` (and controls derived from it, like `ButtonControl`), `IntegerControl`|
|`uint`|`"UINT"`|Unsigned 32-bit integer value|`AxisControl` (and controls derived from it, like `ButtonControl`)|
|`float`|`"FLT"`|32-bit floating-point value|`AxisControl` (and controls derived from it, like `ButtonControl`)|
|`double`|`"DBL"`|64-bit floating-point value|

## `usages`

# How do I associate a template with a specific type?

# How can I base one template on an existing template?

# How do I modify the settings of an existing template?

# What are "variants"?
