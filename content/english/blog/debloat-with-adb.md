---
title: 'Debloating My Android Phone with ADB'
draft: false
date: '2024-01-16'
discussions:
    hackernews: https://news.ycombinator.com/item?id=39019252
tags:
    - technology
    - linux
    - productivity
---

My mother recently mentioned that her phone is continually opening certain
windows and prompting her to use services she doesn’t need. After checking her
phone and doing some online research, I found out that many others have also
complained about this issue.

Apparently, she was referring to pop-ups triggered by a pre-built program
called "SIM Menu". This program basically allows operators to send
notifications and even generate pop-ups on your phone. And the frustrating part
is that most of these pop-ups seem to promote irrelevant services. If you
accidentally click "OK" when one of these pop-ups appears, you get charged by
your provider. It is a carefully set up trap designed to make you accidentally
subscribe to their unnecessary services and pay money.

Since I prefer simplicity and like to have control over the tools I use. I was
already thinking about removing the bloatware on my phone that came
pre-installed and can't be deleted through the interface. Hell, even some of
the fundamental apps, like a gallery or file manager, are filled with ads. It
gets on my nerves. So, I was already planning to learn how to remove these
bloatware and the situation I described earlier was the final straw for me.

I've searched for tools to debloat Android phones and found ``adb``, Android
Developer Bridge. It's basically a program which allows you to directly create
a shell session for your phone, similar to connecting to a remote server via
SSH (the only difference is that instead of connecting through the internet,
the connection is made via USB).

Since Android is essentially an OS based on the Linux Kernel, the shell you
connect to will most likely be a variant of the sh dialect. So, for those who
are familiar with working in UNIX environments, its very convenient to remove
or install packages and customize your phone this way.

In this essay, I document this process of removing bloatware from my phone
as a reference for future use. Hopefully, you find it helpful as well.

# Enabling USB Debugging Mode

While it is possible to establish an ADB connection to your phone over Wi-Fi, I
chose to use ADB through a USB connection as using Wi-Fi involves certain
security risks. For instance, others on your Wi-Fi network could potentially
connect to your phone, especially if your phone doesn't have proper security
settings. Not to mention that you would need to set up your phone using ADB
through a USB connection first before initiating a Wi-Fi connection anyways.
Which kind of makes using WiFi seem more pointless as the reasons for
preferring Wi-Fi over USB are usually just convenience or not having a USB
cable available.

To connect your phone using adb via a USB connection, you need to enable USB
Debugging mode on your phone. I won't go into the specifics of this process as
it varies by phone. But for my own phone, I simply navigated to the "About"
section and tapped "MIUI Version" multiple times to switch into Developer Mode.
Then, I searched for the "USB Debugging" option and enabled it.

# Connecting to your phone via a Shell Session

Once you've installed `adb`, you can view the manual by typing `adb help`. You
can also refer to the [online
documentation](https://android.googlesource.com/platform/packages/modules/adb/+/refs/heads/master/docs/user/adb.1.md).

To connect to your phone via a shell session, simply type `adb shell` and
you're good to go. Most standard UNIX commands such as `ls`, `cat`, `echo`,
`grep`, and more can be used.

Keep in mind that you don't need to enter a shell session just to run specific
commands. You can also use the `adb shell <cmd>` pattern to run your particular
command `<cmd>`. This approach simply creates a shell session, executes your
command, and closes the session, while forwarding the stdout to your current
shell session.

# Uninstalling Bloatware

The Command Line Interface (CLI) tool used in Android for interaction with the
Android Package Manager is `pm`. This tool basically allows you to list,
install, or uninstall software packages.

To list the currently installed packages, execute the following command:

```
pm list packages
```

To view only the default apps on Android, enter this command:

```
pm list packages | grep 'android'
```

At this point, I would advice you to search for other unwanted pre-installed
apps known to come with your phone's brand as well as android packages which
are known to be bloatware. Take a list of these apps for future reference so
you can easily repeat this process if you need it later again.

To delete a specific app from your phone, you first need to identify its
package name. You can accomplish this by searching for it on the internet.
However if you can't still find it, you should be able to locate the app's "apk
package code" directly on your phone. This process can vary depending on the
phone you're using, so I suggest you look up how to find package codes for
applications on your specific phone model.

Also, be careful to not to delete anything critical for your system to work. Do
not delete a package if you are not sure that it is not something system
critical.

Once you get the package.name, you can just run the following command in the
adb shell:

```
pm uninstall --user 0 package.name
```

Here, `--user 0` specifies the user for which you want to uninstall the
package. User 0 is typically the device's default or primary user. When I have
run the command `pm uninstall` without this, it would say package is
successfully deleted but the package would still remain on my phone.

Keep in mind that we could have also used the following command:

```
adb shell pm uninstall --user 0 package.name
```

Or even:

```
adb uninstall --user 0 package.name
```

# Automating the Process

Remember that I told you to make a list of the packages you remove. This was
for to make the debloating process easier if you need to do it a second
time on your phone.

I use a specific [git
repository](https://github.com/kugurerdem/android-bloatwares) for this purpose.
In this repository, I have a file named `bloatwares.txt` that contains the
package names of certain prebuilt applications for Mi, Xiaomi, Android, and
third-party applications.

If I ever need to debloat my phone, or anyone else's, all I need to do is to
run the following command:

```
cat bloatwares.txt | xargs adb shell pm uninstall -k --user 0
```

# Conclusion

If you haven't already started, I strongly encourage you to consider debloating
your phone. Install `adb` on your computer, connect to your phone using it,
identify packages that seem unnecessary, and free your device from the unwanted
'guests.'

Go ahead and take back at least a partial ownership of your phone by getting
rid of these intruders!
