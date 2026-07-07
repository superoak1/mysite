*/ ---
title: "Bayesian Networks in Game AI"
date: 2026-05-12
tags: ["Blog", "AI", "Game Dev"]
draft: true
---

# Beyond Behavior Trees: The Case for Bayesian Networks in Game AI


## **Overview**

Game AI hasn't changed much in twenty years. Most NPCs — guards, enemies, companions — still run on the same basic logic they always have: a set of rules that says "if X happens, do Y." It works well enough, and honestly, there are good reasons it became the standard. But for developers interested in pushing what NPC behavior can be, there's a more expressive tool worth understanding.

This article makes the case for Bayesian Networks (BNs) — a tool from statistics and AI research that lets NPCs reason under uncertainty rather than simply react to triggers. It also proposes Psyche, a middleware platform that could bring this kind of AI to game developers without requiring a statistics degree to use it.
This isn't an argument that Bayesian Networks are strictly better than what exists, or that they should replace everything. They're a different tool with a different set of strengths — and like any tool, their value depends entirely on how and where you use them.

### **1. The Problem With How Game AI Works Today**

Most NPC behavior in games is built on one of two systems:

Finite State Machines (FSMs) — the older approach, around since the early 1990s. The NPC is always in one "state" (patrolling, alert, attacking) and switches between them based on specific triggers. Simple, reliable, and easy to debug. Once you know the triggers, the NPC is completely predictable — but in many games, that predictability is by design.

Behavior Trees — the more modern standard, popularized by Bungie in Halo 2 (2004) and later adopted by studios like Crytek. More flexible and better organized than FSMs, but sharing the same fundamental characteristic: they're deterministic. The same input always produces the same output.

It's worth being fair to these systems. That predictability is actually a feature — it makes AI easier to test, balance, and communicate to players. A guard who always investigates after three seconds is frustrating once you figure it out, but it also means designers know exactly what players are working against. Determinism has real value.

The limitation isn't that these systems are bad. It's that they start to struggle when behavior needs to depend on many interacting variables at once, or when you want NPCs to respond differently to the same situation depending on context, history, and personality. Scripting every possible combination of conditions becomes unwieldy fast. That's where a different approach starts to look attractive.

### **2. What Bayesian Networks Actually Are**

A Bayesian Network is a way of modeling how beliefs update when new information arrives. Instead of hard rules, it works with probabilities.
The core idea in plain terms:

You start with a belief about something. New evidence comes in. You update your belief based on how likely that evidence would be if your belief were true.

The math behind this is called Bayes' theorem:

**P(H | E) ∝ P(E | H) · P(H)**

**Breaking that down:**

P(H) — how likely you think something is before seeing any evidence (your "prior belief")

P(E | H) — how likely the evidence would be if your belief is correct

P(H | E) — how likely your belief is after factoring in the evidence (your "updated belief")

Applied to a guard NPC: instead of "heard noise → investigate," the guard asks "given that I heard this noise, how likely is it that something hostile is happening?" The answer depends on context — time of day, prior suspicion level, how loud the noise was, whether they've been deceived before.

Here's where it's important to be honest: a Bayesian Network can absolutely produce deterministic output. If you set up a network where "noise heard" always pushes the threat probability to 95%, the guard investigates every single time — functionally identical to a behavior tree. A BN isn't magic. Poorly designed, it produces the same mechanical behavior as anything else.

The real advantage isn't the framework itself — it's what the framework makes easier to build well. Three things specifically:

1. Handling many interacting variables without scripting every combination
A behavior tree can handle "heard noise → investigate." It struggles when the decision depends on noise volume and time of day and prior suspicion level and whether an ally just reported something and the guard's personality type. The number of conditions you'd need to script grows exponentially. A BN handles all of that within one structure — variables interact probabilistically, and you don't have to define every possible combination manually.

2. Reasoning from incomplete information
Behavior trees generally have access to everything the game engine knows. The determinism is partly a side effect of that complete information. A BN lets you deliberately give an NPC access to only partial information and have it reason from that incomplete picture. The uncertainty in the output reflects genuine uncertainty in the input — the NPC doesn't know everything, and its behavior reflects that.

3. Probabilistic intent sampling
When the network outputs a probability distribution over possible actions — say, 70% investigate, 20% call for backup, 10% ignore — you can sample from that distribution when deciding what actually happens. The same conditions can produce different outcomes on different occasions, without any randomness feeling arbitrary, because the probabilities reflect the actual weights of the situation.

None of these advantages are automatic. They require intentional design. But they're meaningfully harder to achieve with deterministic systems.

## **3. Where This Has Already Been Used in Games**

Probabilistic reasoning has appeared in games, though it's rarely labelled as such, and the claims you'll find online are often exaggerated. It's worth being specific about what's actually documented.

The Sims uses a probability-weighted utility system to decide what a Sim does next — it evaluates needs, weights options, and selects actions based on those weights rather than fixed rules. It isn't a Bayesian Network in the technical sense, but it works on the same underlying principle of probabilistic decision-making under competing priorities. The result is behavior that feels varied and organic across long play sessions.

Plague Inc. models disease transmission through a probabilistic population network, with transmission rates, resistances, and country responses all interacting dynamically. The developer worked with the CDC and WHO on accuracy. Again, not a Bayesian Network specifically — but the game's realism comes from the same core idea of interconnected probabilities rather than fixed rules.

In academic research, the applications are more direct and better documented. Synnaeve & Bessière (2010) built a Bayesian AI to control an avatar in World of Warcraft, training it to make combat decisions by learning from recorded human gameplay sessions. The same team later applied multi-scale Bayesian modeling to StarCraft, handling unit-level control, tactical positioning, and strategic planning within one probabilistic framework. These are research implementations, not shipping game features — but they demonstrate the approach working in real game environments.
The honest takeaway: proven commercial examples of Bayesian Networks in shipped game AI are limited. Most documented cases are either adjacent techniques (probabilistic but not strictly BN) or academic research. That's part of the argument for why the tooling gap matters — the approach hasn't had a real chance to be tested at scale in production.

## **4. What BN-Driven NPCs Could Actually Look Like**

The practical approach is a stack of focused networks, each handling a different part of NPC cognition, rather than one massive network trying to do everything:
Sensing  →  Believing  →  Personality  →  Intent  →  Action

**Sensing** — The NPC assigns a confidence level to what it perceives rather than a binary detected/not-detected. A noise heard from far away in a loud environment is weak evidence. The same noise up close in a quiet room is strong evidence. The same event carries different weight depending on conditions.

**Believing** — The NPC maintains running probability estimates about the world: Where is the player probably located? How much of a threat are they? Are my allies reliable? Is this area safe? These update as new information arrives, and they decay over time when no new information reinforces them.

**Personality** — A paranoid guard and a relaxed guard hear the same noise and reach different conclusions — not because they follow different scripts, but because their starting probability estimates differ. A paranoid NPC weights the same evidence more heavily toward threat. A cowardly one already leans toward retreat before anything happens. Personality is expressed through prior probabilities, not separate behavior logic.

**Intent** — The network outputs a distribution over possible actions rather than a single decision. That distribution is then sampled to determine what actually happens. This is where genuine behavioral variety comes from — not randomness for its own sake, but probability reflecting the actual weight of the situation.

**Action** — Standard game systems handle execution. Pathfinding, animation, combat — these don't change. The BN handles what the NPC wants to do. Keeping these concerns separate is what makes the approach computationally viable.

The social layer is where the design space gets particularly interesting. NPCs sharing information with each other — but with that communication modeled probabilistically — means trusted allies update your beliefs more than strangers, rumors degrade in accuracy as they pass through multiple NPCs, and a skeptical guard might not act on a panicked colleague's report. These dynamics emerge from the network without being individually scripted.

That said: all of this requires careful design. A poorly structured BN with bad priors produces NPC behavior just as unconvincing as a badly written behavior tree. The framework doesn't guarantee quality — it expands what quality can look like.

## **5. The Missing Infrastructure — Introducing Psyche**

The reason Bayesian game AI hasn't been tested seriously in production isn't that the math doesn't work. It's that the tooling to make it accessible doesn't exist.
Every probabilistic AI tool currently available was designed for researchers and statisticians. Game designers — the people who define how NPCs think and feel — have no practical way to work with these systems without significant technical expertise. That barrier is the real obstacle, not the underlying technology.
Psyche is a proposed middleware platform designed to address it. Three layers:

### **Psyche Studio — the design tool**

A designer describes an NPC in plain terms: "This guard is paranoid, loyal, and holds grudges." Studio translates those descriptors into a working probability network. Designers can run simulated scenarios — "What does this NPC do if they hear a noise but see nothing?" — and observe how beliefs resolve, without writing code or launching the game. A library of personality archetypes provides starting points that designers can adjust.

The key design challenge here is significant: translating natural language personality descriptors into mathematically meaningful prior distributions is a hard problem. The Studio layer would need careful validation to ensure "paranoid" in the designer's head corresponds to something sensible in the network. This is solvable, but it's not trivial.

### **Psyche Runtime — the engine inside the game**

The inference engine running in the background during gameplay. Calculations happen on background threads to avoid impacting framerate. The system scales by running full inference for nearby NPCs and lighter approximations for distant ones. NPCs can share evidence through simulated communication channels with configurable reliability.

It's worth noting: this runtime would need to be built to handle the performance constraints of real game environments, which are significantly more demanding than the research contexts where BN inference has been validated. Approximation methods would be necessary at scale, and those introduce their own tradeoffs in accuracy.

### **Psyche Insights — the debugging layer**

A toggleable overlay showing each NPC's belief state updating in real time during playtesting. Designers can see what each NPC believes, when beliefs shift, and why a decision was made — something essentially invisible in conventional behavior tree AI. A decision recorder allows replay of any NPC's reasoning history.
This layer might actually be the most immediately valuable part of the platform, even before the runtime is fully mature. Visibility into NPC cognition is useful regardless of the underlying AI architecture.

## **6. What This Changes for Players — And What It Doesn't**
The downstream effect of well-implemented BN-driven AI is NPCs that are harder to reduce to a pattern — guards whose suspicion accumulates across a session, factions that notice your playstyle over time, antagonists that adapt rather than repeat.
Deception becomes a real mechanic. If NPCs reason probabilistically about evidence, players can deliberately manipulate what they believe — planting sounds in the wrong location, using disguises, feeding misinformation through NPC social networks. That's a category of gameplay that scripted AI can approximate but not fully deliver.
The broader applications extend to other genres:

RPGs — Relationships that evolve based on accumulated history, not conversation flags
Strategy games — Enemy factions that read your playstyle and respond to it
Horror games — Antagonists that learn your behavioral patterns and exploit them
Simulation games — Populations where social dynamics and collective behavior emerge from the system rather than from scripts

It's important to be realistic about what this doesn't solve. Bayesian NPC AI is not going to make games feel like interacting with real people. It's a more expressive tool for a specific problem — the combinatorial complexity of context-dependent NPC behavior — not a general solution to immersion or narrative quality. A game with compelling characters still needs good writing, good animation, and good sound. The AI layer is one piece of a much larger picture.

## **7. The Challenges Worth Taking Seriously**
Several real technical problems stand between this approach and production use:

**Computational cost** — Exact inference in large networks is expensive. Keeping individual networks small and focused, and using approximation methods where needed, manages this — but approximation introduces accuracy tradeoffs that need to be validated per use case.

**Temporal modeling** — Games unfold continuously over time. Most BN inference is designed for discrete update steps, not continuous belief evolution. Handling belief decay, time-delayed evidence, and ongoing world changes in a way that feels natural is an open problem.

**Multi-agent scale** — Thousands of NPCs sharing evidence in a large open world introduces synchronization complexity that hasn't been fully worked out in real-time contexts.

**Designer accessibility** — Even with a well-designed tool, probabilistic systems are less intuitive to tune than behavior trees. A designer who can't predict what their network will do in a given situation can't design around it effectively. Tooling quality and training matter enormously.

These aren't reasons to abandon the approach. They're the actual engineering problems that would need to be solved to ship it. Acknowledging them honestly is more useful than pretending the path is straightforward.

## **8. Conclusion**

Behavior trees and finite state machines became the standard for good reasons. They're understandable, testable, and reliable. The argument here isn't that they should be discarded — it's that they leave a specific design space underexplored: NPC behavior that depends on many interacting variables, partial information, and context that's too complex to script exhaustively.

Bayesian Networks are a proven tool for exactly that problem, with documented applications in robotics, medicine, and — in limited but meaningful ways — games. The math works. What hasn't existed is the tooling layer that makes it practical for the people who build games to use.

Psyche is a proposal for what that layer could look like. The showcase project is the proof of concept. Whether it succeeds or fails, the experiment is worth running — because the alternative is another decade of guards who investigate after exactly three seconds.

## **References & Further Reading**

Pearl, J. (1988). Probabilistic Reasoning in Intelligent Systems: Networks of Plausible Inference. Morgan Kaufmann.

Millington, I. & Funge, J. (2009). Artificial Intelligence for Games, 2nd Ed. Morgan Kaufmann.

Synnaeve, G. & Bessière, P. (2010). "Bayesian Modeling of a Human MMORPG Player." 30th International Workshop on Bayesian Inference and Maximum Entropy, Chamonix, France. (arXiv:1011.5480)

Synnaeve, G. & Bessière, P. (2012). "Multi-scale Bayesian Modeling for RTS Games: An Application to StarCraft AI." INRIA / CNRS.

Tozour, P. (2002). "Fuzzy Logic and Bayesian Networks." AI Game Programming Wisdom. Charles River Media.

Isla, D. (2005). "Handling Complexity in the Halo 2 AI." Game Developers Conference Proceedings.

Norsys Software Corp. (2025). Netica Application & API Documentation. norsys.com
Iovino, M. et al. (2022). "A Survey of Behavior Trees in Robotics and AI." Robotics and Autonomous Systems, vol. 154.


***This article reflects independent research into probabilistic AI architectures for game development.*** \* 
