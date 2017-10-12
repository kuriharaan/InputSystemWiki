# ... check if the space key has been pressed this frame?

```C#
    Keyboard.current.space.wasPressedThisFrame
```

Same kinda deal works for other devices.

```C#
    Gamepad.current.aButton.wasPressedThisFrame
```

# ... find all connected gamepads?

Multiple ways.

You can ask `Gamepad`.

```C#
    var allGamepads = Gamepad.all;
```

Or you can go through the list of InputDevices directly.

```C#
    InputSystem.devices.Select(x => x is Gamepad);
```

Or you can use a path to get all devices using the "gamepad" template (or any template based on the gamepad template):

```C#
    InputSystem.GetControls("/<gamepad>");
```

Finally, you can match devices by name (which is a bad idea, though, because there is no guarantee that a gamepad device has "gamepad" in its name):

```C#
    InputSystem.GetControls("/gamepad*");
```

# ... know when a new device has been plugged in?

```C#
    InputSystem.onDeviceChange +=
        (device, change) =>
        {
            if (change == InputDeviceChange.Added)
                /* New Device */;
            else if (change == InputDeviceChange.Disconnected)
                /* Device got unplugged */;
            else if (change == InputDeviceChange.Connected)
                /* Plugged back in */;
            else if (change == InputDeviceChange.Removed)
                /* Remove from input system entirely; by default, devices stay in the system once discovered */;
        }
```

# ... create a simple fire-type action?

One way to do so is directly in code:

```C#
    // Create action that binds to the primary action control on all devices.
    var action = new InputAction(binding: "*/{primaryAction}");

    // Have it run your code when action is triggered.
    action.performed += (action, control) => Fire();

    // Start listening for control changes.
    action.Enable();
```

The second way is to simply have a serialized field of type InputAction in your MonoBehaviour like this:

```C#

public class MyControllerComponent : MonoBehaviour
{
    public InputAction fireAction;
    public InputAction walkAction;
}

```

In the editor, you will be presented with a nice inspector that allows you to add bindings to the actions and choose where the bindings go without having to fiddle around with path strings.

Note that you still need to enable the action in code and hook up your response. You can do so in the Awake() method of your component.

```C#

    void Awake()
    {
        fireAction.performed => Fire;
        walkAction.performed => Walk;
    }

    void OnEnable()
    {
        fireAction.Enable();
        walkAction.Enable();
    }

    void OnDisable()
    {
        fireAction.Disable();
        walkAction.Disable();
    }

    void Fire(InputAction action, InputControl control)
    {
        //...
    }

    void Walk(InputAction action, InputControl control)
    {
        //...
    }

```

    ////TODO: the second way I'm still working on ATM (the editors and stuff)

    ////TODO: I'm also working on a way to nicely package up actions in action sets

# ... require a button to be held for 0.4 seconds before triggering an action?

Put a HoldModifier on the action. In code, this works like so:

```C#

    var action = new InputAction(binding: "*/{PrimaryAction}", modifiers: "hold(duration=0.4)");

```

To display UI feedback when the button starts being held, use the `started` callback.

```C#

    action.started += (a, c) => ShowGunChargeUI();
    action.performed += (a, c) => FinishGunChargingAndHideChargeUI();

```

    ////TODO: still working on the modifier mechanics

# ... separate the actions in my game from user-overridable bindings?

Put your actions in one JSON file and put your default bindings in another JSON file. At runtime, load the actions and then load either the default bindings or a customized version from the user's profile.

```C#

    ////TODO: still fleshing out the APIs for this

```

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

////TODO: need a way to give devices an opportunity to feed events; ATM you have to make that happen yourself and events will only go in the next update this way

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

# ... have my own template used when the native backend discovers a specific device?

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
                "processors" : "deadzone(min=0.125,max=0.925)"
            },
            {
                "name" : "rightStick",
                "processors" : "deadzone(min=0.125,max=0.925)"
            }
        ]
    }
```

You can do the same in your C# state structs.

```C#
    public struct MyDeviceState
    {
        [InputControl(processors = "deadzone(min=0.125,max=0.925)"]
        public StickControl leftStick;
        [InputControl(processors = "deadzone(min=0.125,max=0.925)"]
        public StickControl rightStick;
    }
```

In fact, the gamepad template already adds a deadzone processor which will take its min and max values from `InputConfiguration.DefaultDeadzoneMin` and `InputConfiguration.DefaultDeadzoneMax`.

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

# ... record events flowing through the system?

```C#

    var trace = new InputEventTrace(); // Can also give device ID to only
                                       // trace events for a specific device.
    
    trace.Enable();

    //... run stuff

    var current = new InputEventPtr();
    while (trace.GetNextEvent(ref current))
    {
        Debug.Log("Got some event: " + current);
    }

    // Also supports IEnumerable.
    foreach (var eventPtr in trace)
        Debug.Log("Got some event: " + eventPtr);

    // Trace consumes unmanaged resources. Make sure to dispose.
    trace.Dispose();

```

# ... see events as they are processed?

```C#

    InputSystem.onEvent +=
        eventPtr =>
        {
            // Can handle events yourself, for example, and then stop them
            // from further processing by marking them as handled.
            eventPtr.handled = true;
        };

```

# ... see what devices I have and what state they are in?

Go to `Windows >> Input Debugger`.

    ////TODO: working on having device setups from players mirrored 1:1 into the running input system in the editor (including having their input available in the editor)