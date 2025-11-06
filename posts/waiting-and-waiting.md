---
title: I Spend Most of My Time At Work Waiting for the AI, the CI and Code Reviews
description: Agentic coding sped up coding so much that all that remains is waiting.
date: 2025-11-06
layout: layouts/post.njk
image: /img/waiting.png
---

<img src="/img/waiting.png">

The diagram abrove represents an approximation of the process I have to go through to
complete a programming task at work. By complete I mean from picking it up to be completely
done with it. The process I follow is gitops + code review that I presume to be a quite modern
industry standard. This process was tolerable last year, but now that I switched to agentic
engineering, the coding part that took me 5 hours shrank to a bit more than one, and what remains
is mostly waiting.

The worst part is that it's not the 'wait until tomorrow and do something else in the meantime' kind
of waiting, but short, unpredictable waits that leave little room for doing something productive. There
seems to be room for doing another task in parallel but the mental context switches for doing so are
extremely taxing. There's also checking mails, meetings, and code reviews to be done but those don't fit
neatly into the downtimes and there's still all the context switches.

A few years ago programming meant being concentrated for long periods of times, taking walks to think and
answering mails when you felt like taking a break. Now it's running node's task queue in your head and mine
does not like it. I think simply dropping in agentic programming in the current workflow is not sutainable.

So how might a future agentic first workflow look like ?

The agentic engineering part of the workflow itself is full of annoying waits already and there are two different schools
that produce different kinds. In the first, the prompts are very specific and the AI produces little code
in a relatively short time that is carefully reviewed. This is the workflow I am currently using, and for this way
of working I do not necessarily desire smarter models, but faster, so that I can keep better focus. On the other hand
the 'YOLO' school uses very high level prompts and lets the AI run unattended for longer, producing bigger diffs at once.
The waiting time there are big enough to start other prompts in parallel, and that may be the definitive way to go once
the models get better.

Regardless how you you generate your code the main bottleneck is in the code reviews. Agentic engineering produces too much
code too fast to leave room for efficient review. And what's the point of reviewing code that hasn't been written by the
engineer ? Hasn't the code actually already been reviewed by the agentic engineer, why wait for a second review ? Shouldn't we review the prompts instead ? One could imagine working with much bigger tasks and PR chunks, let's say 5K-10K line diffs, were we review the prompts and not the code. This way we keep the current way of working but with larger throughput per task.

What I don't like about that way of thinking is that it leaves the review back and forth which also generates multiples CI runs for the same task, which is also starting to become a bottleneck. I also think smaller tasks, smaller diffs, and quicker iteration time are all good things, and there's an opportunity to make the cycle time even faster.

We could completely drop manual reviews. The CI would have an agentic code quality gateway that would do the job of the 'is this good enough to go to prod' part of the review. And if the agentic coding tool is configured in the same way as the quality gateway, it is unlikely to be triggered and cause repeated CI back and forth

The 'is this the maintainable way to do things' part of the review would be done asynchronously, post deploy, in the Slack channel and not in the diffs, and the agreed upon fix would be done as another independent task later on.

That leaves unsolved the problem of developpers being confused about the requirements of the task and making functionally nonsensical changes. This is often currently caught at code review, but since not being confused about requirements is almost becoming the dev's last remaining task, this might maybe be better solved at performance reviews.

In any case any solution will require big changes to the PR pipeline tools like github and gitlab. I am surprised there is so much work being done on the code generation and so little on the rest of the pipeline while that part is now clearly the bottleneck.
