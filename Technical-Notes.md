This page is for collecting notes on specific technical issues.

# Delta Controls

Delta controls (like e.g. mouse motion deltas) turn out to be unexpectedly tricky beasts. The two unusual aspects about them is that a) they want to *accumulate* (if the app gets sent two mouse deltas from the system and they end up in the same input update, you want them to add up and not just see only the second delta) and b) they want to reset back to zero between input updates. Since mice are usually sampled at higher frequencies than GFX refreshes and since deltas are commonly used to drive sensitive things such as mouse look, handling this correctly actually matters.

My first thought was: let's not complicate the state system and deal with the problem in the code that generates mouse state events (i.e. the platform layer). However, this turns out to be surprisingly tricky to implement correctly. The problem is that knowing when to accumulate and when to reset is intricately tied to which mouse state events go into which input updates -- something that is hard for the OS layer to know. Simply accumulating during a frame and resetting between frames will lead to correct behavior for dynamic updates (which span an entire frame) but will not lead to correct behavior for other update types.

This led me to conclude that these controls are best supported directly in the state and control system -- which is unfortunate as accumulation and resetting not only make state updates more complicated but also complicate state change detection which actions rely on.