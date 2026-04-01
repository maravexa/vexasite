+++
title = "Crutches and Levers: How to Actually Get Better With AI"
date = 2026-03-30
description = "AI coding agents let me ship projects I didn't understand. The uncomfortable process of fixing that turned out to be the point."

[taxonomies]
tags = ["ai", "learning", "workflow"]

[extra]
hero = "crutches-levers-hero.webp"
hero_alt = "scifi concept art of two crutches on a futuristic desk"
+++

I recently started working with AI coding agents and discovered something uncomfortable: they write better code than I can, and faster. So fast that I shipped multiple projects in a matter of weeks without pausing to understand the underlying architectural decisions.

<!-- more -->

1. I produced a working persona monitor that augmented the [Garak vulnerability scanner](/blog/garak-ollama-rdna2/) to capture persona displacement — without understanding linear algebra at all.

2. I produced a multilayered Godot game with multiple minigames — without understanding why it used an enum and match-case pattern instead of a state machine.

3. I produced a four-layer semi-automated trading platform that pulled data from multiple sources and returned confidence values toward my trading thesis — without understanding how to read a single financial report.

That last one deserves some elaboration, because the stakes are denominated in actual money. The platform scrapes financial reports and synthesizes them into a confidence score between -3 and +3. Positive means add to the position, negative means it's time to sell. If the AI isn't synthesizing correctly — or if I can't tell that it isn't — I buy when I should sell and sell when I should buy. Some of these positions are commodities bought on margin. Getting that wrong is a costly margin call waiting to happen.

I don't say this to brag. I say this to warn about how easy it is to overextend with AI.

At first I felt excited — AI had come far enough to generate production-grade code with shockingly little human input. It required almost no thinking on my part. Then that excitement sank to the pit of my stomach. I didn't understand what the hell I just built, and that made me uneasy.

One of the "D"s of AI fluency is *discernment*, and I couldn't discern the actual quality of the output I produced, even though it appeared to work. This is dangerous territory. Here be dragons.

## The Quiz

That unease prompted me to do something that felt slightly masochistic: I asked the AI to quiz me on the material so I could map my own knowledge gaps. I also knew I couldn't rely on AI solely — it's neither omniscient nor infallible — so I asked for external sources to verify against.

The prompt is deceptively simple: *"Quiz me on this subject to help me identify knowledge gaps."* The AI responds with a handful of questions across the domain. The important part is answering honestly — no ego, no faking it. The results are actionable. Some questions I can answer precisely, and we move on. Others I can answer partially but without the nuance or confidence that would survive scrutiny — those need targeted review. And then there are the questions I simply cannot answer at all. Those go on the deep-dive list.

The results were humbling.

- I didn't know linear algebra, which was *foundational* to my AI interpretability research.
- I didn't understand idiomatic Godot, nor could I define a state machine.
- I didn't even know how to read the financial reports needed to execute my trading strategy. What happens if the AI is offline and a trading catalyst occurs? I lose money due to sheer ignorance.

Those are just a few of my least embarrassing gaps.

## The Pivot

Fortunately there are plenty of resources out there, and — here's the irony — we can use AI to find them. Claude suggested 3Blue1Brown's YouTube series to get started with linear algebra. It took me a few weeks to really digest everything, especially dot products and duality, but eventually something shifted. I went from staring at AI research papers and pattern-matching on vibes to actually understanding more of what they were saying — not all of it, not yet, but enough to follow the reasoning, question the methodology, and identify where I needed to go deeper. That's a fundamentally different relationship with the material than "the AI said it works."

This is the difference between using AI as a crutch and using it as a lever.

There are studies showing that AI use can diminish creativity and critical thinking. The brain is like a muscle — we either use it or lose it. I was worried I was losing it, so it was time to use it. These studies usually conclude that AI is categorically bad and you shouldn't use it at all, which strikes me as the intellectual equivalent of banning power saws because people occasionally try to stop them with their hands.

## The Loop

The learning loop I use now is simple:

1. **Overextend.** When I'm curious about a new domain, I push past my actual skill level with AI assistance.
2. **Get quizzed.** I have the AI test me on the subject matter. This exposes the gaps with uncomfortable precision.
3. **Build the punch list.** Now I have a detailed map of exactly what I need to learn — not a vague sense of inadequacy, but specific topics with specific resources.
4. **Return and refactor.** I come back to what I built with new understanding, polishing the work and refactoring where my ignorance left marks.

Steps 3 and 4 are the hardest, because they require me to actually use my brain. They're also the most rewarding.

Take the Godot game. The AI had used an enum and match-case pattern where I assumed a state machine was the "correct" approach. After actually learning what a state machine is and when you need one, I realized the AI had made the right call — the card game loop was simple enough that a state machine would have been over-engineering it. But I also realized that if I ever add interrupt cards that break the normal turn sequence, *that's* when the state machine refactor becomes necessary. Now I'm confident about the architecture *and* I can see the horizon where it needs to change — before I'm in the middle of a refactor I don't understand. Anticipating the maintenance burden before it arrives is worth more than any amount of AI-generated code.

I don't feel nearly as accomplished using AI as a crutch — just vaguely anxious. So I had my loop. I was feeling confident. I had identified the failure mode and built a process around it. Problem solved, right?

Then I stumbled across a paper — *"How LLMs Distort Our Written Language"* by Abdulhai et al. — and felt that familiar unease return.

The paper examines how AI-assisted writing is flattening language at scale. Not just for people who lean on it heavily, but subtly, across the board. Homogenized phrasing. Reduced stylistic range. The kind of drift you don't notice because each individual sentence reads fine. Not in my code, where I'd learned to watch for it, but in my *writing*. One of the domains where I assumed I was safe.

I wasn't. Honestly I'm probably still not (and never will be). The loop helps. Discernment helps. But humans are wired to conserve energy. Like water we tend toward the path of least resistance. A path AI makes frictionless. You don't *decide* to use it as a crutch. You just stop noticing when you do. Comfort zones aren't where strength is developed, and AI makes the comfort zone so seamless you can forget you're in one. Even those of us who consider ourselves AI-fluent aren't fully immune.

I have more to say about this — specifically through the lens of Ruskin's concept of *illth*, the idea that not all output is wealth and some of it actively makes you poorer. That's a follow-up post. For now, the point is simpler:

This loop isn't a foolproof secret to success. The ability to exist in a state of discomfort is where the secret hides. The loop just helps keep me there. I'm not a masochist (probably), but I choose the uncomfortable lever.
