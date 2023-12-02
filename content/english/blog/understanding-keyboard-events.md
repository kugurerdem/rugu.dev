---
title: 'Developer Story: Key Remapping & Keylogging'
draft: false
date: '2023-12-02'
---

In this essay, I describe how I made a Node.js module to listen to keypresses
across the system on Linux machines using X. This experience helped me grasp
how the OS and Window Managers handle keyboard inputs, clarifying the reasons
behind an unexpected behavior I had encountered before, which I also mention in
the essay.

If you're interested in learning more about how keyboard events are handled,
this essay might be of interest to you.

## An Issue About Remapping Keys

As a person who uses VIM for many things I do, seamlessly transitioning between
VIM modes is essential for my workflow. However, the default key for returning
back to normal VIM mode is the Escape key, necessitating me to remove my hands
from the keyboard in order to reach it.

This is exactly why many VIM users, including myself, remap the Escape key to
the Caps Lock key. This adjustment is particularly beneficial for those who are
already accustomed to pressing the Shift key for uppercase characters.

However, I've encountered a minor issue with this setting: Certain applications
seem indifferent to the remappings I've configured. Occasionally, when I press
the Caps Lock key, the operating system interprets it as an Escape key press,
while the specific program I'm using still recognizes it as the original
physical key pressed.

I have encountered this problem both in my Linux computer, and Windows
computer. So the problem itself is OS-agnostic. However I've unintentionally
identified the reason behind this occasional discrepancy while I was working on
a recreational project.

## Linux and Keyboard Events

Recently, a friend asked me if it's possible to create macros using Node.js. I
confidently said, 'Sure, it's probably easy.' After a quick search, I found a
desktop automation library called robotJS and wrote a simple script where
specific keys are pressed regularly by the script.

However, I started wondering if it was possible to trigger those keypresses
after a user presses a certain key. To achieve this, I needed to listen to
keypress events on a system-wide level. I searched for suitable Node.js
libraries for this task on Linux, but I couldn't find one that worked
seamlessly.

There were libraries like ``iohook``, but they seemed to lack support for
listening to Linux keyboard events in the latest versions of Node.js. Some
solutions only focused on capturing keyboard events within the current window
associated with the process.

I stumbled upon a library called ``xev-emitter`` but it didn't provide what I
needed as it mainly dealt with listening to xevents of a specific X client.

After some contemplation, I decided to create my own Node.js module using
``xinput`` underneath, just for the sake of it and out of curiosity. ``xinput``
is a Unix tool that allows listening to keyboard events and provides an
interface to monitor events from connected keyboards.

For instance, running the command ``xinput`` gives me a list of available input
devices connected to my PC:

```
⎡ Virtual core pointer                    	id=2	[master pointer  (3)]
⎜   ↳ Virtual core XTEST pointer              	id=4	[slave  pointer  (2)]
⎜   ↳ 2.4G Mouse                              	id=10	[slave  pointer  (2)]
⎜   ↳ 2.4G Mouse Consumer Control             	id=11	[slave  pointer  (2)]
⎜   ↳ Synaptics TM3336-004                    	id=14	[slave  pointer  (2)]
⎣ Virtual core keyboard                   	id=3	[master keyboard (2)]
    ↳ Virtual core XTEST keyboard             	id=5	[slave  keyboard (3)]
    ↳ Power Button                            	id=6	[slave  keyboard (3)]
    ↳ Video Bus                               	id=7	[slave  keyboard (3)]
    ↳ Power Button                            	id=8	[slave  keyboard (3)]
    ↳ 2.4G Mouse                              	id=9	[slave  keyboard (3)]
    ↳ 2.4G Mouse System Control               	id=12	[slave  keyboard (3)]
    ↳ Ideapad extra buttons                   	id=13	[slave  keyboard (3)]
    ↳ AT Translated Set 2 keyboard            	id=15	[slave  keyboard (3)]
    ↳ 2.4G Mouse Consumer Control             	id=16	[slave  keyboard (3)]
```

`xinput` also has a command type that lets you listen to a specific input
device. For instance, if I use `xinput test 15`, it listens to the device with
the specified ID 15. When I run the command `xinput test 15` and then press the
'a,' 's,' and 'd' keys on my keyboard, the output I get is as follows:

```
key press   38
key release 38
key press   39
key release 39
key press   40
key release 40
```

Now, with these two commands, we can iterate through all the input devices
related to keyboards and listen to them. We can create a script that first
lists the available input devices, filters them, and then runs the command
`xinput test` for each of them.

However, there is still a minor problem. How do we understand which key is
pressed just by looking at the numbers that xinput gave us? How can we know
that 38 stands for the key 'a'?

The numbers provided by xinput are known as X Key Codes. These codes represent
the physical keys pressed on the X layer. They are essentially similar to Linux
Input Event Codes, which the Linux Operating System generates to represent the
physical keys pressed. For reasons I'm not aware of, X Key Codes are
incremented by 8 compared to Linux keycodes (see:
https://wiki.archlinux.org/title/Keyboard_input#Identifying_keycodes_in_console).

Now, the challenge lies in making sense of each X Key Code. We need a mapping
between the X Key Codes and their corresponding keys. However, what they
correspond to can be configured by users. In fact, I've configured this using
the command `setxkbmap -option "caps:swapescape"`. So, although pressing the
same keys on a physical keyboard will result in the same key codes, the
interpretation by your operating system or window management server can be
configured. Therefore, the correspondence of each keycode with what you've
pressed might vary from one environment to another. In the X protocol, you can
view the mapping between X Key Codes and X KeySyms by running the command
`xmodmap -pke`.

This is essentially what I did in the Node.js module I created to listen to
keyboard events using X:

1) Obtain the list of available input devices by running `xinput` as a
subprocess.

2) Filter the devices; you don't need all of them, just their IDs.

3) For each ID, run the command `xinput test id`.

4) Use the result of `xmodmap -pke` to understand the semantic meaning assigned
to each physical keypress, known as a KeySym.

If you're curious, you can check out the module I created:
https://github.com/kugurerdem/node-xinput-event

## Probably a Better Approach

After implementing the module mentioned earlier, I discovered the existence of
a Linux utility called showkey that allows listening to pressed keys.

It's also possible to create a similar script using showkey under the hood. In
fact, this might have been a better approach compared to what I did above
because it operates on a more fundamental level than X.

Similar to how we mapped between X Key Codes and their corresponding X Key
Codes, we could create a mapping between Linux Event Codes and their meanings
by examining the Linux source code:
https://github.com/torvalds/linux/blob/master/include/uapi/linux/input-event-codes.h

Moreover, using scripts like xinput as subprocesses under our script might not
be the optimal approach for implementing an EventEmitter library to listen to
system-wide keypresses. The conventional way is likely to interact with the X
server using an X client. Unfortunately, I couldn't build the x11 library on my
computer and chose not to delve into it much.

## Conclusion

The series of experimental processes I went through greatly enhanced my
understanding of what happens behind the scenes when I simply press a key on my
physical keyboard.

Returning to the initial scenario I described, when you're developing a
program, you can act upon the values of key syms or key codes. While the key
codes might remain the same, the key syms—the meanings attached to those key
codes—can differ. It appears that some applications focus on key codes,
disregarding your local options.

Essentially, at the kernel layer, there are only keycodes. Your operating
system assigns meaning to these keycodes through specific configuration files,
which you can either directly modify or use another program for modification
(in this case, the X Window Management server). Since it's generally more
convenient to alter settings in the window management layer, most people
configure their preferences through utilities provided by their window manager,
and the window manager handles the interaction with the OS.

This serves as a compelling example of how casually experimenting with things
can significantly contribute to one's understanding of the core concepts they
are dealing with.
