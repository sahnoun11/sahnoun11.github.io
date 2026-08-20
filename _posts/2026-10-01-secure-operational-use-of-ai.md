---
title: "Secure Operational Use of AI — Shadow AI, Governance, and Controls That Survive Real Users"
date: 2026-10-01 09:00:00 +0100
categories: [AI Security, AI Systems Security Specialist]
tags: [shadow-ai, ai-governance, nist-ai-rmf, eu-ai-act, data-leakage, dlp, ai-security, ciso, owasp-llm-top-10]
description: "Part 6 of the AI Systems Security Specialist series: the governance and operational side — shadow AI and the real breach numbers, the NIST AI RMF and EU AI Act, and the controls that actually survive contact with real users."
image:
  path: /assets/img/aisec/part6-operational.svg
  alt: Secure Operational Use of AI
pin: false
toc: true
comments: true
series: ai-systems-security-specialist
series_name: "AI Systems Security Specialist"
series_part: 6
series_total: 8
series_hub_url: "/posts/ai-systems-security-specialist-series-overview/"
---

> **Part 6 of the [AI Systems Security Specialist series](/posts/ai-systems-security-specialist-series-overview/).** The first five parts secured AI systems you build. This part is about AI your organisation *uses* — often faster than it secures it.
{: .prompt-info }

---

## TL;DR

The biggest AI security problem in most organisations in 2026 isn't a clever exploit. It's **shadow AI**: employees pasting sensitive data into consumer AI tools nobody approved, and teams shipping AI features nobody reviewed. The numbers are stark — per IBM's *Cost of a Data Breach 2025*, **20% of breached organisations were compromised through shadow AI**, adding **~$670,000** to the average breach, and **97% of organisations with an AI-related breach lacked proper AI access controls.**

This part is the governance and operational layer: what shadow AI actually costs, the frameworks that structure a response (NIST AI RMF, EU AI Act), and — the part that matters — the specific controls that survive contact with real users instead of being bypassed on day one.

---

## 1. Shadow AI: the real numbers

The primary source is IBM's *Cost of a Data Breach Report 2025*, and the figures are worth quoting exactly because they reframe the priority:

| Metric | Value |
|---|---|
| Breached orgs compromised via shadow AI | **20%** |
| Added cost per breach from shadow AI | **~$670,000** |
| Global average breach cost | $4.44M |
| US average breach cost | $10.22M |
| Orgs lacking any AI governance policy | **63%** |
| AI-breached orgs missing proper access controls | **97%** |
| Employees using consumer GenAI | 57% |
| Employees admitting they exposed sensitive data | **33%** |
| Employees using unapproved AI apps on work devices | 36% |

Read the employee rows together: a third of employees are using consumer AI, a third of *those* admit to leaking sensitive data into it, and over a third are doing it on work devices with unapproved tools. This is not a fringe risk. It's the median employee behaviour.

And it maps directly to Part 01's data-flow model. When an employee pastes a customer record or internal doc into a consumer chatbot, that data enters *someone else's* context window, *someone else's* logs, *someone else's* provider infrastructure and observability stack (Part 01, §12–13) — outside every control your organisation has.

### The upside is symmetric

The same IBM data shows the defensive payoff: organisations using AI *extensively in their own security operations* reduced breach costs by up to **$1.9M** and shortened breach lifecycles by roughly **80 days**. AI is both the risk and, deployed deliberately, part of the mitigation — which is Part 07's subject.

---

## 2. Why shadow AI happens (and why banning it fails)

Shadow AI isn't a discipline problem. It's an incentive problem: AI tools make people meaningfully more productive, the approved alternative is usually slower or absent, and the data-exposure consequence is invisible at the moment of use. Given that, a blanket ban doesn't eliminate shadow AI — it just drives it further off managed devices and out of your visibility.

> **The governing principle:** you cannot ban your way out of shadow AI. You provide a sanctioned path that's good enough that the shadow path isn't worth the friction, and you instrument the boundary so you can see what's crossing it.
{: .prompt-tip }

---

## 3. The frameworks

Two bodies of work structure a serious operational response.

### 3.1 NIST AI Risk Management Framework (+ AI 600-1)

The **NIST AI RMF** organises AI risk management into four functions — **Govern, Map, Measure, Manage** — and the **AI 600-1 Generative AI Profile** applies them specifically to generative AI. It's voluntary, it's practical, and it's the natural backbone for an internal AI governance programme:

- **Govern** — policy, roles, accountability. (The 63% who have none start here.)
- **Map** — inventory AI systems and uses, including the shadow ones. You can't manage what you haven't mapped — the same lesson as Part 01's architecture review, at organisational scale.
- **Measure** — assess risks (this is where Parts 01–05 plug in as the technical assessment).
- **Manage** — prioritise and apply controls.

There's also emerging work on an **agentic profile** for the RMF, extending it to the autonomy risks from Part 04.

### 3.2 EU AI Act

If you operate in or serve the EU, the **AI Act** is binding law, not guidance, with a risk-tiered structure (unacceptable / high / limited / minimal risk) and staged obligations. The prohibitions on unacceptable-risk uses and the initial GPAI obligations have already taken effect, with high-risk-system obligations phasing in — though note the timeline has been actively debated in 2026, with the "Digital Omnibus" proposing deferrals and simplification of some high-risk obligations.

> Because the EU AI Act timeline shifted during 2026, confirm the current dates against the official [EU AI Act timeline](https://artificialintelligenceact.eu/) before making compliance commitments. The staged structure is stable; specific application dates have moved.
{: .prompt-warning }

The practical point for a security team: governance frameworks give you the *structure* (map, classify, control, document) and, in the EU, the *legal requirement* to do the technical work the rest of this series describes.

---

## 4. Controls that survive real users

Frameworks tell you *what*. Here's the *how* — the operational controls that hold up when real people use them, ordered by how much risk they retire per unit of friction.

**1. Provide a sanctioned enterprise AI path.** An approved tool with a data-processing agreement, retention controls, and no-training-on-your-data guarantees. This is the single highest-leverage control because it removes the *reason* for shadow AI. Everything else assumes it exists.

**2. Data-loss prevention at the AI boundary.** DLP tuned for AI destinations — detecting sensitive data heading into AI tool domains and either blocking or redacting it. This is the technical enforcement of "don't paste customer data into ChatGPT," and it works whether or not the employee remembers the policy.

**3. Discovery and inventory.** You can't govern shadow AI you can't see. Network-level and endpoint discovery of AI tool usage, feeding the NIST **Map** function. Most organisations are stunned by the first inventory.

**4. Access controls on AI systems.** The 97%-missing-them statistic. Authentication, authorisation, and least privilege on every internal AI deployment — the Part 01 controls, applied as policy rather than per-project.

**5. Redaction before AI processing.** For sanctioned tools handling sensitive data, PII detection and redaction before the context window (Part 01, §12) — as an org-wide capability, not something each team reinvents.

**6. Governance policy with teeth.** Clear rules on what data may go into which tools, backed by the technical controls above. Policy without enforcement is the 63% who "have a policy" and still get breached.

**7. Runtime monitoring / AI-TRiSM.** The AI trust, risk, and security management market was projected to reach **$5.7B by 2026**, yet only about **23%** of organisations had implemented AI runtime controls. Runtime monitoring of AI interactions — inputs, outputs, tool calls — is the observability layer that makes the rest enforceable, and it's where the field is investing.

The through-line: **provide a good path, instrument the boundary, enforce deterministically.** Same philosophy as Part 03, moved up to the organisational layer.

---

## 5. Where this connects to the technical series

Operational security isn't separate from the technical work — it's the same controls at a different altitude:

- **Shadow AI leaks** are Part 01 data flows crossing outside your control. Redaction and DLP are the boundary controls.
- **Internal AI deployments** are exactly what Parts 01–05 assess. Governance decides *which* get assessed and *how thoroughly*.
- **Agent adoption** brings Part 04's excessive-agency risks into the enterprise; the Rule of Two becomes procurement and design policy.
- **AI in defence** (Part 07) is the payoff side of the same IBM data.

Governance without technical assessment is paperwork. Technical assessment without governance doesn't scale past the systems someone happened to look at. You need both.

---

## Key takeaways

- **Shadow AI is the dominant operational risk:** 20% of breaches, ~$670K added cost, 33% of employees admitting leakage, 63% of orgs with no governance policy (IBM 2025).
- **You can't ban your way out.** Provide a sanctioned path good enough to beat the shadow path's convenience, then instrument the boundary.
- **NIST AI RMF (Govern/Map/Measure/Manage) + AI 600-1** is the practical backbone; the **EU AI Act** is binding law in the EU — confirm its shifting 2026 timeline before committing.
- **The controls that survive real users:** sanctioned enterprise path, AI-boundary DLP, discovery/inventory, access controls, redaction, enforceable policy, runtime monitoring.
- **Same philosophy as the technical parts, one level up:** good path, instrumented boundary, deterministic enforcement.

---

## Coming next

**Part 07 — AI in the SOC**, on 8 October. The defensive payoff: agentic alert triage, detection engineering with LLMs, what the AI-SOC numbers actually say, and the failure modes that never make the vendor deck.

[Series hub](/posts/ai-systems-security-specialist-series-overview/) · [RSS](/feed.xml)

---

## Sources

- [Shadow AI: 20% of Breaches, $670K Cost (IBM Cost of a Data Breach 2025) — Shattered.io](https://shattered.io/shadow-ai-breaches-670k/)
- [Shadow AI Statistics: Key Data Points Every CISO Needs in 2026 — Airia](https://airia.com/blog/shadow-ai-statistics-key-data-points-every-ciso-needs-in-2026/)
- [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework)
- [NIST AI 600-1 Generative AI Profile — AI Governance Institute](https://aigovernance.com/policy/nist-ai-600-1-generative-ai-profile)
- [EU AI Act — official timeline and text](https://artificialintelligenceact.eu/)
- [EU AI Act Update: Timeline Relief, Targeted Simplification, and New Prohibitions — Covington Inside Privacy](https://www.insideprivacy.com/artificial-intelligence/eu-ai-act-update-timeline-relief-targeted-simplification-and-new-prohibitions/)
- [OWASP GenAI LLM Top 10 2026 — OWASP GenAI Security Project](https://genai.owasp.org/resource/owasp-genai-llm-top-10-2026/)
