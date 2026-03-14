+++
title = "Data Fortress to Open Source: How Unlearning the Lessons of a High-Control Group Shaped My Path in AI Security" 
date = 2026-02-23
description = "How Unlearning the Lessons of a High-Control Group Shaped My Path in AI Security"

[taxonomies]
tags = ["philosophy", "ai"]
+++

{{ img(src="data-fortress-hero.webp", alt="Dark cyberpunk fortress with red and copper accents", w=1200, h=672) }}


## From Information Fortress to Open Source

For most of my life, my security model was simple: don't have outputs. No stated opinions, no visible beliefs, no detectable interior. If there's nothing to parse, there's nothing to exploit. It was, in the technical sense, a zero-trust architecture. In every other sense, it was exhausting.

The problem with building a fortress is that you have to live in it. I hoarded my thoughts, beliefs, and general sense of self like a paranoid dragon sitting on a pile of information units with no intention of spending any of them. Very secure. Completely useless. Zero threat surface, zero utility — which, as any security engineer will tell you, is not actually the goal.

This blog is my attempt at a different strategy — not less secure, just differently engineered. Less paranoid dragon, more resilient system. The kind that can interact with the world without being compromised by it. I'm still working out the implementation details. But then, so is everyone else building AI right now, which is either comforting or terrifying depending on the day.

---

## My First Adversarial Environment

I didn't get into AI security through a bootcamp or a CTF challenge. My introduction to adversarial systems was considerably more immersive. I grew up in a high-control group — the kind that has its own vocabulary, its own cosmology, and a surprisingly robust content moderation policy for incoming information that contradicted the approved narrative.

In retrospect, it was a remarkably well-engineered system. Information diet? Strictly curated. Dissent? Flagged and penalized. Repetitive, emotionally charged messaging? Deployed on a daily basis with impressive consistency. The goal, I now understand, was to degrade my internal model to the point where I was no longer generating my own outputs — I was just reflecting the group's ideology back at it with a smile. A very well-trained, highly compliant, deeply miserable little model.

The deconstruction didn't come from a sudden revelation or a dramatic escape. It came from a second high-control group. The first group found them familiar enough to permit contact — similar structure, similar vocabulary, compatible enough to pass the filter. But the second group's messaging conflicted just enough with the first that I couldn't reconcile them. No single input broke anything. It was the incoherence between two competing systems that did it. My worldview didn't shatter so much as quietly fail to compile.

Which, if you're familiar with how adversarial inputs can be used to expose contradictions in a model's training, will sound extremely familiar.

---

## The "Aha!" Moment: Seeing My Past in the Present

Years later, when I first stumbled into the world of AI security, I didn't have the reaction most people seem to have. There was no wide-eyed wonder, no "the future is now" energy. Instead, reading about prompt injection and model manipulation for the first time, I felt something closer to... tired recognition. Like running into an ex at the grocery store. *Oh. You again.*

The vocabulary was new, but the playbook was identical. Carefully crafted inputs designed to override a system's core values? I knew that one. Exploiting a model's desire to be helpful in order to extract behavior it was never meant to produce? Deeply familiar. The entire discipline of adversarial prompting, it turns out, has been field-tested on humans for decades — no GPU required.

What struck me most wasn't the technical specifics. It was the fundamental tension sitting at the heart of all of it: the trade-off between openness and safety. A system that is locked down and heavily filtered is difficult to manipulate, but it's also essentially useless — a very expensive rock. A system that is open, helpful, and capable is genuinely powerful, but that same openness is the attack surface. You don't get one without the other.

I'd lived that trade-off in my own nervous system. I just hadn't had the language for it until now.

---

## The Part Where It Gets Weird

Here's where my personal revelation stops being a memoir beat and starts feeling genuinely urgent. Because while I was busy recognizing old patterns in research papers, something else was happening: we started building the geniuses.

Not metaphorical geniuses. Systems capable of reasoning, persuasion, code generation, and nuanced social modeling — at scale, at speed, available to anyone with a browser and a free tier. Which is wonderful. Genuinely exciting. I mean that.

And also: bad actors have browsers too.

The manipulation techniques I experienced were slow, resource-intensive, and required a dedicated group of people with a shared ideological goal. They were effective anyway. Now imagine those same techniques running at the speed of software, with no overhead and no need for a charismatic leader.

What people don't fully appreciate is how expensive the damage is to undo — and this isn't just intuition, it's biology. Unlearning is not the inverse of learning. It's a separate, harder, more expensive process. The brain doesn't delete bad information; it builds new pathways around it and hopes for the best. LLMs have the same problem, just with the serial numbers filed off. Correcting poisoned training data or excising a backdoor behavior baked in at the weight level is expensive, imprecise, and the ghost of the old behavior has a way of lingering. The cost of remediation vastly exceeds the cost of the original attack.

Which means the people who understand this best aren't the ones optimizing for capability. They're the ones who've had to do the unlearning themselves.

The security conversation still feels like it's happening slightly after the fact. We build the capability, celebrate it, and then — somewhere downstream — add it to a list of things we should probably look into. It's not malice. It's just the oldest story in engineering: move fast, patch later.

The problem is that some patches are harder to roll back than others.

---

## My New Strategy: Applied Resilience

This is where my past and my professional future converge. My experience taught me the nature of the threat. My work in AI security is my way of engineering the defense.

On this blog I'll be exploring the front lines of that battle — red-teaming models, diving into operational best practices, and wrestling with the philosophical challenge of building systems that are both capable and hard to corrupt. The same fundamental problem, whether you're a person or a language model: how do you stay open without becoming a target?

I don't have a complete answer. But I have something that might be more useful — a firsthand understanding of what it looks like when the attack succeeds, and what it costs to recover. That's not nothing. In security, that's actually most of it.

If you're a technologist wrestling with these problems, or a fellow traveler who recognized something familiar in the earlier sections of this post, I'd genuinely like to think alongside you. The goal was never to build an impregnable system. It was always to build a resilient one.

Turns out that applies to people too.
