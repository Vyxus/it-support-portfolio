# Active Directory Domain Lab

## Problem
Simulates a small business standing up its first domain: centralized identity, department-based access control, and policy enforcement across workstations.

## What I Built
- DC01 built and promoted to the first domain controller for `helpdesklab.local`
- DNS and DHCP roles configured on DC01 (scope `192.168.50.100–200`)
- CLIENT01 built and domain-joined
- Organizational Units created: Sales, IT, Management
- Users created (`jsmith` in Sales, `kalaoui` in IT) with matching security groups
- Three Group Policy Objects built and linked:
  - **Security-Baseline** — password policy (12-char minimum, complexity, 90-day max age), 10-minute inactivity lock
  - **Sales-Drive-Map** — maps `S:` to the Sales share
  - **Block-USB-Storage** — denies removable storage access
- NTFS permissions locked down on department shares, tested and confirmed

## How It Works
![Domain in ADUC](./screenshots/Active%20Directory%20Users%20and%20Computers%20showing%20the%20domain.png)
![CLIENT01 domain-joined](./screenshots/CLIENT01%20successfully%20domain-joined.png)
![OUs created](./screenshots/OUs%20created%20(Sales,%20IT,%20Management)%20in%20ADUC.png)
![The 3 GPOs](./screenshots/The%203%20GPOs%20(Security-Baseline,%20Sales-Drive-Map,%20Block-USB-Storage)%20shown%20in%20GPMC.png)

Permission testing: `jsmith` (Sales) is correctly denied access to the IT share, while retaining access to the Sales share — confirming NTFS permissions and group membership are enforced correctly per department.

![jsmith denied IT share](./screenshots/jsmith%20IT%20drive%20access%20denied.png)
![jsmith accessing Sales share](./screenshots/jsmith%20sales%20drive%20accessed.png)

## Challenges & Troubleshooting
- **CLIENT01's first local account came out as a standard user, not an admin.** Root cause: VirtualBox's "Proceed with Unattended Installation" option auto-provisions a local account that isn't reliably added to `BUILTIN\Administrators`. Fixed by disabling unattended installation and going through interactive OOBE manually, then verifying group membership with `whoami /groups | findstr "S-1-5-32-544"` immediately after first login.
- Domain join failed repeatedly until the underlying local-admin issue above was identified and fixed — the join dialog's generic error masked what was actually a local privilege problem, not a networking or domain-trust issue.

## What I'd Do Differently / Next Steps
- Script user/group/OU creation with a PowerShell provisioning tool rather than doing it by hand (see the mail-server and file-server projects for a similar automation mindset applied elsewhere)
- Add fine-grained password policies per OU rather than one domain-wide baseline
