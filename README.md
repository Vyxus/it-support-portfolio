# HelpDeskLab / IT Support & Infrastructure Portfolio

**Akram El Asouly**  IT Support Professional · Agadir, Morocco
📧 [AkramEIAsouly@proton.me](mailto:AkramEIAsouly@proton.me) &nbsp;|&nbsp; 🎓 Google IT Support Professional Certificate

![Active Directory](https://img.shields.io/badge/Active_Directory-0E1E2F?style=flat-square) ![pfSense](https://img.shields.io/badge/pfSense-24405F?style=flat-square) ![Windows Server](https://img.shields.io/badge/Windows_Server-24405F?style=flat-square) ![Linux](https://img.shields.io/badge/Linux%20%2F%20Ubuntu-00C2A8?style=flat-square) ![Azure](https://img.shields.io/badge/Azure_Arc_%2F_Sentinel-0E1E2F?style=flat-square) ![Networking](https://img.shields.io/badge/Networking%20%2F%20TCP--IP-24405F?style=flat-square)

A self-built, multi-server IT infrastructure lab simulating a real small-business / MSP client environment. Built end-to-end in VirtualBox: network edge, Active Directory with high availability, file/print services, a ticketing system, a mail server, and a live SIEM with hybrid cloud management via Azure Arc.

Every project below includes real screenshots, configuration details, and an honest account of what broke and how it was diagnosed. Not just a clean happy-path walkthrough. That diagnostic process, more than any single service, is the point of this repo.

---

## About me

I hold the **Google IT Support Professional Certificate** and am based in **Rabat, Morocco**, currently looking for IT Support / Junior Sysadmin roles. I built HelpDeskLab to go beyond scripted tutorials: seven services that actually depend on each other on one network, with the kind of subtle, multi-layered failures a real environment produces. Traced to root cause with logs and debug output rather than guesswork.

Reach me at **[AkramEIAsouly@proton.me](mailto:AkramEIAsouly@proton.me)**.

## Quick look

<table>
<tr>
<td><img src="images/hero-pfsense.png" width="280"/><br/><sub>pfSense — network edge dashboard</sub></td>
<td><img src="images/hero-killtest.png" width="280"/><br/><sub>Live DC01 kill test — DC02 keeps the domain online</sub></td>
<td><img src="images/hero-arc.png" width="280"/><br/><sub>Three servers connected via Azure Arc for hybrid SIEM</sub></td>
</tr>
</table>

## Architecture

Everything sits on one lab network, behind one firewall:

![Architecture diagram](images/architecture-diagram.png)

pfSense is the edge every other server sits behind on `192.168.50.0/24`. DC01, DC02, and FILESRV01 are additionally onboarded to Azure via Arc, feeding a Log Analytics workspace that Microsoft Sentinel monitors.

## Projects

| # | Project | What it demonstrates | The debug |
|---|---------|----------------------|-----------|
| 01 | [Firewall & Network Edge (pfSense)](./01-firewall-pfsense) | WAN/LAN segmentation, NAT, firewall rules | Interface assignment vs. VirtualBox host-only adapters |
| 02 | [AD Domain Lab](./02-ad-domain-lab) | OUs, department security groups, 3 GPOs, NTFS permissions proven both ways | A VirtualBox unattended-install account that looked like admin but wasn't |
| 03 | [Second DC & High Availability](./03-second-dc-ha) | DHCP failover, a real kill test (DC01 powered off, DC02 keeps the domain alive) | **The DC rename cascade — 6 independent root causes.** [Full breakdown ↓](#the-dc-rename-cascade) |
| 04 | [File & Print Server](./04-file-print-server) | Migrated shares, FSRM quotas, GPO-deployed printer | Stale IPv6 DNS placeholders + a silently-skipped migration step |
| 05 | [Ticketing System (osTicket)](./05-ticketing-system) | Full LAMP stack, full ticket lifecycle | A cryptic HTTP 500 traced to a silently-rejected MySQL password change |
| 06 | [Mail Server (Postfix/Dovecot/Roundcube)](./06-mail-server) | Real SMTP delivery + IMAP webmail | **A Dovecot config override hiding the inbox.** [Full breakdown ↓](#chasing-a-ghost-in-the-mailbox) |
| 09 | [SIEM with Microsoft Sentinel](./09-siem-sentinel) | Azure Arc hybrid onboarding, [live KQL detections](./09-siem-sentinel/kql-detections) | A 3-layer DNS → auth → RBAC chain, plus a documented platform limitation |

## Troubleshooting highlights

The two deepest diagnostic stories in the lab, in brief — full write-ups live in their project folders.

### The DC rename cascade

Renaming a domain controller mid-project (`DC1` → `DC01`) cascaded into **six independent, hard-to-connect failures**: a Kerberos error that was actually about stale DNS, a DHCP authorization failure, a loopback self-authentication quirk, a stale AD computer object, a DHCP scope missing failover DNS options, and a GPO that accidentally locked out the DCs themselves. Each layer needed its own diagnosis — fixing one didn't fix the next.
→ [Read the full breakdown in 03-second-dc-ha](./03-second-dc-ha)

### Chasing a ghost in the mailbox

Roundcube threw *"Internal error occurred"* loading the inbox, even though Postfix logs confirmed every message was delivered. Permission and package fixes made partial progress but never solved it. Reading Dovecot's own verbose debug trace (`doveadm -Dv force-resync`) revealed a `mail_inbox_path` override in `10-mail.conf` silently pointing IMAP at an empty directory — a completely different location than where mail was actually landing. Every individual symptom pointed to a plausible-but-wrong cause; only the debug log revealed the real one.
→ [Read the full breakdown in 06-mail-server](./06-mail-server)

## Skills & technologies

**Windows Server & Identity** — Active Directory Domain Services · DNS & DHCP (incl. failover) · Group Policy Objects · NTFS permissions & security groups · FSRM storage quotas

**Networking & Security** — pfSense firewall / NAT / edge routing · network segmentation & subnetting · Microsoft Sentinel SIEM & KQL · Azure Arc hybrid management · detection engineering

**Linux & Services** — Ubuntu Server administration · LAMP stack (Apache/MySQL/PHP) · Postfix & Dovecot mail delivery · osTicket helpdesk platform · service/log-level debugging

**Diagnostic method** — root-cause isolation over guesswork · reading verbose/debug logs · systematic elimination of causes · documenting scope limits honestly · live failure testing (kill tests)

## Notes on scope

Two originally-planned modules — VLAN segmentation and scripted endpoint hardening — were deliberately scoped out due to time, in favor of finishing the seven services above to a high standard rather than spreading thinner across nine. Everything listed in the table was fully built, tested, and documented.

## About this lab

Built solo, VM-to-VM, using VirtualBox for every server. The goal was to reproduce the kind of infrastructure a small business or MSP client environment would actually run, and to practice diagnosing and fixing real issues along the way — rather than following a scripted tutorial.

---

📧 **[AkramEIAsouly@proton.me](mailto:AkramEIAsouly@proton.me)** · Morocco · Open to IT Support Specialist / Junior Sysadmin / Helpdesk Technician roles
