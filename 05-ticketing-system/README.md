# Ticketing System (osTicket)

## Problem
A helpdesk needs a way to intake, track, and resolve support requests rather than handling everything ad hoc over chat or email. This project builds and proves out that workflow end-to-end.

## What I Built
- TICKET01 built (Ubuntu Server) with a full LAMP stack (Apache, MySQL, PHP)
- osTicket 1.18.4 installed and configured
- Departments and Help Topics configured and mapped
- A staff/agent account created
- Full ticket lifecycle proven: public submission → staff reply → status update

## How It Works
![osTicket dashboard/login](./screenshots/osTicket%20installed,%20dashboardlogin.png)
![A submitted ticket](./screenshots/A%20submitted%20ticket%20(public-facing).png)
![Staff reply to ticket](./screenshots/Staff%20reply%20to%20that%20ticket.png)

## Challenges & Troubleshooting
- **The osTicket web installer returned HTTP 500 on the database step.** Root cause: an earlier `ALTER USER` attempt had been silently rejected by MySQL's `validate_password` policy, meaning the password never actually changed — so the installer was authenticating with a password that had never actually been set. Fixed by choosing a password that satisfied the complexity policy and confirming it worked with a direct CLI login test before retrying the web form. This was a good reminder not to assume a command succeeded just because it didn't loudly error at the time.

## What I'd Do Differently / Next Steps
- Integrate osTicket against Active Directory for staff login (LDAP auth), rather than a local-only account
- Add SLA rules and auto-escalation for tickets left unanswered past a threshold
