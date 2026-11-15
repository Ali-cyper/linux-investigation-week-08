# W8-C1 — Linux Authentication Logs

**Date:** 2026-11-03 · **Host:** web-srv-02 · **User:** sara (accepted login) · **Time spent:** not recorded
**Alert:** Multiple failed authentication attempts followed by a successful login (SIEM rule)

## Evidence — `/var/log/auth.log`
```
Nov  3 02:14:07 web-srv-02 sshd[4121]: Failed password for invalid user admin from 203.0.113.45 port 51822 ssh2
Nov  3 02:14:11 web-srv-02 sshd[4123]: Failed password for invalid user test from 203.0.113.45 port 51830 ssh2
Nov  3 02:14:15 web-srv-02 sshd[4125]: Failed password for invalid user oracle from 203.0.113.45 port 51838 ssh2
Nov  3 02:14:20 web-srv-02 sshd[4127]: Failed password for root from 203.0.113.45 port 51846 ssh2
Nov  3 02:14:24 web-srv-02 sshd[4129]: Failed password for root from 203.0.113.45 port 51852 ssh2
Nov  3 02:14:29 web-srv-02 sshd[4131]: Failed password for invalid user ubuntu from 203.0.113.45 port 51860 ssh2
Nov  3 02:14:33 web-srv-02 sshd[4133]: Connection closed by 203.0.113.45 port 51860 [preauth]
Nov  3 08:52:41 web-srv-02 sshd[4388]: Failed password for sara from 10.10.5.23 port 49211 ssh2
Nov  3 08:52:58 web-srv-02 sshd[4388]: Accepted password for sara from 10.10.5.23 port 49211 ssh2
Nov  3 08:52:58 web-srv-02 sshd[4388]: pam_unix(sshd:session): session opened for user sara by (uid=0)
Nov  3 08:53:20 web-srv-02 sudo:     sara : TTY=pts/0 ; PWD=/home/sara ; USER=root ; COMMAND=/usr/bin/systemctl status nginx
Nov  3 09:41:05 web-srv-02 sshd[4388]: pam_unix(sshd:session): session closed for user sara
```
**SIEM Notes:** (1) `203.0.113.45` is external and unseen in this environment. (2) `10.10.5.23` is in the internal VPN pool. (3) `sara` is a sysadmin on the web team per the HR directory.

## Final report
**Verdict:** FP · **Confidence:** Medium · **Impact:** No Observed Impact · **Action:** Monitor

**Executive Summary.** On 2026-11-03 at 02:14, an external IP (203.0.113.45) made six failed SSH password attempts against five usernames (admin, test, oracle, root, ubuntu) at roughly four-second intervals, then disconnected; there is no Accepted entry from that IP in this log. Hours later, `sara`, a sysadmin, logged in from the internal VPN address 10.10.5.23 after one failed attempt, ran one read-only sudo command (`systemctl status nginx`), and closed the session after about 48 minutes. The two activities share no source, account or technique, so the SIEM rule appears to have correlated unrelated events. Assessment: False Positive for this alert, Medium confidence, no observed impact.

**Pattern:** automated credential guessing across multiple usernames. Password reuse cannot be shown from `auth.log`, so "password spraying" is not claimed.

**Timeline:** 02:14:07–02:14:29 six failures → 02:14:33 disconnect → 08:52:41 sara fails once → 08:52:58 accepted + session opened → 08:53:20 sudo `systemctl status nginx` → 09:41:05 session closed.

**Evidence ranking**
- Strong: no `Accepted` from 203.0.113.45 in this log (strong on "no success on this host", not beyond it); sara's session from an internal VPN address with a single read-only sudo command.
- Medium: six failures in about 22 seconds across five usernames (proves an attempt, not a compromise).
- Weak: SIEM Notes 1 and 3 (context only).

**Benign explanation:** sara mistyped her password once, then succeeded, and checked nginx as part of her job. The external IP is background internet noise that failed.
**Missing evidence:** VPN logs for 10.10.5.23 at 08:52; whether SSH is exposed to the internet; any Accepted from 203.0.113.45 on other hosts; raw firewall logs.
**What would change my mind:** an Accepted login from 203.0.113.45 on any host; sara's session with no matching VPN record; commands such as `useradd`, `crontab` or `curl … | bash` after 08:53.

**SIEM Notes review**
| Note | Explains | Does not explain |
|---|---|---|
| 1 | Why the IP is unfamiliar | That it is malicious |
| 2 | Why the source address is internal | That the person logging in was really sara (needs VPN/MFA logs) |
| 3 | Why a sudo `systemctl status` is plausible for her | Identity or authorization of this particular session |

## Evolution
- **First submission:** FP, Low, No Observed Impact, Monitor.
- **Issues found in review:** "password spraying" claimed without evidence; VPN logs ranked as evidence although absent from the case; Time Spent was the gap between two log lines; sudo command not mentioned; Confidence Low contradicted an FP verdict reached with no contrary evidence.
- **Final:** same verdict; Confidence raised to Medium with a stated reason.

**Lesson:** name a pattern only when a log line proves it.
