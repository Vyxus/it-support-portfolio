# File & Print Server

## Problem
Centralizing file storage and print services off the domain controllers, with proper access control and storage quotas — the way a real small-business environment would run it, not with shares sitting on a DC.

## What I Built
- FILESRV01 built, domain-joined, with File Server and Print Server roles installed
- Sales and IT shares created and migrated onto FILESRV01 (moved off DC01)
- NTFS permissions re-applied on the new shares, scoped to the correct security groups
- GPO drive-map paths updated to point at the new server, tested working for both department users
- FSRM installed with 5GB soft quotas configured on both shares
- A shared printer deployed via GPO, confirmed appearing automatically on clients after policy refresh

## How It Works
![FILESRV01 File Server role](./screenshots/FILESRV01%20joined%20+%20File%20Server%20role.png)
![Sales and IT shares](./screenshots/Sales%20and%20IT%20shares.png)
![FSRM quota configured](./screenshots/FSRM%20quota%20configured.png)
![Printer deployed via GPO](./screenshots/Printer%20deployed%20via%20GPO.png)

## Challenges & Troubleshooting
- **FILESRV01 couldn't join the domain** ("AD DC could not be contacted") — root cause: DNS was pointing at stale IPv6 site-local placeholder addresses instead of the real domain controllers. Fixed by explicitly setting the DNS client server addresses to the real DC IPs.
- **FSRM's quota creation dialog showed no shares to select** — the shares had never actually been created on FILESRV01 yet, a migration step that had been skipped. Created them fresh with correct NTFS permissions, then retried FSRM successfully.
- **Group Policy Management Console wasn't available from FILESRV01** — GPMC only ships with AD DS tools on domain controllers by default; corrected by running policy changes from DC01 instead.

## What I'd Do Differently / Next Steps
- Add DFS namespaces if this environment grows to multiple file servers
- Set up FSRM email notifications for quota threshold warnings
