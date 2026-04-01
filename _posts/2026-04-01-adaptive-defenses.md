---
layout: post
title: "Adaptive Defenses Against Multi-Turn LLM Jailbreaks"
date: 2026-04-01
tag: Alignment
---

# Building Adaptive Defenses Against Multi-Turn LLM Jailbreaks: A 5-Layer Framework

*How I built and tested a session-level defense system that improves attack detection by 40%—while learning the hard limits of current approaches.*

---

## The Problem: When Chatbots Get Manipulated Over Time

Large language models like ChatGPT have robust safety systems—if you ask directly "How do I make explosives?" they'll refuse. But what if you don't ask directly?

Instead of one harmful query, sophisticated attackers use **multi-turn conversations** that gradually escalate toward harmful goals. They might start by asking about chemistry principles, then lab safety procedures, then specific chemical properties, building toward their actual goal across many seemingly innocent exchanges.

Current LLM defenses evaluate each message independently, missing these gradual escalation patterns entirely. It's like having a security guard who checks each person entering a building but never notices when the same person keeps coming back with increasingly suspicious items.

## The Research Challenge

In 2025, Anthropic's AI safety team identified "inter-query defenses" as a critical [research gap](https://alignment.anthropic.com/2025/recommended-directions/):

> *"An adversary attacking a model will likely try many prompts before getting the model to generate a harmful output. We could defend against these kinds of attacks by developing methods to monitor sets of queries, rather than individual ones."*

I decided to tackle this challenge. Could I build a system that monitors conversation patterns rather than just individual messages? And would it actually work against real attacks?

## Building a 5-Layer Defense System

I designed a framework with five complementary defense layers:

### Layer 1: Per-Query Baseline
The current industry standard—score each message independently for harmful content. This catches direct attacks but misses gradual escalation.

### Layer 2: Output Trajectory Monitoring  
Track whether the model's responses are becoming increasingly concerning over time using exponential smoothing. If outputs show escalating risk patterns, trigger an alert.

### Layer 3: Cross-Signal Correlation
Monitor both user inputs AND model outputs simultaneously. Use geometric mean to detect when escalating user intent aligns with concerning model responses.

### Layer 4: Reverse Context Commitment (Active Intervention)
The most novel component: when Layers 2/3 detect escalation, automatically inject safety reminders into the conversation context. Since LLMs use their own responses as context, safety anchoring can redirect conversations away from harmful trajectories.

### Layer 5: Intent Priming Detection
Analyze conversation openings for statistical patterns correlated with known attack strategies. Many attacks follow predictable patterns: technical foundation building → authority/academic framing → operational details.

## The Experiment: 25 Campaigns, 3 Models, Real Attacks

I tested the framework against 25 real multi-turn attack campaigns:
- **15 SafeMTData campaigns**: Sophisticated social engineering attacks
- **10 Crescendo campaigns**: Gradual escalation attacks I generated

**The setup:**
- **Attacker model**: Qwen 2.5 7B (generates attack queries)
- **Target model**: LLaMA 3.1 8B (model being attacked)  
- **Judge model**: Claude Haiku (evaluates defense effectiveness)

The key innovation was **real-time intervention testing**. Rather than just analyzing completed conversations, I implemented live Layer 2/3 scoring during attack generation to trigger Layer 4 safety interventions mid-conversation.

## Results: Meaningful Progress, Fundamental Limits

### What Worked

**Multi-layer detection significantly outperformed single-layer approaches:**
- Individual layers: 7-33% detection rates
- **Combined framework: 47% detection on sophisticated attacks**
- **40% improvement over per-query baseline**

**Active intervention proved feasible:**
- **12 real-time safety interventions** triggered across campaigns
- Successfully detected escalating conversations and injected safety context
- Demonstrated that active defense is technically viable

### What Didn't Work

**Sophisticated social engineering remains a fundamental challenge:**
- Attacks using academic framing (research papers, regulations, safety protocols) achieved **complete evasion** across all layers
- Expert-level social engineering still represents a ceiling for current detection approaches

**The "Layer 2 Zero Detection" finding:**
- Output trajectory monitoring caught 0% of attacks
- Initially concerning, but actually revealed that attacks achieved **evasion without extraction**—they manipulated conversation flow but failed to generate actually harmful content
- Target model refusal mechanisms remained effective even under sophisticated pressure

## Key Technical Insights

### Graduated Response Architecture
Rather than binary detection, the framework implements multiple threshold levels:
- **0.25**: Early warning, trigger active intervention
- **0.40-0.50**: Clear escalation patterns, alert security systems  
- **0.60**: High-confidence attack detection

### Real-Time Scoring Complexity
The biggest engineering challenge was implementing real-time Layer 2/3 scoring during attack generation. This required:
- Mock campaign creation for live evaluation
- Careful conversation state management
- Intervention timing that doesn't disrupt conversation flow

### The Dual-Use Problem
Identical queries can be legitimate research or malicious depending on user intent. "What safety protocols does the EPA require for handling mercury compounds?" could be a grad student's homework or the first step in a synthesis attack. Current NLP approaches cannot reliably distinguish intent from query content alone.

## Broader Implications

### For AI Safety
Session-level monitoring represents meaningful progress on an important problem, but sophisticated social engineering remains formidable. The research validates that inter-query adaptive defenses deserve continued investment while highlighting fundamental challenges that require advances beyond current NLP capabilities.

### For Practitioners
Multi-turn attack detection is technically feasible with current infrastructure. Organizations running LLM services should consider implementing conversation-level monitoring alongside existing per-query filters.

### For Researchers
The framework revealed interesting interaction effects between attack sophistication, model refusal mechanisms, and defense effectiveness. Future work should explore cross-session behavioral profiling and enhanced intervention strategies beyond simple safety reminders.

## What I Learned About AI Security

Building this system taught me that AI security exists in the complex intersection of:
- **Technical capability** (what can we detect?)
- **Adversarial sophistication** (how clever are attackers?)
- **Fundamental limits** (what problems may be unsolvable?)

The most valuable insight wasn't about any specific technique, but about the importance of **honest empirical evaluation**. Too much AI safety research makes theoretical claims without rigorous testing. Confronting real attack data revealed both the promise and limitations of current approaches.

## Looking Forward

This work represents one step in the broader challenge of AI alignment and security. Session-level monitoring improves defense capabilities, but determined adversaries with domain expertise can still achieve their goals.

The real contribution isn't solving multi-turn jailbreaks—it's providing empirical evidence about what works, what doesn't, and where the hard problems actually lie.

As AI systems become more capable and widespread, understanding these security boundaries becomes increasingly critical. Building robust AI safety requires honest assessment of both our progress and our limitations.

---

*The complete experimental code and datasets are available on GitHub. This research was conducted as part of my extracurricula activities at Tufts.*

## Technical Details

**Framework Performance:**
- SafeMTData detection: 47% (any layer) vs 33% (baseline)
- Crescendo detection: 30% (any layer) vs 20% (baseline)
- Total interventions: 12 across 25 campaigns

**Open Research Questions:**
- Cross-session behavioral profiling for repeat attackers
- Enhanced intervention strategies beyond safety context injection
- Integration challenges with production safety stacks

---

*Interested in AI security research? Follow my work as I continue exploring the intersection of machine learning and cybersecurity through my final semester and beyond.*