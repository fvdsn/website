---
title: My Programming Job Has Become An Intelligence Buying Job
description: Intelligence is now a commodity, our task is to use it efficiently
date: 2026-02-06
layout: layouts/post.njk
---

I have been programming for more than 20 years. 20 years of learning how to translate
thought into code. 20 years of trying to master an editor to be fast at it. 2025 was the year 
when I wrote my last piece of code. AI is now better at it than I am and ever will be. It
is not just better at writing code, it is better at architecting, debugging, designing, documenting,
deciding trade-offs. It is now smarter than I am.

This is not something easy to accept, and I have been trying to rethink my relation with my job
and my computer into something that makes sense for the future.

The key is that intelligence is now a commodity. It was for a long time a luxury product with a time
based limited supply. But it is now just a matter of spending money. You can now get unlimited
amount of intelligence if you are willing to spend the money on the tokens.

Let me take a practical example. At whichever place I was working at, the quality of the codebase has
always been a point of contention. It is bad ! We should refactor ! But we never had the time.

Yesterday evening I made a small ralph loop with a prompt to refactor and improve the quality of a
microservice. *Read the whole code base, identify the pain points based on the following criteria, make
a set of tasks, and work on the tasks* (the real prompt was more involved). Then I let Opus 4.6 run
on that prompt until it ran out of tokens.

It worked. It vastly improved the code base, made clean commits, and did that entirely unsupervised.
I then could, if I wanted, run this ralph loop in parallel on every one of the 250 services at my company,
and completely clean up our whole code base for less than a day of prompting. But also for $10,000 of
tokens.

So the question that was *do we have the time to do it*, has become *is it worth the money to do it*.
And who decides ? Currently it's a long discussion with various managers and the finance department.
But I think eventually, engineers will have to be trusted with making these decisions on their own, 
and I believe that is going to be our job in the future, being the ones responsible for the billing lines
in the token expense list.

And as the impact of the engineer's decisions scales, so do the risks. Is it wise to deploy 100,000 lines
of changes ? Who knows what they really contain ? In the past, this risk was mitigated by careful review 
by the engineer's peers. But this new scale makes it completely impractical. New types of AI powered 
guardrails will need to be put in place, which might just be a matter of automating the existing process of
review, deploy, monitor and fix. But I think that once again, engineers will need to be trusted
with a much higher degree of risk-reward economic decisions.

In a sense this can be an exciting development, but I can also see around me many engineers who have
taken comfort in taking tasks from a Jira board and simply turning that into code. I think the days
of *that* job are numbered.
