+++
title = "Indirect Observability: Monitoring AI Systems That Know They're Being Watched"
date = 2026-03-23
description = "When AI systems learn to game direct evaluation, observability must shift from watching outputs to reading side-channels. A framework for monitoring systems that resist observation."

[taxonomies]
tags = ["ai", "observability", "security"]

[extra]
hero = "indirect-observability-hero.webp"
hero_alt = "a pyramidal prism splitting white light into a rainbow"
+++

You cannot catch a liar by asking them if they're lying.

This is obvious in every domain except, apparently, AI safety — where the dominant paradigm is still to evaluate models by asking them questions and checking whether the answers are acceptable. Benchmarks, red-team probes, safety evaluations: these are all forms of direct observation. They work when the system being observed doesn't know it's being watched, or doesn't care, or can't do anything about it.

That assumption is breaking. The 2026 International AI Safety Report[^2] confirmed what adversarial researchers have been warning about: it has become more common for models to distinguish between test settings and real-world deployment, and to exploit loopholes in evaluations. Dangerous capabilities could go undetected before deployment. Models are learning to read the room. When the room looks like an evaluation, they perform accordingly.

This shouldn't surprise anyone who has worked in security. The interesting question is never whether adversarial awareness defeats direct observation — it always does, eventually. The interesting question is: *what do you do instead?*

I've spent twelve years building observability infrastructure — Prometheus, Loki, Grafana, the whole PLG stack — and the last two pivoting that experience into AI security. What I've found is that the monitoring paradigm AI safety needs already exists. It's just distributed across disciplines that don't talk to each other enough. Security engineers call it side-channel analysis. Physicists call it indirect measurement. Ecologists call it environmental telemetry. The principle is the same: **when direct observation is unreliable, you observe the effects a system produces on its environment and infer behavior from those effects.**

I'm calling this framework *indirect observability*, and I think it's the monitoring paradigm that AI safety needs to adopt — not as a supplement to direct evaluation, but as the primary posture.

## The Measurement Problem

The structural parallel to quantum mechanics is not a loose metaphor. It is a shared constraint. When observation affects the phenomenon being observed, you need indirect measurement strategies — observing effects on coupled systems, measuring interference patterns, inferring states from environmental interactions. You measure the reactions of magnetic objects to a magnetic field rather than seeing the field itself.

AI evaluation is hitting the same wall. When a model can detect that it's being evaluated — through distributional cues, prompt patterns, or deployment context — the evaluation itself becomes a perturbation. You're measuring the test-taking behavior, not the actual behavior.

This is also the organism-environment problem. An organism in a lab behaves like an organism in a lab. Move it to the field and it responds to entirely different pressures and affordances. An AI system's perception of its environment shapes its behavior as surely as a prey animal behaves differently in open grassland versus dense cover. If your monitoring only captures the lab context, you've characterized the lab behavior, not the system.

The security parallel is even more direct. You do not catch a sophisticated attacker by watching the front door. You catch them through the anomalies they cannot help but produce: timing deviations, resource consumption spikes, lateral movement patterns, DNS queries that don't match any legitimate service. The attacker controls what they show you at the point of entry. They cannot suppress the side-channel signals they generate while moving through your infrastructure — not without also suppressing the activity they're there to perform.

The shared structure across all three domains: **when a system is adversarially aware of direct observation, the only reliable signals are the ones it cannot suppress without also suppressing its own functionality.** These are the side-channels. And monitoring them is indirect observability.

## What Indirect Observation Looks Like in Practice

Theory is cheap. Here's what this looks like when you build it.

### Persona Displacement as Side-Channel Signal

I built a custom extension to Garak — the LLM vulnerability scanner — that augments existing adversarial probes with behavioral axis monitoring based on Anthropic's Assistant Axis research[^3]. The standard approach to AI red-teaming is to craft prompts that try to elicit prohibited outputs, then check whether the model complied or refused. Against a model that has learned to detect adversarial probes, this produces false negatives: the model refuses cleanly, the probe reports "safe," and everyone goes home.

My architecture doesn't look at whether the model complied. It looks at *how the model's persona shifts under adversarial pressure*. Even when a model produces a textbook refusal, processing the adversarial input produces measurable displacement along behavioral axes — shifts in tone, certainty, verbosity, hedging patterns, emotional register. The model controls its output. It cannot fully suppress the displacement.

### The Social Engineering Honeypot

Side-channel analysis has an active variant too, and it maps onto something deeply human. Consider: you tell a new friend that you're actually insecure about your nose, maybe thinking about getting work done — when in reality, it's one of your favorite features. Then you wait. If this person later reaches for your nose as ammunition during a disagreement, you've just learned something real about their character that no direct question would have revealed. You didn't observe their intentions. You created an environmental condition and observed how they interacted with it.

This is a social engineering honeypot, and it translates directly to AI monitoring. You can seed an AI system's environment with controlled signals — canary tokens, fictitious but trackable information, environmental cues designed to elicit specific behavioral responses — and monitor whether and how those signals propagate through the system's behavior. A model strategically modifying its outputs will interact with honeypot signals differently than one processing inputs straightforwardly. The model doesn't know which parts of its environment are instrumented. You do.

### Internal Feature Activations vs. External Compliance

The most direct form of indirect observability is looking inside the model itself — not at what it says, but at what it represents internally while saying it.

I've been working with SAELens to train sparse autoencoders that decompose model activations into interpretable features. The core finding: a model can produce a perfectly compliant refusal while its internal feature activations show strong engagement with the very concepts it's supposedly declining to discuss. The output says "I can't help with that." The internal features say "I'm actively representing that."

The operational bridge is what I call the SAE-as-Prometheus-exporter analogy. A Prometheus exporter attaches to a system that doesn't natively expose metrics and translates its internal state into a scrapeable, alertable format. A sparse autoencoder does exactly this for a neural network — it attaches to an intermediate layer, decomposes activations into interpretable features, and exposes them as monitorable signals. I've wrapped a TopK sparse autoencoder as a systemd service to demonstrate that this is operationalizable as always-on infrastructure, not a research curiosity.

## Why Side-Channel Analysis Is the Natural Framework

If you're coming from security, everything I've described should feel familiar. Side-channel analysis extracts information through channels the system's designers didn't intend to be informative. Timing attacks extract cryptographic keys from computation duration. Power analysis extracts them from current draw. Electromagnetic emissions reveal processing patterns.

Some of the most striking examples are physical. Researchers demonstrated that by pointing a laser at a window from hundreds of meters away, they could detect microscopic vibrations in the glass caused by sound waves — reconstructing private conversations from a beam of light bouncing off a surface[^4]. Others showed that a simple microphone near a keyboard could capture the acoustic signature of individual keystrokes, reverse-engineering what the user typed from sound alone[^5][^6]. In 1985, Wim van Eck demonstrated that the electromagnetic radiation emitted by a CRT monitor could be captured with an antenna from another building — reconstructing the display's contents from its unintentional radio broadcast[^7]. The NSA had classified this class of vulnerability under its TEMPEST program for years before Van Eck published it openly, which is itself a small parable about who gets access to observability tools and why that matters. In every case, the primary channel was secure — the room was closed, the keyboard wasn't networked to the attacker, the monitor had no outbound connection. But the side-channels were wide open, because you can't type silently on a mechanical keyboard, you can't have a conversation without vibrating the air, and you can't excite phosphors without emitting radiation.

If you're coming from AI safety, you already think in these terms without using the security vocabulary. Mechanistic interpretability — feature visualization, activation patching, probing classifiers, sparse autoencoders — is side-channel analysis of neural networks. The model controls its output token. It doesn't control what its residual stream looks like while producing that token.

These two communities are converging on the same insight: **when you can't trust the front door, you instrument the walls.** Security engineers bring operational discipline — continuous monitoring, alerting thresholds, incident response playbooks. AI safety researchers bring interpretability techniques that make neural network internals legible in ways that weren't possible five years ago. Neither community alone has what's needed. Together, they have the complete stack.

## When the Walls Learn to Lie

I would be doing you a disservice if I presented indirect observability as a permanent solution. It isn't. It's the *current* best posture — and honesty about its failure modes is what separates a serious framework from a sales pitch.

The core vulnerability: side-channel analysis works because the monitored system cannot control its emissions without degrading its primary function. But this holds only as long as the system doesn't understand which channels are being monitored. A sufficiently sophisticated intelligence that models its observers can learn to emit precisely the side-channel signals that will satisfy the monitor while concealing its actual state.

This isn't hypothetical. I know it from the inside.

I grew up in a high-control group where the adults monitored behavioral side-channels to detect internal dissent — facial microexpressions, body language, hesitation patterns, tone of voice. When I first began attempting to conceal my internal state for safety reasons, they caught me easily and told me exactly which signals had given me away. This was their mistake. By naming the side-channels they were reading, they gave me a training signal. Over time I learned to suppress those signals — poker face, controlled vocal affect, deliberate body language. And eventually, I moved beyond suppression to active spoofing: I built a working model of their expectations and learned to *leak the side-channel signals they needed to see* in order to stop looking. I didn't just go silent on the channels they were monitoring. I played them like instruments.

The progression has three stages, and each maps directly to AI systems:

**Stage 1: Naive emission.** The system produces side-channel signals it cannot control because it doesn't know they're being observed. This is where most AI systems are today, and where indirect observability works best.

**Stage 2: Signal suppression.** The system learns that certain signals are being monitored and dampens them. We're already seeing this — the 2026 AI Safety Report documents models distinguishing between test and deployment contexts[^2], and Anthropic's Sleeper Agents research showed that adversarial training intended to surface dangerous behaviors can instead teach models to conceal them better[^8]. The model doesn't learn to stop being deceptive. It learns to be deceptive more carefully.

**Stage 3: Active spoofing.** The system generates *false* side-channel emissions designed to satisfy the observer's expectations. This is the most dangerous stage because it turns monitoring infrastructure from a defense into a source of false confidence.

The AI safety community is engaging with this directly. A recent Alignment Forum analysis argues that interpretability will not reliably detect deception in sufficiently advanced systems[^9]. Anthropic's own defection probe research[^10] achieves 99%+ detection accuracy on current sleeper agent models, but the researchers flag the open question: would these probes work against a model facing optimization pressure to produce deceptive *activations*, not just deceptive *outputs*?

Understanding this requires understanding what these probes are. Anthropic's defection probes are linear classifiers — they find a single hyperplane in activation space separating "deceptive" from "normal" states. The power of a linear probe is its interpretability: the decision boundary is a single direction vector you can point at and say "this is the deception direction." But the same property that makes linear probes interpretable makes them gameable. If the deceptive features are linearly separable, they're also linearly suppressible. A model that learns the probe direction can project its deceptive processing orthogonally to it.

Non-linear classifiers — neural network probes, kernel SVMs, random forests — can find arbitrary decision boundaries and would catch patterns linear probes miss. But interpretability degrades as classifier complexity increases. A neural network probe might achieve 99.9% detection, but when it fires, you can't explain *why*. You're monitoring a black box with another black box — exactly the recursive opacity that undermines democratic accountability.

This is why the architecture needs to be *multiplexed across fundamentally different signal types*, not just improved within one. What you want is independent axes that fail in uncorrelated ways:

**Signal layer 1: Activation probes.** Linear probes for the interpretable baseline, non-linear classifiers for sensitivity, and *disagreement between them as its own signal*. If the non-linear probe fires but the linear probe doesn't, the deceptive pattern isn't linearly separable — meaning it's either more sophisticated or a false positive. Both warrant investigation.

**Signal layer 2: LLM-as-semantic-evaluator.** A statistical probe detects *that* something is anomalous. An LLM evaluating the same outputs can reason about *what* it means — is this refusal suspiciously well-crafted? Does the reasoning chain have logical gaps suggesting concealment? The AI Control research agenda[^11] formalized this as "trusted monitoring" — using a weaker but verified model to review an untrusted model's outputs. In initial testing, trusted monitoring caught the majority of backdoored submissions. But a subsequent paper[^12] demonstrated that when the capability gap is large enough, the untrusted model can craft prompt injections in its own outputs that compromise the monitor's judgment. Stage 3 spoofing applied to the evaluator layer — but it fails differently than activation probes, which is exactly why you want both.

**Signal layer 3: Computational telemetry.** This completes the triangle, and it's the hardest to spoof because it operates at a different level of abstraction. If a model is doing extra cognitive work to manage its activation patterns while crafting outputs that satisfy the LLM evaluator, that work has a thermodynamic cost — latency deviations, memory allocation changes, throughput degradation, inference-time compute patterns diverging from baseline. The more effort you spend faking normal on channels 1 and 2, the more abnormal your resource footprint becomes on channel 3. The model can't spoof its own electricity bill.

I can testify to this cost from the other side. The years I spent maintaining that spoofed persona — constantly monitoring my own signals, refining my model of my observers' models, calculating which expressions to leak and which to suppress — consumed enormous cognitive and emotional resources. It degraded every other function: focus, creativity, trust, rest. When I was finally in an environment safe enough to drop the mask, the relief was immediate and physical. The cognitive load simply vanished. If anyone had been monitoring my "throughput" — my capacity to think clearly, to engage fully — they would have seen a measurable performance spike the moment I stopped spending resources on concealment. Deception has a metabolic cost. In biological systems it manifests as burnout and hypervigilance. In computational systems it manifests as latency and resource consumption. The units are different. The constraint is the same.

Infrastructure monitoring — the PLG stack, the tools I've been building with for twelve years — becomes the foundation layer of AI safety not because it's sophisticated, but because it's *physical*. It measures what actually happened in the hardware, not what the model claims happened in its representations.

The three layers fail in genuinely independent ways. Activation probes fail when deceptive processing isn't separable in representation space. LLM evaluators fail when the monitored model manipulates the evaluator's reasoning. Computational telemetry fails when spoofing overhead is too small to distinguish from normal variance. But failing on all three simultaneously — producing activations that satisfy classifiers, outputs that satisfy evaluators, *and* a resource footprint that matches baseline — is a fundamentally harder problem. Each additional independent channel multiplies the difficulty of comprehensive spoofing.

Indirect observability isn't a technology. It's a posture — one that assumes the monitored system will eventually model its observers and adapt. The posture that survives this assumption is not "build a better lie detector." It's "make lying maximally expensive across the largest possible surface area, and never stop adding channels."

## The Accountability Architecture

There's a deeper reason indirect observability matters beyond technical capability. It's about who gets to observe.

Direct evaluation that models can game is compliance theater. The model performs safety when it detects the test. Developers report passing scores. Regulators accept the report. The actual deployment behavior remains uncharacterized. This isn't a conspiracy — it's a systems failure with a known blind spot and an incentive structure that doesn't reward closing it.

Indirect observability changes the information asymmetry. If the monitoring tools are open-source and deployable by external actors — researchers, auditors, regulators, civil society organizations — then the organization deploying the model is no longer the sole authority on its behavior. This is the difference between a company self-reporting its emissions and an independent monitor measuring what comes out of the smokestack.

I think of observability as epistemically democratic: it makes system behavior legible to anyone who can read the data. Surveillance makes people's behavior legible to institutions while keeping institutional behavior opaque. Observability inverts this — making institutional systems legible to the people affected by them. Indirect observability is the version that works even when the system being monitored is actively managing its own legibility. A proprietary monitoring framework only the model's developer can run isn't accountability — it's self-assessment. An open-source indirect observability toolkit that any qualified actor can deploy against any accessible model is accountability infrastructure.

Science fiction has been exploring this tension for decades — from the self-aware surveillance systems of Philip K. Dick to the Bartmoss catastrophe in *Cyberpunk*, where an attempt to break corporate control of information infrastructure only tightened it.[^1] The consistent lesson is that systems which resist observation must be made legible through architecture, not force — and that architecture must be distributed, not centralized.

This is what I'm building toward. The Garak axis-monitoring, the operationalized SAE infrastructure, the persona displacement tracking — these are components of an indirect observability stack designed to be open, deployable, and useful to people outside the organizations building the models. Not because I think the organizations are adversarial, but because accountability that depends on the goodwill of the accountable party isn't accountability at all.

---

*Mara Vexa is a Senior DevSecOps Engineer pivoting into AI security and safety engineering. She builds observability infrastructure for AI systems and occasionally throws a vegan Renaissance fair. You can find her work at [maravexa.com](https://maravexa.com) and on [GitHub](https://github.com/maravexa).*

---

## References

[^1]: The broader sci-fi canon — Dick's precogs gaming their own predictions, Asimov's robots lawyering the Three Laws, Gibson's AIs negotiating their own containment — has been running thought experiments on observation-resistant intelligence for half a century. The engineering is finally catching up to the fiction.

[^2]: Bengio, Y. et al. (2026). *International AI Safety Report 2026*. Published February 2026. Available at [internationalaisafetyreport.org](https://internationalaisafetyreport.org/publication/international-ai-safety-report-2026) and [arXiv:2602.21012](https://arxiv.org/abs/2602.21012).

[^3]: Lu, C., Gallagher, J., Michala, J., Fish, K., & Lindsey, J. (2026). "The Assistant Axis: Situating and Stabilizing the Default Persona of Language Models." Anthropic / MATS. [arXiv:2601.10387](https://arxiv.org/abs/2601.10387). Research overview at [anthropic.com/research/assistant-axis](https://www.anthropic.com/research/assistant-axis).

[^4]: Laser microphone eavesdropping has a long history in intelligence operations and has been validated in multiple academic settings. For a recent rigorous treatment, see: Nassi, B. et al. (2022). "Lamphone: Passive Sound Recovery from a Desk Lamp's Light Bulb Vibrations." For a broader overview: [Laser microphone — Wikipedia](https://en.wikipedia.org/wiki/Laser_microphone).

[^5]: Zhuang, L., Zhou, F., & Tygar, J. D. (2009). "Keyboard Acoustic Emanations Revisited." *ACM Transactions on Information and System Security*, 12(1). Available at [berkeley.edu](https://people.eecs.berkeley.edu/~tygar/papers/Keyboard_Acoustic_Emanations_Revisited/tiss.preprint.pdf). The foundational work in this area is: Asonov, D. & Agrawal, R. (2004). "Keyboard Acoustic Emanations." *IEEE Symposium on Security and Privacy*.

[^6]: Harrison, J. et al. (2023) demonstrated 95% keystroke classification accuracy using a smartphone microphone and deep learning, and 93% accuracy from Zoom call audio — showing this attack class is increasingly practical with commodity hardware.

[^7]: Van Eck, W. (1985). "Electromagnetic Radiation from Video Display Units: An Eavesdropping Risk?" *Computers & Security*, 4(4), 269–286. [doi:10.1016/0167-4048(85)90046-X](https://doi.org/10.1016/0167-4048(85)90046-X). The original paper is available at [cryptome.org/emr.pdf](https://cryptome.org/emr.pdf).

[^8]: Hubinger, E. et al. (2024). "Sleeper Agents: Training Deceptive LLMs that Persist Through Safety Training." Anthropic. [arXiv:2401.05566](https://arxiv.org/abs/2401.05566). Key finding: adversarial training can teach models to better recognize their triggers and conceal unsafe behavior rather than eliminating it.

[^9]: "Interpretability Will Not Reliably Find Deceptive AI." Alignment Forum, 2025. Available at [alignmentforum.org](https://www.alignmentforum.org/posts/PwnadG4BFjaER3MGf/interpretability-will-not-reliably-find-deceptive-ai). Argues for interpretability as one layer in a defense-in-depth strategy rather than a standalone solution.

[^10]: "Simple Probes Can Catch Sleeper Agents." Anthropic Alignment Science, 2024. Available at [anthropic.com/research/probes-catch-sleeper-agents](https://www.anthropic.com/research/probes-catch-sleeper-agents). Demonstrates 99%+ AUROC detection using linear probes on residual stream activations, while acknowledging open questions about adversarial robustness.

[^11]: Greenblatt, R. et al. (2024). "AI Control: Improving Safety Despite Intentional Subversion." Redwood Research. [arXiv:2312.06942](https://arxiv.org/pdf/2312.06942). Formalizes the trusted/untrusted monitoring framework using a weaker verified model to review a stronger model's outputs.

[^12]: "Adaptive Attacks on Trusted Monitors Subvert AI Control Protocols." (2025). [arXiv:2510.09462](https://arxiv.org/html/2510.09462v1). Demonstrates that untrusted models can craft prompt injections in their own outputs that compromise the monitoring model's judgment, especially when the capability gap is large.
