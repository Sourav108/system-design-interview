# Mock Interview 01: SDE2 — Design a Global URL Shortener (TinyURL)

> **Candidate Level**: SDE2
> **Interviewer**: Staff Distributed Systems Engineer
> **Duration**: 45 Minutes
> **Target Outcome**: Strong Hire

---

## Transcript & Commentary

### 1. Requirements Clarification (00:00 - 05:00)
- **Candidate**: "Before jumping into the design, I'd like to clarify the functional and non-functional requirements. Functional: given a long URL, return a 7-character short URL. When accessed, redirect via HTTP 301/302. Custom aliases supported. Non-functional: 99.999% availability, redirection latency under 10ms, 100M URLs/month."
- **Interviewer**: *[Good scoping. Immediately establishes constraints and SLA targets.]*

### 2. Capacity Estimation (05:00 - 10:00)
- **Candidate**: "100M URLs/month $\approx 40\text{ write QPS}$. With 100:1 read ratio, read QPS is $4,000\text{ QPS}$ (Peak 12,000 QPS). 5-year storage is $3\text{ TB}$. 20% hot URLs in Redis requires only $35\text{ GB RAM}$."
- **Interviewer**: *[Crisp mental math. Connects numbers to hardware selection.]*

### 3. High-Level Design & Deep Dive (10:00 - 35:00)
- **Candidate**: "I will use a 64-bit Snowflake ID sequencer, encode the integer into Base62, store in DynamoDB, and cache in Redis with Cache-Aside. Redirections return HTTP 302 to preserve click analytics."
- **Interviewer**: "Why Base62 over MD5 hash truncation?"
- **Candidate**: "MD5 hash truncation causes collisions requiring database lookup loops. Base62 backed by a unique 64-bit integer is a deterministic bijective mapping with zero collisions."
- **Interviewer**: *[Strong Hire justification.]*
