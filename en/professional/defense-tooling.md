# Defense Tooling

A defender's toolkit spans detection (SIEM, EDR, network sensors), investigation (forensics, case management), and automation — and much of it is free or open source. This article organizes common options by category and, more importantly, by when to use them. Free tools still cost time to deploy and tune, so choose what you can realistically operate.

## SIEM / XDR

| Tool | What it is | When to use |
|---|---|---|
| Wazuh | Free, open-source SIEM and XDR: endpoint agents, log ingestion, rules, dashboards | Self-hosted visibility for labs and small-to-medium environments |
| Elastic Security | Built on the Elastic Stack; log search, detection rules, dashboards in one | You already run Elastic and want tight search plus detection together |
| Splunk (free tier) | Splunk Enterprise free license tier with a daily volume limit | Learning SPL and enterprise-style searching — verify current free-tier terms |

## Endpoint: EDR Alternatives

| Tool | What it is | When to use |
|---|---|---|
| Velociraptor | Endpoint DFIR and hunting at scale: collect artifacts and run queries across many hosts | Rapid scoping, artifact collection, fleet-wide hunting |
| osquery | Query endpoints with SQL (`SELECT ... FROM processes`) | Lightweight cross-platform visibility; scheduled compliance-style queries |

Both add endpoint visibility but are not full drop-in replacements for a commercial EDR's live response — verify current features against your needs.

## Network Detection

| Tool | What it is | When to use |
|---|---|---|
| Suricata | Open-source IDS/IPS: signature-based detection on traffic | Detect or block known malicious traffic at the perimeter or a segment |
| Zeek | Network metadata and logs (connections, DNS, HTTP, TLS) without signatures | Behavioral investigation and detection when signatures miss |

Suricata tells you "this matched a rule"; Zeek tells you "what happened on the wire". Together they cover signature and metadata needs.

## DNS and Proxy Filtering

| Tool | What it is | When to use |
|---|---|---|
| Pi-hole | Network-wide DNS sinkhole with a dashboard | Home labs and small networks; fast feedback on malicious domains |
| blocky / CoreDNS with plugins | Configurable DNS proxies and resolvers with blocklists and logging | Environments wanting scriptable DNS filtering |

DNS blocking is cheap, early containment — check what your network team permits in a real environment.

## Hunting and Forensics

| Tool | What it is | When to use |
|---|---|---|
| TheHive + Cortex | Open-source security case management (TheHive) with observable-analysis workers (Cortex) | Turning alerts into shared cases and automating lookups |
| Autopsy | Disk-forensics platform: timelines, keyword search, file analysis | Deep dives on a disk image |
| Plaso (log2timeline) | Super-timeline generator from many artifact sources | Building a cross-artifact timeline fast |

## Deception and Automation

- **Honeytokens:** fake but believable artifacts — a decoy credential in a repository, a fake admin share, a decoy user account — that should never be touched. Any access is high-signal with almost no false positives. Deploy them, alert on them, and never make them real.
- **SOAR / orchestration (e.g. Shuffle):** connect tools so repetitive triage steps run automatically — enrich an IP, open a case, post to a channel. Automate repetition, not judgment; keep irreversible actions behind human approval.

## Which Tool for Which Job

| You need to... | First choice | Why |
|---|---|---|
| Centralize logs and alert on them | Wazuh or Elastic Security | Free, self-hosted, rule-driven |
| Query one host's history quickly | osquery or Velociraptor | Direct endpoint queries without full EDR |
| Collect evidence at scale | Velociraptor | Targeted artifact collection across many hosts |
| Detect known-bad traffic inline | Suricata (IPS mode) | Signatures with blocking |
| Understand what a host did on the network | Zeek logs | Rich metadata for hunting |
| Block malicious domains cheaply | Pi-hole (home) / DNS filtering (org) | Early, low-effort containment |
| Coordinate an incident with a team | TheHive plus your SIEM | Shared case state and evidence |
| Automate repetitive triage | Shuffle or your own scripts | Fewer manual steps |
| Know instantly when a fake credential is used | Honeytokens | High-signal alerting |

## Install Notes

All tools above have official documentation — start there and verify current versions, supported platforms, and configuration guidance before deploying. Design first (what you will monitor, where data goes, who reads alerts), then install: a deployed-but-untuned tool is just overhead. Treat any quick-start commands you find online as untrusted until checked against the official docs.

## The Golden Rule

Tools only defend what you actually monitor and tune. Coverage beats quantity: five log sources you watch daily beat fifty integrations nobody queries. Every tool needs an owner, a tuning cadence, and alerts that someone is ticketed or paged for — otherwise it produces data, not defense.

## See also

- [SIEM Basics](../fundamentals/siem-basics.md)
- [Network Defense](../fundamentals/network-defense.md)
- [Defense Labs & Ranges](defense-labs.md)
- [Writing Sigma Rules](../detection/sigma-rules.md)
