# Investigation Report Template

```
Case ID:
Date:
Host:
User:
Alert / Trigger:
Time Spent: (my own working minutes, not the log time range)

Verdict: TP / FP / Undetermined
Confidence: Low / Medium / High   (one-sentence reason)
Impact: Confirmed / Likely / Unknown / No Observed Impact
        (split: operational vs security/data if they differ)

Executive Summary (3–5 English sentences)
- What happened?
- What evidence was found?
- What is your conclusion?

Timeline
-

Evidence Ranking (strong / medium / weak ON WHAT?)
- Strong:
- Medium:
- Weak:

Benign Explanation (the strongest normal explanation, then what it does NOT explain):

Missing Evidence:

What Would Change My Mind? (a verifiable piece of evidence to look for, in both directions)

SIEM Notes (for each note: what it explains / what it does not explain)
- Note 1:
- Note 2:

Recommended Action: Close / Monitor / Escalate (one-sentence reason)
```

## Working method
1. Read the brief; write the one question the case must answer.
2. Mark every log line with Who / What / When; build the timeline or process tree.
3. Two columns: Facts (a log line proves it) and Assumptions.
4. Prosecutor (3 strongest malicious points) vs Defender (3 strongest benign points).
5. Verdict sentence: "My verdict is ___ because ___, but I cannot confirm ___."
6. Fill the template, then run the pre-submit checklist.
