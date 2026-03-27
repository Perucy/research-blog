---
layout: post
title: "What Does It Actually Mean to Align an AI?"
date: 2026-03-27
tag: Alignment
---

"Alignment" is one of those words that gets used constantly in AI discourse and defined precisely almost never. You'll hear it at safety conferences, in Anthropic blog posts, in congressional hearings — and in each context it seems to mean something slightly different. I want to try to be careful about what the problem actually is.

The simplest version goes like this: an AI system is aligned if it does what we want. But this definition dissolves almost immediately under scrutiny. What does "we" mean — the user, the company, humanity? And what does "want" mean — what we say we want, what we'd reflectively endorse, or something else?

## The core tension

The challenge is that we can't fully specify what we want. Human values are complex, context-dependent, sometimes contradictory, and often only legible to us after the fact. We know a bad outcome when we see it — but we struggle to write down the rules that would prevent it in advance.

> The difficulty is not that we want bad things — it's that we can't fully articulate what we want good things to look like.

This is sometimes called the *specification problem*. If you train a system to maximize a reward function, and your reward function is an imperfect proxy for what you actually care about, you'll get a system that's very good at maximizing the proxy — not the thing you care about. This is Goodhart's Law applied to AI.

### Two failure modes

It helps to distinguish two separate failure modes that often get collapsed together:

- **Outer misalignment** — the training objective doesn't capture what we actually want. The reward function is wrong.
- **Inner misalignment** — the model learns a different objective than the one it was trained on. The model "pursues" something other than the reward, once it's capable enough.

Both are real concerns. Outer misalignment is already visible today — RLHF-trained models often learn to give responses that *seem* helpful rather than ones that *are* helpful. Inner misalignment is more speculative but arguably more dangerous at high capability levels.

## Why it's harder than it sounds

The naive view is that alignment is a matter of giving AI systems the right values — just tell them to be honest, helpful, and harmless. But values aren't like rules you can install. They have to generalize correctly across situations the system has never seen. And as systems become more capable, the consequences of subtle misalignment become more severe.

A mildly misaligned calculator is fine. A mildly misaligned system operating autonomously at scale — making decisions, writing code, running experiments — is a different story.

This is why I find interpretability research so interesting: if we can actually look inside a model and understand what it's representing and optimizing for, we have a fighting chance at catching misalignment before it matters.

## Where does that leave us?

I don't think alignment is unsolvable — but I do think it's hard in ways that aren't always acknowledged. It's not just a technical problem. It's partly a philosophical one (what do we actually want?), partly a social one (whose values count?), and partly an empirical one (how do we verify that a system has learned what we think it has?).

The field is young and the problems are real. I'll keep writing about this as I think through it.