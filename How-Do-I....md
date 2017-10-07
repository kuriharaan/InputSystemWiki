# ... wait for any button to be pressed on any device?

One way is to use actions.

    var myAction = new InputAction(binding: "/*/<button>");
    myAction.onPerformed += (action, control) => Debug.Log("Button pressed!";
    myAction.Enable();

However, this is dirt inefficient. The amount of processing an action has to do is directly correlated with the amount of controls it is targeting. Targeting every single button of every single device will yield a ton of controls and result in high processing overhead. The keyboard alone will contribute a ton of buttons each of which will have to be processed individually.

A more efficient way is to just listen for any activity on any device and when there was activity, find out whether it came from a button.

    ... still being worked on; can already listen on whole devices but you won't know what control caused the state change.

# ... check if the space key has been pressed this frame?

    Keyboard.current.space.wasPressedThisFrame

# ... find all connected gamepads?

Multiple ways.

Can use a path to get all devices using the "gamepad" template:

    InputSystem.GetControls("/<gamepad>");

Or match devices by name:

    InputSystem.GetControls("/gamepad*);

Or you can just go through the list of InputDevices directly.

    InputSystem.devices.All(x => x is Gamepad);

# ... create a device?

    InputSystem.AddDevice("Gamepad");

The given string is a template name.

# ... create my own custom devices?

Two possible ways. If you are okay with using one of the existing C# InputDevice classes in code to interface with your device, you can just build on an existing template using JSON.

    {
        "name" : "MyDevice",
        "extend" : "Gamepad", // Or some other thing
        "controls" : [
             // ... customize control setup
        ]
    }

You simply register your template with the system and then instantiate it.

    InputSystem.RegisterTemplate(myDeviceJson);
    var device = InputSystem.AddDevice("MyDevice");

Alternatively, you can create your own InputDevice class and state layouts in C#.

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

To create an instance of your device, register it as a template and then instantiate it

    InputSystem.RegisterTemplate("MyDevice", typeof(MyDevice));
    InputSystem.AddDevice("MyDevice");

# ... choose a different state layout for my gamepad?

Extend the "Gamepad" template and customize its controls.

     {
        "name" : "MyGamepad",
        "extend" : "Gamepad",
        "controls" : [
             // Say you want to store the sticks as shorts instead of floats.
             {
                 "name" : "leftStick/x",
                 "format" : "SHRT",
                 "offset" : 4
             },
             {
                 "name" : "leftStick/y",
                 "format" : "SHRT",
                 "offset" : 6
             },
             {
                 "name" : "rightStick/y",
                 "format" : "SHRT",
                 "offset" : 8
             },
             {
                 "name" : "rightStick/y",
                 "format" : "SHRT",
                 "offset" : 10
             }
             // You can also freely add new controls or change the templates on existing
             // controls and so on and on.
        ]
    }

# ... have my own template used when the native backend discovers a specific type?

Simply describe the device in the template.

     {
        "name" : "MyGamepad",
        "extend" : "Gamepad",
        "device" : {
            // All strings in here are regexs and case-insensitive.
            "product" : "MyController",
            "manufacturer" : "MyCompany"
        }
     }