---
title: 'Switching from Arch to NixOS'
draft: false
date: '2024-07-23'
discussions:
    hackernews: https://news.ycombinator.com/item?id=41066431
tags:
    - highlights
    - technology
    - linux
---

I’ve switched my desktop computer environment from Arch Linux to NixOS and used
it for about a month. I want to share my migration experience in case it might
interest or even help others.

I also want to thank my friends [Onur](https://github.com/onsah) and
[Mert](https://github.com/mk-nlp) for encouraging me to switch to NixOS and
providing help.

# Why Migrate from Arch in the first place?

As someone who likes playing with tools to understand how they work as well as
to match my preferences and ergonomic choices, I find myself frequently changing many
configurations on my system. However, it didn't take long for me to realize
that I need a system to save and possibly automate these configuration
processes to save time in the future.

A common solution is to create a git repository, often called "dotfiles," where
you can store your configurations and changes. This approach helps you avoid
repeating the process of configuring the same stuff over and over when
switching to new host machines. You turn your home folder into a repository
itself by initializing git directly inside of it, set the remote address, and
pull the content. You can check out what my dotfiles looked like before
switching to NixOS [here](https://github.com/kugurerdem/legacy-dotfiles).

Yet, even with this "dotfiles" approach there are some problems:

- It's very easy to forget to add some configuration files from your computer
  to the repo because, in a typical Linux setup, these files are often
  scattered in different locations, and unstaged changes can be easily
  overlooked. This is especially true for the home directory, where there are
  many unstaged files by default, making it easier to miss the ones you want to
  stage. I've had several instances where I realized I was missing some
  configuration files from my old computer in the Git repo after formatting my
  PC.

- Your dotfiles are likely to become more complex over time, requiring you to
  document how to configure certain aspects to avoid confusion the next time
  you set up your environment.

- Even if your dotfiles repo is perfect, there's no guarantee your system will
  work the same when you rebuild it. Changes might have occurred to some of the
  packages that your dotfiles repo relies on. As a result, you might encounter
  issues regarding package upgrades or even conflicts. This problem isn't
  specific to rebuilding systems from a dotfiles repo but also affects regular
  users who just want to just update their systems.

In addition to these problems mentioned, although not very often, I would
encounter situations in Arch Linux where I had to look up an error message, a
specific bug, or a non-backward compatible update for some of the apps I use.
Why? Because a system update broke something! While this may not seem like a
big problem, it can be very inconvenient if you're in a hurry to get a job
done. In these kind of situations, I often had to either fix the problem
immediately or ignore it for a while and fix it later. Rolling back to the
state of my computer before the system update and postponing the task of
addressing the issue introduced by the update was not an option.

This was the moment when I remembered that the tools (Nix and NixOS) my friends
had been recommending could be useful to me.

# What is Nix/NixOS?

In essence, Nix is a package manager (there's also a programming language
called Nix, which can sometimes be confusing). What sets Nix apart from its
alternatives is the way it manages packages and much more, including your home
folder. It is designed to provide reliable and reproducible package management
by isolating packages from each other in a smart way, preventing issues like
dependency conflicts. It also allows users to configure their computers using
its configuration files, through its programming language. So, you're not
limited to downloading specific versions of packages with their dependencies,
but you can also configure other files on your computer, such as your dotfiles.

NixOS, on the other hand, is a Linux distribution that uses Nix as its default
package manager. It integrates Nix's features to manage the entire system at
both the system and user levels.

To stay within the scope of this essay, which is to share my Nix and NixOS
experiences rather than explain their inner workings, I'll stop here. However,
if you're curious to learn more, I found [nix.dev](nix.dev) and
[nixos.wiki](nixos.wiki) particularly helpful for learning more about Nix and NixOS.

# The Learning Curve and Initial Trial

Since NixOS fundamentally provides a much different user experience than most
of the other Linux distributions. I thought it would be wiser to first try
NixOS in a VM instead of directly trying to figure out stuff after installing
the distro on my host machine.

I can confidently say that during this period of testing NixOS on a VM, I had
more troubles related to `QEMU` and network bridging than problems related to
understanding how Nix works. The same goes for the installation process as
well, for some reason `Ventoy` did not work properly with the NixOS iso image
while formatting the disk with `dd` just worked fine.

My initial goal was to make the VM I was running function exactly like my host
machine. This way, once I got NixOS working as intended in the VM, I could
replicate the setup on my host machine. I just needed to copy the configuration
files from the VM to the host machine and run a few Nix and NixOS commands. And
this was exactly what happened when I switched to my host machine. Easy peasy.
:)

In the end, it took me around 4-5 days, working 2-3 hours each day, to learn
Nix and NixOS and replicate about 95% of my Arch dotfiles in the VM. When I
installed NixOS on my host computer, I simply cloned my nix-config repo, ran a
few commands as described, and boom! Everything was set up. :) It was such a
nice experience.

# Initial Impressions and Experience

Here are my first impressions after using NixOS for about a month:

- At first, it feels like the knowledge you've gained from using conventional
  FHS Linux distros becomes redundant, as you no longer configure programs by
  directly modifying their configuration files in the file system. Instead, you
  use the settings provided by NixOS and home-manager (a standardized Nix
  program that allows users to manage and configure their home environments
  through Nix files without root privileges).

  Because most configurations are done through the settings provided by the
  packages, it initially seemed like this might prevent users from
  understanding what's happening under the hood.

  However, after using NixOS for a while, I realized this was not true. The
  abstraction that NixOS packages provide doesn't hide everything from the user
  to avoid confusion with irrelevant details. Instead, it offers a way to
  configure your environment the Nix way, so the resulting configuration files
  are created by Nix.

  Most of the prior knowledge I had about configuring the programs I use was
  easily transferable to the NixOS domain. Also, you don't have to configure
  every dotfile through Nix. In fact, home-manager allows you to source files
  to desired destinations (see the home.file.*.source option for home-manager).

- The documentation is not in great shape. The Nix wiki is certainly not as
  good as the Arch wiki. Sometimes, it's outdated, and other times, it's not
  detailed enough. This is why it's very important to learn the Nix programming
  language well so you can easily read the options available for a package you
  want to install. Once you understand the fundamentals of the Nix programming
  language, the code itself becomes the documentation.

- It is very confusing to have many alternatives for certain tasks. For
  example, there are two different ways to install home-manager (standalone
  installation vs. system modules), and how you install it affects the way you
  interact with it later. Another example is Nix flakes, which are meant to
  replace channels (an imperative way of downloading packages) but are still
  considered an experimental feature by NixOS.

  To be fair, having to choose between many options is already an issue in
  Linux (though many see this as a feature), and NixOS seems to have the same
  problem.

- From my experience so far, most Nix packages are designed to allow
  self-contained setups and installations, including plugins. Here are a few
  examples:
    - When installing Firefox or Chromium browser packages, you can set which
      plugins you want to be installed by default.
      [Example](https://github.com/kugurerdem/dotfiles/blob/963f8a99545fa648e31edf085299fd96802e7d04/home-manager/home.nix#L137)
    - For my default password manager `pass` (a standard UNIX password
      manager), the plugins I wanted to integrate with pass can be defined
      through a derivation attribute `withExtensions`.
      [Example](https://github.com/kugurerdem/dotfiles/blob/963f8a99545fa648e31edf085299fd96802e7d04/home-manager/home.nix#L52)
    - When installing the Minecraft launcher prismlauncher, you can declare
      which JDKs should be available and used by the launcher by simply
      overriding one of the package attributes.
      [Example](https://github.com/kugurerdem/dotfiles/blob/963f8a99545fa648e31edf085299fd96802e7d04/home-manager/home.nix#L81)
    - For Neovim, you can declare which dependencies and plugins you want to
      install out of the box using the `extraPackages` and `plugins` options of the
      home-manager's `programs.neovim` option.

  These are just a few examples, and I am sure this is a standard for many
  other programs. I really like this. Dependencies used only
  by certain programs are self-contained within the program that will use them.

  You can even override some of the derivation attributes for the package you
  are installing so that it is not installed from the git source repository
  defined in the nixpkgs repo, but from your own source repository. I used this
  technique to install and set up my window manager dwm using my own git fork
  of dwm.
  [Example](https://github.com/kugurerdem/dotfiles/blob/main/home-manager/dwm.nix#L4)

- The configuration files and the nix-config repo I now have are much
  more elegant and simpler than my previous dotfiles repo. It's much easier to
  organize configurations in a modular way now.

- The rollback mechanisms that Nix provides (combined with the ease of using
  other people's configurations) make trying new things (like different window
  managers, desktop environments, programs, or even other people's setups) very
  appealing.

- In Arch, there were a few instances where I had to use additional package
  managers like `yay` to access the AUR (Arch User Repository) alongside the
  official Arch repository. I also recall compiling and building some tools,
  like `fzf`, from scratch. I haven't needed to do any of this while using
  NixOS.

  Overall, I've had a better experience with the Nix package manager itself
  compared to using pacman and the AUR.

# To Conclude

In the end, I really think Nix and NixOS are very strong tools for achieving
reliable and reproducible system configurations and package management.
Unfortunately, though, I don't think the benefits I've gotten in this one month
of using NixOS so far justified the cost I've initially spent and continue to
spend learning Nix and NixOS.

But since I currently have no workload and enjoy the learning process, I
don't see a serious problem here.

Ultimately, whether the benefits of learning a particular technology outweigh
the costs depends on how much you take full advantage of its features. So, I
believe that if I experiment with more setups, try different programs, or start
managing servers with Nix, I will begin to see a better return on this
investment from what I have learned so far. :)
