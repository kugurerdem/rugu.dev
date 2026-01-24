---
title: 'Introducing: HackerNews Domain Stats'
date: 2025-05-17
images:
  - https://raw.githubusercontent.com/kugurerdem/hn-domain-stats/refs/heads/master/overview.png
---

You can quickly try the app through [hnstats.rugu.dev](https://hnstats.rugu.dev) \
Or, see the source code [kugurerdem/hn-domain-stats](https://github.com/kugurerdem/hn-domain-stats)

---

I like hanging out on Hacker News, reading essays. I also enjoy submitting essays I like that haven’t already been shared there. This way, the author gets recognition and others get to read something they might enjoy too. Plus, more karma never hurts. 😀

For that, I've been using an extension called [What HN Says](https://github.com/pinoceniccola/what-hn-says-webext). The extension basically lets you see whether an HN discussion already exists for the page you're viewing. If it does, it lists them and lets you open them in a new tab; If not, it makes submitting the link easy by opening the HN submission page with the title and URL prefilled. Despite being a simple app, I've been using it for a while and enjoying it.

Recently, I thought the same Algolia API that this extension uses could also be used for constructing additional analytics related to a specific domains. These analytics could be anything like: The total number of upvotes and comments the domain has received so far, how much time has passed since its first submission, how many unique users have submitted links from it, a graph of submission, upvote, and comment counts over time, and so on.

I'd certainly be interested in seeing HN analytics for my own site from time to time. I looked out whether such an app already exists or not, but after not finding one, I thought why not develop one?

So, here, I introduce [HN Domain Stats](https://hnstats.rugu.dev). It is a very simple application that analyzes and visualizes how domains perform on Hacker News over time.

Here's what the app looks like after analyzing a domain (in this case, rugu.dev):

![Overview](https://raw.githubusercontent.com/kugurerdem/hn-domain-stats/refs/heads/master/overview.png)

If there are any features you'd like to see, feel free to reach me out through [issues](https://github.com/kugurerdem/hn-domain-stats) or e-mail.

Also, I would appreciate any feedback if you're willing to share one.
