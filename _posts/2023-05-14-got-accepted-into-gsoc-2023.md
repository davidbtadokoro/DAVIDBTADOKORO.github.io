---
layout: post
title: Got accepted into GSoC 2023!
date: 2023-05-14 21:40:00 -0300
categories: [gsoc23]
tags: [gsoc23, kw]
---

A little more than a week ago, I received an email saying that my Kworkflow project proposal to GSoC 2023
was accepted. For those that don't know, I'll take a step back and explain what is Kworkflow, the
GSoC 2023 program and what exactly entails being accepted into it.

## kw

The first thing is that the Kworkflow (or just kw) project, in simple terms, aims to be the best bud of the Linux kernel
developer. If you ever experimented with tinkering with the Linux kernel, you may know that many repetitive
(and dangerous) actions can be automated and that this "category of development" would welcome a unified ecosystem.
As the project's [website](https://kworkflow.org) states "_kw has a simple mission: reduce the environment and setup overhead of developing for Linux. kw is composed of different software components unified in a single interface_". In my opinion, the project is doing this
masterfully. 

For the past year or so, I've been continuously contributing to the project, so I get that my opinion
may be biased toward my community. But that is the second thing: open-source projects. kw introduced me to the notion of
communities built around a software project and how this approach is (there goes another biased opinion)
enormously advantageous when compared with other forms of writing collaborative software.

> I invite everyone to take a look at [kw's GitHub repo](https://github.com/kworkflow/kworkflow) and, who knows, maybe start contributing!
{: .prompt-tip }

## GSoC 2023

GSoC 2023 stands for "_Google Summer of Coding 2023_". As this name suggests, it is the 2023 edition of
something related to coding during the summer supported by Google. Jokes aside, it doesn't go far from that.
In my understanding, the program aims to connect open-source mentor organizations with contributors
where the latter proposes projects to the former, and the accepted ones are developed during the summer
(in the northern hemisphere) in 12-22 weeks. It is a great initiative (although there is nothing "initial" to it,
as the first edition was in 2005) that no doubt results in tons of great experiences (and software, of course).

## My project

In the context of kw's mission, there is a missing link in the Linux kernel developer workflow that kw
doesn't cover. The Linux kernel is collaboratively developed using mailing lists. If you want to
propose changes to the kernel, there comes a time when you have to send your patches in the form of emails to
the mailing lists and interact with these to collect feedback, make adjustments, and so forth. If you are on
the other end, i.e., you are the one reviewing the sent patches in the lists, you also have to interact
with those lists. Either way, even if you are just a curious person wanting to peek into the discussions
of developers from the whole world, you still have to interact with the public mailing lists.

For the sending part, kw already has a wonderful feature called `kw mail` that makes the whole process
of converting a set of commits into patches and sending them to the right mailing list just a single
command. However, for the interaction part, kw has a simple prototype feature named `kw upstream-patches-ui`
that is a long way from being ready.

My project aims to fully develop `kw upstream-patches-ui` and refine it to allow a kernel
developer to just rely on kw and a text editor for his/her workflow. TLDR, my goal is to integrate kw with
the [lore.kernel.org](https://lore.kernel.org) archives and provide a much more friendly interface with
the public mailing lists. At first the UI will be terminal based using `dialog`. However by using the
Model-View-Controller pattern, adding other UIs (like a web one) will not be so complicated.

## Conclusion

I'm happy for being accepted into GSoC 2023, even more, because I'll be contributing to kw
that really "feels like home". I can't predict the future but I truly believe that, if everything
goes well, the kw project will have a marvelous addition in about 12-22 weeks.
