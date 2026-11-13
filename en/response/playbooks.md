# Playbooks & Checklists

A playbook is a pre-written, step-by-step procedure for handling one specific incident type, so responders do not improvise during a live event. Playbooks compress hard-won experience into repeatable actions, decision points, and communication. They complement the general IR process: the process tells you which phase you are in, while the playbook tells you exactly what to do for this kind of incident.

## Playbook vs the IR Process

The incident response process is generic and applies to every incident. A playbook is specific and exists per incident type — ransomware, phishing, account compromise, data exfiltration, malware on a server. You follow the process always, and pick the playbook that matches the pattern you are seeing.

## Playbook Anatomy

| Section | Contents |
|---|---|
| Trigger / detection | Which alert or condition starts the playbook |
| Triage steps | Confirm it is real, gather context, classify severity |
| Containment actions | Ordered steps: isolate, block, reset |
| Escalation | Who to call, when, and with what information |
| Communication | Templates for internal escalation and stakeholder updates |
| Recovery verification | How to confirm the threat is gone and services are healthy |
| Lessons | What to feed back into detection and into the playbook itself |

Every playbook also needs a named owner, a version and date, and a link to the detection rule that triggers it.

## Example Playbook: Suspicious Login / Possible Account Compromise

**Trigger:** SIEM alert for impossible travel, or a successful login from an unseen country/ASN for user account X.

1. **Confirm and enrich (within 5 minutes).** Read the alert details: source IP, ASN and reverse DNS, geolocation, user department, time of day. Check whether this IP appears in other alerts.
2. **Review the account's recent activity.** Failed attempts before success? New device registered? MFA method added or changed? Mailbox rules created, especially forwarding rules? Unusual file downloads?
   - **Decision point A** — MFA method or mailbox rule was changed during this session: treat as confirmed compromise, skip to step 4.
   - **Decision point B** — activity is clean and the user recently traveled or uses a VPN: likely benign, but still verify with the user through a separate channel (never email the possibly compromised account).
3. **Classify severity.** Standard user with no suspicious actions: Low/Medium. Privileged account, or any persistence change: High/Critical.
4. **Contain.** Revoke sessions and tokens using your identity platform's documented process, reset the password, disable any recently added MFA methods, remove rogue mailbox rules. Isolate the endpoint too if malware is suspected.
   - **Decision point C** — revoking sessions for a shared or service account may cause outages: check dependencies first and involve the service owner before acting.
5. **Verify scope.** Query the SIEM for the same source IP or ASN and for other accounts with similar behavior in the same window. Any hits: open a wider IR ticket.
6. **Recover and hand back.** Restore the user's access only after verification, notify the user through a verified channel, and document what changed and when.
7. **Post-incident.** Add the source IP or ASN to detection context, review whether the triggering rule needs tuning, and record time-to-contain.

## Building Your First Playbook

1. Pick one incident type you actually see (or credibly expect), such as malware on a workstation.
2. Write down what you did last time, in order, even if it was messy.
3. Cut it to the ten steps that must happen; move the rest to links or an appendix.
4. Add decision points: where would a competent analyst have to make a call?
5. Add escalation contacts and a communication template.
6. Walk it through with a colleague in a tabletop; fix anything they stumble on.
7. Assign an owner and a review date, then link it to the triggering detection rule.

A short playbook that gets used beats a perfect one that does not. If a step needs a tool command, link to the tool's official documentation rather than pasting commands that will go stale.

## Checklists vs Playbooks vs Runbooks

| Type | Scope | Used when | Example |
|---|---|---|---|
| Checklist | Short verification list | Any task, to avoid skipped steps | "Before closing: timeline complete, evidence hashed, stakeholders notified" |
| Playbook | Full procedure for one incident type | A known incident pattern has triggered | Possible account compromise |
| Runbook | Operational procedure for a system or task | Routine ops, automation, or recovery steps | "Rotate service account credentials" |

They layer: runbooks support individual playbook steps, and checklists guard playbook completion.

## Communication Templates

Internal escalation message (adapt to your environment):

```
[IR-141] Possible account compromise - user j.doe (Finance)
Severity: High | Detected: 2026-03-04 14:02 UTC
Known facts: successful login from hosting-provider IP 198.51.100.10;
forwarding rule created 3 minutes later. No malware found yet.
Scope checked: same IP seen on one other account (k.service).
Actions taken: sessions revoked, password reset, rule removed.
Asks: L2/IR to confirm eradication; identity team to review token revocation.
Contact: analyst-jane in #ir-room, ticket IR-141
```

Stakeholder update structure: (1) status and severity, (2) confirmed facts only, (3) impact so far, (4) actions taken, (5) time of next update. Never speculate in writing — always separate confirmed facts from suspicions.

## Testing Your Playbooks

A tabletop exercise is a facilitated discussion where the team walks through a scenario on paper, with no live systems involved. To run a simple one: pick a scenario (phishing leading to password reuse), give the team the opening facts, let them talk through their steps for 30-60 minutes, then discuss what broke. Record every gap as an action item. Keep it blameless and timeboxed, and verify that contact lists and tool access actually work during the exercise. A common cadence is one tabletop per quarter.

## Keeping Playbooks Alive

Review playbooks on a fixed cadence (commonly at least annually) **and update them after every real incident** — each incident exposes steps that were wrong, missing, or too slow. When detection rules change, playbook triggers must change with them. Give each playbook a version and a "last verified" date: a stale playbook is worse than none, because it creates false confidence. Verify current guidance against sources such as NIST SP 800-61 and your organization's own policies.

## Common Mistakes

| Mistake | Consequence | Fix |
|---|---|---|
| Playbook too long (10 pages) | Nobody reads it mid-incident | One page of ordered steps plus links |
| No named owner | It rots silently | Assign an owner and a review date |
| Never tested | Fails exactly when needed | Quarterly tabletop exercise |
| No decision points | Team guesses under pressure | Explicit if/then branches |
| Not updated after real incidents | Same friction repeats every time | Playbook update is a standard action item |
| No escalation contacts inside the playbook | Delays when the on-call person is unreachable | Verify contacts quarterly; list roles, not just names |
| Steps assume tools nobody has access to | Playbook stalls mid-incident | Check access during tabletops |

## See also

- [Incident Response Process](incident-response.md)
- [Alert Triage](triage-alerts.md)
- [Digital Forensics Basics](forensics-basics.md)
- [The Detection Lifecycle](../detection/detection-lifecycle.md)
