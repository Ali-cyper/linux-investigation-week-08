# Quick Reference — Linux Investigation (SOC view)

## Where to look (Debian/Ubuntu; RHEL uses `secure` and `messages`)
| Source | Typical content |
|---|---|
| `/var/log/auth.log` (`/var/log/secure`) | sshd logins, sudo, PAM sessions |
| `/var/log/syslog` (`/var/log/messages`) | systemd service events, CRON runs, application errors |
| `journalctl -u <unit>` | per-service logs |
| `last`, `lastb` | successful / failed login history |

## Log patterns worth knowing
| Pattern | Meaning |
|---|---|
| `Failed password for <user> from <ip>` | valid user, wrong password |
| `Invalid user <user>` / `Failed password for invalid user` | username does not exist on the host |
| `Accepted password|publickey for <user> from <ip>` | successful authentication (note the method) |
| `Connection closed by <ip> [preauth]` | client left before authenticating |
| `pam_unix(sshd:session): session opened/closed` | session start/end (gives duration) |
| `sudo: <user> : TTY=… ; PWD=… ; USER=root ; COMMAND=…` | privileged command by a user |
| `CRON[pid]: (<user>) CMD (<command>)` | a cron job ran |
| `systemd[1]: Stopping/Stopped/Started/Failed with result` | service lifecycle |

## SSH patterns (what the log can and cannot prove)
- **Brute force:** many attempts against one or few accounts.
- **Password spraying:** one or few passwords across many accounts. `auth.log` does not show the passwords, so spraying cannot be proven from it alone. Safer wording: *automated credential guessing across multiple usernames*.
- Compare to a baseline: source IP, auth method (key vs password), time of day, session length.
- A failed-then-accepted sequence is not proof of compromise; look at who, from where, and what happened inside the session.

## Process investigation
- Fields: process name, PID, PPID, user, command line, start time, parent.
- Build the tree from PID/PPID before judging. Live host: `ps -ef --forest`, `pstree -p`.
- Patterns that need context (never a verdict by themselves): `curl|wget … | bash`, `base64 -d | bash`, execution from `/tmp` or `/dev/shm`, hidden (dot-prefixed) files, `chmod +x` followed by execution, interpreters with `-c`.
- Decode before judging. Base64 is **encoding**, not encryption.

## Persistence locations
- systemd: `/etc/systemd/system/*.service`, `systemctl list-unit-files --state=enabled`, `systemctl cat <unit>`
- cron: `crontab -l`, `/var/spool/cron/crontabs/`, `/etc/crontab`, `/etc/cron.d/`, `cron.hourly|daily`
- Others: shell rc files, `~/.ssh/authorized_keys`, `/etc/rc.local`
- Unit file lines to read: `ExecStart` (what runs), `User` (privilege), `Restart` (resilience), `WantedBy` (starts at boot), `After`.
- Persistence alone does not prove compromise; check who created it, when, whether a change ticket covers this host, and what the script does.

## PCAP checklist
Source/destination IP · protocol · ports · DNS queries and answers (TTL explains missing repeat queries) · SNI · timing regularity · bytes in vs out.
- **Beaconing hint:** equal intervals + near-identical sizes to the same destination.
- A download (large inbound) is not exfiltration; a single connection does not prove C2.
- Classify as: Suspicious Network Activity / Confirmed Malicious Activity / Undetermined. Always link back to host evidence (which process, which cron job).

## Definitions
- **Verdict:** TP (real malicious or unauthorized activity), FP (alert fired but activity is benign), Undetermined (evidence cannot decide).
- **Confidence:** how well the evidence supports the verdict. **Impact:** what is known to have happened. Keep them separate. Split Impact into *operational* and *security/data* when they differ.
- **Evidence strength:** always "strong/medium/weak **on what**" (attempt, success, compromise, impact).
- **Fact vs assumption:** if no log line proves it, it is an assumption.
- **Temporal ≠ causal.** **Absence of evidence ≠ evidence of absence.** **Unknown reputation ≠ malicious.**

## Glossary
PID/PPID (process / parent process ID) · baseline (normal behaviour for this user/host) · persistence (survives logout or reboot) · beaconing (periodic call-outs) · preauth (before authentication) · TTL (DNS cache lifetime) · SNI (hostname in the TLS handshake) · OCSP (certificate status check).
