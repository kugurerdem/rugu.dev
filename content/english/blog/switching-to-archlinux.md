---
title: 'My Experience on Switching to Arch Linux'
draft: false
date: '2023-01-29'
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

After seeing what I saw, I thought to myself, 'If I were able to develop and
refactor code at that speed, I would save a lot of time, I could have spend more
time on thinking about the actual stuff with less friction as possible. I want
to have this power.' So I have look to [his CV](https://gwn.wtf/resume.html) and
realized that he was knowledgeable on topics that many developers, including
myself, struggle with. Influenced by this, I started asking him (he was our
consultant, after all) as many questions as I could and focused on the resources
he suggested and the technologies he used.

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

## What is special about Linux?

Linux is a free and open-source operating system (OS) based on the Linux kernel,
a piece of software that sits between the hardware and programs that you run.
Maybe you already know it but the vast majority of servers are already running
in Linux. Android, the operating systems that most phones use, is also a
specific fork of the Linux Opearting System. By this means, even a developer
uses Windows or MacOS, it is still thought among many programmers that it is
very beneficial to learn some fundamental aspects of Linux regardless.

Many developers already seem to agree on the importance of learning Linux to
some extent for System Administration is out of the question. The variety of
views seem to raise when it comes to use Linux on a personal computer. From the
view of an average computer user, it might not matter much whether one uses
Linux, Windows or MacOS but as your values and needs become more nuanced it
might be more reasonable for you to switch to Linux over its alternatives.

The first aspect that comes to my mind between Linux and other well-known
operating systems is that Linux is a free and open-source operating system,
while Windows and Mac OS are proprietary operating systems that must be
purchased. This has several implications:

- You can set up Linux on a computer with no cost except some finite amount of
  time where you have to spend some money in order to use MacOS or Windows.
- Since both MacOS and Windows are closed-source, we don't know for sure what
  they do under the scenes. Windows, for example has a builtin keylogger.
  Although Apple does not have such a claim, there is nothing preventing them to
  do something like that.
- By being open source, linux is highly customizable and allows users to modify
  and tailor their environment to meet their specific needs. In contrast,
  Windows and Mac OS are more limited in terms of customization. So Linux is a
  great choice for users who want to have control over their operating system
  and want to modify or customize it to meet their specific needs.
- By being Open Source, the source code of Linux is checked and maintain by a
  HUGE group of experienced developers. This might also have some implications
  on its security.

Another aspect that I like to focus on its Linux's user base. As with many
products, to use a product is often not merely using it but also being part of a
group. By using Linux, you also become part of a group that it contains a lot of
experienced software developers, hacker-minded people. To me, this is very
important because when you become part of a group, it is more likely for you to
get some of the habits of the group whether whether you intended so or not. By
switching to an environment where there are more experienced people you become
more exposed to their content.

## What is unique about Arch Linux?

Before diving into this question, I shall clarify what a linux distribution is.
In essence a linux distribution is a version of the Linux operating system that
is packaged with additional software and tools.

So an oversimplicated, but an intuitive formula can be given as:

``DISTRIBUTION = KERNEL + SOME ADDITIONAL DEPENDENCIES``

In essence, there is nothing preventing you from doing what one can do in one
linux distribution in another by changing the 'SOME ADDITIONAL DEPENDENCIES'
part of the formula above. These additional dependencies vary from package
manages to init systems, and the initial softwares that come with the
distribution.

If you look at the amount of possible distributions one can choose, you can be
buffled. Why are there so many different linux distributions? Because, there are
lots of people who have different goals when using an OS and therefore needing
different dependencies. There are also lots of organizations and communities
that share different perspectives of what a constitutes a good linux distro.

While distros like Ubuntu focus on being more friendly and welcoming to new
users, some distros, like debian focus on stability and some like arch, focus on
a certain combination of being minimal, cutting-edge and active.

Despite the fact that there is no objective metric to determine which one of
these distributions is the best, they can be superior to each other given a
certain context. For example, if you are working on a production server for an
important application you might want to choose a linux distribution that does
not give major updates regularly to become stable. If you are a new person who
switches to Linux you might prefer a friendly linux distro such as Ubuntu for
their user friendly community. If you are a person who only wants the programs
that they use and nothing more than that, a distro which comes with less
dependencies as much as possible might be a good choice for you.

To me, Arch Linux comes forward by being a *modern*, *minimalist* distribution
that have a very knowledgable *community*. You may be already heard that Arch
Linux is a cutting-edge distribution, this is to say that it adapts new updates
and releases of a wide variaty of programs very quickly. In Arch Linux besides
its own package manager `pacman` an organization called Arch User Repository is
also providing a possibility for installing programs, this repository is
basically very frequently maintained by Arch Linux users, the community. I think
it would not be unfair to say that AUR is one of the most active Linux
repository, if not the most. One of the widely known things about Arch Linux is
that when installed, you dont have anything but a terminal and some of the
programs you specified arch installer to download. This is related to Arch Linux
being minimal. Instead of building many packages into your system by default
that my contain several programs that you might need at the expense of
downloading lots of programs that is certain you will not use, it just leaves to
customization part on the users' shoulders. This way you can install and focus
on only what you need.

This article will not just be about my journey of switching to arch linux but
also embracing its philosophy.

## Some Reflections

### Getting used to it

It was hard to change the environment that I used to (Windows) with another
environment that almost for all the things you want to do you are the one who
needs to take the responsibility of deciding how things should go. During this
process, I noticed that there are lots of things that we take it granted but by
itself an actual thing: Screen lockers, clipboard functionalities, power
management, multimedia keys... All of these things are handled by certain
processes that are running on the background. Your typical computer user might
not be aware that these are actual programs that are needed to be set up but
when you are building an environment on a minimalist distribution, your
awareness of such things tend to increase.

### Variety of Solutions

When you want to do something on Linux, there are a lot of alternative programs to let
you do it. And because of this, one is left with the question "which program should one
prefer first?" I must admit that I still often get stunned by this question and
already lost enough time on it.

>'I am using X as a Window Server Protocol but I heard that Wayland is a newer
>protocol, should I switch to it?'
>
>'A program called pipeware for audio handling is recommended but some suggest
>something called pulseaudio, which one shall I use?'
>
>'Shall I use vim or neovim? I heard that vim is organized by one person whereas
>neovim is more community driven.'

Don’t get me wrong. I am not implying that these questions are meaningless. They
are indeed meaningful after a point where small nuances between them start to
matter.

I think the best approach here would be to just choose a tool that solves the
problem without thinking too much and continuing on with what we do. It is a
mistake to find the ‘best’ program for your beginners. In many cases, going with
the ‘sufficient’ solution is also ‘best’ in terms of your time spent on finding
a program.

### Some beginner mistakes
Another mistake I see rookies do and I did is to try to do things without actually understanding them. It is not at all uncommon amongst new Linux users to break their system by copying and pasting the commands they saw on some forum. They install programs that do the same stuff, they use different package managers that configure several dependencies and configuration files which result in one manager breaking the changes made by another, and so on.

While it is understandable that people want to do things in the same way they are used to in their previous settings I think this behavior also prevents them from getting used to handling things in Linux: For example, instead of seeking programs that enable installation stuff through certain programs with GUI, it is better to learn how one’s native package manager is working through a terminal, how to build stuff from source code with makefiles and so on.

Sometimes the solutions people come up with to imitate their previous workspace are so complex that it is better to not solve the ‘problem’ in the first place. One example is from me: Since I love MSPaint, when I needed to draw things, I was running a Windows instance on QEMU and initiating MSPaint with a bash alias that I have set. This was a very bad idea as just switching to another Linux program for drawing would be just easier.

### Dual Boot

Setting up a dual-boot computer with Linux and Windows may seem like a good idea
for those looking to learn Linux, but I think that it can be a bad idea. A few
years ago, I made the mistake of setting up Linux Mint alongside Windows.

Firstly, dual-booting increases the amount of maintenance that one has to do.
Not only do you have to keep both operating systems updated and running
smoothly, but you also have to know how to synchronize your files between the
two systems.

Secondly, having the option to fall back on Windows when encountering problems
can prevent one from fully immersing themselves in the Linux experience and
learning how to troubleshoot and solve issues on their own. It's similar to
trying to learn how to swim while having a floatation device. It's better to
dive in and learn the Linux operating system without the crutch of Windows as a
backup.

In my opinion, the only good reason for still wanting to have Windows installed
on a computer is for either playing video games or for using special programs
that are not available on Linux.

### GUI vs Terminal

Linux users often find themselves using terminal programs because of the
flexibility and power it offers for advanced users. Terminal programs, which are
controlled through a command-line interface (CLI), allow users to enter commands
in order to perform actions, unlike programs with a graphical user interface
(GUI) which typically have a visual interface such as buttons and menus. While
GUI programs have the advantage of ease of use for non-technical users and quick
access to common tasks through visual cues, terminal programs offer more
flexibility and can be programmed and automated through scripting, making it a
preferred choice for advanced users.

Even when using GUIs, it is common to find ourselves performing repetitive tasks
manually. With simple programs that are designed to perform one specific task
well and are also available to use programmatically, you can use them in for
loops, conditionally, and pipe the output of one program to another. This
approach can be a game changer, especially for those who want to become power
users. Here are a few examples of where this approach has saved me a significant
amount of time:

- Recently, I had a batch of 113 weirdly rotated images, but I was able to
  rotate them all to the desired orientation using the following code:
  ```
    for file in *.jpg;
    do
        convert $file -rotate 90 rotated-$file;
    done
  ```
- I used `yt-dlp` to easily download youtube videos and playlists. It
  was one of the most comfortable downloading experiences I had.

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
github repository](https://www.github.com/kugurerdem/dotfiles).

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
vim. As a result, I needed a mechanism to be able to benefit from both of these
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

The script for this can be found [here](https://github.com/kugurerdem/dotfiles/commit/8eb5c90e3db4fe4e553e9caea23607c88333c0ce#diff-b756de9b4e56d77950c9933ad361337ca35b7a23f479d34d9ac28a8ac29db497).

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
