---
layout: post
title: Starting collaborative work in kw patch-hub
date: 2024-08-14 15:00:00 -0300
categories: [rust dev, software engeneering, lore]
tags: [rust, kw, kw patch-hub, lore, free software]
---

In my [last post]({% post_url 2024-07-28-how-the-new-kw-patch-hub-is-shaping-up
%}), I talked about how the rebuilding of `kw patch-hub` was going. With the
start of the second semester in my post-graduation and the enrollment in the
_Advanced Topics on Software Engeneering_ subject, I came in contact with
three great developers who will help me in this rebuilding effort. This marks
the start of the new `kw patch-hub` as a dedicated Free Software project, and
this post is meant to talk about how this exciting collaborative work started.

<br>

## **kw patch-hub as a dedicated Free Software project**

As a product of my [GSoC23 project]({% post_url 2023-08-26-gsoc23-final-report
%}), I've developed a prototype of `kw patch-hub`, a _Terminal User Interface_
to [lore.kernel.org](https://lore.kernel.org/), aiming to streamline some
workflows of Linux kernel developers (especially the maintainers).

This TUI app was implemented just like any other `kw` feature, which (in simple
terms) means that:

1. Its implementation resides in the `kw` [main project
codebase](https://github.com/kworkflow/kworkflow).
2. It was developed entirely in Bash script.

`patch-hub` was a feature that significantly diverged from the other features
from `kw`, as it was a screen-driven feature with a much broader scope. So,
point number one became an issue because the feature started growing
uncontrollably and in dissonance with the rest of the codebase.

Point number two represented another dead-end but in the sense of the
performance of Bash script for the tasks entailed in the app. Even with much
time spent on optimization, the prototype had an unacceptable performance.

A few weeks before, during the weekly meeting of `kw`, we decided to create a
[dedicated project](https://github.com/kworkflow/patch-hub) for the new
`patch-hub` (which I explain in detail in [this post]({% post_url
2024-07-28-how-the-new-kw-patch-hub-is-shaping-up %})) inside the kworkflow
organization.

## **Advanced Topics on Software Engeneering**

To quickly introduce the context in which this collaborative work sprouted, I am
coursing a discipline called _Advanced Topics on Software Engeneering_, which,
in a nutshell, is intended for students experienced in software engineering and
free software.

Every student/group is supposed to interact with a free software community and
foster collaborative work in the ecosystem. What I mean by this mouthful is
that, in my group's case, we are kickstarting a new free software project.

This free software project is the new `kw patch-hub`, and although I used the
word "kickstarted", it inherits much of the processes and practices of its
"parent" project, the original `kw`. This, coupled with my experience in
maintaining `kw`, means we won't waste much time assessing how to bootstrap the
project and focus more on building it.

## **The Dream Team**

My group is composed of four integrants:

- **davidbtadokoro** (me)
- **OJarrisonn**
- **th-duvanel**
- **lorenzoberts**

I've omitted their real names to preserve their privacy and only used their
GitHub nicknames.

I know my three colleagues from past interactions, and I can confidently say
that their technical and (most importantly) collaborative skills are more than
adequate for our task.

## **Heating-up this community**

Besides my work on the project, which has been continuous since the start of
July, all three colleagues have already produced meaningful contributions,
considering that we had our first conversation just the week before.

**OJarrisonn** had two merged PRs:

1. [Use clap for Cli arg parsing](https://github.com/kworkflow/patch-hub/pull/20)
2. [README: Mention b4 and git-email as required dependencies](https://github.com/kworkflow/patch-hub/pull/24)

**th-duvanel** also had two merged PRs:

1. [src: app: fix: removed obligation to write the full list name](https://github.com/kworkflow/patch-hub/pull/27)
2. [src: main: fixes empty bookmark list](https://github.com/kworkflow/patch-hub/pull/28)

**lorenzoberts** studied the codebase to find many weak points, and we had a
tremendous offline discussion on those.

## **Conclusion**

I couldn't ask for a better start for the new `kw patch-hub`!

I feel like my team and I will have great experiences working together and,
hopefully, get `patch-hub` into the state it deserves.

Shortly, I will probably post about what milestones we aim to reach with updates
on the development.
