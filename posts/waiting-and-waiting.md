---
title: I Spend Most of My Time At Work Waiting for the AI, the CI and Code Reviews
description: Agentic coding sped up coding so much that all that remains is waiting.
date: 2025-11-06
layout: layouts/post.njk
image: /img/waiting.png
---

<img src="/img/waiting.png">

The diagram above approximately represents the process I have to go through to complete a
programming task at work. This is the result of using the standard workflow that GitHub and Gitlab
recommend. Which means picking up an issue, making some code changes, creating a merge request for
these changes, and getting the CI checks, and approval from my colleagues, before it is merged and
automatically deployed.

As you can see the process is grossly inefficient, most of the time on a task is spent waiting, either
for the CI or for code reviews. The first reason for this state of affairs is that the
coding part used to take a lot more time. When the coding was done by hand it would take me a whole 8h
to get 500 lines of code out, now with agentic coding, I can get 1000 lines in an hour. The CI and review
waiting times were acceptable overheads, but now they're the bulk of the "work".

The second reason is that the code review back and forth is by itself an extremely inefficient process.
You do not know when you will get a review, and just one nitpick wastes a whole dev cycle.

The fact that code reviews are inefficient is not new, and there are different strategies to go around them.
On one hand you have pair programming and mob programming where the code is looked at by multiple people before
it is even pushed. On the other hand you have reviews *after merge*.

In a review after merge process, the code is merged first, and reviews come later, with the changes implemented
in new coding tasks. This means the original merge request is not blocked, and does not need multiple rebases
and CI that are likelier to happen the longer the request is stuck in review. This seems like the most
promising way to solve our efficiency problem for agentic coding, but the unfortunate part is that this requires appropriate
tooling and neither Github nor its alternatives have put the appropriate tooling in place.

I can say I am a bit surprised, given how much work is being done to speed up the code generation that
Github and co keep pushing a merge workflow so obsolete that it is now the main bottleneck in development work.
In the meantime if you have any solution I would be happy to hear about it in the comments.
