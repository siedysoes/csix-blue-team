# Alert Triage

Alert triage is the daily work of deciding whether a detection is a real threat, how large its scope is, and what to do next. Most SOC time is spent here, and the quality of these decisions determines whether real incidents get missed. This article describes a practical workflow for analysts working an authorized defensive queue.

## The Daily Reality of the Queue

The queue never empties. Alerts arrive in bursts, duplicates cluster around one noisy rule, and at least one alert is always more ambiguous than you would like. The job is not to deeply investigate everything — it is to reach the correct disposition quickly: escalate, close, or defer, each with notes the next analyst can follow. Disposition quality beats raw speed; a wrong "close" is far more expensive than a slow escalation.

## The Triage Decision Tree

```text
[Alert fires]
     |
     v
[Context enrichment]  -- asset criticality, user, IP, process lineage, history
     |
     v
[Is it real?] -- no --> [Close as FALSE POSITIVE: document the WHY]
     |
    yes / suspicious
     v
[Scope]  -- same indicator on other hosts? other accounts? same time window?
     |
     v
[Classify severity]  Critical/High -> escalate to L2/IR now
     |                 Medium       -> escalate with notes, keep monitoring
     |                 Low          -> handle or ticket within SLA
     v
[Act: escalate or close]  -- ALWAYS leave notes: what you checked,
                             what you found, what you did
```

## Enrichment Sources

| Source | What it tells you | Example question |
|---|---|---|
| Asset criticality | Business impact of the host | Is this a domain controller or a kiosk? |
| User department / role | Whether the user is a likely target | Is this finance, an executive assistant, or a shared account? |
| IP reputation / reverse DNS | Nature of the remote endpoint | Known hosting provider, residential ISP, or Tor exit? |
| Process lineage | Whether the behavior is plausible | Did WINWORD.EXE just spawn powershell.exe? |
| Previous alerts on same host | Pattern and history | Has this host had three alerts this week already? |

## Worked Example: Suspicious PowerShell Alert

Alert: PowerShell launched with an encoded command on WS-2207 at 10:42.

1. **Check the parent process.** The EDR shows `WINWORD.EXE` spawned `powershell.exe`. Word documents should not spawn shells — already suspicious.
2. **Check the command line.** The `-enc` argument carries a base64 blob; the EDR's decoded view shows a download of an executable from an external host. Now it is very likely malicious.
3. **Check the user and host.** User is an accountant on a finance workstation — a plausible target. No prior alerts on this host, so this may be the entry point, not a repeat.
4. **Check the remote side.** Reverse DNS and reputation on the download host show a bulletproof hosting provider with recent malicious submissions. Reputation supports the finding but does not replace steps 1-3.
5. **Scope it.** Query the SIEM for the same domain, IP, and file hash across the fleet; check for other PowerShell-with-encoded-command alerts in the last two hours. One more host hits.
6. **Decide.** Confirmed malicious, two hosts affected → High. Escalate to L2/IR, isolate both hosts per playbook, and hand over a complete note.

Example escalation note:

```
Alert: PowerShell encoded command, WS-2207 (Finance), 10:42
Parent: WINWORD.EXE -> powershell.exe -enc <decoded: download of payload>
User: j.doe (Finance). Host had no prior alerts.
Scope query: same hash found on WS-2211 at 10:47. Domain blocked at proxy.
Actions: both hosts isolated per ransomware/malware playbook; passwords pending reset.
Handing to L2 for eradication support. Ticket IR-417.
```

## Closing False Positives Properly

Document WHY — future you thanks you, and so does detection engineering. A useful closure reason states the benign mechanism and its evidence:

```
Closed: FALSE POSITIVE. powershell.exe invoked by SCCM client
(C:\Windows\CCM\...) on a fixed schedule; binary is vendor-signed;
pattern seen 40x on 12 managed hosts. Suggest narrowing the rule to
non-CCM parents. Feeding to detection tuning backlog.
```

Vague closures ("looks fine", "benign") force the next analyst to redo the work and hide tuning opportunities. Never close on reputation alone — a clean IP behind a real attack is the classic miss.

## Escalation Criteria

| Condition | Escalate to | Urgency |
|---|---|---|
| Confirmed malicious execution or active C2 | L2 / IR | Immediate |
| Same indicator on multiple hosts or accounts | IR | Immediate |
| Privileged or service account suspected compromised | IR + identity team | Immediate |
| Anomalous mass data access or download | IR + data owner | Same shift |
| Suspicious but evidence insufficient for a call | L2 for second opinion | Within the shift |
| Cannot verify because tooling or access is missing | L2 | Do not guess |

## Speed vs Thoroughness

Timebox simple alerts (for example, 5-10 minutes for a routine L1 decision) and escalate when you run out of budget rather than sitting on the alert. Prioritize by asset criticality and potential severity: the domain controller outranks the conference-room PC. Depth belongs to L2 and IR — your job is correct disposition, correct priority, and complete notes. If you cannot decide, escalating is always cheaper than a wrong close.

## Common Triage Mistakes

| Mistake | Consequence | Fix |
|---|---|---|
| Closing on reputation alone | Missed attack behind a "clean" IP or CDN | Check process lineage and command line every time |
| Ignoring scope | Close one alert, miss ten other infected hosts | Always run one fleet-wide scope query |
| No notes or vague notes | Next analyst repeats the work; audits fail | Record what you checked and why you decided |
| Treating every alert as critical | Burnout; real incidents get buried | Classify severity consistently |
| Pivoting forever on one alert | Queue grows unmanaged | Timebox and escalate when stuck |

## See also

- [Incident Response Process](incident-response.md)
- [Tuning & False Positives](../detection/false-positives.md)
- [Log Sources That Matter](../fundamentals/log-sources.md)
- [Playbooks & Checklists](playbooks.md)
