# W8-C5 — Linux Persistence Investigation

**Date:** 2026-11-09 · **Host:** monitor-srv-02 · **User:** omar · **Time spent:** not recorded
**Alert:** New systemd service created and enabled via sudo

## Evidence
```
Nov  9 14:02:11 monitor-srv-02 sshd[7010]: Accepted publickey for omar from 10.10.5.52 port 51200 ssh2
Nov  9 14:02:11 monitor-srv-02 sshd[7010]: pam_unix(sshd:session): session opened for user omar by (uid=0)
Nov  9 14:05:30 monitor-srv-02 sudo: omar : TTY=pts/0 ; PWD=/opt/monitoring ; USER=root ; COMMAND=/usr/bin/nano /opt/monitoring/push_metrics.sh
Nov  9 14:09:44 monitor-srv-02 sudo: omar : TTY=pts/0 ; PWD=/opt/monitoring ; USER=root ; COMMAND=/usr/bin/chmod +x /opt/monitoring/push_metrics.sh
Nov  9 14:11:02 monitor-srv-02 sudo: omar : TTY=pts/0 ; PWD=/opt/monitoring ; USER=root ; COMMAND=/usr/bin/tee /etc/systemd/system/metrics-push.service
Nov  9 14:11:20 monitor-srv-02 sudo: omar : TTY=pts/0 ; PWD=/opt/monitoring ; USER=root ; COMMAND=/usr/bin/systemctl daemon-reload
Nov  9 14:11:25 monitor-srv-02 sudo: omar : TTY=pts/0 ; PWD=/opt/monitoring ; USER=root ; COMMAND=/usr/bin/systemctl enable --now metrics-push.service
Nov  9 14:26:50 monitor-srv-02 sshd[7010]: pam_unix(sshd:session): session closed for user omar

syslog:
Nov  9 14:11:25 monitor-srv-02 systemd[1]: Created symlink /etc/systemd/system/multi-user.target.wants/metrics-push.service → /etc/systemd/system/metrics-push.service.
Nov  9 14:11:25 monitor-srv-02 systemd[1]: Started Metrics Push Agent.

/etc/systemd/system/metrics-push.service:
[Unit]
Description=Metrics Push Agent
After=network.target
[Service]
ExecStart=/opt/monitoring/push_metrics.sh
Restart=always
User=root
[Install]
WantedBy=multi-user.target
```
**SIEM Notes:** (1) `omar` is on the infrastructure team and has sudo rights. (2) Approved change CHG-4471 "Deploy metrics exporter on monitoring servers" does not list individual hostnames. (3) The content of `push_metrics.sh` was not collected. (4) Network summary shows the service contacted only `10.10.9.20:9091` (raw logs not provided).

## Final report
**Verdict:** Undetermined · **Confidence:** Medium · **Impact:** Persistence: Confirmed / Security impact: Unknown · **Action:** Monitor (submitted); the reviewer suggested Escalate because the service runs an unreviewed script as root

**Executive Summary.** On 2026-11-09 `omar` logged into monitor-srv-02 by SSH from 10.10.5.52 and, using sudo, opened `/opt/monitoring/push_metrics.sh` in nano, made it executable, created `/etc/systemd/system/metrics-push.service`, reloaded systemd and enabled and started the service. The unit runs the script as root, restarts it automatically and starts it at boot, and a network summary shows it contacted only 10.10.9.20:9091. The activity is consistent with approved change CHG-4471, but the ticket does not name this host and the script content was not collected. Assessment: Undetermined, Medium confidence.

**Evidence ranking**
- Strong: the sudo entries (who, what, when: `tee` into the unit directory, `daemon-reload`, `enable --now`); the unit file (strong on "persistence established with root privileges", not on malice).
- Medium: syslog symlink and "Started" lines (confirm the enablement); Note 4 (internal destination only, and only a summary).
- Weak: Note 1 (role and sudo rights are context); Note 2 (approval exists, but the host is not named).

**Benign explanation:** an infrastructure engineer deploying the approved metrics exporter; running a monitoring agent as a systemd service is normal.
**Missing evidence:** content of `push_metrics.sh`; service logs; raw network logs for 10.10.9.20:9091; full target list of CHG-4471.
**What would change my mind:** the script sends data to an external IP or opens a reverse shell, or the host is not in CHG-4471 → likely unauthorized; the script only pushes metrics to 10.10.9.20:9091 and the host is in the ticket → FP.

**SIEM Notes review**
| Note | Explains | Does not explain |
|---|---|---|
| 1 | Why omar could do this | That this action was authorized |
| 2 | That a metrics deployment was approved | That this host was a target |
| 3 | That the script is unknown | What it does |
| 4 | An internal destination | The nature or content of the traffic |

## Evolution
- **First submission:** SIEM Notes review only (a good format), then the full report.
- **Issues found in review:** the unit file was not analysed line by line; "modified the script" when the log only proves it was opened in nano; Time Spent left blank.
- Self-reported AI assistance on this case: about 30% (analysis and English).

**Lesson:** read the unit file and say what each line implies.
