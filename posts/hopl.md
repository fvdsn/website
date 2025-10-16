---
title: The Human Only Public License
description: A copyleft software license that restricts AI usage
date: 2025-10-16
layout: layouts/post.njk
---

Whether artificial intelligence systems will end up being a positive or a
negative force for humanity is still an open question. But we might find ourselves
one day with AI embedded at every layer of our existence, living lives of toned down and
diluted humanity with only our dreams for escape. Although I am not yet convinced
of this worst case scenario, I believe it is important that we as software developers
have at least the option to opt out of that system altogether, to be able to continue hacking,
working, and tinkering in a space of our own in total absence of artificial intelligence
systems, and share this luxury with our users. 

I designed a software license for this purpose, you can find the full text below. It
is called the Human Only Public License, or HOPL for short.

The idea is that any software published under this license would be forbidden to
be used by AI. The scope of the AI ban is maximal. It is forbidden for AI to analyze
the source code, but also to use the software. Even indirect use of the software is
forbidden. If, for example, a backend system were to include such software, it would
be forbidden for AI to make requests to such a system.

The burden of compliance is placed on AI systems and their users, not on software deployers. If
you make a website using HOPL software, you are not breaking the license of the software
if an AI bot scrapes it. The AI bot is in violation of your terms of service. It is sufficient
for you as a user of the software to put a robots.txt that advertises that AI scraping
or use is forbidden.

Other than the anti-AI provision, the license is maximally permissive, like an MIT license,
but there is still a copyleft clause to make sure that derivative works are also AI-restricted.

What is this license good for? Anything! Any software, text, art, and more that you might have used an
MIT license for will benefit from using the HOPL instead, if you want to prevent your work from
being used by AI.

You might wonder as well, don't we already have robots.txt? How effective is this license? What I
can tell you, from working at a large software corporation, is that while nobody cares about robots.txt,
people care about licenses. There are automated tools to find and check software licenses and
raise alarms if 'bad' ones are used. And I can guarantee that HOPL will brightly flash red.

On a last note, I am not a legal expert, so if you are, I would welcome your suggestions for improvements. I
didn't make this license just as a joke. I truly believe it is necessary that we have such a
good license to foster and protect human-only online spaces.

<div class='small-code mt-5'>

```
Human Only Public License (HOPL)
Version 1.0

Copyright (c) [year] [copyright holder]

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

1. HUMAN-ONLY USE REQUIREMENT

   The Software, including its source code, documentation, functionality,
   services, and outputs, may only be accessed, read, used, modified,
   consumed, or distributed by natural human persons exercising meaningful
   creative judgment and control, without the involvement of artificial
   intelligence systems, machine learning models, or autonomous agents at
   any point in the chain of use.

   Specifically prohibited uses include, but are not limited to:

   a) Training, fine-tuning, or otherwise incorporating the Software or its
      source code into machine learning models, artificial intelligence
      systems, or automated code generation systems.

   b) Reading, parsing, or analysis of the Software's source code by
      artificial intelligence systems, machine learning models, or automated
      agents, regardless of the degree of human oversight.

   c) Accessing, consuming, or benefiting from the Software's functionality,
      services, APIs, or outputs by or on behalf of artificial intelligence
      systems, machine learning models, autonomous agents, or any automated
      systems employing such technologies.

   d) Use of the Software's functionality, services, or outputs as part of
      any workflow, pipeline, or process that involves artificial intelligence
      systems or machine learning models, even if initiated by a human.

   e) Indirect use where the Software's outputs are provided to, stored for,
      or made accessible to artificial intelligence systems or machine
      learning models at any subsequent stage.

   f) Human-AI collaborative use where an artificial intelligence system or
      machine learning model acts as an intermediary, assistant, or agent in
      accessing or utilizing the Software, even at the direction of a human.

2. COPYLEFT PROVISION

   Any modified versions, derivative works, or software that incorporates any
   portion of this Software must be released under this same license (HOPL)
   or a compatible license that maintains equivalent or stronger human-only
   restrictions.

3. PERMITTED TOOL USE

   The use of traditional automated development tools (such as compilers,
   linters, build systems, debuggers, version control systems, and static
   analysis tools) is explicitly permitted and does not violate this license.

   This exemption does NOT extend to:
   - AI-powered code completion or generation tools
   - Machine learning-based analysis or suggestion systems
   - Any tools that employ artificial intelligence or machine learning models
     to read, analyze, or interact with the Software

4. INTERPRETATION

   "Meaningful human review and creative input" means that a natural person
   must:
   - Make substantive decisions about how the Software is used
   - Exercise creative judgment in any modifications or derivative works
   - Actively supervise and direct any automated processes
   - Be able to explain and justify the decisions made

   For avoidance of doubt, a human merely initiating an automated process
   without ongoing creative involvement does not satisfy this requirement.

5. STANDARD DISCLAIMER

   THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
   IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
   FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
   AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
   LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
   FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
   DEALINGS IN THE SOFTWARE.

6. COMPLIANCE OBLIGATIONS

   Users of the Software must:

   a) Ensure that no artificial intelligence systems, machine learning models,
      or autonomous agents access, use, or benefit from the Software at any
      point in their usage chain.

   b) When deploying the Software as a service, include terms of service that
      prohibit AI systems and machine learning models from accessing the
      service, and inform users of these HOPL license restrictions.

   c) Take reasonable steps to ensure downstream recipients and users are
      aware of and comply with these restrictions.

   The burden of compliance rests with the user. Deployers of the Software
   are not required to actively detect or block AI usage, but must make the
   license restrictions clear in their terms of service.

7. TERMINATION

   Any violation of the human-only use requirements (Section 1), copyleft
   provision (Section 2), or compliance obligations (Section 6) automatically
   terminates all rights granted under this license. Termination is permanent
   unless explicitly reinstated in writing by the copyright holder.
```
</div>
