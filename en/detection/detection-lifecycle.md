# The Detection Lifecycle

A detection is a product, not a one-off script: it is imagined, built, shipped, used by analysts, and eventually changed or retired. Treating rules as living assets is what separates a detection program that improves each quarter from one that quietly rots into noise. This page walks through the full lifecycle, from hypothesis to retirement, and the metrics that tell you whether your detections are working.

## Why "Write and Forget" Fails

- **Noise accumulates.** Every rule written without historical testing fires on legitimate activity. A handful of noisy rules is enough to push real alerts off the screen.
- **Environments drift.** Applications change, agents are upgraded, log fields move or get renamed, and a rule that matched perfectly in January silently matches nothing in June.
- **Attack paths change.** Adversaries adopt new tools and techniques; coverage that was complete last year has gaps today.
- **Trust erodes.** When analysts repeatedly see alerts that are nothing, they stop investigating carefully. The cost is measured in missed real incidents.

## The Lifecycle Phases

| Phase | What happens | Exit criteria |
|---|---|---|
| 1. Hypothesis | State the attacker behavior you want to catch, and the data that would prove it | A testable question, written down |
| 2. Research and ATT&CK mapping | Find the technique, real-world write-ups, and the log fields that expose it | Technique ID and data source confirmed |
| 3. Write rule | Author the detection (Sigma or native query) | Rule passes syntax validation |
| 4. Test against logs | Run it over historical data, sample matches by hand | Known-bad matches, known-good does not |
| 5. Deploy | Convert, schedule or stream it, route output to the SOC | Alert arrives in the queue with context |
| 6. Alert and triage | Analysts investigate; findings feed back to the rule owner | Runbook exists; decisions are recorded |
| 7. Tune | Fix false positives, adjust severity, add scoped exceptions | Alert volume is sustainable |
| 8. Retire or improve | Replace stale rules, re-scope after infrastructure changes | Reason documented in version control |

Phases 6 to 8 loop back into phase 1: a triaged alert often becomes the next hypothesis.

## From Hypothesis to Rule

A good hypothesis names attacker behavior and the evidence it would leave. Bad ones describe feelings ("something looks off").

| Hypothesis | Data needed | Rule idea |
|---|---|---|
| Attackers will use encoded PowerShell to hide payloads | Process creation with full command lines (Sysmon 1 or Security 4688) | Flag `-EncodedCommand` followed by a long base64 string |
| Persistence will be added through Run keys | Registry value-set events (Sysmon 12/13) | Alert on new `Run`/`RunOnce` values set by unusual accounts |
| Credential theft will target LSASS memory | Process access events (Sysmon 10) or EDR process telemetry | Alert on rare processes opening `lsass.exe` |
| Our build agents legitimately run PowerShell all day | Same process logs plus an asset inventory | Inverse rule: alert if PowerShell runs outside known admin hosts |

## Mapping to MITRE ATT&CK

Mapping each rule to a technique ID turns a pile of rules into a measurable coverage picture. A **coverage matrix** lists the techniques you care about and which detections, data sources, and hunts touch each one; empty cells are your blind spots. Verify every technique ID against https://attack.mitre.org/ before publishing a matrix, because remembered IDs are easy to get wrong.

| Technique | Data source | Rule | Status |
|---|---|---|---|
| T1059.001 PowerShell | Process creation with command lines | Suspicious PowerShell Download Cradle | Deployed |
| T1547.001 Run key persistence | Registry value set | Registry Persistence via Run Key | Testing |
| T1003.001 LSASS memory access | Sysmon 10 / EDR process access | Suspicious LSASS Access | Draft |
| T1021.001 RDP logons | Security 4624, LogonType 10 | (none) | Gap: hunt first |

Coverage is not the goal by itself. One well-tuned rule with good telemetry beats ten rules that never fire because the underlying data is missing.

## Detection as Code

Treat rules like software: store them in Git, require a pull request for every change, and run automated checks before merge.

- **Version history:** every tuning change has an author, a date, and a reason.
- **Review:** a second pair of eyes catches broken field names and over-broad conditions.
- **CI:** validate syntax (`sigma check`), lint tags and IDs, and run the query against a stored sample of known-bad and known-good logs.
- **Deployment:** a pipeline converts and pushes rules to the SIEM instead of hand-editing console searches.

## Quality Attributes

| Attribute | Plain language | Cost when it is poor |
|---|---|---|
| Precision | Of the alerts this rule raises, how many are real (`TP / (TP + FP)`) | Alert fatigue and lost credibility |
| Recall | Of the real attacks of this type, how many did we catch (`TP / (TP + FN)`) | Silent, successful intrusions |
| Timeliness | Delay between attacker action and alert (MTTD) | More time for the attacker to act |
| Stability | How much alert volume varies week to week | Constant emergency tuning |
| Context | Evidence included in the alert (user, parent process, host role) | Slow, guesswork triage |

Precision and recall trade against each other: tightening a rule cuts false positives but can also cut real detections. Always re-measure both after a tuning change.

## Metrics for the Detection Program

| Metric | How to compute | What it tells you |
|---|---|---|
| ATT&CK coverage % | Techniques in scope with at least one tested rule, divided by techniques in scope | Where the blind spots are |
| Alert-to-incident ratio | Alerts raised divided by confirmed incidents | Whether noise is drowning real signals |
| MTTD | Median time from event to alert | How fast you detect |
| Rule health | Rules with zero alerts in 30 days, or a sudden spike | Broken pipelines or new behavior |
| False positive rate per rule | FP alerts divided by total alerts, per rule | Which rules to tune first |
| Hunt conversion rate | Hunts that produced a new or improved rule | Whether hunting pays off |

## Worked Example: One Detection From Idea to Tuned Rule

1. **Hypothesis.** "Adversaries will use PowerShell to download payloads, and our endpoints log full command lines."
2. **Research and mapping.** Confirm T1059.001 at attack.mitre.org; check 30 days of process logs to confirm the fields exist and are populated.
3. **Write.** Author the [Sigma rule](sigma-rules.md) with `Image|endswith` plus `CommandLine|contains` selections.
4. **Test.** Run the converted query over 14 days of history. Count matches, then read 20 of them by hand: expect admin scripts and software deployers among the hits.
5. **Deploy.** Convert to the SIEM query language, schedule it, and route alerts to a tuning queue rather than to the on-call page. Severity `medium`.
6. **Alert.** Attach a short runbook: what to check, when to escalate, and who owns the rule.
7. **Tune.** After one week, one deployment tool accounts for most hits. Add a narrowly scoped exception for that parent process, document owner and expiry, and re-run the historical test to measure the drop.
8. **Improve.** An alert leads to a hunt for other downloaders, which produces a second rule for a different LOLBin. Both rules enter the ATT&CK matrix; a rule with no data behind it is marked as a telemetry gap, not a success.

## See also

- [Writing Sigma Rules](sigma-rules.md)
- [Tuning & False Positives](false-positives.md)
- [Threat Hunting](threat-hunting.md)
- [Alert Triage](../response/triage-alerts.md)
- [Security Operations Fundamentals](../fundamentals/security-operations.md)
