# Second Domain Controller & High Availability

## Problem
A single domain controller is a single point of failure. This project adds redundancy and proves the environment actually survives a DC going down — not just in theory, but with a live kill test.

## What I Built
- DC02 built (Server Core) and promoted as a second domain controller, replicating cleanly with DC01
- DHCP failover configured between DC01 and DC02 (Load Balance mode, 50/50 split)
- A full "kill test": DC01 powered off entirely, with confirmation that:
  - DC02 issued a fresh DHCP lease to CLIENT01
  - DNS resolution continued to work via DC02
  - Domain authentication continued to function
- DC01 brought back online with replication forced and verified clean
- `MaxClientLeadTime` tuned from the 1-hour default down to 5 minutes for faster failover pickup

## Lab Context
- **High-availability pair:** DC01 + DC02
- **Core services covered:** AD DS replication, DNS continuity, DHCP failover
- **Failure model tested:** planned full outage of DC01 with live client validation through DC02

## How It Works
![DC02 promoted](./screenshots/DC02%20promoted.png)
![DHCP failover configuration](./screenshots/DHCP%20failover%20configuration.png)
![The kill DC01 test](./screenshots/The%20kill%20DC01%20test.png)
![Replication status clean](./screenshots/Replication%20status%20clean.png)

## Challenges & Troubleshooting
This was the single most involved debugging chain in the whole lab. DC01 was originally built as `DC1`, then renamed mid-project — `Rename-Computer` only updates so much automatically, and the rename cascaded into several independent, hard-to-connect failures:

1. **`Enter-PSSession` to DC02 failed with a Kerberos error** ("domain isn't available") — root cause was unrelated to Kerberos itself; ruled out WinRM, TrustedHosts, and clock skew before finding the real issues below.
2. **DHCP server authorization failed with Access Denied / RPC unavailable** — DHCP's "server in DC" authorization record in AD still referenced the old hostname, and DC01's own DNS zone had no A-record at all for the new name, only a stale one for the old. Fixed by manually correcting the DNS records, then re-registering DHCP against the corrected name.
3. **Querying DC01's own DHCP scope failed with Access Denied even as a genuine Domain Admin** — turned out to be a loopback self-authentication quirk querying a computer by name over CIM/WMI. Fixed by omitting the computer-name parameter when the command already runs locally on the target.
4. **The AD computer object still showed the old name** even after the Windows-level rename succeeded — fixed via `Rename-ADObject` and `Set-ADComputer -DNSHostName`.
5. **DHCP failover worked for leases but clients still got stuck on stale/APIPA addresses after DC01 went down** — the DHCP scope's DNS option only listed DC01, with no DC02 fallback. Fixed by explicitly setting both DNS servers in the scope option.
6. **A GPO was accidentally scoped to the domain root**, silently applying a 10-minute inactivity lockout to the domain controllers themselves. Fixed by unlinking it from the domain root and re-linking only to the intended OUs.

## Validation Performed
- Confirmed healthy replication state after DC02 promotion and after DC01 recovery
- Confirmed new DHCP leases could be issued while DC01 was powered off
- Confirmed DNS resolution and domain authentication remained functional during the outage
- Confirmed post-recovery synchronization returned environment to steady state

## What I'd Do Differently / Next Steps
- Practice FSMO role seizure/transfer as a follow-up exercise
- Document a clean hostname-rename runbook so this specific cascade doesn't need to be re-diagnosed from scratch if it comes up again
