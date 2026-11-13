# Threat Hunting

Threat hunting is the proactive, hypothesis-driven search for attacker activity that produced no alert. Because no detection program covers everything, hunting is how you find the blind spots instead of waiting for them to find you. It matters most on mature teams: the better your alerts, the more valuable the questions no one has asked yet.

## What Hunting Is (and Is Not)

Hunting starts with a question about attacker behavior, not with a dashboard. It is not waiting for alerts to arrive, and it is not ad-hoc grepping with no stated question. A hunt is a time-boxed investigation with a written hypothesis, a defined data source, and a documented outcome that feeds back into detection.

## Hunting Maturity Model

Based on the SANS Hunting Maturity Model (verify details if you formalize your own program):

| Level | Name | Characteristics |
|---|---|---|
| H0 | Minimal | Data sits in silos; work is alert-driven only; no proactive search happens |
| H1 | Minimal automated | Some centralized telemetry; occasional hunts; little repeatability or documentation |
| H2 | Programmatic | Hunts are scheduled, hypotheses are tracked, results reliably become new rules |
| H3 | Optimized | Rich telemetry plus automation and threat intelligence; hunts are measured and improved |

Most teams should aim for H2: a recurring hunt cadence with recorded hypotheses matters more than advanced tooling.

## The Hunt Cycle

| Phase | Actions | Output |
|---|---|---|
| 1. Hypothesis | Pick a technique or behavior; define what would confirm or refute it | A written, testable hypothesis |
| 2. Collect and prepare | Confirm the data source, retention window, and time range | Query-ready data |
| 3. Investigate | Run queries, pivot between endpoints, enrich with asset context | A list of findings with evidence |
| 4. Iterate | Narrow or replace the hypothesis based on what the data showed | New or refined hypothesis |
| 5. Document | Record queries, findings, and gaps | Hunt report plus possible new detection rule |

## Writing Hypotheses

A weak hypothesis cannot fail, so it cannot teach you anything. A strong hypothesis names a behavior, a scope, and the evidence that would settle it.

| Weak hypothesis | Strong hypothesis | Why it is stronger |
|---|---|---|
| "Something suspicious in PowerShell" | "Adversaries living off the land will use `certutil.exe` to download files on our Windows fleet" | Names a binary, a behavior, and a scope |
| "Maybe there is lateral movement" | "An attacker with valid credentials will use WMI to run commands on multiple servers, visible as `wmiprvse.exe` spawning `cmd.exe`" | Names the technique, parent-child relationship, and threshold |
| "Check DNS for anything weird" | "DNS tunneling will appear as repeated queries to a single domain with subdomains longer than 40 characters" | Defines an observable, measurable pattern |
| "Look for new accounts" | "Persistence will be created through local accounts added outside change windows on servers" | Ties behavior to a process and a time scope |

A good habit: write the hypothesis, then write down in one sentence what evidence would prove it wrong before you run a single query.

## ATT&CK-Driven Hunts

Use your coverage matrix (see [The Detection Lifecycle](detection-lifecycle.md)) as a hunt backlog. Techniques with no rule, no telemetry, or an untested rule are hunt candidates; the hunt tells you whether the behavior appears in your environment and what data you would need to detect it. When hunting a technique, verify its ID at attack.mitre.org first, then ask: do we even log what this technique leaves behind?

## Common Hunt Ideas

| Hunt idea | ATT&CK reference (verify) | Data source | First query angle |
|---|---|---|---|
| Suspicious PowerShell usage | T1059.001 | Process creation with command lines (Sysmon 1 / 4688 / EDR) | Command lines containing `-EncodedCommand`, `FromBase64String`, `IEX` |
| Unusual service creation | T1543.003 | System 7045 or Security 4697 | Services created outside change windows or by non-admin accounts |
| Lateral movement via WMI | T1047 | Process creation, WMI-Activity logs | Processes spawned by `wmiprvse.exe` |
| DNS anomalies and tunneling | T1071.004 | DNS query logs | Long subdomains, rare domains, high query volume per host |
| RDP lateral movement | T1021.001 | Security 4624 with LogonType 10 | Accounts logging on to many hosts in a short window |

## Tooling

Hunting is mostly query work over telemetry you already have. SPL for Splunk and KQL for Microsoft Sentinel or Defender are the common SIEM languages; EDR consoles expose the same style of endpoint queries; osquery turns endpoints into SQL tables; and Velociraptor adds fleet-wide forensic hunting with VQL. Example KQL angle for the PowerShell hunt:

```kql
DeviceProcessEvents
| where FileName in~ ("powershell.exe", "pwsh.exe")
| where ProcessCommandLine has_any ("Invoke-WebRequest", "DownloadString", "-EncodedCommand")
| summarize Hits = count() by AccountName, DeviceName, bin(Timestamp, 1h)
| where Hits > 5
```

And a safe osquery angle for local inspection of your own machines:

```sql
SELECT p.pid, p.path, p.cmdline, u.username
FROM processes p JOIN users u ON p.uid = u.uid
WHERE p.cmdline LIKE '%DownloadString%' OR p.cmdline LIKE '%-EncodedCommand%';
```

## When You Find Something

Closing the loop is the point of hunting. Validate the finding, scope it across the fleet, and if it is malicious, hand it to the incident response process with your evidence. Then write or improve a detection rule so the same behavior alerts next time, and note any missing telemetry as a data-source gap. A hunt that ends only in a Slack message has done half the job.

## Documentation and Sharing

A short, consistent hunt report keeps knowledge in the team instead of in one analyst's shell history:

- Title, date, hunter, and time box
- Hypothesis, including what would refute it
- Scope: hosts, accounts, time range, data sources used
- Queries run, copy-pasteable so the next hunt can repeat them
- Findings with evidence (host, user, timestamp, artifact)
- Detections created or updated, and ATT&CK techniques covered
- Gaps found (missing logs, blind spots) and next steps

## See also

- [The Detection Lifecycle](detection-lifecycle.md)
- [Writing Sigma Rules](sigma-rules.md)
- [Tuning & False Positives](false-positives.md)
- [Alert Triage](../response/triage-alerts.md)
- [Defense Labs & Ranges](../professional/defense-labs.md)
