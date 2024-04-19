---
title: 'Incorporating AI Tools Into My Terminal Workflow'
archive: true
date: 2023-12-18
updated: 2024-01-15
discussions:
    hackernews: 'https://news.ycombinator.com/item?id=38689075'
tags:
    - technology
    - productivity
---

For those who may not be aware, Neovim is to me what a lightsaber is to a Jedi.
It forms an essential part of my routine, as I use it for nearly all my tasks
involving text. Be it drafting an essay, sending an email, or coding, Neovim is
my go-to tool.

Moreover, I have a deep admiration for the UNIX philosophy and its command-line
interface programs. It's quite fascinating to observe how these small,
uncomplicated UNIX programs, designed to do one thing flawlessly, interact
effectively using piping mechanisms. Tools like sed, grep, awk, count, cut, and
many others, often prove to be incredibly useful for text processing.

I can confidently state that both Neovim and UNIX have proven themselves
invaluable in my work.

However, like many others, I have been introduced to another set of efficient
tools for dealing with text, known as Large Language Models or LLMs. I've spent
several months experimenting with tools like Co-pilot and ChatGPT, and I've
found them to be highly beneficial for text-based tasks.

Naturally, I wanted to utilize the true potential of all these tools in my
interactions including text. For this reason, I began searching for Neovim plugins
and command-line interface programs capable of integrating these AI tools.

The process of integrating Co-pilot was relatively simple thanks to a
[plugin](https://github.com/github/copilot.vim) available on Neovim.

However, incorporating ChatGPT into my workflow wasn't as straightforward as I
had hoped. I looked into several neovim plugins, like
[ChatGPT.nvim](https://github.com/jackMort/ChatGPT.nvim), which allow
interaction with ChatGPT through Neovim. However, the majority of these plugins
seemed like an overkill compared to what I expect from them. They also had many
features designed to simplify the programming process, a job that Co-pilot
already handles for me. Additionally, I would want llms to be accessible not
only in vim but also within my regular terminal environment. I would appreciate
it as a command-line interface tool, which would enable piping, giving
arguments, and flags for more complicated tasks. Unlike Co-pilot, I would like
to use a tool like ChatGPT in a more widespread context.

Hanging around Twitter, I recently saw a post from
[Gary](https://twitter.com/garybernhardt/status/1735090271690637803), giving a
credit to Simon Willison's library, [llm](https://github.com/simonw/llm). I was
surprised to find out that this library was exactly what I was looking for as
well. It was a command-line interface tool that allowed me to interact with
LLMs through my terminal, which is exactly what I wanted. I could pipe the
output of any command into llm, and it would return the result of the input.
For instance, I could pipe the output of a command like `cat` into `llm`, and
it would return a response from the AI model, which you could pipe or redirect
into another command or file.

## Examples

Here are some of the examples that comes to my mind on how you could use the
``llm`` tool:

```bash
git diff | llm 'Recommend 5 different commit messages for these change'
```

```bash
cat essay.txt | llm 'Summarize what these are about'
```

Furthermore, if you keep finding yourself using the same prompts over and over
again, you can create templates for them.

```bash
# Create a template for finding synonyms of a word
llm --system 'What are the synonyms of the following prompt' --save synonyms

# Create a template for rephrasing text
llm --system 'Fix grammar mistakes and rephrase the text' --save rephrase

# Create a template for finding titles for given content
llm --system 'Recommend 5 titles for the following prompt' --save titleize
```

You can later use these templates by passing the ``-t`` flag to the command.

```bash
# Rephrase the text which are copiod in your clipboard
xsel -b | llm -t rephrase

# Find synonyms of the word 'serenity'
echo 'serenity' | llm -t synonyms

# Find appropriate titles for your document
cat vi-llm.md | llm -t titleize
```

You can also further specify system messages, choose language model you want to
interact with, and many more things, which you can examine on the
[documentation](https://llm.datasette.io/en/stable/help.html) of the app.

## The Readline Issue

So, I was quite happy for finding this tool, but the only thing I did not like
about it was that when you try to use chat mode with ``llm chat`` command, the
readline would break my initial GNU readline settings defined in my
`~/.inputrc` file.

When delved into the source code of the repo, I have seen that other people
have been encountering the same
[issue](https://github.com/simonw/llm/issues/376).

I figured out that the issue is likely caused by the readline libraries used to
build the ``llm chat`` command overriding the default readline settings.
Because I'm not very familiar with these Python libraries, I decided not to try
fixing the issue by changing the source code. Instead, I have decided to use
the ``rlwrap`` command to address this problem. Basically, ``rlwrap`` is a
program that allows you the wrap the readline of the programs that you run so
that you can still use the application's readline as it was respecting your
shell's readline settings.

I know that by the time you, the reader, come across this, the issue may
already be fixed. However, the purpose of this piece is not just to provide a
solution to this particular problem, but to share how I approached solving it
and what I learned from the experience.

## The vi-llm Wrapper

Anyways, the problem with the ``rlwrap`` solution was that, yeah, it allowed me
to use my shell's readline settings, so I could use vi keybindings when giving
prompts, but I still could not copy, highlight, and modify the answers that are
given to me, or the previous prompts that I have give. For this, I have built a
shell script called [vi-llm](https://github.com/kugurerdem/vi-llm) based around
one of my favorite unix utils [vipe](https://joeyh.name/code/moreutils/), and
llm. ``vi-llm`` is basically a wrapper for llm that gets all of its prompts
from Vim, enabling an interactive communication with ChatGPT using llm, by
letting you input a message through the vim editor, then sending that message
to the LLM interface. subsequently displaying any logs received from the
interface right back in your text editor, repeatedly, until the user quitting
the vim editor without doing any changes. In essence, it operates similarly to
a chat interface. You type in messages (or commands) which get sent to the LLM
system, and any response from the LLM system gets displayed back to you. This
cycle continues, enabling continuous, interactive communication with the LLM
from your command line.

Here is a quick showcase:

![vi-llm-showcase](https://raw.githubusercontent.com/kugurerdem/vi-llm/master/showcase.gif#center)

If you are interested, you can check out the [github repo.](https://github.com/kugurerdem/vi-llm)

## Conclusion

I've been using Copilot and ChatGPT for months and now I am using this tool for
a few days now. Now I have all the tools that I have needed in order to utilize
my workflow even more, a strong autocompletion tool such as Copilot, a program
that allows me to interact with large language models through shell: llm, and
finally, a wrapper vi-llm based around llm for my personal use case.

I hope that this essay was helpful or at least interesting to some of you.
