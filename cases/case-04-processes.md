# W8-C4 — Linux Process Investigation

**Date:** 2026-11-06 · **Host:** build-srv-03 · **User:** nadia · **Time spent:** 45 min
**Alert:** Encoded command piped to a shell interpreter (`base64 -d | bash`)

## Evidence
```
Nov  6 09:12:03 build-srv-03 sshd[1544]: Accepted publickey for nadia from 10.10.5.77 port 50312 ssh2
Nov  6 09:12:03 build-srv-03 sshd[1544]: pam_unix(sshd:session): session opened for user nadia by (uid=0)

TIME      USER   PID   PPID  COMMAND
09:12:03  nadia  1562  1544  sshd: nadia@pts/1
09:12:04  nadia  1563  1562  -bash
09:20:15  nadia  2210  1563  /usr/bin/python3 -m http.server 8000
09:31:02  nadia  2305  1563  /bin/bash ./deploy.sh
09:31:03  nadia  2311  2305  /usr/bin/curl -s http://10.10.8.15/artifacts/build-211.tar.gz -o /tmp/build-211.tar.gz
09:31:05  nadia  2318  2305  /bin/tar -xzf /tmp/build-211.tar.gz -C /opt/app
09:31:09  nadia  2322  2305  /bin/bash -c echo ZWNobyAiZGVwbG95ZWQi | base64 -d | bash
09:31:09  nadia  2323  2322  /usr/bin/base64 -d
09:31:09  nadia  2324  2322  /bin/bash
09:31:09  nadia  2325  2324  /bin/echo deployed
```
**SIEM Notes:** (1) `nadia` is a release engineer and this server is part of her responsibilities. (2) `10.10.8.15` is the internal artifact server. (3) `deploy.sh` was last modified 2026-11-05 08:50 by `nadia`. (4) Firewall summary shows no outbound connections to external IPs 09:00–10:00 (raw logs not provided).

## Process tree
```
sshd (1544)
└── sshd: nadia@pts/1 (1562)
    └── -bash (1563)
        ├── python3 -m http.server 8000 (2210)
        └── deploy.sh (2305)
            ├── curl (2311)
            ├── tar (2318)
            └── bash -c (2322)
                ├── base64 -d (2323)
                └── bash (2324)
                    └── echo deployed (2325)
```
Decoded payload: `ZWNobyAiZGVwbG95ZWQi` → `echo "deployed"`.

## Final report
**Verdict:** FP · **Confidence:** Medium · **Impact:** No Observed Impact · **Action:** Monitor (until `http.server` and `deploy.sh` are verified)

**Executive Summary.** On 2026-11-06 at 09:12, `nadia`, a release engineer, logged in by public key from 10.10.5.77. At 09:31 `deploy.sh`, which she had edited the day before, downloaded `build-211.tar.gz` from the internal artifact server, extracted it to `/opt/app` and ran an encoded command that decodes to `echo "deployed"`. The decoded payload is harmless and every process sits under her own SSH session. One item stays unexplained: a `python3 http.server` on port 8000 (PID 2210), started at 09:20 as a separate branch from `deploy.sh`. Assessment: False Positive, Medium confidence, no observed impact; monitor pending verification of the HTTP server and the script.

**Evidence ranking**
- Strong: decoded payload is harmless; unbroken PID/PPID chain under nadia's session.
- Medium: Notes 1 and 3 (role and the day-before script edit are context; they do not prove this run was intended).
- Weak: Note 4 (summary only, external traffic only, raw logs missing).
- Unexplained: the HTTP server on port 8000.

**Benign explanation:** a routine deployment run by the responsible engineer; the base64-to-bash step is an odd way to print a status message but harmless once decoded.
**Missing evidence:** her prior deployment baseline; whether `deploy.sh` contained the base64 line before the 11-05 edit; contents of `/opt/app` after extraction; any connection to port 8000; raw firewall logs.
**What would change my mind:** script history shows the base64 line was added by someone other than nadia; a connection to port 8000 from an unknown host or data served from it; no earlier use of `http.server` in her sessions.

## Evolution
- **First submission:** FP, High (the wording contradicted itself), parent process written as `cmd.exe`, base64 called encryption, `http.server` not mentioned.
- **After review:** correct Linux tree, Medium confidence with a reason, the HTTP server flagged as unexplained, Action changed from a conditional Close to Monitor.

**Lesson:** trace PID/PPID first; Linux hosts have no `cmd.exe`.
