# W8-C2 — Linux System Logs (syslog)

**Date:** 2026-11-04 (Wednesday) · **Host:** app-srv-01 · **User:** devops · **Time spent:** not recorded
**Alert:** Critical service stopped outside approved maintenance window

## Evidence — `/var/log/syslog`
```
Nov  4 02:58:14 app-srv-01 sshd[3302]: Accepted publickey for devops from 10.10.5.40 port 40118 ssh2
Nov  4 03:12:01 app-srv-01 CRON[3410]: (root) CMD (/usr/local/bin/backup.sh)
Nov  4 03:12:05 app-srv-01 systemd[1]: Stopping PostgreSQL RDBMS...
Nov  4 03:12:06 app-srv-01 systemd[1]: postgresql.service: Deactivated successfully.
Nov  4 03:12:06 app-srv-01 systemd[1]: Stopped PostgreSQL RDBMS.
Nov  4 03:13:02 app-srv-01 myapp[1893]: ERROR: database connection refused, retrying in 30s
Nov  4 03:13:32 app-srv-01 myapp[1893]: ERROR: database connection refused, retrying in 30s
Nov  4 03:14:02 app-srv-01 systemd[1]: myapp.service: Main process exited, code=exited, status=1/FAILURE
Nov  4 03:14:02 app-srv-01 systemd[1]: myapp.service: Failed with result 'exit-code'.
Nov  4 03:14:07 app-srv-01 systemd[1]: myapp.service: Scheduled restart job, restart counter is at 1.
Nov  4 03:25:40 app-srv-01 systemd[1]: Starting PostgreSQL RDBMS...
Nov  4 03:25:42 app-srv-01 systemd[1]: Started PostgreSQL RDBMS.
Nov  4 03:26:10 app-srv-01 systemd[1]: Started MyApp Service.
```
**SIEM Notes:** (1) Approved maintenance window is Sundays 01:00–02:00. (2) `backup.sh` was last modified 2026-10-28 by `devops`. (3) No EDR or IDS alerts for this host in the period.

## Final report
**Verdict:** Undetermined · **Confidence:** Medium · **Impact:** Operational: Confirmed / Security: Unknown · **Action:** Escalate

**Executive Summary.** On 2026-11-04 the user `devops` logged in by public key at 02:58 from 10.10.5.40. At 03:12:01 cron ran `/usr/local/bin/backup.sh` as root and PostgreSQL was stopped four seconds later; the database stayed down for about 13.5 minutes and `myapp` exited with status 1 at 03:14:02. The log does not show who or what issued the stop, and the timing match with the cron job is correlation, not proof of causation. The activity is outside the approved Sunday window and `backup.sh` was edited by `devops` a week earlier. Assessment: Undetermined, Medium confidence; operational impact is confirmed, security impact is unknown.

**Evidence ranking**
- Strong: syslog lines showing PostgreSQL stopping at 03:12:05–06 (strong on *that and when it stopped*, not on who caused it); `myapp` failure at 03:14:02 (strong on operational impact).
- Medium: CRON entry four seconds earlier (temporal link only); timing outside the approved window (policy deviation, no real baseline).
- Weak: Note 3, no EDR/IDS alerts (absence of evidence is not evidence of absence); Note 2 (who maintains the script, not what it does).

**Benign explanation:** a root cron backup that stops the database for a cold backup, then restarts it. This is common, but the Wednesday 03:12 timing and the ~13-minute gap before restart are not explained by it.
**Missing evidence:** contents of `backup.sh`; root crontab; whether the same entry runs on other days; who issued the stop (journal/auditd, sudo entries); why restart took ~13 minutes.
**What would change my mind:** `backup.sh` contains `systemctl stop postgresql` and the same cron entry runs on other days → benign scheduled behaviour; no such command and no cron entry for it → unauthorized stop becomes likely; sudo/systemctl entries by `devops` near 03:12 → attribute the action to devops.

## Evolution
- **First submission:** TP, Medium, "No Observed Impact".
- **Issues found in review:** claimed devops had root and ran the script (the log shows `CRON (root)` and a publickey login only); "restarted after 30 seconds" (30 s is the app retry interval; the database was down ~13.5 minutes); Impact ignored the failed application; Time Spent was the DB downtime.
- **Final:** Undetermined, with the verdict sentence "insufficient evidence to confirm malicious activity or unauthorized execution".

**Lesson:** temporal correlation is not causation; write only what the log proves.
