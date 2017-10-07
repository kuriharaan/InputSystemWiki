# ... Find All Connected Gamepads?

Multiple ways.

Can use a path to get all devices using the "gamepad" template:

    InputSystem.GetControls("/<gamepad>";

Or match devices by name:

    InputSystem.GetControls("/gamepad*);

Or you can just go through the list of InputDevices directly.

    InputSystem.devices.All(x => x is Gamepad);

# ... Create a Device?

    InputSystem.AddDevice("Gamepad");

The given string is a template name.

# ... Create My Own Custom Device?

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