---
title: 'NixOS Hates Precompiled Programs (Learn How To Fix It)'
date: '2024-12-21'
---

I've been using NixOS for about six months and am generally satisfied with my experience. However, in this essay, I won't talk about how great NixOS is, but rather about one common issue that many users, including myself, have faced or will face in the future.

**In NixOS, most pre-compiled programs will not work out of the box.** In this essay, I’ll share my experiences on this issue and explain why it happens, along with some approaches I’ve found very helpful to overcome it. Hopefully, this will help others avoid some of the frustrations I’ve encountered.

# The Problem

Let's say that you're working on a project. Maybe using Node. You decide to try out a library. You write some code. And then, once you try to run your script, you get the following error:

```fish
Error: Failed to launch the browser process!
/home/rugu/.cache/puppeteer/chrome/linux-131.0.6778.108/chrome-linux64/chrome: error while loading shared libraries: libdbus-1.so.3: cannot open shared object.
```

Being a smart person, you suspect this might be related to NixOS. You begin researching the issue, only to find people suggesting case-specific solutions such as [\[1\]](https://www.reddit.com/r/NixOS/comments/1gzygtd/comment/lz1qd0t) and [\[2\]](https://www.reddit.com/r/NixOS/comments/1gzygtd/comment/lz5cq99). Sure, in this case, using the Chromium installed on your system instead of the built-in version that comes with Puppeteer, and specifying the `executablePath` property to its path in your Node code when initializing Puppeteer can save the day.

**But as a smart person, you also understand that this is a case-specific solution and wonder what you would do if the library that you use did not allow such flexibility.**

The problem is that, while case-specific solutions like the one mentioned above may work, you can still encounter the same issue whenever you try to run a precompiled program that expects certain files and libraries to be located in specific paths on your filesystem.

So, as I said in the beginning of the essay, **in NixOS, most pre-compiled programs will not work out of the box.** And now we need to understand why this happens so that we can also understand the solutions better.

## Why it happens?

Most UNIX-like systems follow the [Filesystem Hierarchy Standard](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard) (FHS), which defines where programs can find resources or place files. For example, system-wide configuration files are typically in `/etc`, executables in `/bin` or `/usr/bin`, and shared libraries in `/lib` or `/usr/lib`, and so on...

The problem with this approach, and part of why some people prefer systems like NixOS, is that it tends to be messy: The files are scattered across the system, there is no single source of truth, the structure relies on conventions rather than strict enforcement, you can have problems such as version conflicts across different programs that rely on different versions of the same programs, and so on.

The approach that the Nix tools take to resolve these issues is to store every package under a folder called `/nix/store`, in a way that each package has its isolated file structures instead of relying on shared directories. We can say that this directory is a [Merkle tree](https://en.wikipedia.org/wiki/Merkle_tree): package paths are derived from hashes of their contents, dependencies, and inputs. And, if any of these things change, the path for the package also changes.

Now, even when using tools built around `/nix/store` (such as the `Nix package manager`, `nix-build`, `nix-shell`, or `home-manager`), you can still modify most system files imperatively and apply "dirty" solutions in certain parts of your system. However, once you switch to NixOS, the `/nix/store` management approach extends to the entire system, including system files, universally shared resources, shared libraries, and so on.

This is why precompiled programs designed for conventional UNIX systems usually don't work in NixOS. These programs expect specific libraries to be located in certain paths, but NixOS manages system dependencies and files in a self-contained manner within `/nix/store`. As a result, most of these programs can't find what they need unless they are packaged with Nix, run in an environment that mimics FHS or are guided by NixOS itself.

## Dynamic Linking

When you run a precompiled program that depends on shared libraries (those not statically built into the program but loaded at runtime), the dynamic linker (`ld.so` on Linux) is invoked to load the necessary libraries.

The linker checks a set of directories, like `/lib`, `/usr/lib`, and others specified in environment variables such as `LD_LIBRARY_PATH`, to find the required libraries. It also performs caching to help speed up the loading of dynamic libraries.

In the case of `puppeteer` or similar programs, the program is precompiled and expects certain dynamic libraries. However, the linker cannot find these libraries.

# Workarounds

The solutions I've come across so far to work around this problem are:

(1) Look for whether the program you are trying to run is already available in [nixpkgs](https://search.nixos.org/packages) \
(2) Package the program you want to run as a Nix derivation with the necessary dependencies. \
(3) Use the Nix derivation function [buildFHSEnv](https://ryantm.github.io/nixpkgs/builders/special/fhs-environments) \
(4) Use the [nix-ld](https://search.nixos.org/options?channel=24.11&show=programs.nix-ld) options in NixOS.

Among these options, (1) is typically the best solution. If the program you're trying to run is already packaged in `nixpkgs` (which is often the case), it saves you a lot of time.

Unfortunately, though, there will be times when you can't find what you need in `nixpkgs`. In that case, (2) — building your own Nix derivation — is also an option. It's helpful to the community, and it's also the most modular approach as a package in the `nixpkgs` repository can still work on a traditional UNIX system without a `home-manager` or `NixOS`. The downside is that this approach will likely take more time than the other options.

If you are short on time, or not that much of an altruistic person, you can consider using (3) or (4).

The `buildFHSEnv` function allows you to create an environment that mimics the traditional UNIX filesystem hierarchy (FHS), you can then use this environment, in a nix-shell for example. This enables you to place your program and its dependencies in expected locations like `/usr/bin` or `/lib` without fully packaging it.

On the other hand, `nix-ld` (option 4) is a NixOS-specific solution that helps with dynamic linking. It ensures the program can find the libraries in `/nix/store` without needing to rebuild or repackage everything.

If you're in a non-NixOS environment and don't want to mess with your shared libraries, `buildFHSEnv` might be the way to go. But if you're already using NixOS, I think `nix-ld` just makes more sense. It resolves the issue at the same layer where it originated, NixOS itself. You can specify the libraries that you want to be available on your whole system in your `configuration.nix` file, and can also use `NIX_LD*` [environment variables](https://github.com/nix-community/nix-ld?tab=readme-ov-file#usage), for example maybe with `shell.nix` to make certain libraries available in the ephemeral shell that you intend to work in without affecting your global system.

Whichever option you choose, except for the first one, you'll still need to figure out which Nix packages to install to make the necessary libraries available to the program.

## Finding which packages to install

To find the necessary Nixpkgs packages for dynamic libraries, you can use tools like `ldd`, `nix-index`, and `nix-locate`.

Run `ldd` on the executable to list the shared libraries it needs. Then, you can use `nix-locate` to find the corresponding Nixpkgs packages. But, to use `nix-locate`, you first need to run `nix-index` or use a pre-generated `nix-index` database (like I did, since `nix-index` consumed a lot of memory on my system for some reason).

Now, the issue with `nix-locate` is that it can return a lot of results. Not just those that provide it, but also the ones that depend on it. For example:

```fish
[I] rugu@nixos ~/bin> nix-locate libdbus-1.so.3 | wc -l
206
```

We would not want most of these packages. So, to resolve this issue, I've experimented with `nix-locate` a bit and found the following specific flags to narrow down the results to what we essentially want:

```fish
[I] rugu@nixos ~> nix-locate --top-level --whole-name --minimal --at-root "/lib/libdbus-1.so.3"
dbus.lib
```

Great, this means that for "libdbus-1.so.3", we can just install `dbus` package from Nixpkgs!

To automate the process of finding dynamic libraries using `ldd` and then using `nix-locate` to identify which Nixpkgs provides them, I created a [small script](https://github.com/kugurerdem/nix-config/blob/23f0ae804112672e1c8c334efa126b410ed874d7/home-manager/dotfiles/.local/bin/nixldd). It lists the dynamic libraries and which nixpkgs provide them, all at once. To use it, you can simply run it as follows:

```bash
nixldd $PROGRAM_PATH
```

Example (I didn't include the full output because it's too long):

```fish
[I] rugu@nixos ~/bin> nixldd /home/rugu/.cache/puppeteer/chrome/linux-131.0.6778.108/chrome-linux64/chrome
libdl.so.2:
 iconv glibc_multi glibc_memusage
libpthread.so.0:
 saw-tools iconv glibc_multi glibc_memusage
  ...
libc.so.6:
 iconv glibc_multi glibc_memusage
```

Now, we know which Nix packages to install to provide the required dynamic libraries.

I also considered creating a tool that returns the optimal combination of packages to install, but could not find time to work on it yet. Maybe I can try to do that in the future.

# Conclusion

So, these were some of the insights and tricks I’ve learned over time about making precompiled programs work in NixOS while using it in my development environment. I hope they help you as well.

