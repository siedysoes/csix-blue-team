# Incident Response Process

Incident response (IR) is the disciplined set of activities used to detect, contain, eradicate, and recover from a security incident while preserving evidence. A shared process matters because incidents create time pressure, and fragmented decisions cause missed containment windows and lost forensic data. This article follows the lifecycle described in NIST SP 800-61 for teams defending systems they own or are authorized to protect.

## What Counts as an Incident

A security incident is an event or chain of events that actually or potentially violates the confidentiality, integrity, or availability of systems or data, or breaks organizational policy. Note the word *potentially*: you do not need proof of data exfiltration to declare an incident — credible threat is enough.

| Category | Definition | Example |
|---|---|---|
| Event | Any observable occurrence; most are benign and only meaningful in context | A user VPNs in from a new city after a conference |
| Incident | An event (or chain) that harms, or threatens to harm, confidentiality, integrity, or availability | That session is followed by mass file downloads at 03:00 |
| Near-miss | Something nearly became an incident but was blocked or reported before harm | User clicked a phishing link but reported it; no credentials entered |

Borderline cases should be declared as incidents and downgraded later if needed — the cost of an unnecessary investigation is much lower than a missed breach.

## The IR Lifecycle

The NIST SP 800-61-style lifecycle is a loop, not a one-way street: findings during detection may send you back to containment, and lessons learned feed the next round of preparation.

| Phase | Core activities | Deliverables |
|---|---|---|
| Preparation | Playbooks, roles, tooling, logging coverage, training, tabletop exercises | IR plan, on-call roster, working SIEM, tested backups |
| Detection & Analysis | Alert triage, scoping, severity classification, evidence collection, timeline building | Incident ticket, scope statement, severity rating, initial timeline |
| Containment, Eradication & Recovery | Short-term containment, remove persistence, patch, restore, monitor for reinfection | Containment record, cleaned or rebuilt systems, restored services |
| Post-Incident Activity | Lessons-learned meeting, metrics, detection and playbook updates | Lessons-learned report, action items with owners and dates |

### Detection & Analysis in Practice

Triage starts with "is this real?" and immediately moves to "how wide is it?" (see [Alert Triage](triage-alerts.md)). During scoping, ask which hosts, accounts, data, and time windows are affected. Two habits prevent most downstream pain: collect volatile evidence before touching the affected machine, and record every action with timestamps as you go.

Severity classification (adapt names and thresholds to your organization):

| Severity | Definition | Example | Response target |
|---|---|---|---|
| Critical | Active broad impact: exfiltration, spreading ransomware, domain compromise | Encryption spreading across file servers | Immediate |
| High | Confirmed compromise with lateral-movement potential | C2 beacon confirmed on a server | Minutes to hours |
| Medium | Confirmed malicious activity, narrow scope | One workstation ran a loader; no spread | Hours to a day |
| Low | Suspicious but narrow, single-user impact | One malware block with no execution | Within SLA |
| Informational | Benign or fully blocked with no effect | Outbound connection to a known-bad IP, blocked | Record only |

### Containment Strategies

Contain first, but choose the trade-offs consciously — some actions destroy evidence or cause disruption that could have been deferred:

| Strategy | Short-term (immediate) | Long-term follow-up | Trade-offs |
|---|---|---|---|
| Network isolation | Disconnect host or move to a quarantine VLAN | Block C2 indicators at the perimeter | Fast and simple; may alert the attacker and can cut off live EDR monitoring |
| Credential reset | Reset the compromised account, revoke sessions | Password rotation, MFA review | Stops account abuse quickly; disruptive, may break service accounts if done blindly |
| Indicator blocking | Firewall, proxy, or DNS blocks on C2 domains and IPs | Intel-driven tuning | Low impact; attacker may pivot to new infrastructure |
| Evidence hold | Preserve evidence, coordinate before remediation | Forensic review, disclosure if required | Maximizes evidence; needs documented approval |

### Eradication & Recovery

Remove persistence (scheduled tasks, services, registry run keys, WMI subscriptions, rogue accounts), patch the initial access vector, restore clean data from verified backups, then monitor restored systems for indicators of the same intrusion. Recovery is not done until you verify normal behavior — rebuild rather than clean when confidence is low.

### Post-Incident Activity

Hold a lessons-learned meeting within about a week, while memory is fresh. Useful questions: What was the initial access vector? Where did our detection first fire, and how long did that take? What did we do that delayed us? What single change would catch this faster next time? Keep it blameless — the goal is to fix process and detection gaps, not to identify someone to punish; people stop reporting when they fear blame. Output is a written report plus action items with named owners and dates.

## Documentation Throughout

Write the timeline as you go, not from memory afterward. A minimum entry format:

```
[YYYY-MM-DD HH:MM TZ] actor: action taken -> result (evidence: host, artifact, ticket)
2026-03-04 14:02 UTC  analyst-jane: isolated WORKSTATION-1148 via EDR -> host offline, ticket IR-412
2026-03-04 14:06 UTC  analyst-jane: collected memory image -> mem_1148.E01, sha256 3f9c... on evidence share
```

Every action: who, when, what, why, and where the evidence lives. This timeline feeds the incident report, the lessons-learned meeting, and any legal or insurance process that follows.

## Worked Mini-Scenario: Ransomware on One Workstation

| Phase | Actions |
|---|---|
| Preparation (already done) | Ransomware playbook exists; EDR deployed; backups tested; escalation contacts known |
| Detection | EDR alerts on shadow-copy deletion and mass file rename on WORKSTATION-1148 |
| Analysis | Triage confirms encryption in progress; scope check finds one host, one user; severity = High |
| Containment | Isolate host (keep EDR management channel if supported); reset the user's password; block the C2 domain at the proxy |
| Eradication | Collect memory and disk image; find initial vector (phishing email with macro); remove persistence; scan fleet for the same indicators; rebuild host from a known-good image |
| Recovery | Restore user files from backup; return host to production; monitor the same indicators for 7 days |
| Post-incident | Lessons: phishing reached the inbox, macro execution not blocked, no easy reporting path — actions: tighten mail rules, enforce macro policy, promote the report button |

## Common Mistakes

| Mistake | Why it hurts | Better approach |
|---|---|---|
| No playbook, improvising under pressure | Inconsistent decisions, missed steps | Prepare a playbook per incident type before you need it |
| Wiping the machine too early | Destroys evidence; root cause never learned | Contain, image, then rebuild |
| No timeline during the incident | Report written from memory; disputes later | Timestamp every action as it happens |
| Declaring an incident too late | Attacker gains time to spread | Declare early, downgrade later |
| Treating containment as recovery | Infection returns after reconnection | Verify eradication, monitor post-recovery |
| Skipping the lessons-learned meeting | The same incident recurs | Hold it, write it down, assign owners |

## See also

- [Alert Triage](triage-alerts.md)
- [Digital Forensics Basics](forensics-basics.md)
- [Playbooks & Checklists](playbooks.md)
- [Security Operations Fundamentals](../fundamentals/security-operations.md)
