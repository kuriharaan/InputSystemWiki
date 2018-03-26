> Input controls are at the heart of the input system. However, if you are only interested in high-level input actions, it is not necessary to have a deeper understanding of what they do and how they work. If, however, you want to manage devices yourself and/or write code that directly works against their controls, the information here may prove useful.

# What do they do?

An `InputControl` is essentially a named value provided by the system. Each such value has a specific type that is associated with the control. For example, the "leftStick" control of a `Gamepad` provides a `Vector2` value.

Input controls form hierarchies which can decompose values into more fine-grained parts. For example, "leftStick/x" is a child control of "leftStick" which provides access to just the X component of the vector.

# How are they created?

# How are their values computed?
