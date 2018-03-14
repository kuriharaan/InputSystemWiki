# What do they do?

They describe the arrangement of control hierarchies and the correlation of those controls to state stored in raw memory.

Example: the "Stick" template creates a hierarchy with a StickControl at the top, six AxisControls underneath ('left', 'right', 'top', 'down', 'x', and 'y'), and a default state of two floats with 'left', 'right' and 'x' reading from the first float and 'top', 'down', and 'y' reading from the second float.

# How do I create them?

In one of three ways:

1. C# reflection
2. JSON
3. Manual C# construction code

All three ways produce the same format: an instance of InputTemplate. These are created on the fly and on demand during control creation and then thrown away.

Any InputTemplate, regardless of which way it was created, can be turned back into JSON. This is used for sending templates over the wire (e.g. when debugging player input in the editor).

## C# Reflection

## JSON

## Manual C# Construction Code

# How do they work?
