---
title: "The AI Systems Security Specialist — Series Overview"
date: 2026-08-20 09:00:00 +0100
categories: [AI Security, AI Systems Security Specialist]
tags: [ai-security, llm, series, prompt-injection, rag, ai-agents, mcp, owasp-llm-top-10, ai-soc, threat-modeling, trust-boundaries]
description: "A nine-part weekly series on securing AI systems — from tokens and context windows to trust boundaries, agent tool abuse, red teaming, governance, and the AI-assisted SOC."
image:
  path: /assets/img/aisec/series-hub.svg
  alt: The AI Systems Security Specialist series
pin: true
toc: true
comments: true
---

> **Series hub.** This post is the index for a nine-part run I'm publishing one post a week. Bookmark it — every part gets linked here as it goes live.
{: .prompt-info }

---

## Why I'm writing this

Most of what gets published about "AI security" falls into one of two buckets.

The first is **hype**: breathless posts about how AI is going to end cybersecurity as we know it, with no technical content and no threat model. The second is **narrow tricks**: a clever jailbreak, a funny screenshot of a chatbot being convinced it's a pirate, a one-off prompt that leaked a system prompt. Both are entertaining. Neither helps you assess a real AI deployment when your manager drops one on your desk and asks whether it's safe to roll out.

What's missing is the middle: **the architecture**. Where the components sit, how data actually moves between them, which of those movements cross a trust boundary, and what breaks when one of them is compromised. That's the layer where real AI security work happens, and it's the layer this series is about.

I've been working through the AI systems security space for a while now — reading the primary research, tearing apart deployments, and going deep on the offensive side in my [Web LLM Attacks](/posts/Web-LLM-Attack/) posts. This series is the structured version of that, built the way I'd want to learn it if I were starting today.

---

## Who this is for

You'll get the most out of this if you're:

- A **penetration tester or red teamer** who keeps getting AI applications in scope and wants a repeatable methodology instead of improvising prompt injection payloads.
- A **blue teamer or SOC analyst** whose organisation just deployed a RAG chatbot and nobody can tell you what it's connected to.
- A **security engineer** being asked to review or sign off on an internal AI product.
- Someone **learning AI security seriously** rather than collecting jailbreak screenshots.

You do **not** need to be a machine learning engineer. You need to be comfortable with APIs, application architecture, reading logs, and thinking in terms of data flows and trust. If you can threat-model a web app, you can threat-model an AI system — the primitives are different, but the discipline is the same.

---

## What makes AI systems different

Before the roadmap, here's the thesis the whole series rests on. Four properties make AI systems genuinely different from the software you're used to securing:

**1. Instructions and data share one channel.** In a traditional application, code is code and input is input; they live in separate places and the CPU knows the difference. In an LLM, the system prompt, the user's message, a retrieved document, and the output of a tool call all arrive as the same undifferentiated stream of tokens. The model has no cryptographic way to tell them apart. This single fact generates most of the attack surface.

**2. Behaviour is probabilistic, not deterministic.** Traditional software: if X, then Y, 100% of the time. An LLM: if X, then *probably* Y. The same input can produce different outputs. You cannot write a test that proves the system will never do something, only tests that show it usually doesn't.

**3. The decision-maker is also the attack surface.** In an agentic system, the model decides which tools to call. An attacker who can influence what's in the context window can influence those decisions — and therefore the actions of the whole system.

**4. The blast radius scales with autonomy.** A prompt injection against a chatbot produces bad text. The same injection against an agent with `send_email`, `write_file`, and `query_database` produces real-world consequences before any human sees the output.

OWASP put the conclusion better than I can, in the framing that accompanied the 2026 edition of their LLM Top 10:

> *"Stop trying to build a model that cannot be fooled. Build the system around it, so that when the model is fooled — and it will be — nothing important breaks."*

That is the entire discipline in two sentences. Security in AI systems is an **architecture** problem, not a model-behaviour problem. Everything in this series follows from it.

---

## The roadmap

![Roadmap of the nine-part AI Systems Security Specialist series](/assets/img/aisec/series-roadmap.svg)
_Nine posts across five sections. Fundamentals first, then offence, then defence, then operations._

### Section 1 — Orientation

**Part 00 — Series overview** *(this post)*
Why architecture is the right frame, who the series is for, and how the parts fit together.

### Section 2 — AI Systems Security Fundamentals

**Part 01 — Inside the AI Stack** · *27 August 2026*
The foundation post, and the longest one. What an LLM actually is and isn't, tokens and context windows as a security surface, inference parameters, the five-layer stack, model endpoints, orchestrators and the agent loop, RAG's two pipelines, embeddings and vector stores, the principal hierarchy, trust boundary mapping, sensitive data flows, and logging as a secondary exposure surface. Ends with a full worked architecture review of a deliberately vulnerable RAG chatbot.

**Part 02 — Prompt Injection & Abuse Techniques** · *3 September 2026*
The offensive counterpart. Direct and indirect injection, the tokenizer-level tricks that defeat string filters, multi-modal payloads, encoding evasion, context flooding, system prompt extraction, and the exfiltration channels that turn a text-output bug into a data breach. Built around real 2025–2026 incidents rather than toy examples.

### Section 3 — Securing & Assessing AI Systems

**Part 03 — Secure AI Systems Engineering** · *10 September 2026*
What actually works. Information-flow control, dual-LLM patterns, spotlighting, deterministic egress control, capability scoping, and the honest answer to "can prompt injection be fixed?" (No — but that's the wrong question.)

**Part 04 — AI Agents & Tool Abuse** · *17 September 2026*
Agent autonomy as a risk multiplier. The confused deputy problem at machine speed, MCP as an attack surface, tool poisoning and rug pulls, memory poisoning, multi-agent trust, and agent identity.

**Part 05 — AI Security Testing & Validation** · *24 September 2026*
Turning all of the above into a methodology. Scoping an AI engagement, building a threat model, the tooling landscape, why static benchmarks lie, adaptive adversary testing, and how to write findings that a developer can actually action.

### Section 4 — Operationalizing AI for Security

**Part 06 — Secure Operational Use of AI** · *1 October 2026*
The governance and shadow-AI side. What happens when your own organisation adopts AI faster than it secures it, and which controls survive contact with real users.

**Part 07 — AI in the SOC** · *8 October 2026*
Using AI defensively. Agentic alert triage, detection engineering with LLMs, what the numbers actually say about AI SOC performance, and the failure modes nobody puts in the vendor deck.

### Section 5 — Capstone

**Part 08 — Reviewing a Real AI System, End to End** · *15 October 2026*
Everything applied to a single target: reconnaissance, component mapping, data flow tracing, vector store inspection, trust boundary assessment, exploitation, and a report you could hand to a client.

---

## Ground rules for this series

A few commitments, so you know what you're getting:

**Everything is sourced.** Where I cite a statistic, a CVE, or a research result, there's a link. Where something is my own assessment or opinion, I say so.

**No live exploitation of third-party systems.** Every hands-on section targets deliberately vulnerable applications or local lab builds. Techniques are presented for defensive understanding and authorised testing. If you use them anywhere you don't have written permission to test, that's on you, and it's a crime in most jurisdictions.

**Architecture before payloads.** I'll show you attacks, but always after establishing why the architecture permits them. A payload you can copy is worth one engagement; understanding why it works is worth all of them.

**I'll be honest about uncertainty.** This field moves fast and a lot of it is unsettled. Where the research disagrees, or where I'm extrapolating, I'll flag it rather than flattening it into false confidence.

---

## Reference frameworks

Three bodies of work sit behind the series, and they're worth having open in a tab as you read:

| Framework | What it gives you | Where it fits |
|---|---|---|
| [OWASP GenAI LLM Top 10 (2026)](https://genai.owasp.org/resource/owasp-genai-llm-top-10-2026/) | Ranked risk taxonomy for LLM applications, cross-checked against thousands of real incidents | Vocabulary and prioritisation across every part |
| [MITRE ATLAS](https://atlas.mitre.org/) | ATT&CK-style tactics and techniques for adversarial AI, now including agentic coverage | Mapping findings to adversary behaviour (Parts 02, 04, 05) |
| [NIST AI RMF + AI 600-1](https://www.nist.gov/itl/ai-risk-management-framework) | Governance and risk management structure, plus a generative AI profile | Governance and reporting (Parts 06, 08) |

The 2026 OWASP list is worth internalising early, because the ordering itself tells a story:

| # | Risk | Movement from 2025 |
|---|---|---|
| LLM01 | Prompt Injection | — |
| LLM02 | Sensitive Information Disclosure | — |
| LLM03 | Excessive Agency | ▲ 3 |
| LLM04 | Supply Chain | ▼ 1 |
| LLM05 | Data and Model Poisoning | ▼ 1 |
| LLM06 | Unbounded Consumption | ▲ 4 |
| LLM07 | Misinformation | ▲ 2 |
| LLM08 | Hidden Context Exposure *(was System Prompt Leakage)* | ▼ 1 |
| LLM09 | Vector and Embedding Weaknesses | ▼ 1 |
| LLM10 | Improper Output Handling | ▼ 5 |

Look at what moved. **Excessive Agency** and **Unbounded Consumption** — both fundamentally about *blast radius* — climbed hard. **Improper Output Handling** — fundamentally about *sanitisation* — fell five places. The list is telling you that the field has stopped believing it can filter its way out of this problem and started building containment instead.

---

## Publishing schedule

One post a week, Thursdays, from 27 August through 15 October 2026.

| Date | Part |
|---|---|
| 20 Aug 2026 | 00 — Series overview *(you are here)* |
| 27 Aug 2026 | 01 — Inside the AI Stack |
| 3 Sep 2026 | 02 — Prompt Injection & Abuse Techniques |
| 10 Sep 2026 | 03 — Secure AI Systems Engineering |
| 17 Sep 2026 | 04 — AI Agents & Tool Abuse |
| 24 Sep 2026 | 05 — AI Security Testing & Validation |
| 1 Oct 2026 | 06 — Secure Operational Use of AI |
| 8 Oct 2026 | 07 — AI in the SOC |
| 15 Oct 2026 | 08 — Capstone: Reviewing a Real AI System |

If you want them as they land, the [RSS feed](/feed.xml) is the reliable way. You can also find me on [GitHub](https://github.com/sahnoun11) and [X](https://twitter.com/Sahnounoussama5).

Part 01 goes live next Thursday. It's the longest thing I've written for this blog, and it's the one everything else depends on.

See you then.

---

## Sources

- [OWASP GenAI LLM Top 10 2026 — OWASP GenAI Security Project](https://genai.owasp.org/resource/owasp-genai-llm-top-10-2026/)
- [OWASP 2026 LLM Top 10: "The model will be fooled" — Help Net Security](https://www.helpnetsecurity.com/2026/08/06/owasp-2026-llm-top-10-released/)
- [OWASP LLM Top 10 (2026): What Changed and How to Test — HackerDNA](https://hackerdna.com/blog/owasp-llm-top-10)
- [MITRE ATLAS](https://atlas.mitre.org/)
- [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework)
