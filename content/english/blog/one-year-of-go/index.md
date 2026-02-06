---
title: One Year of Using Go
date: 2026-02-06
images:
  - gophers-crochet.jpeg
---

It has been about a year since I decided to learn Go, and more than half a year since I started working at an HFT company that uses it. This is not a very long time with the language, but some of my developer friends have already asked me about my impressions of Go.

In this blog post, I want to share my overall experience with it. I will first explain why I felt the need to switch to Go in the first place. Then, I will describe what I did to learn it and how I improved over time. Finally, I will discuss my current impressions of Go, including what I like, what I do not like, and whether I would recommend it to others based on their goals.

{{< center >}}
![](./gophers-crochet.jpeg#75persize)
(Two Gopher amigurumis stitched by my aunt, using patterns from [here](https://github.com/Acyony/gopher-pattern))
{{</ center >}}

# Looking for an Alternative to JavaScript

To give some context, until very recently, I was using JavaScript for most of the paid full stack projects I worked on. This was largely due to its high development velocity, mainstream adoption, and large ecosystem. On the frontend, I was primarily building on top of the React ecosystem, and on the backend, I was using Fastify. Overall, JavaScript worked well for those projects. There were only a few cases where it was not sufficient due to higher performance requirements. Even then, I was able to work around those situations by offloading the heavy parts to services written in faster languages like C++.

**So, if JavaScript is good enough for most projects I worked on, why did I feel the need to try another language?**

Well, I enjoy trying out new things. I also felt it would be valuable to have a programming language in my toolbox that expands the set of problems I can handle. I find functional programming interesting, and I like the mental model it provides. So, I could have chosen to invest more time in Clojure or Haskell to deepen my existing understanding of them. I could also have spent more time with Python; its ecosystem is very mature in areas like data science and machine learning. Trying frameworks like Ruby on Rails or Laravel was also appealing, given how productive their ecosystems can be.

**But the problem with all of these options was that anything I could build with them, I could already build with JavaScript.** I felt that the marginal gains from spending more time learning another high level programming language would be low.

Because of this, I wanted to try something that is closer to the machine than JavaScript. I believed this would help me better understand how things work under the hood. It would also prepare me for situations where performance matters more.

# Why Go and Not Others (Rust, Zig, etc.)?

You might be thinking, "All of this explains why you wanted something closer to the machine than JavaScript, but it does not explain why you chose Go over Rust, Zig, C, C++, or something else." That is fair. The answer is simple. Being closer to the machine and having better performance was important, but it was not my only priority. I was also thinking about developer velocity, how long it would take to learn the language well enough to build useful things, how widely adopted it is in the industry, and the overall philosophy behind it.

**Rust was tempting.** It is very fast and widely used. It is also interesting because it managed to solve a hard problem in a practical way: achieving memory safety without using a garbage collector. Still, I was hesitant because of its learning curve. I was also not sure if its philosophy and style would fit the way I want to work. I had a tendency to build things on top of abstractions, even when they are not needed. This sometimes lead me to spend time solving problems that are not a real priority, instead of focusing on problems that actually add value (see Why I am Leaving NixOS After a Year). Since I realized this, **I started trying to keep things as simple as possible on a cognitive level. Rust felt like a language that encourages abstraction, not just because it allows it, but because many of its core ideas depend on it.** That was another reason why I was hesitant about it.

**Zig was also tempting.** I was already following Andrew Kelley and his blog, and I have a friend, [Halil](https://github.com/nikneym), who liked both Zig and Go and introduced me to Go in the first place. He often described Zig as a more modern and slightly opinionated version of C. He also mentioned that many Go developers tend to like Zig, and many Zig developers tend to like Go, often pointing to people like [Mitchell Hashimoto](https://github.com/mitchellh) and
[Tim Culverhouse](https://github.com/rockorager). Zig was clearly very interesting. However, once **I realized that it was still under active development, without a stable release, and that its ecosystem was not as mature as Rust or Go, I felt it might be too much for me at that moment.** I was also not sure if I wanted to go as low level as Zig or C right away, before having something more suitable for most mid tier tasks.

As for C++, it was the language we used in our data structures and algorithms courses in college. I never developed a good feeling for it. It was more of a language I would use only if I really had to.

This left Go as the option that made the most sense to me at that moment. It was much more performant than JavaScript, while still offering good developer velocity, at least according to people already using it. It had a large ecosystem and placed a strong emphasis on simplicity. Because of this focus, the syntax was clear and it was easy to pick up quickly.

> Clear is better than clever.
>
> \- Rob Pike, Co-inventor of Go (from [Gopherfest 2015](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=875s))

**None of this means that I am against using Rust, Zig, C++, or other languages. I can easily see myself using any of them in the right context. At the time, I was simply looking for a language that matched my goals and constraints better, and Go fit those needs best.**

# The Learning Process

I learn best by both reading and getting my hands dirty. So, I decided to learn the Go syntax and core concepts just enough to start building things. Then, while experimenting and trying things out, I would continue reading more material to better understand what I was doing right, what I was doing wrong, and why.

To get started as quickly as possible, I followed the [Tour of Go](https://go.dev/tour/welcome/1) and also read [Introducing Go](https://www.oreilly.com/library/view/introducing-go/9781491941997/) by Caleb Doxsey. Both resources were short yet sufficient to help me understand the syntax and get a general sense of what Go is capable of. Right after finishing them, I was ready to move on to actually building something.

I had been thinking about building my own fitness application for a while. Most existing apps either felt like overkill for what I needed or were missing features I actually wanted. I also saw it as a good opportunity to build the API in Go and get more comfortable with the language. I looked into different API libraries and ended up choosing the [go-chi](https://github.com/go-chi/chi) router. It stood out as one of the simplest options, both in terms of staying close to the standard library and having a small codebase of under a thousand lines. Using a minimal library felt like a good way to learn Go better and focus more on the language itself.

While building my fitness app, I also kept reading books such as [Learning Go: An Idiomatic Approach to Real World Go Programming](https://www.oreilly.com/library/view/learning-go-2nd/9781098139285/) by Jon Bodner and [100 Go Mistakes and How To Avoid Them](https://www.oreilly.com/library/view/100-go-mistakes/9781617299599/) by Teiva Harsanyi. I really liked both of them, and I often went back to refactor my codebase as I learned from my mistakes.

While I was learning Go at my own pace and building projects with it, I got a job from an HFT company that was using Go. They gave me a case study to complete in Go, and I was able to do it. I also passed the technical interview and started working as a full-time Go developer. **This is just my impression, but I think Go makes it easier for companies to hire developers from other language backgrounds. If someone is already comfortable with a C-like programming language, their skills tend to carry over to Go quickly. My guess is that this is because Go keeps the language small and predictable, which makes it easy to read and reason about, even for people coming from languages with more complex syntax.**

> The key point here is our programmers are Googlers, they’re not researchers. They’re typically, fairly young, fresh out of school, probably learned Java, maybe learned C or C++, probably learned Python. They’re not capable of understanding a brilliant language but we want to use them to build good software. So, the language that we give them has to be easy for them to understand and easy to adopt.
>
> \- Rob Pike (mentioned by someone in [HackerNews](https://news.ycombinator.com/item?id=18564643]))

With working in an HFT company, I also started to pay more attention to the performance side of things. To that end, I read the book [Efficient Go](https://www.oreilly.com/library/view/efficient-go/9781098105709/) by Bartlomiej Plotka. Overall, I can say that this book really provided a lot of perspective that could be quite useful both for Go and other languages.

# My Overall Impressions on Go

Ok, so far so good. But what about my overall impressions?

Well, I think Go actually provides the experience that it promises. It is simple (not to be confused with easy). It has a very minimal and easy to understand syntax. The language itself kind of feels like a subset of C, but it feels more comfortable because you do not have to think about many things that you need to handle manually in C. It provides great primitives for handling concurrency. It has a great standard library. It is very readable, as a result of being simple and not having many ways to do the same thing. Because of all this, it provides a great developer experience, as long as you are ok with not trying to be too smart with fancy abstractions. In fact, I did not feel much of a decline in my developer velocity since I started using it. To the contrary, I feel like I am moving at a much faster pace when handling things related to backend work and concurrency.

> Simplicity is better than complexity because simpler things are easier to understand, easier to build, easier to debug and easier to maintain.
>
> \- Rob Pike (from [gotocon.com](https://gotocon.com/aarhus-2010/newsrobpike/1))

This is interesting because, in a way, you would expect a programming language with more abstractions, like Node.js, to obviously have an edge when it comes to developer experience, since the point of abstractions is usually to make things much easier. So how can a programming language with a very minimal syntax (that is not as expressive as others), fewer fancy features, and a very conservative way of doing things actually make you move faster?

I think there are many reasons for this, but the most likely one that comes to my mind is that **Go kind of trades expressivity for understandability (and readability), and understanding is usually the main bottleneck when working with others.** Maybe you do not have many ways to write the same thing, and maybe this can reduce your developer velocity in some cases, but it also makes working with other people much easier. It makes understanding code written by others easier as well, and it removes time spent on secondary things, like debating how code should be formatted. (Go has its own formatting tools provided by default.) **I feel like, as a result of this, it is also easier to work with LLMs in Go than in other languages that provide more hacky ways of solving the same problems.** I find it much easier to work with LLMs in Go, because it is easier for me to understand what is going on compared to other languages like Node.js. And since the understanding part of coding is more expensive than the writing part, I feel like some of the things that look like boilerplate in Go (for example, errors being values, and error handling causing a lot of boilerplate code) are actually its strength.

I think Go's standard library being really good also has a lot to do with its developer experience. **The standard library in Go is not cutting edge, but it is very strong and large.** You can almost find anything you need in it. It even has its own string and HTML templating engines, provides standard database interfaces (that many external libraries also build on), includes built-in JSON, XML, and CSV support, a full-featured HTTP server and client, cryptography primitives, context propagation, and many other things you would normally expect from external libraries. Also, **most external libraries also try to be compatible with it.** As a side effect of this, it becomes easier to read and understand code.

Again, **the standards are strong** in Go; you spend less time understanding different approaches to the same problem and more time focusing on the actual logic that matters. It even comes with its own testing, formatting, and documentation tools. This means you can go pretty far using only the standard tools that come with Go itself.

And lastly, I really loved the [Communicating Sequential Processes](https://en.wikipedia.org/wiki/Communicating_sequential_processes) way of thinking when it comes to building concurrent systems in Golang. Maybe there is a slight overhead when using channels, but overall, the mental model it provides makes it very easy and intuitive to design the interacting parts of your systems. I really like how channels work. I love goroutines. I love the concurrency patterns you can apply with them. I love the concurrency tools available in the standard library (I also suggest [Gist of Concurrency](https://antonz.org/go-concurrency/) by Anton Zhinayov for those who are further interested in the topic), and so on.

**Overall, I am glad that I learned Golang. The learning cost was not high, and when you consider what you get for the time you spend on it, it is one of the few technical learnings that easily justified itself.**

Is the language itself perfect? Probably not. I still don't like the lack of non-nilable types. I also think it would have been better if errors were handled through things like type matching and union types instead of multiple return values. I would rather prefer the language to enforce explicit initialization instead of silently filling everything with zero values by default. **But if we are talking about the best language I have used so far, rather than the best language in an ideal world, Go is the way to go.** So far, it was the most balanced language that I used in terms of being performant, simple and providing good developer velocity.
