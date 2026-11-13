# Network Defense

Network defense is the practice of layering controls around traffic so that no single failure turns into a breach, and of watching that traffic so you notice when something slips through. It complements endpoint and log-based defense: even when you cannot read the content of a connection, you can usually see that it happened. This page covers the layered model, firewall and IDS or IPS fundamentals, segmentation, early blocking, and flow-based monitoring.

## The Layered Model

| Layer | Component | Primary job |
| --- | --- | --- |
| Perimeter | Stateful firewall | Permit sanctioned traffic in and out, deny everything else by default |
| Internal | Segmentation (VLANs, ACLs, zero-trust policies) | Contain movement, so a compromised workstation cannot reach every server |
| Inspection | IDS / IPS | Detect or stop known-bad patterns in traffic |
| Early blocking | DNS filtering | Refuse to resolve malicious domains before any connection is attempted |
| Mediation | Proxy / secure web gateway | Control and inspect outbound web requests |
| Visibility | NetFlow, sFlow, network detection | Record who talked to whom, how much, and when, the basis for hunting |

Each layer fails differently, which is the point: an attacker who bypasses one still meets the next, and every layer leaves evidence behind.

## Firewall Basics

**Stateful inspection** is the standard behavior of a modern firewall: it tracks connection state such as new, established, and related, and automatically allows return traffic for connections the firewall itself permitted, instead of needing a separate rule for every reply packet.

Two operating models matter:

- **Explicit allow (default deny)** — nothing passes unless a rule names it. This is the model you want at a trust boundary.
- **Allow-all** — rules such as `any -> any, accept` render the firewall decorative: it blocks nothing and records nothing useful.

**Rule hygiene principles:**

- One rule, one purpose, each with an owner and a comment explaining why it exists.
- No `any/any` permits at boundaries; remove rules that are shadowed by earlier rules or no longer have a business justification.
- Log both allows and denies at boundaries, and review the deny log for reconnaissance attempts.
- Test changes in a maintenance window, keep a rollback ready, and recertify the rule set periodically.

## IDS vs IPS

| Aspect | IDS (intrusion detection) | IPS (intrusion prevention) |
| --- | --- | --- |
| Placement | Tap or mirror port (out of band), sees a copy of traffic | Inline, traffic physically flows through the device |
| Action | Alerts only; a human decides what happens next | Alerts and can drop or reset the connection automatically |
| Main risk | Missed or late reaction to an alert | A false positive blocks legitimate business traffic |
| Best used for | Visibility and tuning where availability outweighs speed | Environments with well-tuned rules and tested exceptions |

Both rely on two detection approaches, often in combination:

- **Signature matching** compares traffic against known patterns. Fast and precise for known threats, blind to anything new.
- **Anomaly detection** compares traffic against a learned baseline of normal behavior. It can flag novel activity, but produces more noise and needs tuning.

## Network Segmentation

A **flat network**, one broadcast domain where every host can reach every other host, turns a single compromised laptop into a path to domain controllers, databases, and backup systems. Attackers do not need sophisticated techniques when the route is already open.

Segmentation closes those routes: workstation VLANs, server zones, a separate management network, and explicit firewall or ACL rules between them, with default deny and only required ports permitted, for example workstations reaching an application server on 443 and nothing else.

The **zero-trust** framing goes further: never grant access based on network location alone. Verify identity and device health explicitly, grant least privilege per application, and assume the network itself is hostile. In practice, segmentation plus strong identity is where most organizations start.

## DNS Filtering and Proxy as Early Blocks

Most malware cannot act until it resolves a command-and-control (C2) domain. Blocking that lookup at the resolver or proxy stops the callback chain before the payload ever connects, a cheap, centrally managed control that needs no endpoint change:

```
malware.exe  ->  DNS query: update-cdn.badexample[.]net
                        |
              resolver policy: known-bad domain
                        |
              NXDOMAIN / block page returned
                        |
        no connection established, no C2 session
                        |
       EDR and DNS logs record the attempted lookup
```

DNS filtering and proxies are early blocks, not complete ones: attackers use direct IP addresses, algorithmically generated domains, and encrypted DNS to evade them. Treat them as one layer that also generates high-value telemetry such as rare domains, lookup bursts, and unusually long DNS names that suggest tunneling, and pair them with EDR and flow monitoring.

## Network Monitoring Basics

**NetFlow and sFlow** produce metadata records rather than packet captures: source, destination, ports, protocol, byte counts, and timestamps for each conversation. That is usually enough to see:

- **Beaconing** — regular, evenly spaced connections to one external host, a classic C2 signature.
- **Data exfiltration** — large or oddly timed outbound transfers, especially to unexpected destinations.
- **Scanning** — one internal host contacting many hosts or ports in a short period.

Metadata scales far better than full packet capture: it is smaller, retains longer, and answers most investigations that start with "who talked to whom, and when?"

## The Encryption Caveat

Modern traffic is encrypted by default with TLS, so without interception you see **metadata, not content**: destination and certificate, connection timing, and byte volumes, but not the URL path or payload. That still answers many questions, but not all of them.

Two workable responses: perform TLS inspection at the proxy where policy and law allow, weighing privacy, consent, and regulatory constraints explicitly; or skip content inspection and rely on endpoint telemetry, since EDR sees the process that made the request, the command line that spawned it, and the file it dropped, which often matters more than the packet bytes.

## Defense in Depth

```
                           Internet
                              |
                     +-----------------+
                     |  Perimeter FW  |   default-deny, stateful
                     +-----------------+
                              |
              +---------------+---------------+
              |                               |
        [ DMZ: web, mail ]            [ Workstations ]
              |                               |
              |                      [ Segmentation:
              |                        VLANs + ACLs ]
              |                               |
              +-------- [ Core: AD, servers ] +
                              |
          +-------------------+-------------------+
          |                   |                   |
     [ IDS / IPS ]      [ DNS filtering ]     [ Proxy ]
          |                   |                   |
          +------- telemetry: logs and flows -----+
                              |
                       [ NetFlow + EDR ]
                              |
                            [ SIEM ]   correlation and alerting
```

Every layer both blocks what it can and feeds evidence upward: firewall denies, DNS queries, proxy events, and flow records become the log sources a SIEM correlates into alerts, which is exactly the material covered in the fundamentals pages on logs and SIEMs.

## Mapping to Earlier Topics

Network devices are first-class log sources: firewall, proxy, and DNS logs are collected, normalized, and correlated in the SIEM, where they join endpoint telemetry from EDR. When a detection fires, the response action may come from any layer, a DNS block, a firewall rule, or host isolation, which is why network defense and security operations are taught together.

## See also

- [Log Sources That Matter](log-sources.md)
- [SIEM Basics](siem-basics.md)
- [Defense Tooling](../professional/defense-tooling.md)
- [Threat Hunting](../detection/threat-hunting.md)
