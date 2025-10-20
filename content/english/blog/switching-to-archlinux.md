---
title: 'My Experience on Switching to Arch Linux'
vault: true
draft: false
date: '2023-01-29'
updated: '2024-01-24'
tags:
    - linux
    - technology
    - productivity
---

About a year ago, I was invited to a pair coding session at the startup where I
was working. The developers were receiving consultancy from a strange person. At
one point, he took control of the screen sharing while reviewing some of the
pull requests that had been made to our codebase. When he shared his screen,
there was nothing but a black screen. Suddenly, a terminal screen appeared with
his keystrokes and he quickly began examining the codebase, providing feedback
on people's code at a speed I had never seen before. He was able to jump between
different files in an instant, examining the diffs that were made in different
git commits.

After seeing what I saw, I thought to myself, ‘If I were able to develop and
refactor code at that speed, I would save a lot of time, I could have spent
more time thinking about the actual stuff with as little friction as possible.
I want to have this power.’ So I have looked at [his
CV](https://gwn.wtf/resume.html) and realized that he was knowledgeable on
topics that many developers, including myself, struggle with. Influenced by
this, I started asking him (he was our consultant, after all) as many questions
as I could and focused on the resources he suggested and the technologies he
used.

The inspiration from this hacker guy, combined with the very precious help from
a friend of mine who had a strong interest in operating systems (he uses Arch in
Qubes OS btw), being libre, and having control over his computer, led me to
switch to Arch Linux. I have also benefited a lot from discussions on hackernews
and from youtubers like Luke Smith which I also heard from the Qubes OS friend
of mine.

In this post, I will first provide a brief overview of Linux and Arch Linux in
particular. Then, I will discuss my reflections on the past few months,
including the downsides and upsides of switching to Arch Linux. Finally, I will
explain the programs that I currently use in my workflow.

## What are my reasons for using Linux?

Unlike MacOS or Windows, Linux is a free and open-source operating system.
Perhaps you are already know that a vast majority of servers actually run on
Linux. Android, which is the operating system most phones use, is a specific
variant of the Linux Operating System. By these means, many developers already
seem to acknowledge the importance of learning Linux for practical reasons.
However, the controversy usually arises when it comes to using Linux on a
personal computer.

For an average computer user, it typically doesn't matter which operating
system they use as long as it doesn't interfere with their daily tasks.

However, in my case, as my views have become more nuanced, switching to Linux
has started to be more appealing.

Here are some of the reasons why I prefer using Linux over MacOS or Windows:

- Both Windows and MacOS forces you their ecosystem by their updates. With each
  update, its more likely that your Desktop Environment is cluttered by a new
  application which Microsoft added and most likely you will not even use. Most
  of the prebuilt stuff that are coming with Windows, I do not use at all. I
  think the same argument also holds for MacOS, as using one of their apps
  usually forces you to use other Apple apps and you to stick with their
  environment.

- Since both MacOS and Windows are closed-source, we don't know for sure what
  they do under the scenes. Windows, for example had a builtin keylogger. If
  you are curious about this, please type "Windows builtin keylogger" to your
  favorite search engine. You will encounter many entries explaining how to
  disable builtin keylogging. Although I do not have sufficient reasons to
  claim that the Apple is doing the same thing, in practice, there is nothing
  preventing these Close Sourced applications to do things like that besides
  legal issues.

- You need to pay some money in order to use both MacOS and Windows, whereas
  Linux is essentially free. You can set up Linux on a computer with no cost at
  all except some finite amount of time you will put in to learn things.

- Linux is highly customizable and allows users to modify and tailor their
  environment to meet their specific needs. In contrast, Windows and Mac OS are
  more limited in terms of customization. This partly makes Linux a better
  choice for users who aim to be a power user, a user who wants to have control
  over their operating system and want to modify or customize it to meet their
  specific needs.

- Linux offers a variety of versions that cater to the different needs of
  various users, which is a significant advantage. These versions, known as
  Linux distros, are essentially Linux systems with additional packages
  specifically designed for certain users. I have not come across a similar
  phenomenon in either MacOS or Windows. Linux provides more options than any
  other proprietary operating system can offer.

- Using a product often means more than just using it; it also means becoming
  part of a group. Using Linux involves you in a community populated by hacker
  minded people. And whether you indend it or not being part of a group
  influences your habits. By immersing yourself in an environment filled with
  more experienced individuals, you become more exposed to their knowledge and
  ideas.

I understand that similar points could be made for the sake of Windows or
MacOS. Examples include Linux not being able to run some proprietary software
that these operating systems can, or Linux not being as convenient because you
often have to figure out most things yourself. I get that. However, all things
considered, the value that Linux provides to me exceeds the values that those
proprietary OSes provide.

## What is unique about Arch Linux?

Essentially, a Linux distro is a version of the Linux operating system that
comes packaged with additional software and tools.

Oversimplicated, but an intuitive formula can be given as:

``DISTRIBUTION = KERNEL + SOME ADDITIONAL DEPENDENCIES``

Theoretically, there's nothing stopping you from doing in one Linux
distribution what you can do in another simply by altering the 'SOME ADDITIONAL
DEPENDENCIES' part of the above formula. These additional dependencies can
range from package managers to init systems, as well as the initial software
that comes with the distribution.

When you examine the variety of potential distributions one can select from, it
can be overwhelming. Why are there so many different Linux distributions? The
answer is because many people have different objectives when using an operating
system and therefore require different dependencies. Additionally, numerous
organizations and communities each have their distinct views on what
constitutes a good Linux distribution.

While distros like Ubuntu focus on being more friendly and welcoming to new
users, some distros, like Debian focus on stability and some like Arch, focus
on a certain combination of being minimal, cutting-edge and active.

To me the key principles that Arch Linux were emphasasising (which can also be
read in their [wiki](https://wiki.archlinux.org/title/Arch_Linux)), were more
appealing than the other mainstream distros available.

Arch Linux essentially distinguishes itself as a minimalist distribution with a
very knowledgeable community. This fact partially explains why Arch is
considered one of the most cutting-edge distributions out there -- its active
community maintains available packages that Arch users can install through a
community-driven repository called the Arch User Repository.

When you install Arch, you don't receive anything but a virtual console and
specific programs that you instructed the Arch installer to download. Unlike
other distributions that build many packages into your system by default, which
might include several programs that you may need but at the cost of downloading
additional unwanted programs, Arch Linux instead puts the customization
responsibility on the user. Which allows you to install and focus solely on
what you need.

## Some Reflections

### Getting used to it

Switching from Windows to Arch Linux was really challenging as in Arch Linux,
it is you that needs to bear practically all responsibilities regarding your
computer. As a result of this transition, I began to appreciate all the
features we often take for granted; screen lockers, clipboard functionalities,
power management, multimedia keys, and so on. These functionalities are usually
managed by specific processes running unseen in the background. The average
computer user might not realize that these are distinct programs that need
setup. However, when you are building your system on a minimalist distribution,
your knowledge of such details tends to increase.

### Variety of Solutions

When you want to accomplish something on Linux, there are many alternative ways
to do it. As a result, you're often left wondering, "which method/approach
should I choose first?" I think that these kinds of questions frequently
puzzled me. Here are some examples:

>'I am using X as a Window Server Protocol but I heard that Wayland is a newer
>protocol, should I switch to it?'
>
>'A program called pipeware for audio handling is recommended but some suggest
>something called pulseaudio, which one shall I use?'
>
>'Shall I use vim or neovim? I heard that vim is organized by one person whereas
>neovim is more community driven.'

Don't misunderstand me. I'm not suggesting that these questions are irrelevant.
They indeed become meaningful when the minor differences between them begin to
matter. However, I believe that the best approach is to simply select a tool
that resolves the problem at hand without overthinking and keep progressing in
our work. It's not beneficial for beginners to obsessively search for the
'best' program. Often, opting for a 'sufficient' solution can also be the
'best' choice, considering the time you might waste finding a program.

### Some beginner mistakes

One common mistake I see among beginners, which I also made myself, is
attempting to do things without understanding them properly. This is especially
common among new Linux users who may wind up breaking their system by copying
and pasting commands from some forum. They tend to install programs that
perform the same functions and use different package managers that configure
multiple dependencies and configuration files. Which often leads to one manager
disrupting the changes made by the other, and so on.

While it's understandable that people may want to work in the same way they're
accustomed to, this habit can also hinder them from getting used to the Linux
environment. For instance, instead of looking for programs that allow
installations through GUI-based applications, it's more beneficial to
understand how to operate the native package manager through a terminal, learn
how to build things from source code using makefiles, and so on.

The solutions people try to replicate their previous workspace can become
overly complicated. In these cases, it can be better not to solve the 'problem'
in the first place. Hell, even I've been guilty of this myself. Since I loved to
use MSPaint, whenever I needed to draw something on Linux, I used to run a
Windows instance on QEMU and start MSPaint with a bash alias that I had set up.
In retrospect, this was a poor solution as it would have been just simpler to
switch to another drawing program designed for Linux.

### Dual Boot

Setting up a dual-boot computer with Linux and Windows could seem like a good
choice for those interested in learning Linux. However based on my experience,
it is usually a bad idea. I made the error of installing Linux Mint alongside
Windows few years ago only to find myself frequently trying to synchronize my
files between the two systems. The maintenance that was required increased due
to the usage of both systems. Moreover, having the option to fall back on
Windows when facing issues prevented me from engaging with the Linux
environment enough. I could not learn how to troubleshoot and resolve issues on
myself.

In my view, what I have described above is like attempting to learn swimming
while using a flotation device. It's probably more effective to dive in and
learn the Linux operating system without relying on Windows as a safety
net.

I guess the only valid reason for wanting to keep Windows installed on a
computer is to play video games or use specific programs that are not available
on Linux. Other than that, I suggest using Linux for most things you do.

### GUI vs Terminal

Linux users often use terminal programs as they sometimes offer more
flexibility and power for certain tasks. The tasks are usually completed by
using command-line interface (CLI) programs which allows users to enter
commands to perform various actions. This is different from programs with a
graphical user interface (GUI) which usually have buttons and menus.

Even when using GUIs, we often end up performing repetitive tasks manually. In
these kinds of situations, using a terminal instead of a GUI program can become
really handy. As simple CLI can easily be used programmatically. They can be
used in loops, conditionally, and to pipe the output from one program to
another. This approach can be a game-changer, especially for those who aspire
to become power users. Here are a few simple examples of where this approach
has saved me a significant amount of time:

- Recently, I had a batch of 113 weirdly rotated images, but I was able to
  rotate them all to the desired orientation using the following code:
  ```
    for file in *.jpg;
    do
        convert $file -rotate 90 rotated-$file;
    done
  ```
- I used `yt-dlp` to easily download youtube videos and playlists. It was one
  of the most comfortable downloading experiences I had.

- I used `pdfcrop` for cropping PDF files.

- I changed the structure of folders that have many files in them by using
  simple for loops alongside with `mv cp rm`

Not to mention how much its easier to install packages that are on AUR or Arch
Repository compared to installing stuff in Windows.

Overall, I strongly believe that the power of interacting with programs through
terminal can increase one's overall productivity.

### Arch Linux manuals

One caveat of using command-line interfaces (CLIs) is that it can be easy to
forget the specifics of the interface. As a result, it is essential for terminal
users to know how to quickly open and find the information they need in manual
pages in order to effectively use CLIs.

Thankfully, most programs in Linux already have their own manuals available
through the `man` command. When I need to use a certain utility or CLI function,
all I do is open the terminal through a keybinding that I have set up and type
`man programname`, then I can quickly scroll through the manual page using VI
keybindings.

Despite the fact that I am already acquinted by heart with some of the most
important flags and utilities of the programs that I use, I am also a lot better
(faster) at finding the stuff I need. It is just as much important, if not more
important, to be able to find the stuff you need by knowing them by heart.

## Current Workflow

All of the configs for the apps below can be found from my [dotfiles
github repository](https://www.github.com/kugurerdem/legacy-dotfiles).

### Window Manager

I do not use any desktop environments. I use a tiling Window Manager (WM), a
type of software that automatically arranges and resizes application windows in
a non-overlapping fashion, without the need for manual dragging and resizing.
The particular WM I use now is called `dwm`, it is one of the tools that are
built by the hacker organization [suckless](https://suckless.org/).

Having switching to a tiling window manager, I now realize that how much of a
hassle was it to manually resize, drag, and select all my application windows.
Besides slowing me down, your average desktop environment also takes a lot of
space with their tilebars and etc. which I might want to use for seeing more
content.

dwm comes with another program called `dmenu` which enables you to select list
of options from the menu and do whatever you want with it. Initially dwm uses
dmenu to make the user easily open the programs they want to open through a
certain shortcut.

I also use `dwmblocks` to control the contents of the info bar on the top left.
I only show Volume, Battery, Memory and Date info there.

### Keyboard Layout

Since I am from Turkey, I need to use Turkish characters in my daily life a lot
especially when interacting with my friends. The thing is I also find English
keyboard layout very productive, especially when it comes to coding and using
Vim. As a result, I needed a mechanism to be able to benefit from both of these
functionalities. For this, I have attached a shortcut to switch between TR and
US layouts.

I have also swapped the Escape key with the CapsLock key as I use the escaping
functionality a lot when using VIM but do not use Caps Lock that much. It is
ergonomically a lot more preferable to use the CapsLock key for the Escape
functionality.

Here are my settings in .xinitrc that imply those changes:

```
# ~/.xinitrc

setxkbmap -option "caps:swapescape"
setxkbmap -model pc104 -layout us,tr -option grp:win_space_toggle
```

I am also aware that I could have used `tr-alt-q` layout which is basically an
English keyboard layout but if you use AltGr, keys like `i,o,u,g,c` turns into
`ı,ö,ü,ğ,ç`. The problem is that the only way I found was to change the keycode
tables through `.Xmodmap` and it was buggy. I could not find a simple and clean
way to implement this layout

### Terminal & Shell

As terminal, I use `st`, so far I have not seen particular advantage of using st
over other possible terminals that I could have use Alacritty or so, I just
needed a terminal that is lightweight and st was one of the possibilities I
could choose.

As shell, I mostly use bash. But I understand using zsh is perfectly fine in a
personal environment as well. The only possible problem that I can think of zsh
is portability problem of the scripts written for it.

### Text editing and programming

I use `neovim` for almost all my works involving text. Neovim is a fork of Vim,
a highly configurable text editor that is designed to be extendible and also
efficient through the maximal use of keyboard both with macros and
shortcuts. It also comes with a powerful syntax highlighting engine and support
for a wide range of programming languages and file formats. As a dialect of Vim,
Neovim is fully compatible with Vim and uses the same configuration files and
command syntax, but it includes additional functionality and improvements that
are not available in the original Vim. dialect of vim.

'Why use neovim instead of vim?' you might ask. Right now, it does not matter to
me whether I use vim or neovim since in both of these the things I want is
available. I use neovim because it was my first decision to go with it and
because of this I already have my files configured for neovim. The reason why I
initially chose neovim over vim was because of a certain workflow video I have
seen on youtube: Vim had not some the plugins that were used in the video. Later
on, I thought that video was full of unnecessary stuff so I gave up on it.

Getting used to vim has significantly improved my speed and comfort when
programming as its command mode is very efficient for text navigation and
manipulation without even having to use mouse or moving your hand much.
When my friends see me getting done stuff in VIM they sometimes refer to it as
'black magic', I like this a lot too. =)

{{< youtube 8n-ylg-pw6s >}}

I should also mention that I have started to convert some of my .odt, .docx
files like diaries, logs, records to plain text just because it gives me to
flexibility to be able to edit/read them through simple text editors such as
vim.

### Terminal Multiplexer

A terminal multiplexer is a software program that allows multiple terminal
sessions to be created, accessed, and controlled from a single terminal window
or console. It enables users to have multiple terminal sessions running
simultaneously, switch between them, and manage them easily.

Since I use terminal for almost all the text work I do including software
development, it is, thus, ergonomically important for me to have a way to manage
different programs through one terminal.

I do this thorugh a program called `dvtm`, an alternative for tmux.

Although I can split screens in vim when doing software development, it does not
give the same flexibity and ease of use the dvtm gives. There are some
programs you might want to see running simultaneously through one terminal
instead in addition to being able to edit/write files. You can do the latter in
vim, but the former is not so trivial to achieve.

Since `dvtm` already solves a problem that vim splits solve, I do not use vim
splits anyways.

Here is an example showcase of dvtm:

![Example DVTM
Showcase](https://raw.githubusercontent.com/martanne/dvtm/gh-pages/screencast.gif#fullsize)


### File Manager

I use `ranger`, considering to switch to `lf` but also don't see a reason for it
since I am already used to `ranger`.

As far as I remember the only thing I have changed in ranger is some of the
priorities on which programs to use when openning files and to enable image
preview mechanism.

Before getting used to ranger I was using a file manager with GUI named
`dolphin`.

### Taking Notes

I used lots of different note-taking apps such as Google Keep, Obsidian, Notion…
The problem is, almost all of these apps come with features that I do not use at
all, I mostly use note-taking applications as a way to remember the things that
I intended to do and for this, all I need is a way to sync my files between my
Phone and Computer. I used telegram for this purpose for a while, but since its
purpose is not this, I then looked for some alternatives.

Meanwhile, I found `gitjournal`, it is a git based note taking application with
a Mobile App. On my phone, I use its own application whereas on my computer, I
just use the `gitjournal` script that I created that updates the notes by
automatically running commands such as `git pull` `git commit` `git push` before
and after opening `nvim` to change note files.

The script for this can be found [here](https://github.com/kugurerdem/legacy-dotfiles/commit/8eb5c90e3db4fe4e553e9caea23607c88333c0ce#diff-b756de9b4e56d77950c9933ad361337ca35b7a23f479d34d9ac28a8ac29db497).

### Password Management and OTP

`pass` and `pass-otp`

Also there is a very simple script called `passmenu` which uses dmenu to fetch
the passwords from pass easily. For passphrase aplet to open, you need `gtk2` or
`gtk3` though.

### Wifi & Bluetooth

I use `bluetoothctl` to connect bluetooth devices and use `networkmanager` &
`nmcli` to connect to the internet.

### Web Browser

I just use `brave` like a normal human being. I like that it has a builtin
adblocker. Since I like moving with vim keybindings, I have also installed an
extension called [vimium](https://vimium.github.io/). This extension helps you to navigate your browser
through vim keybindings.

{{< youtube t67Sn0RGK54 >}}

## Conclusion

Switching to Arch Linux was a challenging experience due to its steep learning
curve. I had to deal with many things that I always used, but never realized
that there were actual programs for those functionalities, such as clipboard,
screen locks, and opening screens. It took some time to get used to it, but now
I am so accustomed to using Arch Linux that I don't even want to use Windows
anymore, except for cases like playing video games (which I also don't do it
much these days).

It's also fun to challenge yourself and succesfully get over those challenges.

