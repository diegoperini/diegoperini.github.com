---
layout: post
title: "Fighting with a language is a bad idea"
description: "Note to myself to stop being a jerk to my computer"
category: "Programming Languages"
tags: ["programming languages", "bad practices"]
---
{% include JB/setup %}
I have the habit of making the same mistakes over and over again not because programming language features are badly advertised, because I get too excited when I am able to find a specific feature I like in a language. My enthusiasm clouds my judgment, then the problem with my approach strikes me at the most inconvenient time. No library fixes a problem introduced solely by bad practice.

#### A Small Thought Experiment
Computer engineering/science education in many schools follows a deductive approach to inspect underlying concepts behind high level abstractions. Many curricula start with an easy to grasp programming language such as Python, PHP or subsets of some Java, C#, C, C++. Then they introduce limits like memory space, I/O delays, network fault tolerance, cache misses, software scheduling. A curious student can easily find out that many programming languages try to hide these limits from its users to become applicable to certain use case scenarios.

Here is my experiment. If there was a school that taught Javascript (on Node.js or io.js without explaining what they are) to its first graders just for writing small console programs, I believe these students would drown under the heavy burden when they tried to use their new skills on a browser.

Browsers are like grumpy parents, they get angry at what you do all the time. No long running loops, functions that do I/O are void functions with callback parameters, no initial file system, HTTP as main network communication tool (which is a monster if you read its history). All these limits are legit and there for security and hardware related reasons.

So, when an algorithm fails to run on your target platform although the language allows you to try it, who is at fault here? Your browser or the language?

#### You are Fighting the Wrong Battle
Language inventors are not stupid. I may hate Java because of reasons but I have no doubt that its creators and maintainers are really smart and hardworking people. They wanted their language to support every platform out there and built tools to render all kinds of problems arise from these platforms irrelevant. When you cleanly hide 1000 problems for your users, you are allowed to introduce a few new ones, okay? I believe a regular user would hardly need to face with those few ones.

The struggle is pointless because the bottleneck is always the environment, not the language.

Your singleton dies not because Java static is buggy, but your Android mobile phone doesn't persist its data on device memory all the time.

Javascript is single threaded because otherwise a browser with many open tabs would quickly deplete all the CPU time available in your pc.

#### Mind Your Manners
That is actually what all it is about. APIs, languages, protocols are only things that computers can understand. Be polite and don't swear at your computer with them.

P.S. In this post, the addressee "you" is me. :)

Cheers
