# W8-PCAP — Light PCAP Exercise (linked to W8-C6)

**Host:** dev-srv-04 (10.10.6.44) · **Window:** 22:58–23:12 on 2026-11-10 · **Time spent:** 40 min
Internal DNS is 10.10.1.10; 10.10.8.20 is the internal package mirror. Payloads are encrypted; only conversation summaries are available.

## Evidence
```
#  TIME      SRC               DST               PROTO/PORT   INFO
1  22:58:03  10.10.6.44        10.10.1.10        UDP/53       DNS A ntp.corp.local -> 10.10.1.5
2  22:58:04  10.10.6.44        10.10.1.5         UDP/123      NTP client sync, 2 pkts
3  22:59:40  10.10.5.88        10.10.6.44        TCP/22       SSH session, ~4 KB both ways
4  23:00:02  10.10.6.44        10.10.1.10        UDP/53       DNS A cdn-sync.example.net -> 198.51.100.14 (TTL 3600)
5  23:00:02  10.10.6.44:51544  198.51.100.14     TCP/443      TLS 1.2, SNI cdn-sync.example.net, out 612 B / in 1,340 B, 0.8 s
6  23:03:15  10.10.6.44        10.10.8.20        TCP/443      TLS, in 2.1 MB / out 38 KB, 41 s
7  23:05:02  10.10.6.44:51702  198.51.100.14     TCP/443      TLS 1.2, SNI cdn-sync.example.net, out 612 B / in 1,338 B, 0.8 s
8  23:07:40  10.10.6.44        10.10.1.10        UDP/53       DNS A ocsp.example-ca.test -> 203.0.113.9
9  23:07:41  10.10.6.44        203.0.113.9       TCP/443      TLS, out 300 B / in 900 B, 0.4 s
10 23:10:02  10.10.6.44:51866  198.51.100.14     TCP/443      TLS 1.2, SNI cdn-sync.example.net, out 612 B / in 1,341 B, 0.8 s
11 23:10:30  10.10.6.44        10.10.1.10        UDP/53       DNS A updates.corp.local -> 10.10.8.20
```

## Classification by conversation
| # | Traffic | Class | Why |
|---|---|---|---|
| 1–2 | DNS + NTP | Normal | Internal name resolution followed by a normal NTP sync |
| 3 | SSH from 10.10.5.88 | Undetermined | The inbound SSH session is confirmed, but it cannot be tied to the earlier tariq session from this capture alone |
| 4, 5, 7, 10 | TLS to 198.51.100.14 | Suspicious | Same SNI, near-identical sizes, exactly five minutes apart (23:00:02, 23:05:02, 23:10:02) |
| 6 | Package mirror | Undetermined | 2.1 MB inbound from a known internal mirror (a download, not an upload), but the responsible process and content are unknown |
| 8–9 | OCSP | Normal | DNS query followed by a short HTTPS exchange, consistent with a certificate status check; the application is unknown |
| 11 | updates.corp.local | Normal | Resolves to the known internal mirror, consistent with updates |

## Final report
**Classification:** Suspicious Network Activity · **Confidence:** Medium (submitted as High; adjusted because the payload is encrypted and the originating process is unproven) · **Impact:** Unknown · **Action:** Escalate

**Link to Linux events:** in W8-C6, tariq downloaded and ran `setup.sh`, which created a cron job that ran `/tmp/.upd.py` every five minutes. The PCAP shows connections to the same external IP every five minutes. The DNS name `cdn-sync.example.net` resolves to 198.51.100.14, the address `setup.sh` was downloaded from. The timing match supports a link but does not prove that `.upd.py` opened these connections.

**Missing evidence:** which process opened the connections (`ss -tp`, auditd, EDR); contents of `setup.sh`, `.cronjob` and `.upd.py`; reputation and ownership of the destination and SNI; whether other hosts contact the same IP; what the encrypted payload carried.
**What would change my mind:** the destination is an approved vendor service and other hosts show the same pattern → toward benign; `.upd.py` is the process behind the connections and its code sends data out → Confirmed Malicious.

## Evolution
- **First submission:** classified "TP" (not a valid PCAP class), said DNS was queried several times for the external IP (it was queried once; the later connections used the cached answer, TTL 3600), counted "packets" instead of connections, and mixed normal traffic into the suspicious evidence.
- **Final:** per-conversation classification table, official class, link to the W8-C6 events.
- Self-reported AI assistance: about 10% (a hint) plus writing help on the classification table.

**Lesson:** classify each conversation separately; look for patterns, not single connections.
