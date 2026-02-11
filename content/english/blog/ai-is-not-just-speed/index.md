---
title: AI Agents Are Not Necessarily About Speed
date: 2026-02-11
images:
  - ai-turtle.png
---

![](./ai-turtle.png#75persize)

I developed a web extension called Mark Scroll Positions about 1.5 years ago. I built it both to solve my own need and to experiment. If you are interested in how it looked and the original idea, you can check [Introducing: Mark Scroll Positions](https://rugu.dev/en/blog/mark-scroll-positions/). But long story short, other people started using it after I published it, even though I did not have high expectations for it. I received some comments on both [Firefox](https://addons.mozilla.org/en-US/firefox/addon/mark-scroll-positions/) and [Chrome](https://chromewebstore.google.com/detail/mark-scroll-positions/echejfhmdgnabmbihbmkdgeajmbojald). A few users even went to GitHub and opened [issues](https://github.com/kugurerdem/mark-scroll-positions/issues).

Most of the issues were simple problems that I could have fixed easily if I had taken the time. For example, adding a dark theme option or fixing the drag and drop feature that was not working properly in Firefox. However, I was a bit tired at that time and focused on other things, so I could not find the motivation to handle these requests. I left the repository as it was.

When I first built the extension, AI agents were not as good as they are now. I was also very skeptical about using them. I was afraid that if I used AI too much, I might lose my ability to write good code. I also worried that the code could become hard to maintain in the future, or that my own skills would get worse.

Recently, I spent a lot of time with my friend [Cem](https://blog.cemunalan.com.tr), and we tried different AI models and agents together. My perspective changed a lot. Seeing a good developer friend of mine working very efficiently with different agents in front of me was something I could not ignore.

So I also started using LLMs and different agents more and more in my workflow. At the same time, I tried to be careful about choosing the right tools and workflows, so it would become easier to put the LLM into a feedback loop. I also try to follow simple principles like making incremental changes. I review the changes made by the agents, so I still work with a human in the loop approach when using LLMs.

In terms of efficiency, for some tasks, AI might still be slower than an experienced developer who is fully focused and writing everything manually. But I think this way of thinking misses the main point. For many people, including me, the main benefit of using AI agents was never just about speed. It was about being able to offload tasks that you do not want to do, or do not have time to do, and focus more on the parts you actually care about. Developer velocity alone does not mean much if you get exhausted and stop. It does not matter if you start fast but then get stuck because of boring tasks and lose motivation. With AI, I feel that this mental barrier becomes lower. That is very valuable for me, even if it does not always increase speed, although in many cases it does.

So I decided to try this approach on the extension that I had already stopped developing, even though there were still users requesting some features. I thought it would be better to offload these tasks to an AI agent instead of not solving them at all.

Over three nights, I probably spent around 3 to 4 hours thinking about Mark Scroll Positions, and the LLM likely spent about the same. In the meantime, the repository changed significantly. It migrated from Parcel to Vite. It moved from raw CSS to Tailwind, because I thought AI would work more easily with it. I removed the SVG icons and started using the Font Awesome library. I modernized the whole UI. I took most of the initiative in the design because Claude did not implement things exactly how I wanted at first. I added dark and light theme support. I fixed the drag-and-drop problems in Firefox popups. I added a sort by latest changes feature and a settings page. I also fixed the broken auto jump feature, where the extension would open the site but not scroll to the correct position. If you are curious about the difference, you can watch the previous version in this [video](https://www.youtube.com/watch?v=BzbMsaQkt34) and explore the latest release on the [Firefox](https://addons.mozilla.org/en-US/firefox/addon/mark-scroll-positions/) or [Chrome](https://chromewebstore.google.com/detail/mark-scroll-positions/echejfhmdgnabmbihbmkdgeajmbojald) stores.

Are any of these tasks very hard or impossible for a human to implement? No. Would I have implemented them on my own? I am not sure I would have found the motivation and time if I was not using an AI agent. And the thing is, this was not the only case where I experienced this. I chose this example because it shows the situation I am trying to describe very clearly. It was a project I had already left behind, even though the required changes were not difficult. With agents, I was finally able to move it forward again.

In the end, speed was not the most important thing that LLMs gave me. It was mostly about reducing mental barriers. And from now on, I intend to use AI agents more and more in my upcoming projects.
