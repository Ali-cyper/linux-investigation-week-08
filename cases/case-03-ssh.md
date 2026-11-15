# W8-C3 — SSH Investigation

**Date:** 2026-11-05 · **Host:** bastion-01 · **User:** mohammed · **Time spent:** 65 min
**Alert:** SSH authentication failures from an external IP followed by a successful login

## Evidence — `/var/log/auth.log`
```
Nov  4 18:05:12 bastion-01 sshd[4410]: Accepted publickey for mohammed from 10.10.5.61 port 52044 ssh2
Nov  4 18:05:12 bastion-01 sshd[4410]: pam_unix(sshd:session): session opened for user mohammed by (uid=0)
Nov  4 19:22:40 bastion-01 sshd[4410]: pam_unix(sshd:session): session closed for user mohammed
Nov  5 04:31:02 bastion-01 sshd[5201]: Invalid user admin from 192.0.2.88 port 60112
Nov  5 04:31:04 bastion-01 sshd[5201]: Failed password for invalid user admin from 192.0.2.88 port 60112 ssh2
Nov  5 04:31:11 bastion-01 sshd[5203]: Invalid user guest from 192.0.2.88 port 60130
Nov  5 04:31:13 bastion-01 sshd[5203]: Failed password for invalid user guest from 192.0.2.88 port 60130 ssh2
Nov  5 04:31:20 bastion-01 sshd[5205]: Invalid user postgres from 192.0.2.88 port 60148
Nov  5 04:31:22 bastion-01 sshd[5205]: Failed password for invalid user postgres from 192.0.2.88 port 60148 ssh2
Nov  5 04:31:31 bastion-01 sshd[5207]: Failed password for mohammed from 192.0.2.88 port 60166 ssh2
Nov  5 04:31:40 bastion-01 sshd[5209]: Failed password for mohammed from 192.0.2.88 port 60180 ssh2
Nov  5 04:33:48 bastion-01 sshd[5214]: Accepted password for mohammed from 192.0.2.88 port 60214 ssh2
Nov  5 04:33:48 bastion-01 sshd[5214]: pam_unix(sshd:session): session opened for user mohammed by (uid=0)
Nov  5 04:41:19 bastion-01 sshd[5214]: pam_unix(sshd:session): session closed for user mohammed
```
**SIEM Notes:** (1) `192.0.2.88` has no prior activity in this environment. (2) `mohammed` is a DevOps engineer on the on-call rota for the week of 2026-11-02. (3) `bastion-01` accepts SSH from the internet and password authentication is enabled.

## Baseline comparison
| | Previous session | Suspect session |
|---|---|---|
| Method | publickey | password |
| Source | 10.10.5.61 (internal) | 192.0.2.88 (external, no history) |
| Time | 18:05 | 04:33 |
| Duration | ~1 h 17 min | ~7 min 31 s |

## Final report
**Verdict:** Undetermined (Suspicious Successful Access) · **Confidence:** Medium · **Impact:** Unknown · **Action:** Escalate

**Executive Summary.** On 2026-11-05 at 04:31, source IP 192.0.2.88 made failed SSH attempts against the invalid users admin, guest and postgres, then two failed password attempts for the valid user `mohammed`; at 04:33:48 the same source logged in as `mohammed` with a password and held a session of about 7.5 minutes. This differs from his previous session (public key, internal address, about 77 minutes). The logs do not show what happened inside the session or whether `mohammed`, who was on call that week, was the person authenticating. Assessment: Suspicious Successful Access (Undetermined), Medium confidence, impact unknown, escalate for user verification.

**Questions from the exercise:** several accounts were tried; the log cannot show whether one password was reused, so spraying is not claimed; a successful login did follow failures; unauthorized use cannot be proven from this log alone.

**Evidence ranking**
- Strong: `Accepted password for mohammed from 192.0.2.88` (strong on successful authentication, not on compromise).
- Medium: guessing attempts on admin/guest/postgres from the same IP in the two minutes before (unusual for the real user, though a shared NAT or a test could explain it); deviation from the baseline; session open/close (gives duration).
- Weak: Note 3 (explains how, not whether); Notes 1 and 2 (context; the on-call rota makes an early-morning login plausible).

**Benign explanation:** `mohammed`, on call, logged in from home or another network with a password after two typos. This does not explain the attempts on admin, guest and postgres unless the IP is shared or someone was testing.
**Missing evidence:** VPN/MFA/IdP records; whether 192.0.2.88 is a known address of his; session activity (wtmp, auditd, shell history); outbound connections from the bastion after 04:33; raw firewall logs.
**What would change my mind:** mohammed confirms through a separate channel that he logged in from that IP → leans benign; he denies it → likely compromise; commands such as `useradd`, `crontab` or `curl … | bash` after 04:33:48; accepted logins from 192.0.2.88 on other hosts.

## Evolution
- **First submission:** "suspicious" with no official verdict, Low, Impact Unknown, Monitor.
- **Second:** Suspicious Successful Access, Confidence written as "mid-high", Escalate, with a correct baseline comparison.
- **Third:** verdict written as TP with no new evidence. TP needs evidence of unauthorized use, which the logs do not contain, so this report keeps **Undetermined (Suspicious Successful Access)**.

**Lesson:** change a verdict only when new evidence appears.
