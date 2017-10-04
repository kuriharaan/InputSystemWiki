//move to google docs

This is my own personal evaluation of what went wrong with the "old new input system."

>I will only speak to my own mistakes (or ones to which I contributed significantly) as well as to mistakes I see as systemic.

## Lack of commitment in early phase

At multiple points I wandered off from the project. This happened right at the beginning where only sporadic effort was made by me as well as when the project was already more thoroughly underway (I went and worked on the testing/faking stuff for a long time).

There was an entire year where nothing happened.

## Getting sidetracked

At one point the concern was raised that "Unity should not get yet another event system". An effort for "unified event handling" was brought to life and I stupidly shouldered the burden of making that happen. Even after multiple meetings it was not clear what the exact objective actually was and what the work to be done was.

This stalled work on critical parts of the input system and ended up taking all momentum out of the project as I was stumbling around and Rune was left with major uncertainty cast over significant portions of the system. I didn't realize how poisonous this was to the project until I found myself jumping on the first opportunity to work on something else entirely. Only with some distance did I realize that I had never made sure I was taking on a well-defined task.

## Lack of inter-team coordination

Communicating with all relevant stakeholders was left solely to the input team. This included talking to a diverse set of teams (labs, XR, platforms, etc) across separate office locations.

While I think we made a decent effort, it clearly fell short and as the person responsible for low-level work, I specifically see my communication with the various platform stake holders lacking.

I think that this can be helped greatly by not leaving it to just the devs to figure this out all by themselves. Code monkeys don't necessarily make the very best communicators and people coordinators :)

## A prototype that was not thrown away

The C# only prototype we did early on proved extremely valuable. It formed a feedback bridge to other teams and allowed us to try out and reshape ideas.

However, there wasn't the step *after* the prototype where you take a step back, take all that you have learnt, ask some hard questions, and progress to writing a production quality system.

This leads me to the next point.

## Incorrect and/or harmful assumptions

Personally, based on the prototype I came away with several assumptions that turned out to be harmful.

One such assumption was that in the HLAPI we need all functionality to be exposed in neat, easy, "Unity style" fashion. This, amongst a verity of things, led to events being classes and having an object-oriented, easy accessibility to them. Out of this flowed a lot of complexity (like the routing mechanics) and more flawed assumptions.

## Putting the pressure on prematurely

When the XR team expressed rapidly evolving needs around input, I made the call that it wouldn't be good to build those on the old input system and that we should use the opportunity to push work on the new input system along and employ the XR team as an in-house user of sorts.

In retrospect, I think this probably was a mistake. One problem is that the XR team in particular has very specific deadlines tied to big platform contracts. Committing to them means committing to their deadlines.

While a lot of good came out of this and the close collaboration with Scott Flynn's team in Bellevue, I think it led to some questions not being asked for the sake of "hey, we need X done by Y".

It also become a problem after being combined with the following...

## Spearheading a parallel effort

Early on it was decided that the C# part of the system would be open source and hosted on Github. Unfortunately, commitment to the XR team and their contracts meant that we had to ship prebuilt binaries with a Unity installation.

At the same time we tried to solve the problem, the new package manager was just about to become a viable delivery platform so we decided to go with that. However, at the time no other package put similar requirements on packman so we ended up prematurely committing to it and spending time on things not directly focused on the input system.

## Compartmentalized ownership

The system got split in two with a low-level native API and a high-level managed API. The division makes good sense.

However, over time it also led to compartmentalized ownership in the team. While this may seem natural and not like a problem, for me personally it meant that I more and more confined myself to my own little sandbox (which was mostly the native part). I think a system that is not eventually owned top to bottom by at least "someone" is always going to show the fact in some ways.
