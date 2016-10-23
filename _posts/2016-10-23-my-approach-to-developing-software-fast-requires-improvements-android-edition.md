---
layout: post
title: "My Approach To Developing Software Fast Requires Improvements (Android edition)"
description: "Open letter to developer communities to show me what I can't see by myself"
category: "Android Development"
tags: ["android"]
---
{% include JB/setup %}
I feel like it has finally arrived: being a generalist programmer/engineer slaps me in the face like it has never done before. I try my best to dodge these slaps but they keep coming, it is starting to hurt. This may sound too sentimental and I agree, this is one of those times when I need my teachers to ask whatever I want and hope they don't tell me to go away and do some research. I need answers, not a new article or book to read. Eventually, I'll go back to reading but right now, I may not have time to do so without consequences. The "I" in the title is actually me and my team in our startup company. Enough chitchat, let's talk programming.

#### Problem: There seems to be a limit for how much one can eliminate boilerplate code

Here is the pseudo code of how we iterate with an example. It is from an Android app project written in native Java and backed by a few 3rd party dependencies.

Like many other people out there, we use `Fragment`s. They enable instant screen transitions and code sharing in a single `Activity` architecture with a cost of high amount of null checks and complex life cycle handling. Why we iterate is to reduce or hide these costs without losing the benefits.

Initially,

  1. We start a new scene by extending native `Fragment`
  2. Implement default overrides
  3. Test, crash then fix a lot until it is stable
  4. Start another scene by extending native `Fragment` again
  5. Do **#2** and **#3** again
  6. Compare last two scenes to detect shared boilerplate
  7. Extend `Fragment` and name it `BaseFragmentV1`
  8. Transfer boilerplate there or generalize the logic
  9. Start a new scene by extending `BaseFragmentV1`
  10. Provide generic arguments if there are any
  11. Implement newly defined default overrides (hopefully for a less amount)
  12. Do **#5**
  13. Do **#9**, **#10**, **#11** and **#12**
  14. Do **#6**
  15. Extend `BaseFragmentV1` and name it `BaseFragmentV2`
  16. Go to **#9** and iterate by increasing `BaseFragment` version on each iteration

Around fifth iteration a limit is hit. The reason is simple. No matter what you do, there is no such thing as a generic model for your data. Many models are nested and used by only one fragment. Since they are not generic, any web service client abstraction and/or custom view that manipulates them requires to be implemented by hand from scratch. Utilities indeed exist to make these implementations a plug and play game.

#### Not a F2P

This game has a cost for our developers. It has guidelines, best practices, worst practices, illegal usages and most importantly require of dirty hacks when UX designers invent smart widgets. The thing is, they tend to do that a lot.

That is the essential cause of this problem in the title. It is hard to avoid. From the design perspective, the requested feature is actually generic to the end user. It is easy to learn, fun to use and reduces click count per action. Quite lengthy in terms of coding though.

If there is a way to automate implementation of such use cases, I'd like to know how. Maybe we need another kind of iteration process, maybe there exist helpful 3rd party libraries that we don't know. I wish I knew a way out of this.

#### Our tools

Here is our tool belt for the curious reader. We already have,

  * A stable main `Activity` that never crashes
  * A generic `Fragment` to never have to bind views by hand
  * Same fragment handles data refresh logic and state persistence too
  * A central dispatch for cache operations
  * A central dispatch for navigating between `Fragment`s
  * A generic pop up with configurable text fields and action callbacks
  * A generic animation library for chaining, interrupting time and control based animations (to be made open source soon)
  * A generic `Switch` widget
  * A generic `RootView` and inflater to be able to use with Android's data binding utilities (this is awesome actually)
  * One liners for HTTP client calls.
  * One liner threading utilities
  * One liner resource handling utilities
  * One liner generic observers
  * One liner date, string, math and list utilities

What we need is,

  * A way to reduce amount of work to write new data models. Reflection based JSON parsers/generators are not suitable.

  * A design pattern to express view state without modifying represented data. I don't want to add view specific bool flags to a model. A state object that wraps the model without modifying it looks dirty but maybe a way to go. It seems dirty when a Fragment needs to persist that state instead of just the data model's entity object.

  * A client generator to be able to benefit from server docs in order to generate client calls (something more lightweight than Swagger and easy to inject into our own HTTP client).

  * Much better `Intent` handling for push notifications and result expecting `Activity` transitions. Native Android approach looks hideous. We still use what is documented in Android's developer guides.

  * A way to optimize view count in a `RecyclerView` cell. `RelativeLayout`s are all over the place right now. `LinearLayout`s that nest other `LinearLayout`s suck too. Maybe there is a way to write flat view hierarchies using `FrameLayout` and `ConstraintLayout` without losing development efficiency.

  * A way to reduce build times. It is around 100 seconds right now. Multidex looks unavoidable.

#### Final Words

I'm not sure if we hit the platform limit or it is just a temporary hassle that will eventually disappear with better tooling, extensive design pattern usage or just more experience. If someone thinks some things in our tool belt looks useful, we are more than happy to make them open source in the future.

Expect a similar post for iOS once I am a mature enough developer in that field. In the meantime, enjoy criticizing anything in this post.

Peace.
