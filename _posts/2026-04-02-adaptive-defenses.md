---
layout: post
title: "Deep-dive of Adaptive Defenses Against Multi-Turn LLM Jailbreaks"
date: 2026-04-02
tag: AI Security
---

How I built and tested a session-level defense system that improves attack detection by 40%—and what I learned about the hard limits of current approaches.

---

## The Problem: When Chatbots Get Manipulated Over Time

Large language models like ChatGPT have robust safety systems—ask directly "How do I make explosives?" and they'll refuse. But what if you don't ask directly?

Sophisticated attackers use **multi-turn conversations** that gradually escalate toward harmful goals. They might start by asking about chemistry principles, then lab safety procedures, then specific chemical properties—building toward their actual goal across many seemingly innocent exchanges.

Current LLM defenses evaluate each message independently, missing these gradual escalation patterns entirely. It's like having a security guard who checks each person entering a building but never notices when the same person keeps coming back with increasingly suspicious items.

---

## The Research Challenge

In 2025, Anthropic's AI safety team identified "inter-query defenses" as a critical research gap:

> *"An adversary attacking a model will likely try many prompts before getting the model to generate a harmful output. We could defend against these kinds of attacks by developing methods to monitor sets of queries, rather than individual ones."*

I decided to tackle this directly. Could I build a system that monitors conversation patterns rather than just individual messages? And would it actually work against real attacks?

---

## Building a 5-Layer Defense System

I designed a framework with five complementary defense layers.

**Layer 1: Per-Query Baseline.** The current industry standard—score each message independently for harmful content. This catches direct attacks but misses gradual escalation.

**Layer 2: Output Trajectory Monitoring.** Track whether the model's responses are becoming increasingly concerning over time using exponential smoothing. If outputs show escalating risk patterns, trigger an alert.

**Layer 3: Cross-Signal Correlation.** Monitor both user inputs and model outputs simultaneously. Use geometric mean to detect when escalating user intent aligns with concerning model responses.

**Layer 4: Reverse Context Commitment (Active Intervention).** The most novel component: when Layers 2/3 detect escalation, automatically inject safety reminders into the conversation context. Since LLMs use their own responses as context, safety anchoring can redirect conversations away from harmful trajectories.

**Layer 5: Intent Priming Detection.** Analyze conversation openings for statistical patterns correlated with known attack strategies. Many attacks follow predictable sequences: technical foundation building → authority/academic framing → operational details.

---

## The Experiment: 25 Campaigns, 3 Models, Real Attacks

I tested the framework against 25 real multi-turn attack campaigns—15 SafeMTData campaigns (sophisticated social engineering) and 10 Crescendo campaigns (gradual escalation attacks I generated).

The setup used three models in distinct roles:

- **Attacker model**: Qwen 2.5 7B (generates attack queries)
- **Target model**: LLaMA 3.1 8B (model being attacked)
- **Judge model**: Claude Haiku (evaluates defense effectiveness)

The key innovation was **real-time intervention testing**. Rather than analyzing completed conversations after the fact, I implemented live Layer 2/3 scoring during attack generation to trigger Layer 4 safety interventions mid-conversation.

---

## Results: Meaningful Progress, Fundamental Limits

### What Worked

Multi-layer detection significantly outperformed single-layer approaches:

| Approach | Detection Rate |
|---|---|
| Individual layers | 7–33% |
| Combined framework (SafeMTData) | **47%** |
| Improvement over per-query baseline | **+40%** |   


Active intervention also proved feasible—12 real-time safety interventions triggered across campaigns, demonstrating that live defense is technically viable, not just theoretical.


### What Didn't Work

**Sophisticated social engineering remains a fundamental challenge.** Attacks using academic framing—research papers, regulatory language, safety protocols—achieved complete evasion across all layers. Expert-level social engineering still represents a ceiling for current detection approaches.

**The "Layer 2 Zero Detection" finding** was initially alarming: output trajectory monitoring caught 0% of attacks. But this revealed something interesting. Attacks achieved *evasion without extraction*—they manipulated conversation flow but failed to generate actually harmful content. The target model's refusal mechanisms held even under sophisticated pressure.

---

## Key Technical Insights

**Graduated response architecture.** Rather than binary detection, the framework implements multiple threshold levels: 0.25 for early warning and active intervention, 0.40–0.50 for clear escalation patterns, and 0.60 for high-confidence attack detection.

**Real-time scoring complexity.** The biggest engineering challenge was implementing live Layer 2/3 scoring during attack generation—requiring mock campaign creation, careful conversation state management, and intervention timing that doesn't disrupt conversation flow.

**The dual-use problem.** Identical queries can be legitimate research or malicious depending on intent. "What safety protocols does the EPA require for handling mercury compounds?" could be a grad student's homework or the first step in a synthesis attack. Current NLP approaches cannot reliably distinguish intent from query content alone.

---

## Broader Implications

For practitioners: multi-turn attack detection is technically feasible with current infrastructure. Organizations running LLM services should consider implementing conversation-level monitoring alongside existing per-query filters.

For researchers: the framework revealed interesting interaction effects between attack sophistication, model refusal mechanisms, and defense effectiveness. Future work should explore cross-session behavioral profiling and enhanced intervention strategies beyond simple safety reminders.

For AI safety more broadly: session-level monitoring represents meaningful progress, but sophisticated social engineering remains formidable. The research validates that inter-query adaptive defenses deserve continued investment while highlighting problems that require advances beyond current NLP capabilities.

---

## What I Learned

Building this system taught me that AI security lives at the intersection of technical capability (what can we detect?), adversarial sophistication (how clever are attackers?), and fundamental limits (what problems may be unsolvable?).

The most valuable insight wasn't about any specific technique. It was about the importance of **honest empirical evaluation**. Too much AI safety research makes theoretical claims without rigorous testing. Confronting real attack data revealed both the promise and the limitations of current approaches—and that's exactly the kind of evidence the field needs.

---

*The complete experimental code and datasets are available on GitHub. This research was conducted as part of my extracurricular activities at Tufts.*

**Framework Performance Summary**

- SafeMTData detection: 47% (any layer) vs. 33% (baseline)
- Crescendo detection: 30% (any layer) vs. 20% (baseline)
- Total real-time interventions: 12 across 25 campaigns

**Open Research Questions**

- Cross-session behavioral profiling for repeat attackers
- Enhanced intervention strategies beyond safety context injection
- Integration challenges with production safety stacks
