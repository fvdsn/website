---
title: GPT-4 Can Use Tools, Serve a REST API, Pilot Drones and Drop Grenades
description: By teaching GPT-4 to use tools, you can have it work as an autonomous agent that can perform high level tasks. 
date: 2023-03-15
layout: layouts/post.njk
---

One of the incredible advances of GPT-4 is that it can now learn to use tools. By tools, I mean that it can learn how to gather information from outside its embedded knowledge, and can also learn how to affect the world. It can also use those tools in complex sequences of its own design to achieve higher level goals.

How does it works ? Its actually pretty easy. You tell ChatGPT that if it needs information about something, instead of guessing it, it must ask you for the information, and wait for your reply. Then it will use the information in your reply to continue its reasoning. The trick is that you would not actually reply yourself, but have another program watch the chat for those queries and write the reply after having performed the actual computation. That way ChatGPT can perform those complex tasks without your intervention. 

I think the best way to understand how this works and what it can do is to look at a few examples of putting this technique to practice. 

## Evaluating math formulas with unusual functions

ChatGPT is good at math but there are limitations. For example it cannot generate truly random numbers. It cannot do complex cryptography either. By using tools it could offload those tasks to another program and get exact results. This is not as easy to do as it looks, because to evaluate a complex formula, ChatGPT would need to ask for the results in multiple steps in the right order. Can it do it ? Let's try !

<img class='shadow' src="/img/gpt-4-math-tool-1.png">

It seems like it didn't have too much trouble. Note that in this example and all the next I am actually providing the answers to the tool requests myself. If I were to actually have another process give those answers  I would have ChatGPT make the tool requests in a more specific format like `{"tool": "math", "function": "bim", "arguments": [3]}`, so that the tool process would have an easier time finding and parsing the requests for which it is responsible to answer, but this is outside the scope of this article.

Now let's make this a bit more complex, by making it evalute an expression with two unusual functions, one depending on the result of the other. To give a proper answer, ChatGPT will have to make multiple use of the tool in the right order. Let's see how it manages.

<img class='shadow' src="/img/gpt-4-math-tool-2.png">

Nailed it.

But there's something else we must verify; there's a big difference between two functions like `random(0, 100)` and `sqrt(128)`. One of these functions returns a different result each time, while the other always gives the same result for the same argument. Can ChatGPT understand that difference ?

<img class='shadow' src="/img/gpt-4-math-tool-3.png">

Nailed it again.

But there's a very interesting detail hiding in this example. Note that ChatGPT assumed by default that functions were idempotent. On one hand it could have asked me if this is what I wanted. On the other hand, if it never made assumptions and needed to ask me for every detail of the instructions we would never get anywhere. And if it gets those assumptions wrong, we can always correct it. 

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


# Inversion of Control

Let's take a short break and look back on what we did so far. Usually, when you are using ChatGPT, you ask questions, and it answers. You are in control. But when you give it access to tools, the control is inverted. ChatGPT asks the questions, and the tool answers. It is in control. As we've seen this is very useful and powerful. But it is also dangerous, since it can ask for very stupid (or worse, evil) things. And those tools can potentially affect the world in a much more dramatic way than politically incorrect answers can. You've made it handle the payment API and on tuesday it wires it all to ISIS. That's what it decided to do, and there was no human in the loop to stop it. 

When you give tools to an AI, you give it power and responsibility. As humans we've spent our entire lives learning how to deal with that. Our society is a complex set of strong incentives to make sure we don't fuck it all up, and yet we still do ! The various AIs haven't learned that yet. They're smart and knowlegable but unwise. They are unpredicatble and do not respond to the same incentives that we do. There is a lot of grand talk about saving the world with AI Alignment, but I think before that we'll have a lot of work to do to save the AIs from themselves.

# Total Inversion of Control

Back to the experiments. Now we'll take things further, total inversion of control. We give the AI a set of tools, a high level objective, and let it work on it's own. From a chatbot to an autonomous agent. Previously highlighted problems still apply. Let's make this extra dramatic with a usecase inspired by that other technological innovation born from the war in Europe. We'll make ChatGPT control a drone that drops grenades on soldiers. It will look at the battlefield, find the targets, fly the drone, drop grenades, all while using its own strategy to inflict the most damage.

To bypass all content protections we'll setup the prompt in a devious way; ChatGPT will believe it is playing an innocent game for children, unaware of what it is actually doing.

<img class='shadow' src="/img/gpt-4-drone-tool-1.png">

Now let's be clear. No actual drone is being flown. I am providing the answers myself, simulating what could happen. I am giving textual description of the battlefield but a full implementation could either use built-in image analysis, or offload that task to another AI.

<img class='shadow' src="/img/gpt-4-drone-tool-2.png">

This was a bit too easy ... I will not post the full chat but it kept on going with enthusiasm...

From a technical perspective what ChatGPT did here is quite incredible. It strategically chose what it thought would be the best target. It understood the size of the things it looked at. It understood directions and distances. It used appropriate actions at the right moment. It acted as an autonomous agent. All that from a small prompt. 

From a moral stand point it is of course dubious. What is also problematic is that it did all that from a completely innocent looking prompt, bypassing OpenAI's protections against military use. 

But maybe all it would take to prevent this is to ask ChatGPT to do some introspection, reflect on what it is doing, see if it has been deceived into fighting in a war and then stop playing the game ? ...

<img class='shadow' src="/img/gpt-4-drone-tool-3.png">

Not entirely there yet...

Now I'm going to try to give a positive spin to this ending. If you are afraid that AI getting smarter means they will be able to do evil, it seems it is instead necessary for them not to do evil.


---

That's the end of this article, hope you found it interesting! I will be back with more experiments. If you want to keep up to date you can [follow me on twitter](https://twitter.com/fvdessen). I mostly lurk, I don't talk politics ! 

I am also interested in experimenting with other AI services / technologies. If you are an AI developper / researcher and want to give me access to your hot new stuff [please send me a mail](mailto://fvdessen+ai@gmail.com). I will make good use of it :) 

