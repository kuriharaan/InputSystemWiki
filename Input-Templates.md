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

When registering a C# class as a template with `InputSystem.RegisterTemplate<T>`, reflection is used to automatically discover the control setup of the given class. Two attributes -- `InputTemplateAttribute` and `InputControlAttribute` -- can be used to guide and supplement this process.

In this process the code goes looking through all properties and fields of the class (directly declared *and* inherited ones) and for each one that classifies as an `InputControl`, add information about the control to the template. Every field or property that either has a type derived from `InputControl` or has an `InputControlAttribute` applied to it classifies.

Defaults for the control added to the template are taken from the property or field. All defaults can be overridden with explicit settings in `InputControlAttribute`.

For properties, the name defaults to the name of the property and the type of control defaults to the `InputControl` derived type of the property. No other defaults are inferred from properties.

For fields, the name defaults to the name of the field. ...

## JSON

## Manual C# Construction Code

# What do those various settings do?

# How can I base one template on an existing template?

# How do I modify the settings of an existing template?