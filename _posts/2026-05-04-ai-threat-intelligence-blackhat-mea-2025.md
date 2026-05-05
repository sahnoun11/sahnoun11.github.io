---
title: "I Spoke at Black Hat MEA 2025 in Riyadh — Here's Everything I Know About AI and Threat Intelligence"
date: 2026-05-04 00:00:00 +0100
categories: [AI Security, Threat Intelligence, Black Hat]
tags: [llm, ai, threat-hunting, rag, blue-team, red-team, blackhat, mitre-attack, cybersecurity, soc, hackthebox, qakbot, transformer, pgvector, groq, ollama]
description: >-
  The full technical writeup of my Black Hat MEA 2025 session in Riyadh: how LLMs and RAG architecture
  are transforming threat intelligence, a deep dive into ThreatLens (open-source),
  and the skills that define the next generation of security engineers.
image:
  path: /assets/img/hero.svg
  alt: "AI and Threat Intelligence — Black Hat MEA 2025 — Oussama Sahnoun"
pin: true
math: false
mermaid: false
---

> **🎤 This post is the full written companion to my session at Black Hat Middle East and Africa 2025, Riyadh, Saudi Arabia.**
> Watch the ThreatLens demo on YouTube: [**youtu.be/9Wmxwix6MYk**](https://youtu.be/9Wmxwix6MYk?si=Be9r7D0z7wNrAcyv)
> Full source code: [**github.com/sahnoun11/ThreatLens**](https://github.com/sahnoun11/ThreatLens)

---

## First — The Moment

Before a single line of code, before the architecture diagrams, before the MITRE ATT&CK mappings — let me show you what getting on that stage actually meant.

![Black Hat MEA 2025 speaker kit — badge, rubber duck in balaclava, commemorative coin, pen](/assets/img/blackhat-gift.jpg)
*The Black Hat MEA 2025 speaker kit. The duck wears a balaclava. The coin reads "CYBERSECURITY" around the edge. These sit on my desk now as a permanent reminder: you shipped something real, and people flew to Riyadh to hear about it.*

That rubber duck in the black balaclava is not just swag. In security culture, the rubber duck is an icon — the [rubber duck debugging](https://en.wikipedia.org/wiki/Rubber_duck_debugging) concept, the idea that explaining your problem out loud (even to a duck) forces clarity of thought. Black Hat making it wear a balaclava and look like it's about to run an operation? That's peak cybersecurity humour, and I love it.

The coin. The badge. The lanyard. Standing in that room in Riyadh, Saudi Arabia, as a Tunisian cybersecurity engineer — that was a long time coming.

This post is me putting *everything* from that session into writing. Not a summary. Everything. The depth I could not fit into 45 minutes, the architecture decisions behind **ThreatLens**, and why I believe the engineers reading this will build the tools that matter most in the next decade.

---

## The Talk: Scope and Intent

**Session title:** *AI and Threat Intelligence: Advancing Predictive Threat Hunting*

**Conference:** Black Hat Middle East and Africa 2025  
**Location:** Riyadh, Saudi Arabia  
**Audience:** Security professionals, researchers, red/blue teamers, engineers  

This was not an AI hype talk. There are enough of those. This was a hard technical session with a clear thesis:

> **The Threat Intelligence pipeline is structurally broken at scale, and Large Language Models with RAG architecture are the most credible engineering solution we currently have.**

Let me prove that thesis, piece by piece.

---

## Part 1 — The Crisis

![The five structural failures of modern threat intelligence programs](/assets/img/ti-crisis-anime.svg)
*Five challenges that are breaking every Security Operations Center right now — regardless of budget or headcount.*

Every SOC I have ever spoken to has the same conversation. They say: *"We have the tools. We have the people. But we are still drowning."*

That drowning is not random. It is structural. There are five specific failure modes, and until you name them precisely, you cannot fix them.

### 1. Volume — The Fire Hose Problem

The average enterprise environment produces logs at **petabyte scale daily**. Network flow records, EDR telemetry, email gateway events, cloud API audit trails, DNS query logs, Active Directory events, firewall logs. Every layer of your stack is generating data constantly.

A 500-person security team cannot process this. A 5,000-person security team cannot process this. The data generation rate grows faster than headcount can ever scale. This is a fundamental asymmetry that only automation can close.

### 2. Context Loss — The Noise Problem

Knowing that `185.220.101.42` is "malicious" is almost useless information standing alone.

A useful intelligence item answers: *Which threat actor? Which campaign? What kill-chain stage? What is the confidence level? Is this actor known to target our industry? What TTPs does this actor use beyond this single IOC?*

Without that context, an analyst cannot prioritize. They cannot decide whether to drop everything right now or add this to the backlog. Raw IOCs without context are noise with a scary label attached. Most threat feeds deliver exactly this.

### 3. Toil — The 60% Problem

Research consistently shows that 40–60% of analyst time goes to manual, repetitive work: pivoting between tools that do not talk to each other, reformatting data between systems, writing the same report structures over and over, manually correlating IOCs across three different platforms, hunting for false positives in a sea of alerts.

This is time directly stolen from the work that actually catches attackers: hypothesis-driven threat hunting, building new detection logic, understanding adversary tradecraft.

### 4. Intelligence Latency — The Tempo Problem

A zero-day exploit can be weaponized and deployed globally **within hours** of discovery. The WannaCry ransomware infected 200,000 systems across 150 countries in a single day. Log4Shell was being actively exploited within 12 hours of public disclosure.

Traditional weekly or monthly threat intelligence reports are structurally incompatible with this tempo. By the time a well-researched, reviewed, and published report reaches your inbox, the initial compromise wave may already have passed through organizations that were not prepared.

### 5. Detection Gaps — The Deepest Problem

This is where I want to spend the most time, because it is the least understood, and it is the problem AI most directly addresses.

Most SIEM detection rules are **signature-based or threshold-based**. Consider this rule:

```
IF failed_login_count > 10
AND time_window = 60 seconds
THEN alert = brute_force_detected
```

This rule completely misses a sophisticated brute-force attack that executes **one attempt every 10 minutes over 72 hours**. Same intent. Same eventual outcome. Completely different pattern. The rule fires on noise; the real attack slides through.

Sophisticated APT groups (Advanced Persistent Threats) operate below detection thresholds by deliberate design. Their techniques include:

- **Living Off the Land (LOLBAS)**: abusing legitimate Windows binaries — `certutil.exe`, `mshta.exe`, `rundll32.exe`, `wmic.exe` — to blend into normal administrative activity
- **Slow-and-low movement**: spreading lateral movement over days or weeks instead of minutes
- **Cloud C2**: using Microsoft Graph API, Google Drive, Dropbox, or Slack as command-and-control channels that blend perfectly into normal enterprise traffic
- **Timestomping and log clearing**: modifying file timestamps and clearing event logs to destroy forensic artifacts

The consequence: industry data consistently shows **average dwell time of 200+ days** before detection. An attacker was living inside your network, pivoting, escalating privileges, and exfiltrating data for six months before anyone noticed.

> **This is not a people problem. It is a data processing and pattern recognition problem. And that is exactly what AI was built to solve.**

---

## Part 2 — AI Fundamentals for Security Engineers

Before we can build AI-powered threat intelligence tools, we need a shared technical foundation. I covered this at Black Hat because too many teams are deploying AI systems they do not actually understand, which means they cannot evaluate their failure modes.

![Slide from the Black Hat MEA session — AI and Intelligence defined](/assets/img/slide-05.jpg)
*Slide 5 from the Black Hat MEA session: defining Intelligence and Artificial in the context of cybersecurity applications.*

**Intelligence** is the ability to analyze information, make informed decisions, and carry out goal-oriented actions. It is the analytical capability applied toward a defined purpose.

**Artificial** — in the AI context — refers to computational algorithms that allow machines to simulate, mimic, or reproduce human-like analytical capabilities at machine scale and speed.

When combined: systems that can process information at the scale no human team can match, while applying something that approximates — and in narrow domains can exceed — human-level pattern recognition.

### How Large Language Models Actually Work

![The Transformer architecture — how LLMs process threat intelligence](/assets/img/llm-transformer.svg)
*The complete Transformer pipeline: from raw security text → tokenization → embeddings → self-attention → hidden layers → structured output. This is what runs inside ThreatLens.*

The LLM architecture shown above follows the **Transformer paradigm**, introduced in Vaswani et al.'s landmark 2017 paper *"Attention Is All You Need."* Here is the engineering breakdown — the level of detail you need to build with these systems, not just talk about them.

#### Stage 1: Tokenization

Text is broken into subword units called **tokens**. A tokenizer does not split on words or characters — it uses a learned vocabulary of subword pieces optimized to balance coverage and efficiency.

The word `"reconnaissance"` might tokenize to `["reconn", "aissance"]`. The phrase `"lateral movement"` might stay as `["lateral", " movement"]`. The critical point: **everything the model processes is tokens**, and the model has no inherent concept of "words" — only token sequences and the statistical relationships between them.

For security applications, this matters because technical terms like `"T1021.003"` or `"EventID_4688"` need to be in or near the training vocabulary to be represented well. This is one reason domain-specific fine-tuning improves security LLM performance.

#### Stage 2: Embeddings

Each token is mapped to a **high-dimensional vector** — typically 768 to 4096 dimensions, depending on the model architecture. These vectors are learned during training and encode semantic meaning in a remarkable way.

The key property: **semantically similar tokens cluster together in the vector space**. The vectors for `"QakBot"`, `"malware"`, `"banking trojan"`, and `"loader"` will be geometrically close. The vector for `"pizza"` will be far away.

This geometric encoding of meaning is what allows embedding-based search to work: when you search for `"lateral movement"`, you retrieve content about `"pass-the-hash"` and `"DCOM abuse"` even if those exact words are not in your query — because they are nearby in the embedding space.

#### Stage 3: Self-Attention — The Core Innovation

This is the mechanism that makes Transformers genuinely powerful, and it deserves a careful explanation.

For a sequence of N tokens, self-attention computes an **N×N relationship matrix** — simultaneously computing how much attention each token should pay to every other token in the sequence.

For the sentence *"The attacker used PowerShell to download the payload from the C2 server"*, the attention mechanism learns that `"attacker"`, `"PowerShell"`, `"download"`, `"payload"`, and `"C2"` are all strongly related to each other, while `"the"` and `"from"` are low-information connectors that deserve less weight.

Multiple **attention heads** run in parallel — each capturing different types of relationships: syntactic structure, semantic similarity, co-reference, technical dependencies. A model with 64 attention heads is simultaneously analyzing 64 different types of relationships across the entire sequence.

This is fundamentally different from older recurrent models (LSTMs, GRUs) which processed tokens sequentially and struggled to maintain long-range dependencies. Attention operates on the entire sequence at once, which is why LLMs can understand the connection between an IOC mentioned at the start of a 50-page report and a tactic described 45 pages later.

#### Stage 4: Parametric Knowledge

After training on **trillions of tokens** of text — including security research papers, CVE descriptions, MITRE ATT&CK documentation, malware analysis reports, threat actor profiles, and security blogs — the model's weights encode a compressed representation of this knowledge.

A cybersecurity-tuned LLM has effectively read more security content than any human analyst could in a lifetime. It has absorbed patterns from thousands of incident reports, hundreds of malware families, all 14 ATT&CK tactics and 200+ techniques, and the TTPs of dozens of named threat actor groups.

This is **parametric knowledge** — knowledge baked into the model weights, available instantly without retrieval. The limitation: it has a **knowledge cutoff**. Events after the training cutoff are unknown to the model. This is the problem RAG solves.

#### Stage 5: Output Generation

The model produces a **softmax probability distribution** over its entire vocabulary — for LLaMA 3.3 70B, that is approximately 128,000 possible tokens. It selects the highest-probability next token, appends it to the sequence, and repeats. This iterative process generates the response one token at a time.

Temperature and top-p sampling control the randomness of this selection. For threat intelligence applications, I use low temperature (0.1–0.2) to maximize determinism and minimize hallucination — we want precise, grounded answers, not creative ones.

---

## Part 3 — Prompt Engineering for Security

![Slide — Prompt-based interactions and the analyst-LLM interface](/assets/img/slide-07.jpg)
*Slide 7: prompt-based interactions as the primary interface between analyst intent and LLM capability.*

A prompt is not just a question. In production security systems, a well-engineered prompt is a **structured specification** that determines the quality, format, and reliability of the output.

The components of a production security prompt:

```
[ROLE ASSIGNMENT]
You are a senior threat intelligence analyst with 15 years of experience 
in APT attribution and malware reverse engineering.

[TASK DEFINITION]
Analyze the following Windows Event Log excerpt and identify all indicators 
of lateral movement, persistence, or privilege escalation.

[OUTPUT SPECIFICATION]
Respond ONLY in JSON with this exact schema:
{
  "findings": [
    {
      "timestamp": "ISO8601",
      "event_id": "integer",
      "technique_id": "T-number",
      "technique_name": "string",
      "tactic": "string",
      "confidence": "0.0-1.0",
      "evidence": "string",
      "recommended_action": "string"
    }
  ],
  "severity": "LOW|MEDIUM|HIGH|CRITICAL",
  "summary": "string"
}

[CONSTRAINTS]
- Never hallucinate Event IDs or ATT&CK technique IDs
- If confidence < 0.6, include the finding but flag it as LOW_CONFIDENCE
- If insufficient data, output {"findings": [], "severity": "UNKNOWN", "summary": "insufficient data"}

[CONTEXT — injected by RAG]
{retrieved_log_chunks}

[QUERY]
{analyst_question}
```

This structure — role, task, format, constraints, context, query — is what separates a toy demo from a production security tool. Each element serves a specific purpose:

- **Role assignment** activates the model's security-domain knowledge and biases outputs toward expert reasoning
- **Output specification** makes parsing deterministic and eliminates freeform text that breaks downstream tooling
- **Constraints** explicitly address failure modes (hallucination, insufficient data)
- **Context** is the RAG-injected retrieved intelligence — the most important element
- **Query** is the analyst's actual question, kept separate for clarity

Key prompt engineering techniques that significantly improve reliability in security contexts:

| Technique | Description | Security Application |
|-----------|-------------|---------------------|
| **Chain-of-thought** | "Think step by step before answering" | Forces the model to reason through an attack chain before concluding |
| **Few-shot examples** | Show 2–3 input/output examples | Teaches the output format and reasoning style |
| **Negative constraints** | "Never invent IOCs" | Reduces hallucination in structured fields |
| **Confidence scoring** | Request a 0–1 confidence per finding | Enables analyst prioritization of uncertain results |
| **Source attribution** | "Cite the specific log line" | Forces grounded responses, enables verification |

---

## Part 4 — Threat Intelligence Maturity

![Slide — What a mature threat hunting program asks](/assets/img/slide-08.jpg)

![Slide — How mature TI programs guide critical security decisions](/assets/img/slide-09.jpg)

A mature **Threat Hunting program** proactively searches for unknown, undetected threats. It operates on hypotheses, not alerts. The fundamental questions it answers:

- Are attackers already inside our environment without triggering any existing rules?
- Which behaviors in our logs indicate lateral movement or privilege escalation that we are not detecting?
- What patterns are hiding in EDR telemetry, network flows, and authentication logs?
- Are there signs of Command-and-Control communication that our tools are missing?
- Where are the gaps in our detection logic that sophisticated actors are exploiting?

A mature **Threat Intelligence program** answers the hardest strategic security questions:

- What are the top three threats specifically targeting organizations in our industry right now?
- Which of the 847 alerts from today represent real malicious activity versus misconfigured systems?
- Which CVEs in our asset inventory are being actively exploited in the wild today?
- How effective are our current controls against the specific TTPs of threat actors that target us?

The **intelligence triad** that enables this:

### Pillar 1: Threat Visibility

You cannot detect what you cannot see. Every unmonitored layer is a blind spot. Production visibility requires sensors at every tier:

| Layer | Technology | What It Captures |
|-------|-----------|-----------------|
| Network perimeter | NGFW, IDS/IPS, NDR | Flow data, connections, blocked events |
| Endpoint | EDR (CrowdStrike, SentinelOne, Defender) | Process trees, memory, file events, registry |
| Email | Secure Email Gateway | Attachments, links, sender reputation, headers |
| DNS | Recursive resolver logging | Domain lookups, DGA patterns, NXDOMAIN spikes |
| Cloud | API audit logs (CloudTrail, Activity Log) | IAM changes, resource creation, unusual API calls |
| Identity | Active Directory, Okta | Auth events, group changes, privilege escalation |
| Application | WAF, SIEM correlation | Web attacks, injection attempts, anomalous requests |

Gaps at any layer mean an attacker operating in that layer is invisible.

### Pillar 2: Processing Capability

Raw data → structured, actionable intelligence. This is where most programs fail at scale.

Converting *"Email with .xlsb attachment received"* into *"Email attachment contains QakBot loader variant using Excel 4.0 macro technique, C2 at 185.220.101.42:443, confidence 0.92"* requires automated processing of:
- Sandbox detonation and behavioral analysis
- YARA rule matching against known malware signatures
- Sigma rule correlation against logged behaviors
- TTP extraction and ATT&CK mapping
- Threat actor campaign attribution

This is precisely what LLM pipelines accelerate by orders of magnitude.

### Pillar 3: Interpretation Capability

The highest-value, hardest-to-automate layer. Taking processed intelligence and answering: *"Given that this is QakBot variant X — is it the top threat to OUR specific organization, with OUR specific control gaps, targeting OUR specific industry, right now?"*

This requires contextualizing against:
- Your industry and the threat actors known to target it
- Your current control posture and its known gaps
- Your crown jewels and their exposure
- The threat actor's current operational tempo and targeting patterns

RAG-enabled LLMs can now contribute meaningfully to this layer — not replace the analyst, but dramatically accelerate their analysis.

---

## Part 5 — The QakBot Intelligence Example

![QakBot kill chain anatomy — from initial access to impact](/assets/img/qakbot-anatomy.svg)
*The complete QakBot kill chain as of 2025. This is what a mature threat intelligence report looks like — and what ThreatLens can query in plain English.*

QakBot is not a footnote. It is one of the most persistent and adaptable threats in the landscape, and it serves as a perfect example of why AI-powered TI is necessary.

![Slide — QakBot intelligence example from the Black Hat presentation](/assets/img/slide-11.jpg)

**Timeline:**
- **2007**: First observed as a banking credential stealer targeting Windows systems
- **2019–2022**: Major infection waves; primary initial access broker for ransomware operators including Conti and Ryuk
- **August 2023**: **Operation Duck Hunt** — FBI-led international takedown seized 700,000 infected machines and disrupted C2 infrastructure across 29 countries
- **December 2023 – 2025**: Resurgence confirmed with new variants adapting delivery techniques post-takedown

A complete production QakBot intelligence report includes:

- **50+ unique malware samples** spanning 2021–2025, including post-Operation Duck Hunt variants
- **70+ IOCs**: SHA256 hashes, C2 IP addresses, domains, registry keys, mutex names, JA3 fingerprints
- **Complete infection chain**: initial access → dropper → main module → credential harvesting → C2 → payload delivery
- **Anti-analysis evasion techniques**: VM detection via CPUID, debugger detection, process hollowing, encrypted string tables, sleep obfuscation
- **2024–2025 adaptations**: OneNote embedding, HTML smuggling, ZIP password-protected delivery to bypass email gateways

**Processing this manually**: 3–5 analyst days for a thorough report.

**Processing this with a ThreatLens RAG pipeline**: 2–5 minutes to query any aspect of the intelligence in plain English.

That differential — 3 days versus 5 minutes — is the business case, the security case, and the career case for AI in threat intelligence.

---

## Part 6 — How LLMs Transform the TI Pipeline

![Slide — Three ways LLMs improve threat intelligence processing](/assets/img/slide-12.jpg)
*Three specific capabilities that LLMs bring to the threat intelligence problem.*

Three concrete capabilities:

**Surface critical data buried in volume.** An LLM can scan a 10,000-line event log and surface the three events that form a lateral movement chain — while ignoring the 9,997 events that are routine operations. It does this in seconds.

**Reduce analyst toil.** Extracting IOCs from a malware report, summarizing a 40-page threat actor profile for an executive briefing, mapping a log snippet to ATT&CK techniques — these tasks that each take an analyst 30–90 minutes can be automated to seconds with well-engineered prompts.

**Accelerate RFI responses.** When a business unit asks: *"Are we affected by CVE-2025-XXXXX?"* — an LLM pipeline can return a structured, contextualized answer in under 10 seconds: *"Yes. Affected asset: SQLSERVER-03. Patch available: yes, KB5040442. Exploited in wild: yes, by APT29 since March 2025. Priority: CRITICAL."*

---

## Part 7 — RAG Architecture: The Technical Core

This is the engineering heart of everything ThreatLens does.

![The complete RAG pipeline — from raw documents to expert threat hunting answers](/assets/img/rag-deep.svg)
*The ThreatLens RAG pipeline: every component, every data flow, every design decision explained.*

**Why RAG?** Because raw LLMs have a hard knowledge cutoff. A model trained in early 2024 has no parametric knowledge of a zero-day discovered in late 2025. Fine-tuning to add new knowledge costs hundreds of thousands of dollars in compute and takes months. **RAG solves this by dynamically injecting current intelligence at query time** — making the system perpetually up-to-date at near-zero cost.

![Slide — RAG architecture from the Black Hat MEA presentation](/assets/img/slide-13.jpg)

![Slide — Full system diagram: documents through to analyst output](/assets/img/slide-15.jpg)

### Step 1 — Document Ingestion

Every intelligence source is treated as a document corpus:
- Windows EVTX log files (parsed via `python-evtx`)
- Linux auth logs and syslog (plain text)
- Threat reports and advisories (PDF via `pypdf2`, web content via `BeautifulSoup`)
- STIX/TAXII intelligence feeds
- Internal incident reports

### Step 2 — Chunking

Documents are split into segments. ThreatLens uses **300-character chunks with controlled overlap**.

Why 300 characters? The `nomic-embed-text` embedding model has a 512-token context window. 300 characters safely fits within this limit, leaving room for the chunk's metadata (source, timestamp, log type) to be prepended without truncation.

Why overlap? Because splitting on a fixed boundary can cut a critical relationship in half:

```
chunk_N:   "...the attacker pivoted via DCOM from WORKSTATION-07 to"
chunk_N+1: "SERVER-01, creating a new local admin account svc_backup2..."
```

Without overlap, a query about "account creation on SERVER-01" retrieves `chunk_N+1` but misses the origin host. With a 50-token overlap, both chunks contain enough context to be independently meaningful.

> **Production gotcha**: Naive fixed-size chunking breaks YARA rules mid-signature and splits multi-line event records at the wrong boundaries. ThreatLens includes a log-aware chunker that respects event record boundaries before applying the character limit.

### Step 3 — Embedding Generation

Each chunk passes through `nomic-embed-text` running locally via **Ollama**. This produces a 768-dimensional dense vector.

Why `nomic-embed-text`?
- Open weights — no API dependency, no data exfiltration
- Strong performance on domain-specific text (competitive with OpenAI `text-embedding-ada-002` on security text benchmarks)
- Runs on CPU — no GPU required, meaning it works on any analyst's laptop
- 512-token context window — appropriate for our chunk size

The output: a float32 array of 768 numbers that encodes the semantic fingerprint of that chunk. Similar chunks will have vectors with high cosine similarity. Dissimilar chunks will have low cosine similarity.

### Step 4 — Vector Database Storage and Retrieval

Embeddings are stored in **PostgreSQL with the pgvector extension**, running in a Docker container:

```bash
docker run -d --name threatlens-db --restart always \
  -e POSTGRES_DB=ai \
  -e POSTGRES_USER=ai \
  -e POSTGRES_PASSWORD=ai \
  -p 5532:5432 ankane/pgvector
```

pgvector adds a new column type (`vector(768)`) and supports **HNSW (Hierarchical Navigable Small World)** indexing for Approximate Nearest Neighbor search. HNSW achieves sub-millisecond retrieval across millions of vectors by organizing them in a hierarchical graph structure — trading a small amount of recall accuracy for dramatic speed improvements over exact nearest-neighbor search.

At query time:
1. Analyst question is embedded using the same `nomic-embed-text` model
2. pgvector executes an ANN search: `SELECT content FROM logs ORDER BY embedding <=> query_embedding LIMIT 5`
3. The `<=>` operator computes **cosine distance** — equivalent to `1 - cosine_similarity`
4. The 5 most semantically relevant chunks are returned in milliseconds

### Step 5 — Augmented Generation

The retrieved chunks are injected into the LLM context window as additional context. The final prompt structure:

```
[SYSTEM]: You are a senior threat intelligence analyst...

[RETRIEVED CONTEXT]:
--- Chunk 1 (similarity: 0.94) ---
02:17:11 EventID 4688 wmic.exe spawned by mmc.exe on SERVER-01...

--- Chunk 2 (similarity: 0.91) ---
02:14:33 EventID 4625 WORKSTATION-07 failed auth to SERVER-01...

--- Chunk 3 (similarity: 0.87) ---
02:19:47 EventID 4720 new account svc_backup2 created by SYSTEM...

[ANALYST QUESTION]: Are there signs of lateral movement?
```

The LLM now generates its response with both:
- **Parametric knowledge**: everything it learned during training about lateral movement patterns, ATT&CK techniques, Windows event IDs
- **Retrieved context**: the specific log events most relevant to this question

Result: a grounded, evidence-based answer with specific timestamps, event IDs, ATT&CK mappings, and recommended actions. Not hallucination — contextualized reasoning from actual log evidence.

---

## Part 8 — ThreatLens: The Open-Source Tool

![ThreatLens UI — the full analyst experience](/assets/img/threatlens-demo.svg)
*ThreatLens running locally: upload a log, ask questions in plain English, get expert-level threat hunting answers in seconds.*

[![GitHub](https://img.shields.io/badge/ThreatLens-github.com%2Fsahnoun11%2FThreatLens-9fef00?style=for-the-badge&logo=github&logoColor=black)](https://github.com/sahnoun11/ThreatLens)

> **Watch the live demo:** [YouTube → youtu.be/9Wmxwix6MYk](https://youtu.be/9Wmxwix6MYk?si=Be9r7D0z7wNrAcyv)

**ThreatLens** is the open-source implementation of everything in this post. It is a free, locally-running AI assistant that analyses Windows Event Logs and Linux logs like a senior SOC analyst — using the full RAG pipeline described above.

### Design Philosophy

> *Built in Tunisia, for the global security community.*
> *100% free stack. No credit card. No cloud subscription. No data leaving your machine.*

This is not an accident. One of the realities of working in the MENA region is that international payment requirements often block access to SaaS security tools that analysts in the US or EU take for granted. ThreatLens is deliberately designed so that **anyone, anywhere, with an internet connection and a laptop, can run enterprise-grade AI threat analysis for free**.

### Architecture

```
Upload log (.evtx or .txt)
         │
         ▼
  ┌─────────────┐
  │   Chunker   │  300-char chunks, event-aware boundaries, 50-token overlap
  └──────┬──────┘
         │
         ▼
  ┌─────────────────────────────┐
  │  Ollama — nomic-embed-text  │  768-dim embeddings, 100% local, no GPU needed
  └──────────────┬──────────────┘
                 │
                 ▼
  ┌──────────────────────────┐
  │  PostgreSQL + pgvector   │  HNSW index, Docker, port 5532
  │  (Docker container)      │
  └──────────────────────────┘
         │  ANN search at query time
         ▼
  Analyst asks question in plain English
         │
         ▼
  ┌────────────────────────────────┐
  │  Groq API — LLaMA 3.3 70B     │  Free tier, ~500 tok/s, grounded by context
  └────────────────────────────────┘
         │
         ▼
  Expert answer with ATT&CK mappings,
  timeline, IOCs, recommended actions 🎯
```

### Full Tech Stack

| Component | Technology | Why This Choice |
|-----------|-----------|-----------------|
| UI | Streamlit | Zero-friction local deployment, Python-native |
| LLM | Groq API — `llama-3.3-70b-versatile` | Free tier, fastest open inference (~500 tok/s), no data retention |
| Embeddings | Ollama — `nomic-embed-text` | 100% local, strong domain performance, CPU-only |
| Vector DB | PostgreSQL + pgvector (Docker) | Production-grade, familiar tooling, HNSW index |
| RAG Framework | phidata | Clean RAG primitives, good pgvector integration |
| Log Parser (EVTX) | python-evtx | Native Windows event log parsing |
| Web scraping | BeautifulSoup4 + requests | URL-based threat intelligence ingestion |

### Supported Log Formats

| Format | Extension | Status |
|--------|-----------|--------|
| Windows Event Log | `.evtx` | ✅ Full support |
| Linux / plain text | `.txt`, `.log` | ✅ Full support |
| JSON structured logs | `.json` | 🔜 Roadmap Q3 2026 |
| CSV exports | `.csv` | 🔜 Roadmap Q3 2026 |
| Syslog (RFC 5424) | `.syslog` | 🔜 Roadmap Q4 2026 |

### Quick Start (5 minutes)

```bash
# 1. Clone
git clone https://github.com/sahnoun11/ThreatLens.git
cd ThreatLens

# 2. Start PostgreSQL with pgvector
docker run -d \
  --name threatlens-db \
  --restart always \
  -e POSTGRES_DB=ai \
  -e POSTGRES_USER=ai \
  -e POSTGRES_PASSWORD=ai \
  -p 5532:5432 \
  ankane/pgvector

# 3. Pull local embedding model (runs 100% on your machine)
ollama serve &
ollama pull nomic-embed-text

# 4. Python environment
python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt

# 5. Get your free Groq API key at console.groq.com — no credit card
export GROQ_API_KEY="gsk_your_key_here"

# 6. Launch
streamlit run app.py
# Open http://localhost:8501
```

### What to Ask ThreatLens

Once your log is uploaded, try these queries:

```
# Discovery
What failed login attempts are in this log?
List all EventIDs present and their frequency.
What happened between 02:00 and 03:00?

# Threat Hunting
Are there any signs of lateral movement?
Show me all privilege escalation events.
Are there signs of persistence mechanisms?
Look for C2 beacon patterns in the timing of outbound connections.

# ATT&CK Mapping
Map all suspicious events to MITRE ATT&CK techniques.
Which events match T1059 (Command and Scripting Interpreter)?
What tactics are represented in this log?

# IOC Extraction
Extract all external IP addresses and flag suspicious ones.
List all new user accounts created.
What processes were spawned by unusual parent processes?

# Summary
Give me an executive summary of the security posture in this log.
What are the top 3 most critical findings?
What should I investigate first and why?
```

---

## Part 9 — The Future of AI-Powered Defense

![The four vectors shaping the next generation of AI-powered cybersecurity defense](/assets/img/future-ai-defense.svg)
*Four trajectories that will define defensive AI over the next 24 months.*

### Vector 1: Autonomous Hunting Agents

The current ThreatLens model is **question-and-answer**: an analyst asks, the system retrieves and generates. The next evolution is **agentic**: the analyst states a hypothesis, and an AI agent autonomously executes the hunting workflow.

```
Analyst: "Investigate whether there is evidence of QakBot in our environment"

Agent:
  1. Query SIEM for Excel process anomalies → result: 3 suspicious events
  2. Run QakBot YARA rules against flagged files → result: 1 match, confidence 0.91
  3. Extract C2 communication from matched host's network logs → result: 2 external IPs
  4. Check IPs against threat feeds → result: both in APT attribution lists
  5. Reconstruct full timeline → result: initial access 14 days ago
  6. Generate incident report with ATT&CK mapping
  7. Return: "QakBot confirmed. Dwell time: 14 days. Recommend immediate isolation..."
```

The primitives for this exist today: LangChain agents, Model Context Protocol (MCP) tool calling, SIEM APIs, ReAct prompting patterns. The integration work is what remains.

**ThreatLens roadmap**: agentic mode with auto-pivot across log sources is planned for 2026.

### Vector 2: Real-Time Intelligence Synthesis

The current model ingests intelligence in batches. The future is **streaming ingestion**: threat feeds, NVD updates, ISAC alerts, and OSINT all processed the moment they are published — chunks embedded, stored, and immediately queryable.

Latency from publication to queryable intelligence goes from hours or days to under 5 minutes. The intelligence gap closes.

### Vector 3: The Adversarial AI Arms Race

I want to be direct about this: **attackers are already using AI**. This is not a theoretical future concern.

- **AI-generated phishing** is operational — personalized spear-phishing at scale that defeats traditional detection heuristics
- **Polymorphic malware** using AI to generate functionally equivalent code variants that defeat signature detection
- **AI-assisted vulnerability research** accelerating the discovery of novel attack surfaces
- **LLM prompt injection attacks** against AI security tools — a threat ThreatLens itself must defend against

The defender's current advantage: the window in which defensive AI deployment sophistication exceeds offensive deployment sophistication. That window is real but shrinking. Build defenses now.

### Vector 4: Explainable AI in Security

A black-box system that says "this traffic is malicious" with no reasoning will not and should not be trusted in enterprise security environments. Every high-stakes security decision needs an evidence chain.

RAG architecture is **inherently explainable**: the retrieved chunks are the evidence. ThreatLens already shows which log segments informed each answer. The next step is making this explicit and auditable — every finding linked to specific source evidence, with confidence scores and alternative hypotheses.

---

## Part 10 — Skills for the Next Generation

![Skills radar for the next-generation security engineer](/assets/img/skills-radar.svg)
*The skill profile of the next-generation security engineer. These are not future skills — they are current job requirements at leading security organizations.*

I said this at Tek-Up University and I'll say it here: the skills required to contribute to this future are not just traditional cybersecurity skills. The next generation of security engineers needs a genuinely hybrid profile:

**Cyber Core** — ATT&CK framework, kill chain methodology, network fundamentals, malware analysis, incident response. You cannot build AI security tools if you do not understand what you are detecting and why.

**SIEM / EDR Proficiency** — Splunk SPL, KQL (Microsoft Sentinel), YARA rule writing, Sigma rules. These are the data sources your AI tools consume. You need to understand the data before you can build models on top of it.

**Vector Database Architecture** — pgvector, Pinecone, Weaviate, Qdrant, ChromaDB. Understanding HNSW, cosine similarity, embedding dimensions, and ANN tradeoffs is now a required skill for security AI engineers.

**RAG System Design** — chunking strategies (fixed-size, semantic, hierarchical), retrieval quality evaluation (RAGAS), hallucination mitigation, context window management, hybrid search (vector + keyword).

**LLM Fine-tuning** — LoRA, QLoRA, PEFT frameworks. Understanding when to fine-tune versus when to RAG (answer: usually RAG, unless domain-specific vocabulary is the bottleneck).

**Prompt Engineering** — chain-of-thought, few-shot examples, structured output, negative constraints, temperature calibration. This is a real engineering skill, not a buzzword.

**Adversarial ML** — how AI systems fail under adversarial conditions, evasion attacks against AI classifiers, prompt injection, data poisoning. If you build AI security tools, you must understand how to attack them.

**AI Evaluation Frameworks** — RAGAS for RAG evaluation, hallucination detection benchmarks, precision/recall for security-specific tasks. You cannot improve what you cannot measure.

---

## Closing

We are not replacing threat analysts with AI. We are giving threat analysts superpowers.

The combination of human expertise — contextual judgment, organizational knowledge, adversarial mindset, institutional memory — with AI's ability to process information at machine scale and recall everything it has ever learned is enormously powerful. Neither alone is enough. Together, they are the most effective threat hunting capability we have ever had.

I built ThreatLens because I wanted to prove this was possible with a completely free stack. No enterprise license. No API credit card. No cloud subscription that only teams in wealthy countries can afford. Just open-source tools, a laptop, and a good idea.

The question is not whether AI will reshape threat intelligence. It already has, and it is accelerating.

The question is who builds the next generation of these tools — and whether the defenders or the attackers are moving faster.

I believe the defenders will win. And I am genuinely excited to see what this generation builds.

---

*Oussama Sahnoun — Cybersecurity Expert*
*Black Hat MEA 2025 Speaker · Riyadh, Saudi Arabia*
*Tek-Up University Guest Lecturer · Tunis, Tunisia*

---

## Resources

**ThreatLens:**
- GitHub: [github.com/sahnoun11/ThreatLens](https://github.com/sahnoun11/ThreatLens)
- Demo: [youtu.be/9Wmxwix6MYk](https://youtu.be/9Wmxwix6MYk?si=Be9r7D0z7wNrAcyv)

**Find me:**
- Twitter/X: [@Sahnounoussama5](https://x.com/Sahnounoussama5)
- GitHub: [github.com/sahnoun11](https://github.com/sahnoun11)
- LinkedIn: [linkedin.com/in/oussama-sahnoun-0ba565131](https://linkedin.com/in/oussama-sahnoun-0ba565131/)
- Blog: [sahnoun11.github.io](https://sahnoun11.github.io/)

**Essential reading:**
- [MITRE ATT&CK Framework](https://attack.mitre.org/)
- [Attention Is All You Need — Vaswani et al. 2017](https://arxiv.org/abs/1706.03762)
- [OWASP LLM Top 10 2025](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
- [RAGAS — RAG Evaluation Framework](https://docs.ragas.io/)
- [Operation Duck Hunt — DOJ Press Release](https://www.justice.gov/opa/pr/us-leads-international-operation-to-seize-qakbot-malware)

---

> If ThreatLens saved you time, a GitHub star takes two seconds and means everything:
> **[github.com/sahnoun11/ThreatLens](https://github.com/sahnoun11/ThreatLens)**
