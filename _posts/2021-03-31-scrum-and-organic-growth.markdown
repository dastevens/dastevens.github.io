---
layout: post
title:  "Scrum and organic growth"
date:   2021-03-31
categories: agile coding design scrum software
---

<p align="center">
  <img src="/assets/images/header-640.131.2.png" width="100%">
</p>

The [agile manifesto](https://agilemanifesto.org/) has been around for twenty years now, and probably the most frequently used approach is using [scrum](https://en.wikipedia.org/wiki/Scrum_(software_development)). It’s the best project management method for software development we have. But there are some pitfalls, and this article is about one of them.

Scrum is great for flexibility, concentrating on business requirements, delivering new features as soon as they are complete, and quickly adjusting when business requirements change. It is driven by business requirements as set out by the product owner and it puts delivering value at the heart of the software development process. Find out what the user needs to be more productive and concentrate on delivering that.

What it completely misses is the long-term cost of making decisions driven by short-term gain. If left to run wild, technical debt can be the unwanted side effect. There is no inherent architectural design and there is no short-term incentive to keep a tidy code-base. And of course none of these components of a healthy software project is visible to the product owner who is running the show.

Does this scenario sound familiar to you: pressure is on you to deliver new features without adjusting the underlying architecture. After a while the code starts getting entangled because developers are encouraged to take the shortest path, and you only realize when your technical debt starts to bite. This can lead to a down-turn in productivity, initially features were being shipped regularly to production, but after a while new features become harder to add. In extreme cases, you might even cut your losses and choose to rewrite, with all the risks of failure that restarting from scratch entails.

So what do I mean by organic growth? I have this mental image of code being like a bunch of plants in a garden. The flowers are the features, while the stems and roots are all the support code needed to support them. The scrum method focuses on features; it nurtures the flowers. But the supporting structures are neglected, they are just not considered valuable. The stems and roots are left to multiply in whatever way makes the most flowers.

Scrum can lead to a disorganized jungle of tangled shoots, whereas we as software developers would prefer to work in a meticulously-maintained French ornate garden. It is a constant battle against entropy, there is an energy barrier to be overcome to relax the code back into its lowest energy state.

So what can we do? Empower developers to keep their backyard tidy. Include them in the scrum preparation process and ask us when time is required for refactoring work and take account of it in the sprint plan.

If no-one is keeping on top of code quality, it will quickly get out of control.
