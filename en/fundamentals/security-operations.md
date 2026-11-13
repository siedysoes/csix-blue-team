# Security Operations Fundamentals

Security operations is the day-to-day work of watching systems, deciding which signals matter, and responding before small problems become incidents. A Security Operations Center (SOC) is the team and the function that carries this out, and its performance is what most blue-team careers are built around. This page covers SOC structure, the lifecycle of an alert, the metrics leaders track, and the reality of shift work.

## What a SOC Is

A SOC is an organized function, meaning a team, a process set, and a toolset, that monitors an organization's systems and detects, investigates, and responds to security events. It is not a room with dark walls and wall displays; it is a repeatable operating model that works at 3 a.m. as well as at noon.

The typical stack is simple to state: **SIEM + EDR + ticketing**. The SIEM collects and correlates logs, the EDR provides endpoint visibility and containment actions, and the ticketing system records every alert, decision, and action so work survives shift changes.

## Tiers of a SOC

| Tier | Focus | Typical work |
| --- | --- | --- |
| L1 — Triage | First look | Review alerts, gather basic context (user, host, reputation), close known-benign events, escalate true positives with notes |
| L2 — Investigation | Depth | Pivot across SIEM and EDR, determine scope, decide on containment, work the incident until resolution |
| L3 — Hunting and engineering | Proactive improvement | Threat hunting, building and tuning detection rules, root-cause analysis, tooling and automation |
| Management | Direction | Staffing and shifts, service levels, metrics and reporting, escalation paths, cross-team coordination |

Tiers describe responsibility, not seniority gates. In a small team, one analyst may triage, investigate, and write a detection rule in the same shift.

## The Alert Lifecycle

Every alert follows the same path, regardless of tool or company:

1. **Detect** — a rule or threshold fires on telemetry and creates an alert.
2. **Triage** — an analyst confirms it is real, relevant, and in scope; known-benign events are closed with a reason.
3. **Investigate** — expand outward: the affected host, the account, related events, and whether other systems are involved.
4. **Contain** — stop the damage: isolate the host, disable the account, block the indicator, revoke the session.
5. **Document** — record what happened, what was done, and what should change so the next alert is faster to handle.

Skipping documentation is the most common shortcut, and it is the one that hurts most during the next night-shift incident.

## Key Metrics

| Metric | What it means | Why it matters |
| --- | --- | --- |
| MTTD (mean time to detect) | Time from the attacker's first action to your alert | Shows how much of the attack you see, and how early |
| MTTR (mean time to respond or resolve) | Time from detection to containment or closure | Shows how fast the team acts once something fires |
| Alert volume | Alerts per shift or per day | Drives staffing; a sudden spike usually means a real event or a broken rule |
| False positive rate | Share of alerts that turn out not to be real incidents | The main driver of alert fatigue; a high rate trains analysts to dismiss alerts |

Organizations define MTTD and MTTR boundaries differently, for example first action versus first log, or detection versus containment versus closure. Agree on the definition before comparing numbers.

## On-Call Reality and Shift Work

Detection does not stop overnight, so SOCs run shifts with weekend and holiday coverage. In practice this means a handover at the boundary of every shift, an on-call rotation for out-of-hours escalation, and a written path for when one analyst is overwhelmed.

Two things make shift work sustainable: handover notes that make context transferable, and clear rules for when to wake someone up. Both are process problems rather than personality problems, and they are worth fixing early.

### What a shift handover must include

- Open incidents and their current containment state.
- Hosts still isolated or accounts still disabled, and who owns releasing them.
- Known noisy or broken rules producing alert floods during the shift.
- Tooling outages: a collector that stopped, a connector that expired.
- Any pending action with a deadline, such as a forensic image scheduled for acquisition.
- Where the outgoing analyst left their notes, in the ticket and not only in chat.

## Runbooks vs Playbooks

- **Runbook** — tactical, per-alert instructions: which console to open, which fields to check, when to escalate. Example: "EDR alert type X: check parent process; if Office parent, escalate to L2."
- **Playbook** — a broader procedure for an incident type, covering roles, containment options, communication, and evidence handling. Example: "Ransomware suspected: isolate, notify executives, preserve forensics, follow recovery order."

Runbooks tell an analyst what to do with one alert. Playbooks tell the whole team what to do with one incident. Both must be short enough to be used under pressure.

## Alert Fatigue

The central problem of SOC work is not a lack of alerts; it is too many alerts that do not matter. When a large share of the queue is noise, real incidents get the same skim treatment as the noise, and the cost shows up as missed detections.

The fix is a loop, not a purchase: measure which rules produce the most false positives, tune or disable them, add suppression for known-good activity, then re-measure. An analyst who can file a tuning suggestion in one click is feeding the SOC its most valuable data stream.

## Worked Example: An Alert Journey

Scenario: an EDR alert fires on a finance workstation, and the alert travels from detection to closure.

```
09:12  DETECT   EDR rule: "suspicious PowerShell spawned by Word" on WRK-117
09:14  TRIAGE   L1 checks process tree: winword.exe -> powershell.exe -enc <blob>
                 User: j.mayer   Host: patch level current   No prior alerts
                 Verdict: not a known software deploy -> ESCALATE to L2
09:21  INVEST   L2 pivots in SIEM: same encoded command on 1 more host (WRK-142)
                 Outbound connection to rare-domain.example, first seen yesterday
09:34  CONTAIN  Both hosts isolated in EDR; domain added to proxy and DNS
                 blocklists; affected user sessions revoked and password reset
09:50  SCOPE    Fleet hunt for the parent-child pattern: no further hits
10:40  DOCUMENT Ticket filled with timeline, indicators, and actions taken;
                 new detection idea filed for the engineering backlog
11:05  CLOSE    Alert closed as a confirmed incident; post-incident review set
```

The timeline is the point: each step has an owner, a decision, and a written record. An alert without those is just a notification.

## Tooling Landscape

Three tool categories carry most SOC workflows: **SIEM** to centralize and correlate logs, **EDR** for endpoint telemetry and containment, and **ticketing** as the system of record for work, with enrichment sources such as threat-intel lookups, DNS, and proxy logs plugged into the same loop.

## See also

- [What Is a Blue Team](what-is-blue-team.md)
- [Alert Triage](../response/triage-alerts.md)
- [Playbooks & Checklists](../response/playbooks.md)
- [Tuning & False Positives](../detection/false-positives.md)
