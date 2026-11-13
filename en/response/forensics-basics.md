# Digital Forensics Basics

Digital forensics is the systematic collection, preservation, and analysis of digital evidence to reconstruct what happened and why. It extends triage by producing defensible facts — a timeline, a root cause, and proof of scope — that a single alert cannot provide. This article covers the principles, artifact sources, and basic tooling for examining systems you own or are authorized to protect.

## What Forensics Adds Beyond Triage

Triage answers "is this bad right now?" Forensics answers: What exactly happened? When did it start? What was accessed? How did the attacker get in? Those answers are required for an accurate timeline, for legal or regulatory reporting, for lessons learned, and for confirming that eradication actually worked. Where triage is a snapshot, forensics is the story.

## Evidence Handling Principles

| Principle | What it means | Practice |
|---|---|---|
| Integrity | Evidence must be provably unchanged | Hash images (e.g. SHA-256) before and after transfer; analyze copies, never originals |
| Chain of custody | Document who handled the evidence and when | Evidence log: item, hash, collector, time, storage location, every transfer |
| Order of volatility | Collect the most ephemeral data first | Memory, then network connections, running processes, disk, then archived logs |
| Least interaction | Every action on a system changes its state | Minimize actions on live hosts; prefer forensic images; record what you did |

**Live acquisition note:** some evidence exists only while a machine is running — RAM contents, mounted encrypted volumes, active network connections. Live acquisition is justified when the system must stay up or full-disk encryption would render a dead image unreadable, but it changes state, so record every command you ran and its output.

## Where Evidence Lives

| Location | Artifacts | What it can show |
|---|---|---|
| Disk - file system | MFT ($MFT), $UsnJrnl, $LogFile | File create/modify/access/delete times and paths, even after deletion |
| Disk - Windows registry | Run keys, services, ShimCache/AmCache, UserAssist, USB history | Persistence, programs executed, devices attached |
| Disk - prefetch | Prefetch files under `C:\Windows\Prefetch` | Which executables ran and when (where enabled) |
| Disk - shellbags | BagMRU keys | Folders browsed in Explorer, including deleted and network paths |
| Disk - browser artifacts | History, cookies, downloads, cache | Web activity, C2 via browser, credential-phish visits |
| Memory | Processes, network connections, loaded modules, injected code | Fileless malware, live C2 connections, code injection |
| Logs - central | SIEM or syslog collectors | Cross-host correlation; survives endpoint tampering |
| Logs - local | Windows Event Logs, Linux syslog/journald | Host-level detail; may be cleared by an attacker |
| Cloud | Provider audit trails (control-plane audit logs, SaaS sign-in logs) | Who changed what, from where, when — often the only record |

Timestamp semantics (creation vs modification, UTC vs local) and artifact locations vary by operating system and version — verify against current references for your environment.

## A Practical Collection Workflow

1. Confirm authorization and scope: which systems, which time window, whose approval.
2. Capture volatile data first: memory, running processes, network connections, logged-in users.
3. Decide live or dead acquisition (see the live acquisition note above) and record the choice.
4. Create a forensic image or targeted collection, then compute a hash immediately.
5. Record the chain of custody: who, when, from where, where stored, hash value.
6. Analyze copies only; keep the original image write-protected.
7. Re-verify the hash after analysis and confirm it still matches.

## Timeline Analysis

Timeline analysis means correlating artifact timestamps into one coherent story: user action, then process execution, then file written, then persistence created, then network connection. The practical work is normalization — timestamps come from different sources with different clocks, time zones, and meanings. Convert everything to one timezone (usually UTC), note clock skew between hosts, and always pair each timestamp with its artifact and your interpretation of it.

Rules of thumb when correlating:

- One artifact proves one fact; a chain of independent artifacts proves a narrative.
- Prefer artifacts the attacker is less likely to have tampered with (central logs outlive local ones).
- Record when an artifact's timestamp is a last-write time, not an event time.
- Mark gaps in coverage explicitly ("no EDR data before agent install") instead of silently interpolating.

## Common Pitfalls

| Pitfall | Consequence | Better habit |
|---|---|---|
| Analyzing the live system instead of an image | Changed timestamps, destroyed evidence | Image first, analyze copies |
| Collecting only the alerting artifact | Missed scope and root cause | Collect per the order of volatility, then narrow |
| Ignoring time zones and clock skew | Events ordered wrongly in the story | Normalize to UTC, note offsets |
| Unlabeled evidence files | Cannot tell which host or when | Naming convention plus evidence log from the start |
| Deleting "irrelevant" data to save space | Missing context during review | Collect more, filter during analysis |

## Tooling

Brief overview; all have official documentation, and versions change often:

- **Autopsy** — open-source digital forensics platform for disk images: file-system analysis, timeline view, keyword search.
- **Plaso / log2timeline** — parses many artifact sources into a single "super-timeline" for correlation at scale.
- **KAPE** — targeted artifact collection: grab only the artifacts you need, quickly, from one or many endpoints (Windows-focused).
- **Volatility** — memory analysis framework: process, network, and injection analysis from a memory dump.

Verify current versions, licensing terms, and usage docs on each project's official site before relying on them in an investigation.

## Worked Mini-Timeline Example

Case: workstation WS-1148, suspected phishing.

| Time (UTC) | Artifact | Interpretation |
|---|---|---|
| 2026-03-04 13:47 | Browser history: `invoice_2026.zip` downloaded | User fetched the attachment from an external site |
| 2026-03-04 13:49 | MFT: `invoice_2026.exe` created in Downloads | Archive extracted; executable present |
| 2026-03-04 13:49 | Prefetch: `invoice_2026.exe` executed | The binary ran on this host |
| 2026-03-04 13:50 | Registry Run key: new `updater` entry | Persistence established |
| 2026-03-04 13:51 | Memory: suspicious child process with a connection to 203.0.113.44 | Likely C2 — escalate and contain now |

Five unrelated observations become a defensible narrative with a clear scoping start point: everything after 13:47 is in scope.

## Legal and Policy Note

This material is educational. Forensic examination and evidence handling must follow your organization's policies and applicable local law, including rules on consent, data protection, employee privacy, and retention. When in doubt, involve your legal or compliance team before collecting or analyzing data.

## See also

- [Incident Response Process](incident-response.md)
- [Alert Triage](triage-alerts.md)
- [Log Sources That Matter](../fundamentals/log-sources.md)
- [Playbooks & Checklists](playbooks.md)
