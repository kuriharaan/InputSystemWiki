# What do they do?

They describe the arrangement of control hierarchies and the correlation of those controls to state stored in raw memory.

Example: the "Stick" template creates a hierarchy with a `StickControl` at the top, six `AxisControls` underneath ('left', 'right', 'top', 'down', 'x', and 'y'), and a default state of two floats with 'left', 'right' and 'x' reading from the first float and 'top', 'down', and 'y' reading from the second float.

When a device is plugged into the system, it looks for a matching template that can be used with the device. This is done by looking at the `InputDeviceDescription` provided for the device by the input runtime (`IInputRuntime`) and comparing it with the descriptions associated with the various templates that have been registered. If a matching template is found, it is "instantiated", i.e. turned into an actual `InputDevice` instance by `InputControlSetup`.

# How do I create them?

In one of three ways:

1. C# reflection
2. JSON
3. Manual C# construction code

All three ways produce the same format: an instance of `InputTemplate`. These are created on the fly and on demand during control creation and then thrown away.

Any `InputTemplate`, regardless of which way it was created, can be turned back into JSON. This is used for sending templates over the wire (e.g. when debugging player input in the editor).

## C# Reflection

When registering a C# class as a template with `InputSystem.RegisterTemplate<T>`, reflection is used to automatically discovered the control setup of the given class. Two attributes -- `InputTemplateAttribute` and `InputControlAttribute` -- can be used to guide and supplement this process.

## JSON

## Manual C# Construction Code

# What do those various settings do?
