This page is for collecting notes on specific technical issues.

# Action Polling vs Callbacks

```C#
    myAction.wasPerformed

    // vs.

    myAction.performed += ctx => ...;
```

Polling has two big advantages:

* Storing up all the state in actions relevant to when a control was triggered (when was it triggered? which control triggered it?) gets complicated quickly and is prone to storing state that is never needed. The callback-based approach can lazily fetch information from the callback context when needed and can thus cheaply add all kinds of context-dependent information.
* Actions are able to observe and perform based on every single state change in the system -- even if those state changes fall into the same frame. A polling-based API will only be able to observe the very latest state change.

However, polling has one huge drawback:

* It gives a natural sync point to allow the system to work asynchronously. I.e. if the user has to do `.wasPressed` we know that this is exactly the point by which we have to have the current input system update C# job completing. With callbacks, the question of when to sync becomes much harder. In the synchronous version, callbacks fire immediately. Moving it to async will unavoidably delay calls but then the questions becomes "to when?"

# Delta Controls

Delta controls (like e.g. mouse motion deltas) turn out to be unexpectedly tricky beasts. The two unusual aspects about them is that a) they want to *accumulate* (if the app gets sent two mouse deltas from the system and they end up in the same input update, you want them to add up and not just see only the second delta) and b) they want to reset back to zero between input updates. Since mice are usually sampled at higher frequencies than GFX refreshes and since deltas are commonly used to drive sensitive things such as mouse look, handling this correctly actually matters.

My first thought was: let's not complicate the state system and deal with the problem in the code that generates mouse state events (i.e. the platform layer). However, this turns out to be surprisingly tricky to implement correctly. The problem is that knowing when to accumulate and when to reset is intricately tied to which mouse state events go into which input updates -- something that is hard for the OS layer to know. Simply accumulating during a frame and resetting between frames will lead to correct behavior for dynamic updates (which span an entire frame) but will not lead to correct behavior for other update types.

This led me to conclude that these controls are best supported directly in the state and control system -- which is unfortunate as accumulation and resetting not only make state updates more complicated but also complicate state change detection which actions rely on.