---
title: 'Things learned from using Plain Text'
date: '2024-10-09'
---

When you interact with a text file using an editor, what you see doesn't necessarily reflect the actual data stored in the file. Sure, the contents of plain text files are byte codes encoded in formats like ASCII, UTF8, or UTF16, and these byte codes are the ultimate source of truth. But in the end, it's still your text editor that chooses how to interpret and represent that ultimate source of truth - binary codes into something recognizable to you. **This means that two different files could look the same, or the same file might appear differently depending on the editor(s) you use.**

Your text editor might highlight (or not) certain parts based on syntax it recognizes, it can control how tabs appear (2 spaces, 4 spaces, or even 8). It decides how to encode the tab key input, whether as `\t` or as a set number of spaces. The same applies when you press the enter key to create a new line—whether it gets encoded as `\n` (UNIX), or `\r\n` (Windows) depends on the editor's configuration.

Your text editor hides details so you don’t have to think too much. However, there are many times when these details leaks through the protection layer that your text editor tries provide. And you often don't notice these complexities until you face them.

So, the main goal of this essay is to share some of my experiences and what I've learned over time about common problems you might encounter with plain text.

## Tabs vs Spaces

Historically speaking, spaces existed long before tabs. In fact, the [reason](https://en.wikipedia.org/wiki/Tab_key) why tabs were initially developed was to reduce the repetitive use of both the space bar and backspace key.

However, people still debate whether to use tabs or spaces in their projects. **This isn't about whether we should use the tab key itself, but rather whether text editors should insert spaces or tabs when we press it.** The debate is about how to encode the blank spaces we see in our editor.

I guess the main advantage of tabs over spaces is the flexibility it provides. When using tabs, you can basically collaborate with many different people prefer to see different levels of indentation, without having to expose their preferences over others. (Sure, there is also the fact that tab character taking less space, but personally I don't think this makes much of a difference especially considering that we are in 2024.)

One problem with tabs, though, is precise editing. Since tabs represent multiple spaces at once, aligning lines may not work as one expects from time to time.

Like, what you perfectly see as OK in one editor where tabs are configured to take 4 columns, can be disgusting in another editor where tabs are configured to take 2 columns:

Editor 1 with tabs configured to take 4 spaces:
```javascript
// 4 tabs, + 3 spaces
function calculate(a, b,
                   c, d) {
    // Some logic
}
```

Editor 2 with tabs configured to take 2 spaces:
```javascript
// 4 tabs, + 3 spaces
function calculate(a, b,
           c, d) {
    // Some logic
}
```

This is also the partly the reason why I personally use spaces most of the time. If you still can end up adjusting the tab width to match others' preferences, what's the purpose of using tabs in the first place?

I don't know whether this is just due to [first-mover advantage](https://en.wikipedia.org/wiki/First-mover_advantage) or not but it also looks like more projects uses [spaces over tabs](https://insanelab.com/blog/notes/spaces-vs-tabs/). So what's the point of going against the tide where there does not seem to be a very powerful advantages anyways?

With all that said, I still believe that in many cases this conversation is somewhat overkill and often doesn't make a significant practical difference. In the end, what truly matters is whether the codebase is consistent—either using tabs or spaces throughout. Aside from that, it shouldn't matter much since these settings can be easily configured in many environments anyways.

## Soft Wrapping vs Hard Wrapping

When using plain text, there will come a point when the text you write becomes too long. In many text editors (Notepad, Notepad++, NeoVim, or even VSCode), the default behavior is for the text to continue growing horizontally until you press Enter to create a new line break. This can be somewhat unuser-friendly compared to most email or messaging clients, where the text automatically wraps, making it much easier to read.

To be more clear, let me show you how non-wrapped and wrapped texts look like.

---
The text that is not wrapped:
```
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Praesent fermentum felis nec elit bibendum, sit amet tempus sapien volutpat. Sed eu congue massa, non condimentum diam. Aenean vel consectetur odio. Suspendisse nec diam ac nisl bibendum tempor. Ut rutrum maximus velit, commodo consectetur nulla auctor ac. Curabitur neque dui, scelerisque in facilisis at, sagittis quis massa. Ut non tempor arcu. Vivamus elit massa, pulvinar vitae tellus ut, lobortis sagittis elit. Aenean vehicula varius eros, vitae pellentesque lorem consequat non. Aenean gravida velit id pellentesque tempor. Aenean ut purus nulla. Curabitur fringilla felis consequat ante condimentum porta. Curabitur id ex in libero rhoncus lobortis sit amet ac ligula.
```

The text that is wrapped:
```
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Praesent fermentum
felis nec elit bibendum, sit amet tempus sapien volutpat. Sed eu congue massa,
non condimentum diam. Aenean vel consectetur odio. Suspendisse nec diam ac nisl
bibendum tempor. Ut rutrum maximus velit, commodo consectetur nulla auctor ac.
Curabitur neque dui, scelerisque in facilisis at, sagittis quis massa. Ut non
tempor arcu. Vivamus elit massa, pulvinar vitae tellus ut, lobortis sagittis
elit. Aenean vehicula varius eros, vitae pellentesque lorem consequat non.
Aenean gravida velit id pellentesque tempor. Aenean ut purus nulla. Curabitur
fringilla felis consequat ante condimentum porta. Curabitur id ex in libero
rhoncus lobortis sit amet ac ligula.
```
---

As you can see, the wrapped text is a lot easier to read compared to the unwrapped one. This is why many people follow the practice of inserting newlines after a certain number of characters is reached (often around 78). In fact, in the text editor I use, NeoVim, it is as simple as to just press `gq` for this line wrapping procedure to happen for the current line you are in.

This procedure of putting actual newlines between text so that it looks wrapped is known as hard-wrapping. This was what I did for a while as well, but there are problems with this approach as well. Since you are actually wrapping a line into multiple lines just because you can read it more easily in the text-editor, you now have the following problems:

- It's frustrating to constantly hard-wrap the text manually, even if you have a shortcut for it.

- **If you copy the wrapped text this way and paste it into another application, like your favorite messaging app, it will likely look unpleasant, especially on smaller devices like smartphones.** This happens because when the screen size is reduced, the messaging app will make sure each line to be wrapped again if they don't fit the vertical window size. I recommend reading [The problems with hard wrapping email body text](https://www.arp242.net/email-wrapping.html) to understand this phenomena a bit better.

- **Once you start adding actual newlines into your text just to make it more readable, you are sacrificing greppability.** You can think of the term "greppability" as a way to describe how well text is suited for efficient searching through tools like grep. In the case of hard-wrapped text, if you search a sentence that is split across multiple lines, the search/find tools like grep might just fail since they usually operate on a line-by-line basis.
\
\
    In fact, this idea of greppability can be so important and time-saving even when you are working on a code-base. For example, in general it is not very optimal to split your error message or log messages into multiple lines as you lose greppability. See [Greppability is an underrated code metric](https://morizbuesing.com/blog/greppability-code-metric/), if you are further interested.

So, what's the solution? **Stop hard-wrapping and just use soft-wrapping**, many modern text-editor's already provide these kind of wrapping solutions. This is what I have started to recently. In NeoVim it is as simple as to use these commands, in other "modern" editors, I am sure there are ways to achieve it through some UI stuff as well.

```
:set linebreak
:set wrap
:set columns=80
```

## Newline

Let's say that you are collaborating with other people through Git, and you just changed one line of code in a file and then pushed it to a branch. And suddenly, your colleagues started complaining about how your commit diff looks like you've changed the entire file.

In this case, if you are not sure what's going on, you might find yourself spending some time just to realize that it is your text editor that automatically converted all the line break characters to its own format.

In fact, I had few friends who use VSCode as their main text editor and experienced the same issue. They were very confused the first time they encountered this kind of an issue. Luckily, I was there to help them understand what's going on. :)

I’m going to take a step back in time, but please bear with me for a bit.

A long time ago, before monitors were common, people interacted with computers using these cool devices known as [teleprinters](https://en.wikipedia.org/wiki/Teleprinter). These were devices that combined a keyboard for input and a printer for output. You would basically type stuff, end when you send your commands to computers, you would get the results just like this:

{{< youtube AwqryPuwl_w >}}

Now, in the context of a teleprinter, what "newline" meant was something like: first physically move the typewriter's carriage back to the beginning of the line (CR: Carriage Return) and then move the paper up by one line (LF: Line Feed)

For these early machines designed to work with teletypes, both CR and LF were required to begin typing on a new line. And this influenced the early days of of both Unix (1971) and MS-DOS (1981) as well. Back then, teletypes were still in use, so it kindof had to influence both operating systems in various ways. It turns out that, Unix developers chose to simplify the process of creating new lines by treating the LF character alone to signify the end of a line, while MS-DOS retained the full CR + LF combination. I guess Microsoft guys were more concerned about legacy and hardware compatibility.

And all these conventions continue to exist even today. Files created in Windows use `\r\n` for newlines and those generated in UNIX-like systems (Linux/MacOS/BSD) use `\n`. However, many modern text editors are capable of reading and parsing both formats across both operating systems anyways so we don't even notice these small differences. Unless we mess up our git diffs :D

**To avoid these situations, you can configure your text editor correctly, use a linter, and create a `.git/config` file with `autocrlf` set to your need.**
