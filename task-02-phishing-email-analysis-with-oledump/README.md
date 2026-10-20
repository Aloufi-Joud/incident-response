# Task 02 · Phishing email analysis with oledump

**Program:** stc × Misk Virtual Work Experience: Cybersecurity Incident Response &nbsp;·&nbsp; **Tools:** oledump.py, olefile, Python

## Task

A suspicious email was reported. Analyse it and its attachment to decide whether it is malicious, then document the incident:

1. Install the tooling: `oledump.py` (Python-based) needs Python plus the `olefile` module.
2. Analyse the attached document with oledump.
3. Determine whether the document contains macros, and if so, read them.
4. Write a cyber-incident report on the finding.

## Overview

Microsoft Office documents are OLE compound files: they are made of **streams**, and a macro lives in its own stream inside the file. `oledump.py` lists those streams and marks the ones that contain a macro with an `M` (or `m`). Two switches do the real work: `-s` selects a stream, and `-v` decompresses it so the macro source becomes readable. That makes oledump the standard first look at a document that might be weaponised, you see whether it carries code before you ever open it in Office.

Typical workflow:

```
oledump.py -h                  # usage
oledump.py PaySlip.doc         # list streams; look for an M beside one
oledump.py -s 8 -v PaySlip.doc # decompress stream 8 and read the macro
```

## The email

| Field | Value |
| --- | --- |
| **Subject** | Pay slip review |
| **From** | Fahad A. Almobarak (posing as a senior cybersecurity incident-response analyst) |
| **To** | Ibrahim A. Alzaid |
| **Attachment** | `PaySlip.zip` |

**Body:**

> Dear Ibrahim,
> Looking forward for your support to verify and validate the information in the attached Pay slip before we submit them to HR and Finance.
> Best regards, Fahad.

This is a textbook lure. It combines **authority** (the sender presents himself as a senior security analyst), **a finance pretext** (a pay slip bound for HR and Finance), and **a call to open an attachment**. All three push the recipient to act quickly and open `PaySlip.zip` without questioning it.

## Analysis

The attachment `PaySlip.zip` contained the document to examine. Following the oledump workflow, I listed the document's streams to check for macros, then decompressed the relevant stream with `-s <n> -v` to read what the embedded code actually did.

The document was not a real pay slip. It was a **phishing simulation**: its content steers the user into opening a PDF and then displays a message telling them they have fallen for a trap. In other words, the "attack" is a safe awareness test, but the delivery, a zipped attachment behind an authority-and-finance lure, is exactly how a real malicious document would arrive.

> The malicious sample itself is not included in this repo. The analysis here reflects the document's behaviour as found during the exercise, and the oledump commands above are the method used to reach it.

## Incident report

- **Title:** Suspicious email analysis
- **Category:** Phishing · **Type:** Incident
- **Incident date:** 16 Nov 2021
- **Summary:** An email arriving from a supposed member of the cybersecurity protection centre asked the recipient to review and approve salary information, with an attachment named `PaySlip.zip`. On examination the file proved to be a **phishing simulation**.
- **Findings:**
  - The file is a phishing simulation built to measure employees' security awareness.
  - Its contents direct the user to open a PDF and then raise an alert that they have fallen for a trap.
- **Risk:** Staff who act on a message like this could, with a real attacker, open an attachment that runs hidden commands and harms the company, all because the sender's authority went unverified.
- **Recommendation:** Intensify awareness training, and stress that attachments from an unexpected or unverified sender should never be opened, authority in the "From" line is not proof of identity.

## Takeaway

The email never needed to be opened to be judged. The lure structure (authority + finance + urgency + attachment) is the tell, and oledump lets you confirm what a document does before trusting it. The strongest control here is the human one: verify the sender through a second channel before acting.
