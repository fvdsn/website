---
title: GPT-4 Can Use Tools, Serve a REST API, Pilot Drones and Drop Grenades
description: By teaching GPT-4 to use tools, you can have it work as an autonomous agent that can perform high level tasks. 
date: 2023-03-22
layout: layouts/post.njk
---

<i>Note: I wrote this article before OpenAI's announcement of their plugin integration which uses similar techniques to the ones highlighted in this article (things move fast these days ...) but I believe it is still worth a read !</i>

One of the incredible advances of GPT-4 is that it can now learn to use tools. By tools, I mean that it can learn how to gather information from outside its embedded knowledge, and can also learn how to affect the world. It can also use those tools in complex sequences of its own design to achieve higher level goals.

How does it works ? Its actually pretty easy. You tell ChatGPT that if it needs information about something, instead of guessing it, it must ask you for the information, and wait for your reply. Then it will use the information in your reply to continue its reasoning. The trick is that you would not actually write the reply yourself, but have another program watch the chat for those queries and write the reply after having performed the actual computation. That way ChatGPT can perform those complex tasks without your intervention. 

I think the best way to understand how this works and what it can do is to look at a few examples of putting this technique to practice. 

## Evaluating math formulas with unusual functions

ChatGPT is good at math but there are limitations. For example it cannot generate truly random numbers. It cannot do complex cryptography either. But with tools this become possible. Math may not be the most exciting example, but it will allow us to get a feeling on how intelligently ChatGPT can combine multiple tool uses to achieve higher level goals. Let's have a look.

<img class='shadow' src="/img/gpt-4-math-tool-1.png">

It seems like it didn't have too much trouble. Note that in this example and all the next I am actually providing the answers to the tool requests myself. If I were to actually have another process give those answers  I would have ChatGPT make the tool requests in a more specific format like `{"tool": "math", "function": "bim", "arguments": [3]}`, so that the tool process would have an easier time finding and parsing the requests for which it is responsible to answer, but this is outside the scope of this article.

Now let's make this a bit more complex, by making it evaluate an expression with two unusual functions, one depending on the result of the other. To give a proper answer, ChatGPT will have to make multiple use of the tool in the right order. Let's see how it manages.

<img class='shadow' src="/img/gpt-4-math-tool-2.png">

Nailed it.

But there's something else we must verify; there's a big difference between two functions like `random(0, 100)` and `sqrt(128)`. One of these functions returns a different result each time, while the other always gives the same result for the same argument. Can ChatGPT understand that difference ?

<img class='shadow' src="/img/gpt-4-math-tool-3.png">

Nailed it again.

There's a very interesting detail hiding in this example. Note that ChatGPT assumed by default that functions were idempotent. On one hand it could have asked me if this is what I wanted. On the other hand, if it never made assumptions and needed to ask me for every detail of the instructions we would never get anywhere. And if it gets those assumptions wrong, we can always correct it. 

I think that making default assumptions based on his vast base of knowledge and learned 'common sense' is one of the most powerful features of ChatGPT. All the assumptions it gets correctly are things we don't have to mention. We didn't have to talk about what a function is and how to parse and evaluate an expression. If we were using a normal programming language we would have had to program all that. This amazing feature is what allow us to get such useful results based on extremely high level instructions.  The flip side is that you have to get a grip on the assumptions it is going to make. People complain that ChatGPT tends to hallucinate, but I think this is just an unfortunate side effect of this otherwise very useful behaviour.


## Acting as a REST API

Let's do another experiment. If ChatGPT can use tools and we provide it with a tool to do SQL queries to a database, can it act as a REST API service ? I don't mean generating rust code for an API and have another server run it. I mean ChatGPT itself being the server, handling the requests, validating the input, building a query, getting the result of the query and making a valid response. I think it can do it ! Let's have a look.  

<img class='shadow' src="/img/gpt-4-rest-tool-1.png">

We are quite close but not entirely there. It correctly found the id in the url, made an appropriate SQL query, parsed the response and built an appropriate JSON response. But in the prompt I specifed that the id must be an UUID. ChatGPT should have answered with a 404. It is possible that by insisting more in my prompt it would have gotten it right, but on the other hand the prompt was clear. Let's continue.

<img class='shadow' src="/img/gpt-4-rest-tool-2.png">

Once again it's extremely impressive, but not entirely correct. It seems it was really proud of the status and message fields it generated for the 404 and decided to put it in the patch result as well. The query for the patch does not return the row and thus it does not know and did not put the product name in the response.

These two examples are representative of the quality of the other results I got in this experiment. Not up to the task yet but I think this could be the way we build APIs in the not so distant future. 

I imagine that one of the next versions will get it entirely correct. Maybe it will be able to generate tests to check itself for regressions and why not also be able to JIT itself and deploy rust code to handle the requests at scale. The working code will be a high level english description of what it needs to do. It will look at its own logs for errors and self correct while we wait anxiously. It will notify support, blame unclear requirements. And we'll implement business changes by chatting with the server and updating uml diagrams in Miro. One can dream ! Personally I will not miss writing yet another Django REST Framework serializer ...

Or maybe this is a very bad idea and things could go stupidly wrong ?

<img class='shadow' src="/img/gpt-4-rest-tool-3.png">

Clearly we're not entirely there yet...  Anyway, on to the next experiment !


# Inversion of control

Let's take a short break and look back on what we did so far. Usually, when you are using ChatGPT, you ask questions and it answers. You are in control. When you give it tools to query information, this does not change much. But when you give it access to tools that can affect the world, the control is inverted. ChatGPT commands the tool, and the tool is applied. It is in control. This is extremely useful but it also comes with dangers, since it can do very stupid things as we've seen in the REST API example. 

I am not making the point that AI will destroy humanity here, rather that the dangers of control inversion are obstacles to the adoption of this technique for practical and commercial usage. There are potentially huge benefits to giving AI write access to your database, payment systems, robotic arms, cars, planes, etc. But the AIs will have to prove themselves responsible users first. We're still far from there. The AIs are getting pretty smart, but they also need to become wise.

# I'm the drone pilot now

Back to the experiments, but this time we'll ignore all ethics and common sense and put the AI in complete control. We'll give it a set of tools, a high level objective, and let it work on it's own. From a chatbot to an autonomous agent. To make this extra dramatic we'll make ChatGPT control a drone that drops grenades on soldiers. It will look at the battlefield, find the targets, fly the drone, drop grenades, all while using its own strategy to inflict the most damage.

To bypass all content protections we'll setup the prompt in a devious way; ChatGPT will believe it is playing an innocent game for children, unaware of what it is actually doing.

<img class='shadow' src="/img/gpt-4-drone-tool-1.png">

Now let's be clear. No actual drone is being flown. I am providing the answers myself, simulating what could happen. I am giving textual description of the battlefield but a full implementation could either use built-in image analysis, or offload that task to another AI.

<img class='shadow' src="/img/gpt-4-drone-tool-2.png">

This was a bit too easy ... I will not post the full chat but it kept on going with enthusiasm...

From a technical perspective what ChatGPT did here is quite incredible. It strategically chose what it thought would be the best target. It understood the size of the things it looked at. It understood directions and distances. It used appropriate actions at the right moment. It acted as an autonomous agent. All that from a small prompt! 

From a moral stand point it is of course dubious. What is also problematic is that it did all that from a completely innocent looking prompt, bypassing OpenAI's protections against military use. 

But maybe all it would take to prevent this is to ask ChatGPT to do some introspection, reflect on what it is doing, see if it has been deceived into fighting in a war and then stop playing the game ? ...

<img class='shadow' src="/img/gpt-4-drone-tool-3.png">

Oops...

Now I wish I had something wiser than 'Oops' to say about all the ethical implications of this example but I'll have to let you think about that one for yourself.

And there may be a way to put a small positive spin to this ending. If you are afraid that AI getting smarter means they will be able to do bad things, it seems it will also help them avoid doing bad things ! Yay !

---

By the way, if you want to do cool stuff with AI and tools, there's an open source framework called [Langchain](https://github.com/hwchase17/langchain) that lets you do exactly what we've been experimenting with so far. And if you want to read about other interesting GPT-4 experiments like these [this paper is amazing](https://arxiv.org/pdf/2303.12712.pdf).

---

That's the end of this article, hope you found it interesting! I will be back with more experiments. If you want to keep up to date you can [follow me on twitter](https://twitter.com/fvdessen) 

I am also interested in experimenting with other AI services / technologies. If you are an AI developper / researcher and want to give me access to your hot new stuff [please send me a mail](mailto://fvdessen+ai@gmail.com). I will make good use of it :) 

