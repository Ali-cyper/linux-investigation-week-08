# Mistake Log — Week 7 carry-over + Week 8

Status key: **Fixed** = not seen again by the end of the week · **Improving** = fewer occurrences · **Recurring** = still appearing.
Typographic slips (for example typing FT instead of FP) are not counted.

## Carried over from Week 7

| ID | Mistake | Week 8 occurrences | Status |
|---|---|---|---|
| W7-1 | Impact marked Confirmed without direct evidence | C2 (first draft), C6 (first draft) | Improving |
| W7-2 | Evidence marked Strong because it looks strange | C2, C4, C6 (time-of-day ranked Strong) | Recurring |
| W7-3 | SIEM Notes ignored or not reviewed | C3 | Fixed from C5 (explains / does not explain format) |
| W7-4 | "What Would Change My Mind" repeated the hypothesis instead of naming evidence | C1, C2, C4 | Improving (specific evidence from C3 on) |
| W7-5 | Copying information from one case to another | C2 (old summary reused), C6 (IP from C5 in the summary) | Recurring |
| W7-6 | Confidence always Medium / badly calibrated | C1 (Low with FP verdict), C4 (High with contradictory wording), C6 and PCAP (High without a stated reason) | Improving |
| W7-7 | Unsupported claims | C1, C2, C3, C4, C5, C6 | Recurring |

## New in Week 8

| ID | Mistake | Where | Correct approach | Status |
|---|---|---|---|---|
| W8-1 | Incomplete submissions: blank Medium/Weak, missing Impact, Benign Explanation or Summary | C2–C6, PCAP | Fill every field of the template; run the pre-submit checklist | Recurring |
| W8-2 | Time Spent written as the log timestamp range instead of my own working time | C1, C2 | Time Spent = minutes I spent investigating | Fixed |
| W8-3 | Non-official terms (`mid-high`, `Suspicious` or `TP` as a verdict where only the official set applies) | C1, C3, PCAP | Verdict: TP / FP / Undetermined. Confidence: Low / Medium / High | Mostly fixed |
| W8-4 | Windows concept on a Linux host (parent process written as `cmd.exe`) | C4 | Read the chain from PID/PPID: sshd → sshd session → bash → script → children | Fixed |
| W8-5 | Verdict changed with no new evidence (TP written for C3 after Undetermined) | C3 | Change a verdict only for new evidence; log Initial → Final and why | Watch |
| W8-6 | Executive Summary made of pasted logs, or unfinished sentences, or mixed language | C3, C4, C6 | 3–5 English sentences: what happened → evidence → conclusion | Recurring |
| W8-7 | Benign Explanation missing or written backwards (why it is NOT normal) | C1, C3 | First argue the strongest normal explanation, then say what it does not explain | Improving |
| W8-8 | Persistence unit file not analysed line by line | C5 | Read ExecStart, User, Restart, WantedBy and say what each implies | Open |
| W8-9 | Temporal correlation or "opened" treated as causation or "modified" | C2, C5 | Write only what the log proves ("opened in nano", "followed within seconds by") | Recurring |
| W8-10 | Base64 called encryption; cron job called a service; "several DNS queries" when there was one | C4, C6, PCAP | Use precise terms; recount from the data | Improving |
| W8-11 | Evidence from outside the case used in the ranking (VPN logs) | C1, C3 | Anything not in the case goes under Missing Evidence | Improving |
| W8-12 | Evidence Ranking without saying "strong on what?" | All | After each ranking write: strong/medium/weak on attempt, success, compromise, or impact | Recurring |

## Habits that worked this week

- Facts / Assumptions two-column split before writing the verdict (C2 onward).
- Prosecutor vs Defender pass before the verdict.
- Verdict sentence: "My verdict is ___ because ___, but I cannot confirm ___."
- Tracing PID/PPID before judging a process (C4 onward).
