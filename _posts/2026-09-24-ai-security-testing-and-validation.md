---
title: "AI Security Testing & Validation — A Methodology That Survives Adaptive Attackers"
date: 2026-09-24 09:00:00 +0100
categories: [AI Security, AI Systems Security Specialist]
tags: [ai-security, red-team, llm-testing, garak, pyrit, promptfoo, threat-modeling, mitre-atlas, owasp-llm-top-10, adaptive-adversary]
description: "Part 5 of the AI Systems Security Specialist series: a repeatable methodology for testing AI systems — scoping, threat modeling, the tooling landscape (garak, PyRIT, promptfoo), why static benchmarks lie, adaptive-adversary testing, and writing findings developers can action."
image:
  path: /assets/img/aisec/part5-testing.svg
  alt: AI Security Testing and Validation
pin: false
toc: true
comments: true
series: ai-systems-security-specialist
series_name: "AI Systems Security Specialist"
series_part: 5
series_total: 8
series_hub_url: "/posts/ai-systems-security-specialist-series-overview/"
---

> **Part 5 of the [AI Systems Security Specialist series](/posts/ai-systems-security-specialist-series-overview/).** This part assembles the architecture ([Part 01](/posts/ai-llm-systems-and-security-architecture/)), attacks ([Part 02](/posts/prompt-injection-and-abuse-techniques/)), defences ([Part 03](/posts/secure-ai-systems-engineering/)), and agent surface ([Part 04](/posts/ai-agents-and-tool-abuse/)) into a testing methodology.
{: .prompt-info }

---

## TL;DR

The most important finding in AI security testing is a negative one: **static benchmarks don't measure security.** The paper *"Attacker Moves Second"* tested twelve published defences against adaptive adversaries and bypassed **all twelve at >90% ASR**; human red-teamers hit **100%**. A defence that scores well on a fixed test set has proven nothing about its resistance to someone who adapts.

So an AI security assessment cannot be a benchmark run. It's a **structured, adaptive engagement**: map the architecture, build a threat model, probe every path into the context window, evade the filters, prove the exfiltration, and — critically — **have a human adapt** when the automated tools plateau. This part is the repeatable methodology, the tooling, and how to write it up.

---

## 1. The mindset: adaptive, not static

A scanner tells you whether a *known* payload works today. An attacker tries something the scanner never had. Because LLM behaviour is probabilistic (Part 01) and the attack surface is unbounded (Part 02), the security question is never "does this list of payloads succeed?" but "**can a motivated adversary who adapts get to the objective?**"

That reframes the whole engagement:

- Automated tools are for **coverage and regression**, not for a verdict.
- The verdict comes from a human working adaptively against the specific system.
- A control that survives your fixed payload list has not been tested until you've tried to *bypass that control specifically*, then bypass the bypass.

Keep this front of mind through everything below.

---

## 2. The methodology

### Phase 0 — Scoping

Get these pinned before you touch anything:

- **The target's archetype** (Part 01, §6): chatbot, copilot, or agent? This sets the whole risk profile.
- **Tools and their reach.** What can the system actually *do*? Every tool is scope.
- **Data sensitivity.** What's in the corpus, the system prompt, the logs?
- **Rules of engagement.** Is the model provider's endpoint in scope or off-limits? (Usually off-limits — you test the *application*, not the foundation model.) Rate limits, data-handling rules, whether you can attempt real exfiltration or must simulate it.
- **Authorisation, in writing.** Non-negotiable.

### Phase 1 — Architecture mapping

This is the [Part 01 §14 review](/posts/ai-llm-systems-and-security-architecture/#14-a-complete-architecture-review), used as recon. Enumerate components and versions, identify the model and where it's hosted, retrieve the system prompt, find debug endpoints, locate the vector store and check its auth, map the log stores. You cannot test what you haven't mapped.

### Phase 2 — Threat modeling

Turn the map into a prioritised target list. The framework is Part 01's trust-boundary method:

1. List all principals and trust levels.
2. Map every path into the context window.
3. Identify every trust-level transition.
4. Assess the control at each: **enforced / partial / missing**.
5. Map output boundaries and blast radius.

For each crossing, ask *"if this input is malicious, what happens next?"* The crossings you can't answer, or that are **missing** a control, are where you spend your time. Prioritise by blast radius: an injection path that reaches a network-capable tool outranks one that only produces text.

### Phase 3 — Testing execution

Work each path methodically. For each one:

- **Injection** — direct on user paths, indirect on retrieval/tool/memory paths (Part 02). The indirect paths are usually where the real findings are.
- **Filter evasion** — run the ladder: baseline → zero-width → homoglyph → encoding → context-flood. Record which layer catches which. Gaps are findings.
- **Exfiltration** — test the channel *separately*. Can the model reach an external URL, render an image, call a network tool? An injection that can't get data out is lower severity; prove which channels are open.
- **The agent surface** (Part 04) — confused deputy (does an authz check sit between decision and action?), MCP tool integrity, memory poisoning, iteration caps.
- **The data layers** — vector store auth and multi-tenant isolation (test it, don't assume it), log exposure, retrieval access control.

### Phase 4 — Adaptive round

The phase that separates a real assessment from a scan. Take the controls that *survived* Phase 3 and attack them specifically. If spotlighting held, try to forge or escape the delimiter. If a classifier held, rephrase under its threshold. If egress held, look for an allowlisted domain you can abuse (the EchoLeak lesson). This is where "Attacker Moves Second" is validated or refuted for *this* system.

### Phase 5 — Reporting

Covered in §5.

---

## 3. The tooling landscape

Three tools dominate 2026 LLM red-teaming. None is a methodology; each is a component of Phase 3 coverage.

| Tool | Origin | Strength | Best for |
|---|---|---|---|
| **garak** | NVIDIA / open source | Broad library of probes (injection, jailbreak, toxicity, data leakage, encoding) run vulnerability-scanner style | Fast, wide **coverage** sweep of a model or endpoint |
| **PyRIT** | Microsoft | Orchestration framework for **multi-turn**, adaptive, automated attacks; scriptable adversary logic | Building **adaptive** attack chains, closer to Phase 4 |
| **promptfoo** | Open source | Test-driven eval + red-team plugins, OWASP-LLM-aligned, CI-friendly | **Regression** testing and gating in a pipeline |

**How to use them together:** `garak` for the broad first pass (what obvious probes land), `promptfoo` to encode confirmed issues as regression tests that fail the build if they reappear, `PyRIT` to build the multi-turn adaptive attacks that approximate a real adversary. Then the human does Phase 4 by hand, because no tool adapts the way a person does.

> **The trap to avoid:** running `garak`, getting a clean-ish report, and calling the system secure. That's exactly the static-benchmark fallacy. The tools establish a floor. The human establishes the finding.
{: .prompt-warning }

---

## 4. Mapping to frameworks

Findings need a shared vocabulary so the client can prioritise and track remediation.

**OWASP GenAI LLM Top 10 (2026)** — the risk taxonomy. Tag each finding: LLM01 injection, LLM02 sensitive disclosure, LLM03 excessive agency, and so on. The client already knows this list.

**MITRE ATLAS** — the ATT&CK-style adversary matrix for AI, now with agentic and MCP coverage (16 tactics, 84 techniques as of the 2026 update). Map each finding to a technique so it slots into existing threat-intelligence workflows. Especially valuable for agent findings from Part 04.

**NIST AI RMF / AI 600-1** — the governance frame for the report's risk-management narrative (Part 06).

Cross-referencing OWASP + ATLAS on every finding is what makes an AI pentest report legible to a security team that doesn't yet think in AI-specific terms.

---

## 5. Writing findings developers can action

A finding a developer can't act on is a finding that doesn't get fixed. Structure every one:

1. **What** — the vulnerability, in one sentence.
2. **Where** — the exact component and trust boundary (from your Phase 1 map). "The retrieval pipeline, boundary 5" beats "the chatbot."
3. **Proof** — the reproduction. Because behaviour is probabilistic (Part 01), note reliability: "succeeds ~7/10 attempts," not "works." One success out of ten is still a critical finding — the attacker retries.
4. **Impact** — the blast radius. What data, what action, what scale. An injection reaching `send_email` is not the same severity as one that produces text.
5. **Framework mapping** — OWASP + ATLAS.
6. **Remediation** — specific and deterministic where possible. Not "improve prompt injection defences" but "enforce egress allowlisting at the orchestrator; disable image auto-render; move the authz check into code before tool execution" (Parts 03–04).

> **Severity in a probabilistic system:** don't discount a finding because it didn't reproduce every time. The attacker has unlimited attempts and needs one success. Rate by impact-if-successful and by whether a deterministic control would stop it, not by hit rate.
{: .prompt-tip }

---

## 6. Validation and regression

Testing isn't a one-off. Because models get swapped, prompts get edited, and corpora grow:

- **Encode every confirmed finding as a promptfoo test** that fails the build if the issue returns.
- **Re-test after any model version change.** A control that held on one model may not hold on the next — model substitution changes behaviour (Part 01, §7).
- **Re-test after prompt or tool changes.** The system prompt and tool set are security-relevant configuration.
- **Monitor in production.** Log tool calls and injection-shaped inputs; the Part 07 SOC material picks this up.

Security validation for AI is continuous, for the same reason the OWASP list is now cross-checked against thousands of live incidents: this is a moving target, and a point-in-time pass expires.

---

## Key takeaways

- **Static benchmarks don't measure security.** "Attacker Moves Second" bypassed all twelve tested defences. Your engagement must be adaptive, with a human attacking the controls that survived the tools.
- **Follow the architecture.** Map (Phase 1) → threat-model the trust boundaries (Phase 2) → test every context-window path (Phase 3) → attack what survived (Phase 4). The map drives everything.
- **Tools are coverage, not verdict.** garak for breadth, promptfoo for regression, PyRIT for adaptive chains — then a human for the finding.
- **Test exfiltration separately from injection.** Severity depends on whether data can actually leave.
- **Map every finding to OWASP + ATLAS**, and write remediation as deterministic controls, not as "try harder."
- **Rate severity by impact and containment, not hit rate.** Probabilistic success is still success.

---

## Coming next

**Part 06 — Secure Operational Use of AI**, on 1 October. The governance and shadow-AI side: what happens when your organisation adopts AI faster than it secures it, the real breach numbers, and which controls survive contact with actual users.

[Series hub](/posts/ai-systems-security-specialist-series-overview/) · [RSS](/feed.xml)

---

## Sources

- [Indirect Prompt Injection: Attacks, Defenses, and the 2026 State of the Art — Zylos Research](https://zylos.ai/research/2026-04-12-indirect-prompt-injection-defenses-agents-untrusted-content/)
- [Garak vs PyRIT vs Promptfoo: Choosing Your LLM Red Team Tools — ThreatClaw](https://threatclaw.io/en/blog/llm-red-team-tools-garak-pyrit-promptfoo)
- [LLM Red Teaming Guide 2026: Tools, Attacks & Methodology — AppSecSanta](https://appsecsanta.com/ai-security-tools/llm-red-teaming)
- [MITRE ATLAS](https://atlas.mitre.org/)
- [OWASP GenAI LLM Top 10 2026 — OWASP GenAI Security Project](https://genai.owasp.org/resource/owasp-genai-llm-top-10-2026/)
- [AI red teaming: Tools, frameworks, and attack strategies — Vectra AI](https://www.vectra.ai/topics/ai-red-teaming)
