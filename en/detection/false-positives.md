# Tuning & False Positives

A false positive is an alert that fired on legitimate activity, and enough of them will destroy a SOC's credibility faster than any missed detection. Tuning is the ongoing work of raising precision without lowering recall: removing noise while keeping real attacks visible. This page covers the tuning workflow, the common causes of noise, and how to keep severity and baselines honest.

## Why False Positives Destroy Credibility

Alert volume grows faster than analyst headcount. If a rule fires hundreds of times a day and only rarely describes anything real, analysts learn to dismiss it in a glance, and the one day it matters, it is dismissed too. This is alert fatigue: response slows down, investigations become perfunctory, and quiet dismissal heuristics ("it is always that host") replace actual analysis. The numbers matter less than the pattern: noise compounds, trust decays, and detection value follows it down.

## Precision and Recall, Simply

| Measure | Question it answers | Formula | Fishing net analogy |
|---|---|---|---|
| Precision | Of the alerts we raised, how many were real? | `TP / (TP + FP)` | A net that is too wide catches bycatch instead of fish |
| Recall | Of the real attacks, how many did we alert on? | `TP / (TP + FN)` | A net that is too small lets fish swim through |

Tuning a rule to stop false positives usually costs some recall. Every tightening decision should therefore ask: what real attack could this now miss?

## The Tuning Workflow

1. **Triage the false positive.** Confirm it is legitimate, capture the evidence (host, account, parent process, timestamp), and tag it in the alert queue.
2. **Identify the cause.** Is the source noisy, the pattern too broad, the context missing, or the parser broken? See the table below.
3. **Choose the smallest fix.** Tighten the condition, add a narrowly scoped exception, raise a threshold, or fix the data instead of disabling the rule.
4. **Re-test.** Run the revised query over 7 to 30 days of historical logs and compare match counts and samples before and after.
5. **Document.** Record the change, the reason, the owner, and any exception expiry date in version control alongside the rule.
6. **Monitor.** Watch the rule for two weeks; if volume spikes again, the environment changed and the cycle restarts.

## Common False Positive Causes

| Cause | Symptom | Typical fix |
|---|---|---|
| Noisy source | Most hits come from a few known hosts (backup, build, deployment tools) | Scope the rule or add a documented exception for those assets |
| Broad pattern | `\|contains: '-enc'` also matches `-Encoding` and paths like `C:\encrypted\` | Use `\|endswith`, require two matching fields, or use `\|re` with anchors |
| Missing context | Cannot tell a system account from a user, or a service from interactive use | Enrich the data source (user, parent process, host role) |
| Broken parser | Fields are empty, split, or misnamed; rule matches garbage or nothing | Fix field mapping and re-validate against known samples |
| Legitimate admin activity | Helpdesk or admin scripts trip the rule repeatedly | Narrow exception with an owner and expiry, never rule deletion |
| Threshold too low | `>0` events fires during normal daily churn | Baseline normal volume and raise the threshold above it |

## Exceptions Done Right

An exception excludes a specific, justified source from a rule. It is never "turn the rule off":

```yaml
detection:
  selection:
    Image|endswith:
      - '\powershell.exe'
      - '\pwsh.exe'
    CommandLine|contains:
      - 'Invoke-WebRequest'
      - 'DownloadString'
  filter_backup_agent:
    ParentImage|endswith: '\backupagent.exe'
  condition: selection and not filter_backup_agent
```

Every exception should be scoped (exact process, path, account, or host group), documented (justification, ticket, owner), time-bounded (expiry or review date), and reviewed regularly. Keep a register:

| Exception | Rule | Scope | Justification | Owner | Expiry | Last review |
|---|---|---|---|---|---|---|
| EX-014 | PowerShell download cradle | Parent `backupagent.exe` on `srv-backup*` | Vendor agent checks for updates | SOC lead | 2026/07/01 | 2026/04/01 |

## Baselining

You cannot call activity abnormal without knowing what normal looks like in your environment, not someone else's. Baselines are measured over a representative window (typically 2 to 4 weeks) and refreshed each quarter.

| Metric | How to baseline | What it detects |
|---|---|---|
| Processes per host per hour | 30-day average and top-N list per host role | Rare or never-before-seen binaries |
| Logon volume per account (4624 types) | Daily median and usual source hosts | Credential use from new locations |
| DNS query volume and length | 14-day distribution per network segment | Tunneling and low-and-slow exfiltration |
| New services installed per week | Count over 30 days | Out-of-band persistence |
| PowerShell usage share of endpoints | Percentage of hosts running it in 30 days | Living-off-the-land activity on odd machines |

## Severity Calibration

A practical rule: `severity = impact × confidence`. Impact is the damage if the behavior is malicious; confidence is how sure you are that it is.

| Severity | When to use | Example | Expected response |
|---|---|---|---|
| High | High impact and high confidence | LSASS memory access by an unsigned binary | Immediate triage, possible escalation |
| Medium | Suspicious behavior, confidence still unproven | PowerShell download cradle from an office workstation | Same-shift analyst review |
| Low | Weak signal, useful in aggregate or hunt | Single new autorun value set by a standard user | Dashboard review or batch processing |

Severity must be calibrated against your alert-to-incident ratio: if every rule is high, nothing is.

## The Alert-to-Incident Ratio

Count alerts raised and confirmed incidents over a period, then divide. A ratio in the thousands means noise is setting the pace; a ratio in the single digits may mean you are under-triageing or under-detecting. Benchmarks vary by environment, so track your own trend over time: the goal is a ratio that falls as tuning improves, while incident counts stay flat or rise.

## Worked Example: A Rule That Fired on Everything

The first version only required a PowerShell process with `-enc` anywhere in its command line:

```yaml
detection:
  selection:
    Image|endswith: '\powershell.exe'
    CommandLine|contains: '-enc'
  condition: selection
```

It produced roughly 400 alerts per day. The cause was a broad substring: `-enc` matched `-Encoding UTF8` in legitimate scripts, directory names like `C:\share\encoded\`, and unrelated switches. After triage, the rule was rewritten:

```yaml
detection:
  selection:
    Image|endswith:
      - '\powershell.exe'
      - '\pwsh.exe'
    CommandLine|contains: '-EncodedCommand'
  selection_short:
    Image|endswith:
      - '\powershell.exe'
      - '\pwsh.exe'
    CommandLine|re: '(?i)-enc\s+[A-Za-z0-9+/=]{20,}'
  filter_deploy_account:
    User: 'svc_deploy'
  condition: (selection or selection_short) and not filter_deploy_account
```

Then: re-run both versions over 30 days of history, sample 20 matches from each result set, confirm that the known admin deployment account is excluded with a documented exception, and record the before/after match counts in the commit message. The regular expression dialect depends on the backend, so validate it in your SIEM before deploying.

## Tuning Never Ends

Applications change, new software is deployed, log schemas move, and attackers adapt. A tuned rule is a snapshot, not a finish line: schedule periodic rule reviews, watch per-rule alert volume, and treat every false positive report as data for the next iteration.

## See also

- [The Detection Lifecycle](detection-lifecycle.md)
- [Writing Sigma Rules](sigma-rules.md)
- [Alert Triage](../response/triage-alerts.md)
- [Incident Response Process](../response/incident-response.md)
- [SIEM Basics](../fundamentals/siem-basics.md)
