# Log Sources That Matter

Logs are the raw material of defense: without them there is no detection, no investigation, and no way to prove what happened. The practical skill is knowing which sources reveal the most, in which order to collect them, and what you can never see. This page is a prioritized map of log sources for small and mid-sized environments.

## Why Logs Matter

Every detection rule, hunt, and forensic timeline is built from events someone chose to record. A SIEM with poor inputs produces confident, wrong answers; a modest environment with the right five sources can outperform an expensive one with gaps. Source selection is therefore the highest-leverage decision in a detection program.

## Priority Log Sources

| Log source | Key events | What it tells you |
| --- | --- | --- |
| Windows Security event log | 4624 (successful logon), 4625 (failed logon), 4688 (process creation), 4720 (user account created) | Who logged in and from where, which attempts failed, what processes ran, when accounts were created |
| Sysmon (Windows) | 1 (process create with hashes and command line), 3 (network connection), 11 (file create) | Rich endpoint detail beyond default Windows logging: command lines, outbound connections, file drops |
| Linux auth and audit logs | /var/log/auth.log or secure (sshd, sudo), auditd rules | Login attempts and failures, privilege escalation, command and file access on Linux hosts |
| Firewall and proxy | Allowed and denied sessions, proxy requests and verdicts | Perimeter traffic, exposed services, outbound destinations, blocked and allowed web activity |
| DNS server logs | Queries and responses | Which domains your hosts resolve, an early signal of command-and-control and tunneling |
| EDR telemetry | Process trees, script execution, fileless behavior, isolation status | Endpoint behavior with context, plus the ability to contain a host immediately |
| Cloud audit logs (AWS CloudTrail, Azure Activity, GCP Audit Logs) | API calls, console sign-ins, IAM and key changes | Who changed cloud resources, created credentials, or opened storage to the internet |
| Web server access logs | Requests, status codes, source IPs | Scanning, exploitation attempts, and anomalous request patterns against your applications |
| Email gateway logs | Delivered, quarantined, and blocked messages; URL and attachment verdicts | Phishing delivery, impersonation attempts, and what users actually received |

### Two notes on Windows events

- **4624 and 4625 are the backbone of authentication detection:** repeated 4625 spikes reveal brute force, while 4624 with unusual logon types reveals lateral movement.
- **4688 command-line auditing and Sysmon must be explicitly enabled.** Default installs do not give you the detail most detections assume.

## The Starter Set for a Small Org

If you are building collection from zero, order the work like this:

1. **Identity** — domain controller Windows Security logs (4624, 4625, 4720 and related account-change events), or your cloud identity provider's sign-in logs.
2. **Endpoints** — EDR on every server and workstation you can reach; where EDR is impossible, at minimum collect process creation (4688 or Sysmon event 1).
3. **Perimeter** — firewall logs, including denied connections.
4. **DNS** — your internal resolver's query log, shipped off the host immediately.
5. **SaaS and cloud** — Microsoft 365 or Google Workspace audit logs, plus AWS CloudTrail, Azure Activity, or GCP Audit Logs where applicable.

> **First five sources:** domain controller authentication logs, EDR telemetry, firewall logs, DNS query logs, and your SaaS or cloud audit log. Together they cover identity abuse, endpoint execution, network boundaries, command-and-control callbacks, and cloud control-plane changes, which is what most investigations ask for first.

## What One Event Looks Like

Normalization turns lines like these into structured fields you can aggregate across sources:

```
Windows 4625 (raw, simplified):
  Time=2026-11-12T09:14:02Z  Host=dc01  ID=4625  LogonType=3
  Account=j.mayer  SrcIP=10.20.4.55  Status=0xC000006D

Linux auth.log (raw):
  Nov 12 09:14:07 app01 sshd[4412]: Failed password for j.mayer
  from 10.20.4.55 port 51224 ssh2

Normalized into the same schema:
  event=auth.failure  user=j.mayer  src_ip=10.20.4.55  outcome=failure
```

Once both sides share field names, a rule can say "five failures from one source in five minutes" and mean the same thing for Windows, Linux, VPN, and cloud logins.

## Volume vs Value

Sources differ enormously in size, and bigger is not better:

- **High volume, high value:** process creation and DNS. Noise is manageable once fields are normalized and rules are tuned; dropping these sources to save storage usually costs more in missed detections than it saves.
- **High volume, moderate value:** full proxy and web access logs. Keep them searchable for recent weeks, archive the rest, and filter known-benign destinations at the source where possible.
- **Low volume, critical value:** account creation, IAM changes, firewall denies to management interfaces. These are cheap to keep and should never be sampled away.

Storage decisions should follow investigations rather than budgets alone: ask which sources your last incident actually needed, and keep those hot.

## Retention Basics

As general guidance, keep the sources you investigate with **searchable for at least 90 days.** Investigations routinely reach back weeks, through user-reported delays and attacker dwell time, and many teams find that 90 days is the point where routine pivots stop failing. Archive older data cheaply in object storage or compressed tiers when regulation or policy requires longer periods.

Retention is context-dependent. Confirm requirements against the legal, regulatory, and privacy obligations for your industry and region rather than treating 90 days as a universal rule.

## Confirm Collection Actually Works

Adding a source to a configuration is not the same as receiving its events. Validate each new source with a short checklist:

- Generate a known event yourself, such as a failed login, and find it in the SIEM within minutes.
- Compare counts: roughly how many events per hour should this source produce, and is that what arrives?
- Check that timestamps land in the correct timezone and that fields parsed into the right columns.
- Verify the source keeps shipping when its buffer fills or its network link blips.
- Alert on silence: a source that stops sending should page someone, because quiet looks like healthy.

A source that nobody verifies tends to fail silently, and silent gaps are only discovered during an incident.

## Common Gaps

- **Endpoints without EDR** — legacy systems, unmanaged devices, or neglected servers become invisible exactly where attackers like to live.
- **Unlogged DNS** — clients using public resolvers, or an internal resolver whose query log nobody ships, erase your earliest callback signal.
- **Detail not enabled** — process creation auditing off, Sysmon never installed, command-line auditing disabled: the source exists but is empty.
- **Cloud control plane left local** — CloudTrail or Activity logs that nobody exports can disappear with the account or with an attacker's deletion.
- **Clock skew** — hosts with drifting timestamps make correlation unreliable; synchronize everything to a consistent time source.
- **Logs only on the host** — an attacker with administrative rights clears local logs, so ship them off-box in near real time.

## Verify Before You Build

Event IDs, audit subcategories, and cloud logging offerings change over time, and new sources appear regularly. Check the official Microsoft, AWS, Azure, and GCP documentation, plus your appliance vendors' references, before finalizing parsers, dashboards, or retention policies built on this table.

## See also

- [SIEM Basics](siem-basics.md)
- [Writing Sigma Rules](../detection/sigma-rules.md)
- [Defense Tooling](../professional/defense-tooling.md)
- [Threat Hunting](../detection/threat-hunting.md)
