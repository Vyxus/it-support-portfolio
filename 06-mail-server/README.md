# Mail Server (Postfix / Dovecot / Roundcube)

## Problem
Stand up internal email for the organization with real SMTP delivery, IMAP mailbox access, and a working webmail client — and prove mail actually flows end to end, not just that the services start.

## What I Built
- MAIL01 built (Ubuntu Server)
- Postfix installed and configured (Internet Site, Maildir-based delivery)
- Dovecot configured for Maildir access
- Local mailboxes created for domain users
- Roundcube webmail installed and connected
- Internal mail delivery confirmed working via command line and verified landing in the actual mailbox

## Lab Context
- **Server role/name:** MAIL01
- **Mail flow path:** Postfix (SMTP receive/delivery) → Maildir storage → Dovecot (IMAP) → Roundcube (webmail UI)
- **Mailbox model:** Local user mailboxes for internal organization communication tests
- **Primary dependency:** Consistent Maildir path alignment between MTA and IMAP layers

## How It Works
![Postfix/Dovecot config](./screenshots/PostfixDovecot%20config%20confirmation.png)
![Mail delivery confirmed via command line](./screenshots/Mail%20delivery%20confirmed%20via%20command%20line.png)
![Working Roundcube inbox](./screenshots/jsmith%20inbox%20screenshot.png)

## Challenges & Troubleshooting
This project had the deepest single root-cause chase in the whole lab — a genuinely subtle bug that took several rounds of elimination to fully resolve.

**Symptom:** Roundcube repeatedly threw `Server Error: STATUS/LIST: Internal error occurred` when loading the inbox, despite Postfix logs clearly showing `status=sent (delivered to maildir)` for every test message.

**What it wasn't:**
- Not a missing package (`php-zip` was flagged and installed early, but didn't fully resolve it)
- Not a wrong install path (Roundcube's `public_html` subfolder quirk was identified and worked around)
- Not simple file permissions — `chown`/`chmod` fixes on the home directory and Maildir folder got Roundcube past the login error, but the inbox still showed empty

**Actual root cause, found by reading Dovecot's own debug output (`doveadm -Dv force-resync`):** Dovecot's INBOX namespace was resolving to `/var/mail/jsmith` — a completely separate, empty Maildir-format directory — while Postfix had been correctly delivering every message to `/home/jsmith/Maildir` the entire time. The two never overlapped. This was caused by a `mail_inbox_path` override in `10-mail.conf`, a newer Dovecot config setting that takes precedence over the general `mail_location`/`mail_driver`/`mail_path` settings for INBOX specifically. Commenting out the override and letting the general Maildir settings apply resolved it immediately — confirmed with a fresh test message appearing correctly in Roundcube's inbox.

This is a good example of a bug where every individual symptom (login error, empty inbox, permission denials in the logs) pointed toward related-but-wrong causes, and the real fix only came from reading the mail server's own verbose debug trace rather than guessing at plausible-sounding fixes.

## Validation Performed
- Verified successful SMTP delivery events in Postfix logs
- Verified messages landed in the expected Maildir on disk
- Verified Roundcube could authenticate and display newly delivered inbox messages
- Verified Dovecot namespace/path behavior after removing conflicting override

## What I'd Do Differently / Next Steps
- Add SPF/DKIM records if this were ever exposed to real external mail exchange
- Script new-mailbox provisioning as part of the `New-HelpdeskUser` workflow used elsewhere in this portfolio
