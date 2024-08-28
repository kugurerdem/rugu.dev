---
title: 'Introducing: Mark Scroll Positions'
draft: false
recommended: true
date: '2024-06-03'
updated: '2024-06-05'
tags:
    - technology
    - productivity
discussions:
    hackernews: https://news.ycombinator.com/item?id=40589852
---

Try the extension in
[Chrome](https://chromewebstore.google.com/detail/mark-scroll-positions/echejfhmdgnabmbihbmkdgeajmbojald)
or in
[Firefox](https://addons.mozilla.org/en-US/firefox/addon/mark-scroll-positions/)
\
See the source code
[kugurerdem/mark-scroll-positions](https://github.com/kugurerdem/mark-scroll-positions)

{{< youtube BzbMsaQkt34 >}}
---

I like reading blog posts a lot. While some of them are short and easy to read,
most of them are long and require more time to finish. When reading those long
essays, I take a break most of the time. Thus, when I re-open an essay, I
often  lose the original place where I was reading. And if I
can remember where I was, then I manually scroll back there. This makes the
reading experience less smooth and more time-consuming.

## A Not-So Clever Workaround (Fragmented Identifiers)

I have found a neat trick to work around this problem over time. I was already
using the bookmark feature of my Brave browser, but it was not tracking where I
left reading. So, I would proceed with the following procedure:

(1) Open the inspect mode. \
(2) Click an element that has an ID nearest to where I am at, get the element
ID. \
(3) Append the element ID to the URL of the site in the form of a fragmented
identifier. \
(4) Using the new URL with fragmented ID, either overwrite the existing
bookmark or create a new one.

If you don't know what fragmented identifiers are, they are the part of the URL
that follows the hash symbol (#). Consider the following URL:

{{< center >}}
https://rugu.dev/en/blog/intentional-digital-consumption/#maintaining-balance
{{</ center >}}

Here `#maintaining-balance` is the fragmented identifier and thanks to it the
browser directly knows where to jump when it opens the page.

Anyways, this approach works, but there are some problems with it. First of
all, it requires manual labor which could have been automatized, secondly,
although the fragmented links directly jump to the element with the id
specified in the URL, if there is no element with id close to where you are,
the method fails.

## Seemingly A Better Idea (Storing Scroll Positions)

So, I wanted a tool to save and jump to specific scroll positions on a webpage.

I found some programs that are built for this purpose, but none of them met my
expectations.

The most popular one I found was [Scrroll
In](https://chromewebstore.google.com/detail/scrroll-in/cjgjbjogfodppempgdlppgefojbcmjom),
and even that could be improved a lot: For example, it forces you to name each
saved scroll position with an alert prompt. Why not automatically give a random
name that can be renamed later? The fetch and save UI/UX is confusing—why not
directly show the saved scrolls and allow users to jump to them? Additionally,
it lacks features like adding notes to scrolls and searching through saved
scrolls.

So I decided to build my extension for storing/marking scroll positions on web
pages.

## Introducing: Mark Scroll Positions

Here is my extension built for that purpose, you can download it from
[here](https://chromewebstore.google.com/detail/mark-scroll-positions/echejfhmdgnabmbihbmkdgeajmbojald).

You can save your scroll positions and resume reading later with ease. You can
save as many scroll positions as you want, add notes, rename them, and see and
manage all your saved spots on a separate page.

### Implementation Details

I think there are 3 important aspects for understanding how this project works
under the hood.

#### 1) Interaction between the popup and content scripts

In modern browsers, the environment used by an extension's popup is separate
from the environment of the current tab the user is viewing (i.e., the HTML,
CSS, and JavaScript files of the webpage).

This separation isolates the extension from web content and prevents extensions
from directly accessing and modifying tab content. For this, extensions need
specific permissions like scripting and activeTab to interact between the
extension's popup window and content scripts.

In our case, when the user clicks the "Mark" button in the popup window, we
want to fetch the scroll position information from the active tab. However,
this can only be done in the content script environment. In such scenarios, you
can either create a content script that listens for events from the popup
(using `chrome.runtime.onMessage` and `chrome.runtime.sendMessage`) or inject
content scripts into the page environment (using
`chrome.scripting.executeScript`) when the user clicks the "Mark" button. I
chose the second approach as it seemed cleaner. The same applies to the "Jump"
utility.

This is basically to isolate the extension's environment from the web content
so that extensions cannot directly access and modify tab content. As a
result of this, extensions need specific permissions such as `scripting` and
`activeTab` permissions to make an interaction between the extension's
popup window and the content scripts.

#### 2) The data structure to be saved

If you want your application to be persistent and remember what the user has
done, you need to store data in a persistent form.

I chose to store the details like this:

```
{
    [absoluteURL]: {
        scrolls,
        title,
    }
}
```

So, the data is stored in `chrome.storage.local` consists of keys of absolute
URLs and values of data related to that page.

Each time a new scroll position is saved, the scrolls array is fetched and the
new scroll details are added to it. The same approach is used for deletion and
updates.

#### 3) Deciding on how to implement the jump functionality.

Deciding how to implement the jump functionality was challenging. I could have
simply saved the `window.pageYOffset` value when the user clicks "Mark" and
uses that value with `window.scrollTo(0, offset)` when the user clicks "Jump"
(like [Scrroll
In](https://chromewebstore.google.com/detail/scrroll-in/cjgjbjogfodppempgdlppgefojbcmjom)
does). However, this would fail if the user resized the page or if the author
changed font sizes. So, I decided to save enough information to recalculate the
target offset based on a percentage.

When the user clicks "Mark," I save not only `window.pageYOffset` but also
`window.innerHeight` and `document.body.scrollHeight`. Since
`window.pageYOffset + window.innerHeight` roughly equals
`document.body.scrollHeight` when the user scrolls to the bottom of the page,
we can adapt to page resizes with a normalization procedure when the user
clicks "Jump."

Is it that easy? Unfortunately, no. This method fails when the page gets longer
due to dynamic content updates (like new comments). In this case,
`document.body.scrollHeight` gets bigger, but the offset where the user left
off and should continue to read on doesn't actually change. Here, jumping
directly to the offset works better. You can still adjust the offset value in
comparison to `window.innerHeight`, also known as the viewport.

Currently, my extension uses the first method, but I might add a feature
allowing users to choose which jump method they prefer for certain pages.

## An Alternative Idea (Storing Uniquely Identifiable Text)

Another option is to mark pages based on uniquely identifiable text so the user
can jump to specific text. The problem with this is if the author changes the
page or content. Even changing one word can break the mark. In contrast, if you
save scroll positions, you will still land somewhere close to the initial text.

## Last Thoughts

I believe all these ideas can be improved to create a better marking
application. Maybe a combination of these methods could work, or there might be
even simpler concepts that I have missed.

The main problem is that pages can change, and it's unclear how our application
should adapt to these changes.

Anyway, I hope this application will be useful to some people. It will at least
be useful to me. If you want to contribute, please feel free to send your PRs
to
[kugurerdem/mark-scroll-positions](https://github.com/kugurerdem/mark-scroll-positions)

