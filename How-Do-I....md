# ... wait for any button to be pressed on any device?

One way is to use actions.

```C#
    var myAction = new InputAction(binding: "/*/<button>");
    myAction.onPerformed += (action, control) => Debug.Log($"Button {control.name} pressed!");
    myAction.Enable();
```

However, this is dirt inefficient. The amount of processing an action has to do is directly correlated with the amount of controls it is targeting. Targeting every single button of every single device will yield a ton of controls and result in high processing overhead. The keyboard alone will contribute a ton of buttons each of which will have to be processed individually.

A more efficient way is to just listen for any activity on any device and when there was activity, find out whether it came from a button.

```C#
    ... still being worked on; can already listen on whole devices but you won't know what control caused the state change.
```

# ... check if the space key has been pressed this frame?

```C#
    Keyboard.current.space.wasPressedThisFrame
```

# ... find all connected gamepads?

Multiple ways.

Can use a path to get all devices using the "gamepad" template (or any template based on the gamepad template):

```C#
    InputSystem.GetControls("/<gamepad>");
```

Or match devices by name (which is a bad idea, though, because there is no guarantee that a gamepad device has "gamepad" in its name):

```C#
    InputSystem.GetControls("/gamepad*");
```

Or you can just go through the list of InputDevices directly.

```C#
    InputSystem.devices.Select(x => x is Gamepad);
```

# ... create a device?

```C#
    InputSystem.AddDevice("Gamepad");
```

The given string is a template name.

# ... create my own custom devices?

Two possible ways. If you are okay with using one of the existing C# InputDevice classes in code to interface with your device, you can just build on an existing template using JSON.

```
    {
        "name" : "MyDevice",
        "extend" : "Gamepad", // Or some other thing
        "controls" : [
             // ... customize control setup
        ]
    }
```

You simply register your template with the system and then instantiate it.

```C#
    InputSystem.RegisterTemplate(myDeviceJson);
    var device = InputSystem.AddDevice("MyDevice");
```

Alternatively, you can create your own InputDevice class and state layouts in C#.

```C#
    public struct MyDeviceState : IInputStateTypeInfo
    {
        // FourCC type codes are used identify the memory layouts of state blocks.
        public static FourCC kFormat = new FourCC('M', 'D', 'E', 'V');

        [InputControl(name = "firstButton", template = "Button", bit = 0)]
        [InputControl(name = "secondButton", template = "Button", bit = 1)]
        public int buttons;
        [InputControl(template = "Analog", parameters="clamp=true,clampMin=0,clampMax=1")]
        public float axis;

        public FourcCC GetFormat()
        {
             return kFormat;
        }
    }

    [InputState(typeof(MyDeviceState)]
    public class MyDevice : InputDevice
    {
        public ButtonControl firstButton { get; private set; }
        public ButtonControl secondButton { get; private set; }
        public AxisControl axis { get; private set; }

        protected override void FinishSetup(InputControlSetup setup)
        {
             firstButton = setup.GetControl<ButtonControl>(this, "firstButton");
             secondButton = setup.GetControl<ButtonControl>(this, "secondButton");
             axis = setup.GetControl<AxisControl>(this, "axis");
             base.FinishSetup(setup);
        }
    }
```

To create an instance of your device, register it as a template and then instantiate it

```C#
    InputSystem.RegisterTemplate("MyDevice", typeof(MyDevice));
    InputSystem.AddDevice("MyDevice");
```

# ... choose a different state layout for my gamepad?

Extend the "Gamepad" template and customize its controls.

```
     {
        "name" : "MyGamepad",
        "extend" : "Gamepad",
        // If you customize the state layout, you also need to send state data
        // in the right format. The default "GPAD" format won't fit. So, you
        // have to specify a custom format identifier (a FourCC code) and send
        // state events tagged with this format code.
        "format" : "MYGP",
        "controls" : [
             // Say you want to store the sticks as signed shorts instead of floats
             // and Y happens to go the opposite way on your gamepad.
             {
                 "name" : "leftStick/x",
                 "format" : "SHRT",
                 "offset" : 4,
                 "parameters" : "normalize,normalizeMin=-0.5,normalizeMax=0.5"
             },
             {
                 "name" : "leftStick/y",
                 "format" : "SHRT",
                 "offset" : 6,
                 "parameters" = "invert,normalize,normalizeMin=-0.5,normalizeMax=0.5"
             },
             {
                 "name" : "rightStick/x",
                 "format" : "SHRT",
                 "offset" : 8,
                 "parameters" : "normalize,normalizeMin=-0.5,normalizeMax=0.5"
             },
             {
                 "name" : "rightStick/y",
                 "format" : "SHRT",
                 "offset" : 10,
                 "parameters" = "invert,normalize,normalizeMin=-0.5,normalizeMax=0.5"
             }
             // You can also freely add new controls or change the templates on existing
             // controls and so on and on.
        ]
    }
```

The same principle applies if on your device some buttons are swapped, for example. Simply remap their offsets.

# ... have my own template used when the native backend discovers a specific type?

Simply describe the device in the template.

```
     {
        "name" : "MyGamepad",
        "extend" : "Gamepad",
        "device" : {
            // All strings in here are regexs and case-insensitive.
            "product" : "MyController",
            "manufacturer" : "MyCompany"
        }
     }
```

Note that you don't have to restart Unity in order for changes in your template to take affect on native devices. On every domain reload, changes will automatically be applied so you can just keep refining a template and your device will get recreated with the most up-to-date version.

# ... add deadzoning to my gamepad sticks?

Simply put a deadzone processor on the sticks.

```JSON
     {
        "name" : "MyGamepad",
        "extend" : "Gamepad",
        "controls" : [
            {
                "name" : "leftStick",
                "processors" : "deadzone(0.125,0.925)"
            },
            {
                "name" : "rightStick",
                "processors" : "deadzone(0.125,0.925)"
            }
        ]
    }
```

You can do the same in your C# state structs.

```C#
    public struct MyDeviceState
    {
        [InputControl(processors = "deadzone(0.125,0.925)"]
        public StickControl leftStick;
        [InputControl(processors = "deadzone(0.125,0.925)"]
        public StickControl rightStick;
    }
```

I'm still working on a way to do add a deadzone processor conveniently on the fly to an existing gamepad instance.

# ... give my head tracking an extra update before rendering?

First enable before render updates on your device.

```JSON
    {
        "name" : "MyHMD",
        "extend" : "HMD",
        "beforeRender" : "Update"
    }
```

And then make sure you put extra StateEvents for your HMD on the queue right in time before rendering. Also, if your HMD is a combination of non-tracking and tracking controls, you can update just the tracking, if you want to, by sending a DeltaEvent instead of a full StateEvent.

# ... simulate HMD movement from mouse and keyboard?

    I'm working on a callback that allows state to be updated from state and see the change in the same frame
