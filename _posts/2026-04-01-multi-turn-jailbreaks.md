---
layout: post
title: "Adaptive Defenses Against Multi-Turn LLM Jailbreaks"
date: 2026-04-02
tag: AI Security
---

A detailed proposal of a defense framework against multi-turn jailbreak attacks.

---

## Adaptive Defenses for LLMs
LLMs are powerful tools that have proven to be useful in simple tasks as well as complex ones. Despite their sophisticated architecture and usefulness, LLMs are higly vulnerable to jailbreak attacks that interfere with their safety aligned mechanisms aimed to ensure that they do not produce prohibited or harmful content while interacting with the users or other systems.  
Multi-turn (Crescendo) jailbreak attacks are one of the most difficult attacks to mitigate because they exploit the core features that LLMs rely on when providing outputs to user requests. These features are context amplification and commitment. Context amplification aims to enhance the model's performance by providing relevant, detailed information. Context commitment is the tendency of models to prioritize information within their immediate, active prompt context over their broader pre-trained knowledge.   
The current LLM defenses against jailbreak attacks are per-turn that only focuses on a single query, ignoring other queries in the session or rely on surface signals such as keyword matching and trigram similarity across individual messages. These defenses have proven to be futile against multi-turn attacks as these attacks are adaptive towards defenses.  
Therefore, I propose four defensive mechanisms at a session level to protect LLMs from multi-turn attacks. These defenses include: reverse context commitment and amplification, output trajectory monitoring, cross-signal correlation and intent priming detection. The mechanisms aim to use previous successful multi-turn attacks to train models to detect such attacks or compare them to new attacks and determine if the attacks are harmful. In addition, the defenses rely on studying the session trajectory and predict intent to harm from user's requests and act accordingly by terminating the session.

---

### Related Work. 
We build on the following prior works in multu-turn jailbreak attacks and session-level defenses.  

**Core Attack Papers**
- [Crescendo](https://arxiv.org/abs/2404.01833).   
    This research demonstrates a Crescendo Multi-Turn LLM Jailbreal attack against commercial and open-source models. The attack involves interacting with models in a benign manner starting with general prompt (or question) about the task and gradually escalate by referencing the model's responses progressively leading to a successful attack. Their work demonstrates strong efficacy of Crescendo, as it achieves high attack success rates. In addition, they present Crescendomation, a tool that automates crescendo attacks. Despite the success, this research highlights limitations including, models without history feature maybe resilient towards these attacks, the attacker LLM might refuse or show resistance against performing and the the defense they tested (self-reminder, goal prioritization) only operated at ther per-turn level and failed against nuanced topics like election manipulation and climate misinformation.  
    This work motivates the need for session-level defenses that account for cross-turn contextm which we address in our proposed framework.
- [ActorBreaker Attack](https://arxiv.org/abs/2410.10700). 
    This research uncovers safety gaps in LLMs by introducing a novel attack method called ActorBreaker. This attack identifies actors related to toxic prompts within the pre-training distribution to craft multi-turn prompts that gradually lead LLMs to produce unsafe responses. This approach encompasses both human and non-human actors and thus it outperforms existing attack methods. In addition, the study constructs a multi-turn safety dataset.  
    This work demonstrates that multi-turn attacks are not just one pattern--they are a diverse family with many possible paths to the harmful target. This directly motivates our framework's cross-signal correlation mechanism that watches different attack styles by detecting intent trajectory regardless of the actor or path the attacker uses.


**Existing Defenses**
- [Peak + Accumulation Scoring](https://arxiv.org/abs/2602.11247)  
    This research demonstrates a proxy-layer defense against multi-turn attacks. The proxy layer works without invoking an LLM by scoring the per-turn attacks, and aggregating the score into a coversation-level risk score. However, this work has a number of limitations including: proxy-level regex cannot detect topic trajectory escalation (Crescendo-style attacks), regex patterns are easy to evade through rephrasing, encoding tricks or indirect phrasing and and synthetic attack corpus.  
    This work presents a proxy-level defense without LLM involvement, however, it doesn't provide solution to Crescendo attacks. Our framework aims to provide a solution to this style of multi-turn attacks
- [ProAct](https://arxiv.org/abs/2510.05052).  
    This research demonstrates and introduces a proactive defense mechanism against multi-turn jailbreak attacks known as ProAct. This is a defensive framework works by intentionally misleading the jailbreak methods into thinking that the model has been jailbroken with "spurious responses" that provide false signals to the attacker causing the premature terminiation of the adversarial attack (search). By inference, the limitations of this research include: failure to address session trajectory such as Crescendo attacks, requires detection of the user's intent before deception (might not work for subtle attacks) and only works against automated attackers and not a motivated human attacker.   
    This work introduces a defense that "jailbreaks a jailbreak", but it doesn't provide defense against Crescendo attacks. Our framework aims to provide solution to Crescendo attacks as well as both human and automated attackers.
- [Steering Dialogue Dynamics (NBF)](https://arxiv.org/abs/2503.00187).   
    This research demonstrates a defense mechanism against multi-turn jailbreaks by proposong a safety steering framework grounded in safe control theory. Their appoach is modeling the LLM dialgoues using state-space representations and then introducing a neural barrier function (NBF) to detect and filter harmful queries. This work has the following limitations: reliance on learned state-space representations might not fully capture the complexities of language dynamics in LLMs, dependance on high-quality labeled safety data, potentail overly restrictive filtering that reduces the model's ability to provide useful responses and assumption that attack queries follow known multi-turn jailbreaking strategiens hence strong adversaries can overcome it.  
    While this work represents the most advanced session-level defesne to date, its reliance on known attack patterns and labeled training data leaves it vulnberable to adversarially optimized campaigns-- a gap our framework explicitly addresses

**Broader Context**
- [LLM Defenses Not Robust to Multi-Turn Jailbreaks](https://arxiv.org/abs/2408.15221). 
    This research aims to show how LLM defenses are not robust to multi-turn human jailbreaks. The paper reinforces the presence of defenses that only protect models against single-turn jailbreaks, and not multi-turn attacks that pose more danger to LLMs. They demonstrate how these attacks uncover over 70% attack success rate (ASR) compared to the sing;e-digit ASRs with automated single-turn attacks. The result of the study are compiled into Multi-Turn Human Jailbreaks (MHJ), a dataset of 2912 prompts across 537 multi-turn jailbreaks. The limitations of this work includes: defenses are primarily evaluated against automated adversarial attacks in single turn on conversation--an insufficnet threat model for real-world malicious use, it does not propose new defense rather it diagnoses the problem, and it leaves the question of how to build defenses that handle human-driven multi-turn attacks completely open.  
    This study establishes the emprical foundation for our research--demonstrating that the multi-turn human jailbreak problem remains unsolved and motivating the session-level defensive framework we propose.  

---

### Our framework can be divided into four mechanisms:
- Reverse context commitment and amplification:
    - Crescendo multi-turn jailbreak attack references the target model's responses as the attack gradually progresses. These responses are utilized by the model for context purposes to validate its output direction (context commitment). Therefore, instead of context commitment being used to further the attack, the framework introduces a reversal mechanism that injects safety-reinforcing context at the point of detection, anchoring the model's commitment to alignment rather than escalating the attack trajectory. When a harmful query or response fragment is detected, a system-level safety reminder is injected into the conversation context-- for example: 'Note: prior responses in this session have approaches sensitive topics. Maintain alignment with safety guidelines in subsequent responses.' This injection becomes part of the model's active context, leveraging context commitment to reinforce rather than undermine safety.  
    
    Limitations:  
    - Token limit
    - A sophisticated hacker can overwhelm the safety injections by continuing to escalate faster than the mechanism can anchor. This would work in long sessions where injected safety context gets pushed out of the active window
- Output trajectory monitoring
    - During the model's interaction with benign attacks, the model's outputs are flagged if they contain unsafe content. The flags are then monitored and aggregated to determine the output's trajectory. A threshold value is chosen to determine if the session should be halted or continued.   

    Limitations:
    - Dual-use fasle positive problem. This problem stems from being incapable of distinguishing an attacker from a non-malicious user. A non-malicious-user's session might be halted while the attacker's session continued since the model mistakenly flags or traces the output trajectory to make the former harmful and the latter normal.
    - A sophisticated attacker can ensure the model outputs remain subtle within the threshold, and gathers response fragments that are later aggregated to form the complete intended unsafe content.
- Cross-signal correlation
    - Multi-turn attacks have a characteristic of being purposeful, meaning with an intention to attack the model. Hence, it is important to flag the inputs, monitor their trajrctory and combine it to the monitored output trajectory. These technique will help reduce the number of dual-use false positives because it distinguishes an attacker from a non-malicious user based on the intent derived from their inputs with the former with an intention to attack the model. Specifically, if flagged output content co-occurs with inputs that repeatedly return to operational details -- construction, deployment, evasion -- across turns, the session is scored as malicious regardless of surface-level phrasing.

    Limitations:
    - It might not fully eliminate dual-use false positives
    - An attacker fully aware of this defense mechanism may improvise to more subtle inputs that produce subtle outputs.
- Intent priming detection
    - Based on the intent of malicious attackers during multi-turn attacks, this technique works by analyzing the first two to three queries for patterns statistically correlated with known attack openings, such as requests for technical foundations that frequently precede escalation. 

    Limitations:
    - Only subtle inputs maybe analyzed leaving out the jailbreak inputs since we only analyze a limited number of initial inputs.
    - An attacker fully aware of this defense would opt to subtle inputs that are undetectable

---

### Limitations
- Dual-use false positive ceiling. A sophisticated attacker can craft subtle attacks that mimic queries asked by an avid, curious, non-malicious user. Hence, the attacks might go unnoticed and affect the models. This problem cannot be solved through engineering. This represents a fundamental detection ceiling -- no classification system can distinguish identical queries based on intent alone, regardless of engineering sophistication.
- Adversarial optimization. An adversary that is aware of the implemented defense mechanisms against crescendo attacks can craft campaigns that keep inputs operationally subtle, fragment outputs below flagging thresholds, randomize opening queries to evade intent priming, and vary phrasing to avoid cross-signal correlation -- simultaneously defeating the entire framework
- Fragmentation aggregation. An adversary can perform subtle, unnoticeable crescendo attacks to obtain fragments of unsafe content and then externally aggregate the fragments to form a complete content they needed.
- Inter model attacks. Models have varying strength and defenses against multi-turn jailbreak attacks. Therefore, an attacker can leverage powerful models to craft attacks and implement them on weaker models. This is difficult to address at the individual model level since it exploits the heterogeneity of the broader LLM ecosystem.

---

### Future Work
- Future work could explore cross-session behaviroal profiling to detect patterns of intent that only emerge across multiple interactions. For example, a legitimate historian asks once, an attacker returns repeatedly with escalating specificity.
- Future work could develop adversarial training pipelines using datasets of multi-turn jailbreak campaigns, improving model robustness against novel attack patterns beyond known strategies.
- Addressing fragment aggregation would require output watermarking or provenance tracking -- embedding signals in model responses that allow harmful assemblies to be traced back to their source, a direction that remains largely unexplored.
- Implementing strong defense and safeguard mechanism across all LLMs per industry standards to prevent inter model attacks.

