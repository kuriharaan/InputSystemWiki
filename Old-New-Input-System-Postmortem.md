//move to google docs

This is my own personal evaluation of what went wrong with the "old new input system."

>I will only speak to my own mistakes (or ones to which I contributed significantly) as well as to mistakes I see as systemic.

## Lack of commitment in early phase

At multiple points I wandered off from the project. This happened right at the beginning where only sporadic effort was made by me as well as when the project was already more thoroughly underway (I went and worked on the testing/faking stuff for a long time).

There was an entire year where nothing happened.

## Getting sidetracked

At one point the concern was raised that "Unity should not get yet another event system". An effort for "unified event handling" was brought to live and I stupidly shouldered the burden of making that happen. Even after multiple meeting it was not clear what the objective actually was and what the work to be done was.

This stalled work on critical parts of the input system and ended up taking all momentum out of the project as I was stumbling around and Rune was uncertainty cast over major portions of the system. I didn't realize how poisonous this was to the project until I found myself jumping on the first opportunity to work on something else entirely. Only with some distance did I realize that I had never made sure I was taking on a well-defined task.

## Lack of inter-team coordination

Communicating with all relevant stakeholders was left solely to the input team. This included talking to a diverse set of teams (labs, XR, platforms, etc) across separate office locations.

While I think we made a decent effort, it clearly fell short and as the person responsible for low-level work, I see my communication with the various platform stake holders clearly lacking.

I think that this can be helped greatly by not leaving it to just the devs to figure this out all by themselves. Code monkeys don't necessarily make the very best communicators and people coordinators :)

## A prototype that was not thrown away

The C# only prototype we did early on proved extremely valuable. It formed a feedback bridge to other teams and allowed us to try out and reshape ideas.

However, there wasn't the step *after* the prototype where you take a step bake, take all that you have learnt, ask some hard questions, and progress to writing a production quality system.

## Compartmentalized ownership

