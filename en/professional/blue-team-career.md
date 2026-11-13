# Blue Team Careers & Certs

Blue team roles range from working an alert queue in a SOC to designing detection architecture, and the field values demonstrable skill as much as credentials. This article maps common career stages, roles, and certifications so beginners can plan realistically. Certification details change frequently — verify current details and pricing on the official website of each provider.

## Career Path

| Stage | Typical role | Skills that matter at this stage |
|---|---|---|
| 1 | SOC Analyst L1 | Alert triage, log reading, ticket hygiene, networking basics, following playbooks |
| 2 | SOC Analyst L2 | Deeper investigation, scoping, host and network analysis, escalation quality, some scripting |
| 3 | L3 / Threat Hunter / Incident Responder | Hypothesis-driven hunting, forensics, attacker TTPs (e.g. MITRE ATT&CK) |
| 4 | Detection Engineer / IR Lead | Detection-as-code (Sigma, YARA), tuning, automation, mentoring, delivery |
| 5 | Security Architect / Security Manager | Architecture, risk, strategy, budget, cross-team leadership |

Progression is not strictly linear. People move between blue roles (SOC to IR to detection) and sideways from sysadmin, network, or helpdesk roles, which are common entry points.

## Role Samples

| Role | Focus | Day-to-day |
|---|---|---|
| SOC Analyst | Detect and triage alerts | Work the queue, enrich, escalate or close with notes, report obvious noise |
| Detection Engineer | Build and improve detections | Write and test rules, review coverage gaps, tune false positives, manage rules as code |
| Threat Hunter | Search for hidden activity | Form hypotheses, write queries across logs and endpoints, validate findings |
| IR Responder | Contain and eradicate incidents | Lead or support incidents, collect evidence, build timelines, run lessons learned |
| DFIR Consultant | Forensics and IR across client environments | Engagements, evidence handling, reporting; scope and travel vary by firm |

## Certification Comparison

| Certification | Focus | Format | Suits |
|---|---|---|---|
| CompTIA Security+ | Broad security theory and fundamentals | Multiple-choice with performance-based items | Entry level; a common HR screening baseline |
| CompTIA CySA+ | Behavioral analytics and defense | Multiple-choice, analyst-oriented | SOC analysts early in their career |
| BTL1 (Blue Team Level 1) | Hands-on blue skills: SIEM, forensics, incident response | Practical, lab-oriented | Practitioners who want applied practice |
| GCIA / GCIH (SANS GIAC) | Intrusion analysis / incident handling | Proctored exams tied to SANS training | Analysts with employer-funded training budgets |
| CISSP | Security management and broad domain knowledge | Proctored; requires documented experience | Experienced practitioners moving toward management |

Certifications are not a substitute for skill, but some employers and markets use them as filters. Verify current details and pricing on the official website of each provider before deciding.

## How to Choose

- **Starting from zero, or coming from IT support / networking / sysadmin:** begin with entry-level theory (Security+ style) alongside free hands-on labs.
- **Already in a SOC or IT role and want to get better:** choose practitioner-oriented, hands-on options (BTL1, CySA+, SANS when employer-funded).
- **Aiming at management or architecture in five or more years:** CISSP-style breadth, after you have real operational experience.
- Sequence matters more than brand: one entry credential plus a real portfolio beats a long list of advanced certifications you have not earned.

## Skills That Matter More Than Paper

| Skill | Why it matters | How to practice |
|---|---|---|
| Linux comfort | Most tooling and servers run on it | Break and fix a Linux VM |
| Scripting (Python, bash) | Automates queries, parsing, enrichment | Parse auth logs, call an API, write a small triage helper |
| Reading logs fluently | The core of detection work | Read Windows Event Logs and syslog daily |
| Clear writing | Reports and escalations drive decisions | Write incident notes a stranger could follow |
| Networking fundamentals | Without packets, ports, and DNS, alerts make no sense | Trace a login from client to server |

## Building a Portfolio

- **Home lab:** deploy a SIEM, generate telemetry, write and tune detections (see [Defense Labs & Ranges](defense-labs.md)).
- **Write-ups:** publish lab or CTF write-ups that explain what you did and why — clear writing is itself evidence of skill.
- **Open source:** contribute detection rules, documentation, or fixes to blue-team projects. Small, real contributions beat claims.
- **Keep an achievements list:** incidents handled, rules shipped, time-to-detect improvements, with dates.

## A First-Year Plan

| Months | Focus |
|---|---|
| 1-3 | Foundations: networking, OS basics, Linux, one entry certification track, daily log reading |
| 4-6 | Lab: SIEM plus a few endpoints, basic detections, start scripting in Python or bash |
| 7-9 | Practitioner skills: triage drills on free ranges, first practical certification or equivalent project |
| 10-12 | Portfolio: publish two or three write-ups, contribute a detection rule or tool docs, apply to L1/L2 roles with concrete evidence |

Adjust the pace to your background, but keep labs running throughout — employers ask what you have built and what you have defended.

## See also

- [Defense Tooling](defense-tooling.md)
- [Defense Labs & Ranges](defense-labs.md)
- [What Is a Blue Team](../fundamentals/what-is-blue-team.md)
- [Incident Response Process](../response/incident-response.md)
