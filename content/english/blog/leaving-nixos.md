---
title: Why I'm Leaving NixOS After a Year?
date: 2025-08-03
tags:
  - highlights
discussions:
    hackernews: https://news.ycombinator.com/item?id=44785093
    twitter: https://x.com/kugurerdem/status/1952096414240026911
    linkedin: https://www.linkedin.com/feed/update/urn:li:activity:7357858884037787649/
    programming.dev: https://programming.dev/post/35107436
---

Around a year ago, I published a blog post explaining my overall experience [Switching from Arch to NixOS](https://www.rugu.dev/en/blog/nixos/). You can read it if you're interested in my early experiences, but, to give you a spoiler, that post ends with me saying:

> Unfortunately, though, I don’t think the benefits I’ve gotten in this one month of using NixOS so far justified the cost I’ve initially spent and continue to spend learning Nix and NixOS. \
> --- \
> Ultimately, whether the benefits of learning a particular technology outweigh the costs depends on how much you take full advantage of its features. So, I believe that if I experiment with more setups, try different programs, or start managing servers with Nix, I will begin to see a better return on this investment from what I have learned so far. :)

Well, it's been about a year since I published that post. Since then, I've experimented with more setups, tried different programs, and started managing my own server with NixOS. And... **Contrary to my initial expectations that I would get a better return on investment from NixOS with more usage, the opposite happened.**

# The Pain of Getting Things to Work

So you want to try a new program/service? First, you try the NixOS module and see whether it works the way you want. Oh... it doesn't. Now, you have a few choices:

1. Try to figure out what's wrong with your current configuration (maybe even read the source code of the NixOS module you're using.)

2. Stick to more standard NixOS modules instead of using the one provided for the program you're trying to install. For example, you might create your a systemd unit file and include the program binary using Nix.

3. Just use some containerization technology like Docker, or maybe a sandboxing solution like Flatpak (which is exactly what you can do on any typical FHS-based Linux distro.)

Sometimes, you just can't foresee whether the reason an app isn't running the way you want is due to a problem or limitation in the NixOS module. And if you don't put a time limit on how much you investigate it, you may lose a lot of time. The problem with the second option is that it's not fun to go through that process every time you want to try a new application or service on your system. Especially if it's just because you can't install it the native way, or you just don't like something about the native way. So you end up falling back to the third option, which is something you can just as easily do on any FHS-based distro anyway.

You need to run a program like Electron in your Node project for your new job, and you try to install it through npm and then run it? Well, guess what. [NixOS hates pre-compiled programs](https://www.rugu.dev/en/blog/nixos-precompiled/), so you need to learn more about how to resolve this problem while using NixOS. So you're kind of forced to learn about other specific Nix tools (`nix-ld`, `buildFhsEnv`, creating your derivations, etc.) if you need to use them regardless.

In an FHS system, you'd configure a file and then run the program with that configuration. In NixOS (or home-manager), you now write a DSL based on Nix expressions, which then generates the actual configuration file the program will use. I think this just means another layer to trace when debugging. **When you are using NixOS, and things work, everything works smoothly. It's fine. But when things go wrong, you now have an additional layer to worry about.** Did you misconfigure something at the NixOS level, or did the config generate correctly but there's an issue with the program itself? And so on...

# Problem of (Leaky) Abstractions

I think one of the main issues here is that NixOS just adds an abstraction layer on top of a procedural system. **You interact with it as if it were declarative, but it's not. Under the hood, you're still running programs that were designed for FHS environments, where programs share their libraries, bootstrap themselves, install other precompiled programs, and so on... I believe that almost all my frustrations with NixOS is due to this phenomenon of abstractions.**

As the [Law of Leaky Abstractions](https://www.joelonsoftware.com/2002/11/11/the-law-of-leaky-abstractions/) suggests, *"All non-trivial abstractions, to some degree, are leaky."* And it turns out that NixOS is not immune to this phenomenon either.

I would definitely prefer NixOS over an FHS-based Linux distro if it weren't just a wrapper around Linux, but instead its own system with dedicated tooling and native programs. But I also understand how hard it is to build and sustain a community of people developing software specifically for a niche OS. So, it makes sense that NixOS is designed as a Linux distro. It's similar to how Rich Hickey made Clojure run on the JVM and JavaScript runtimes to take advantage of their tooling and ecosystems, despite sharing very little with them in terms of philosophy.

**But in the end, I feel like the discrepancy between the philosophy of NixOS and the programs and services running under the hood is so great that it is almost impossible to not feel this in some negative way, as a user.**

# The Time Cost

Well, I like NixOS and Nix's philosophy. That's why I stuck with it for over a year and tried to learn as much as I could despite the frustrations. I also think NixOS delivers on most of its promises, so the issue for me isn’t whether it works or not. It’s also not about whether you can make things work (you can). The problem is: at what cost? Tasks that would take very little time on a traditional FHS-based distro can take significantly longer on NixOS. And I really don’t like that.

I understand the benefits of reproducibility and so on, but when I think about how much I've gained from that reproducibility compared to my old dotfiles setup, I realize that I did NOT see any real benefits **in practice**. I am just sure that configuring and making stuff run the Nix way just takes way more time than just using a regular FHS distro.

# Back to Arch, BTW

I still think that NixOS can be useful for some people who especially HEAVILY rely on syncing configurations across multiple devices and TRULY need system-level reproducibility. But I’m definitely not one of those people. I don’t need reproducibility to the extent that NixOS provides, and **I also realized that I'm way more comfortable getting things done in an impure system than dealing with the friction of a pure one that slows me down in almost in every change I do.**

So now, I'm just back to using Arch Linux, btw.
