# Defense Labs & Ranges

Defenders need labs for the same reason attackers practice: you learn detection and response by breaking and observing systems you control. A home SOC lab lets you generate real telemetry, test detections, and rehearse incident response without risking production. This article gives an architecture, build steps, and a 30-day plan for an isolated lab.

## Why Defenders Need Labs

- You can trigger your own alerts and watch the full path: action, log, rule, alert, triage note.
- Safe failure: break detections and rules with no business impact.
- Concrete evidence for interviews and portfolios: a lab you built beats vague claims.
- Practice response: isolate, collect evidence, write a timeline, run a lessons-learned pass.

## Home SOC Lab Architecture

```text
        [ Host-only network: 192.168.56.0/24  (NAT for updates only) ]

 +--------------------+    +--------------------+    +--------------------+
 | Windows Domain     |    | Windows            |    | Linux Server       |
 | Controller         |<-->| Workstation        |    | (web app target)   |
 | (AD, DNS, GPO)     |    | (user, Office)     |    |                    |
 +--------------------+    +--------------------+    +--------------------+
          |                        |                         |
          +-----------+------------+-------------------------+
                      |
              +--------------------+       +------------------------+
              | Wazuh Manager      |<------| Attacker VM            |
              | (SIEM/XDR, agents, | logs  | (Kali or atomic test   |
              |  rules, dashboards)|       |  runner)               |
              +--------------------+       +------------------------+

   [ Analysis host ] -- runs Autopsy/Velociraptor and the SIEM dashboards
```

Keep the attacker VM on the same isolated network so its traffic never leaves your machine.

## Build Steps

| Component | Purpose | Notes |
|---|---|---|
| Hypervisor (VirtualBox, VMware, or Hyper-V) | Hosts all VMs | Take a snapshot before every exercise: that is your undo button |
| Windows Domain Controller | AD, DNS, GPO for realistic identity telemetry | Baseline it with a couple of user accounts |
| Windows workstation | Generates Windows events, PowerShell, Office telemetry | Domain-joined so logon events land centrally |
| Linux server | App target and syslog source | Diverse log formats to practice on |
| Wazuh manager plus agents | Central log collection, rules, dashboards | Free and open source; use official docs for current install steps |
| Attacker VM (e.g. Kali) | Generate safe, controlled attack telemetry | Used only against this lab |
| Host-only networking | Isolation from your real network | Avoid bridged adapters; snapshot the baseline |

## Generating Your Own Telemetry Safely

Atomic Red Team is an open-source project of small test techniques, mapped to MITRE ATT&CK, designed to be executed against systems you own to check whether your detections fire. The defensive workflow: pick a technique your rule should cover, run its test in the lab, confirm the alert and timeline appear, then fix the rule if nothing fired. This is legitimate defensive validation on your own machines — verify the current repository and its safety notes before running anything, disable tests you do not understand, and never point test tooling at systems outside your lab.

## Free Ranges and Platforms

| Platform | Cost | Format | Focus |
|---|---|---|---|
| LetsDefend | Free tier plus paid | Hosted SOC environment | Alert triage and investigation on simulated alerts |
| Blue Team Labs Online | Free tier plus paid | Browser-based labs | Investigation challenges and artifact analysis |
| CyberDefenders | Free tier plus paid | Blue-team CTF-style labs | Forensics and analysis challenges |
| TryHackMe (blue team rooms) | Free tier plus paid | Guided browser-based rooms | Structured learning paths for defenders |
| SANS CyberRanges / NetWars | Varies (often via SANS events or courses) | Instructor-led or event-based ranges | Structured, intensive skill building |

Verify current offerings, free-tier limits, and pricing on each platform's official site — these change frequently.

## Defensive CTFs

CTFs compress skills under time pressure: forensics challenges build artifact fluency, detection challenges build query logic, and incident scenarios build prioritization. Treat write-ups as portfolio material — note which artifact you used, what question it answered, and what you would automate next time. Placement matters less than the write-up; the learning happens when you explain your path.

## A 30-Day Lab Plan

| Days | Focus | Output |
|---|---|---|
| 1-7 | Build and snapshot the lab; install Wazuh; register agents | Dashboard showing real events from three machines |
| 8-14 | Learn your log sources: Windows logon events, PowerShell logging, Linux auth logs | A one-page cheat sheet of the events you actually use |
| 15-21 | Run 5-10 atomic tests; tune rules that stayed silent or fired too much | Five tuned detections with notes |
| 22-30 | Full exercise: inject an incident and respond end to end | Timeline, short incident report, lessons list |

## Safety Note

Run everything inside your isolated lab: host-only networking, snapshots before each exercise, and no test activity against any system you do not own or lack written authorization to touch. Treat the generated attack telemetry as training data for your own defenses — nothing more.

## See also

- [Defense Tooling](defense-tooling.md)
- [Blue Team Careers & Certs](blue-team-career.md)
- [Threat Hunting](../detection/threat-hunting.md)
- [Alert Triage](../response/triage-alerts.md)
