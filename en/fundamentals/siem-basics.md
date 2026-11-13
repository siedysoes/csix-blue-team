# SIEM Basics

A Security Information and Event Management (SIEM) system is where an organization's security telemetry comes together: it centralizes logs from many sources, normalizes them into a common shape, correlates them over time, and raises alerts when patterns look wrong. Almost every SOC workflow passes through a SIEM, which makes its mechanics essential to understand even if you never administer one. This page covers architecture, correlation rules with a worked example, common platform choices, and the factors that decide whether a SIEM helps or becomes an expensive log archive.

## What a SIEM Does

The whole product reduces to a four-step pipeline:

**centralize -> normalize -> correlate -> alert**

1. **Centralize** — pull events from endpoints, servers, cloud services, and appliances into one place.
2. **Normalize** — parse raw lines into structured fields so a Windows logon and a Linux SSH failure can be compared.
3. **Correlate** — evaluate conditions across events and time windows, not just single lines.
4. **Alert** — when a condition holds, create a case for a human with enough context to act.

## Architecture

| Stage | Component | What happens |
| --- | --- | --- |
| Collection | Agent or forwarder (also agentless API pulls) | Ships logs off each host or service; off-box shipping is what makes logs tamper-resistant |
| Parsing and normalization | Parser / ingest pipeline | Extracts timestamp and fields (src_ip, user, event_code) and maps them to a common schema |
| Storage | Hot, warm, and cold tiers | Recent data lives in fast storage for investigations; older data moves to cheaper tiers |
| Correlation | Detection rules | Thresholds and sequences evaluated against the normalized data |
| Dashboards | Visualization | Source health, alert trends, and pivots analysts use during investigations |
| Alerting | Routing and notifications | Sends qualifying events to the case or ticket system and pages analysts |

## Ingestion vs Normalization

**Ingestion** is getting the bytes in: an agent reads a file, or an API returns records, and the SIEM accepts them. **Normalization** is making them meaningful: the raw line is split into fields, timestamps are parsed into a common format, and field names are mapped to a shared schema, for example every authentication result becoming `auth.success=true` or `auth.success=false`.

The distinction matters because ingestion without normalization just produces a searchable pile of text. You can grep it, but you cannot reliably answer "failed logins per source across Windows and Linux" until both sides speak the same field names.

## Correlation Rules

A correlation rule expresses a condition that only matters in combination: **event A within X minutes, plus event B, equals an alert**. Rules generally specify a scope (which sources or assets), a threshold (how many events), a time window (over how long), and a severity.

Single-event alerts, such as a known-bad malware hash being seen, are simple. Sequence and threshold rules catch behavior that is harmless line by line but suspicious in aggregate: repeated failures followed by a success, a beacon repeating on a fixed interval, or an account created and then used from a new country.

## Worked Example: Brute Force Alert

Rule statement: **3 failed logins followed by 1 successful login for the same account from the same source within 5 minutes.** Individually, failed logins happen constantly and a success is normal; together they suggest a guessed password.

Illustrative Splunk SPL (field names vary by implementation, and the search time window supplies the 5-minute bound):

```spl
index=auth (action=failure OR action=success)
| bin _time span=5m
| stats count(eval(action="failure")) as fails count(eval(action="success")) as ok by _time, src_ip, user
| where fails >= 3 AND ok >= 1
```

The alert that reaches the analyst should carry the account, source IP, both counts, and the earliest and latest event times, enough to decide within the first minute whether to escalate.

## Platform Comparison

| Platform | Model | Good fit | Notes |
| --- | --- | --- | --- |
| Splunk | Commercial, license-based | Large or mature teams that value a powerful search language | Capable and widely used; licensing is the main cost driver, verify current pricing |
| Elastic Stack (ELK) | Self-managed free tier plus paid subscriptions | Teams that want to run their own stack and keep data in-house | You operate the cluster: capacity, upgrades, and durability are your job |
| Wazuh | Free and open source, XDR plus SIEM | Budget-limited teams, labs, and mid-size deployments | Bundles detection, integrity monitoring, and incident response features |
| Microsoft Sentinel | Cloud-native SaaS in Microsoft's ecosystem | Organizations already invested in Microsoft identities and endpoints | Managed service; verify current pricing, connectors, and retention behavior |

> Verify before committing: pricing models, editions, and cloud features change frequently. Treat every figure you read as a snapshot and confirm it on the official pricing and documentation pages. This page deliberately lists no exact prices.

## Query Language Basics

You do not need every language, but you should recognize the shape of each.

### SPL (Splunk)

Pipe-based: a search followed by transformations.

```spl
index=windows EventCode=4625 | stats count by src_ip | where count > 10
```

### KQL (Kusto Query Language)

Used by Microsoft Sentinel and Defender, tabular in style: filter with `where`, then aggregate with `summarize`, for example `SecurityEvent | where EventID == 4625 | summarize FailedLogins=count() by IPAddress`.

### Lucene-style queries

Used by Elastic's classic query syntax, and simplified further in Kibana's KQL: field-term boolean matching such as `event_code:4625 AND NOT user:svc_backup`.

The transferable skill is the shape of the question: filter first, then aggregate, then compare against a threshold.

## What Makes a SIEM Succeed or Fail

1. **Data quality.** Missing sources, parsing errors, and clock skew make every rule unreliable, so collection-health monitoring is not optional.
2. **Tuning.** Rules that are too broad drown analysts in noise; rules that are too narrow miss real activity. Both failures look like "the SIEM does not work."
3. **Staffing.** A SIEM without analysts on a schedule is an expensive log archive. Alerts create work, and that work needs owners, runbooks, and a feedback loop back to the rule authors.

Build in that order: get the data right, then tune what fires, then scale the people and process around it.

## See also

- [Log Sources That Matter](log-sources.md)
- [Writing Sigma Rules](../detection/sigma-rules.md)
- [Tuning & False Positives](../detection/false-positives.md)
- [Defense Tooling](../professional/defense-tooling.md)
