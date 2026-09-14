# Alert 8815: Fake Amazon Phishing Email Targeting an Employee

SOC alert triage and phishing investigation in a simulated SOC environment.

| Investigation at a glance | |
| --- | --- |
| Alert ID | 8815 |
| Alert type | Inbound phishing email |
| Severity | Medium |
| Date | 22 June 2026 |
| Environment | TryHackMe SOC Simulator, simulated company "The Try Daily" |
| Tools used | Splunk, TryDetectThis |
| Verdict | **True Positive, threat blocked before impact** |

**[Read the full report with all 19 screenshots (PDF)](Alert-8815-Phishing.pdf)** for the original document exactly as written, with every figure.

---

## Goal

This case study documents the investigation of Alert 8815, a medium severity phishing alert raised by the SOC Simulator platform. I was working as a Tier 1 SOC Analyst for a simulated media company, monitoring an active alert queue and triaging incoming security events.

The alert involved an inbound email sent to an employee that contained a suspicious external link. The investigation required me to determine whether the email was a genuine phishing attempt, whether the employee interacted with the malicious content, and whether any remediation or escalation was needed.

**Objective.** Work through a real phishing alert from start to finish: identify the threat, gather evidence from email and firewall logs in Splunk, validate indicators using threat intelligence, classify the alert correctly, and document all findings in a professional case report.

## Tools used

| Tool | Purpose |
| --- | --- |
| TryHackMe SOC Simulator | Simulation environment providing a live alert queue, documentation, SIEM access, and a virtual analyst workstation |
| Splunk (SIEM) | Searching email logs, firewall logs, and web traffic to trace the phishing email and confirm whether the recipient clicked the link |
| TryDetectThis | Threat intelligence tool on the Analyst VM used to check URLs and domains for known malicious indicators |
| Email log analysis | Reviewing inbound and outbound email logs to identify the phishing email and confirm delivery |
| Firewall log analysis | Confirming whether the recipient's machine attempted to access the malicious URL and whether the connection was blocked or allowed |

## What I did

Picked up Alert 8815 from the queue, assigned myself the alert and reviewed the full details including sender, recipient, subject line, email content, and the embedded URL before doing anything else.

Identified phishing indicators immediately. Searched Splunk for the recipient's email activity to establish a baseline before and after the phishing email arrived. Searched Splunk for her machine IP to see whether it had attempted to connect to any suspicious external URLs. Searched Splunk for the shortened URL directly.

Ran both indicators through TryDetectThis. Made the classification decision, wrote and submitted the full case report.

## Investigation summary

### Step 1: the alert queue

After closing the previous alert, I returned to the queue and could see Alert 8815 waiting, another medium severity phishing alert. I also noticed Alert 8816 in the queue labelled "Access to Blacklisted External URL Blocked by Firewall" with a High severity rating. That was something to come back to.

### Step 2: reading the alert

The alert showed an inbound email sent to Hannah Harris (`h.harris@thetrydaily[.]thm`) at 17:39:54 from `urgents@amazon[.]biz`. The subject line was "Your Amazon Package Couldn't Be Delivered, Action Required." The email body told Hannah to click a link to confirm her shipping information within 48 hours or her package would be returned.

**Phishing indicators spotted at first review:**

1. **Sender domain** `urgents@amazon[.]biz`. Amazon's real domain is amazon.com. The `.biz` suffix is a spoofed lookalike.
2. **Shortened URL.** Legitimate companies never use bit.ly in official communications. Shortened URLs hide the real destination.
3. **Generic greeting**, "Dear Customer". Real Amazon emails address you by your account name.
4. **Urgency pressure**, "48 hours or your package will be returned", designed to make the recipient act without thinking.

### Step 3: Hannah's email activity

Searching for Hannah's email address returned 4 events. Reading them in order painted a clear picture:

- **17:37:15** outbound email about scheduling a virtual meeting. Normal work activity.
- **17:38:14** internal email to IT Support about a colleague's onboarding email not arriving.
- **17:39:54** the phishing email arrived. This is the alert email.
- **17:42:46** outbound business email about next steps on an engagement. Normal work activity.

Nothing in her email activity suggested this was a false alarm.

### Step 4: the firewall logs

Searching for Hannah's IP address returned two firewall events:

- **17:38:46** connected to Google and searched "how to set up payroll system for small business". Allowed. Normal work browsing.
- **17:41:08** attempted to connect to the exact URL from the phishing email. The firewall **BLOCKED** this connection under the rule "Blocked Websites". Destination `67.199.248[.]11` on port 80.

**Key finding.** Hannah clicked the phishing link at 17:41:08, exactly **81 seconds** after the email arrived. Her machine sent a connection request to the malicious URL, but the company firewall blocked it before the destination was reached. No data was transmitted.

### Step 5: confirming the URL scope

Searching directly for the shortened URL in Splunk confirmed **only one event across the entire environment**, the blocked connection from Hannah's machine. No other machine in the company had attempted to access this URL. The threat was contained to her endpoint, and even there it was blocked before reaching the destination.

### Step 6: the sender domain

Searching for the sender domain returned only one result, the phishing email itself. There was no history of this domain communicating with the company before. Not a known vendor or approved sender. Combined with the `.biz` domain spoofing Amazon's real `.com`, this confirmed the sender was impersonating Amazon.

### Step 7: threat intelligence checks

I ran two checks on TryDetectThis:

**The sender domain** returned Status: **CLEAN**. The domain itself is not flagged in threat intelligence. This is common. Spoofed domains are often newly registered and not yet blacklisted. However, the fact that it is not amazon.com is still a clear indicator of impersonation.

**The shortened URL** returned Status: **MALICIOUS**. The actual URL Hannah clicked is confirmed as a known malicious indicator. This sealed the verdict.

**Why the clean sender domain does not clear the alert.** A CLEAN result does not mean the email is legitimate. Threat intelligence tools rate what they have seen before, and a freshly registered spoofed domain will often return CLEAN because it has not been reported yet. The real evidence was the malicious URL that the email was designed to get Hannah to click.

### Step 8: writing the case report

The case report was a True Positive report form, requiring time of activity, affected entities, classification reasoning, escalation reasoning, recommended remediation actions, and the list of attack indicators. All sections completed.

## Results and findings

| Finding | Detail |
| --- | --- |
| **Classification** | TRUE POSITIVE. A real phishing email was delivered and the recipient clicked the malicious link |
| **Email received** | Hannah Harris received a phishing email at 17:39:54 impersonating Amazon |
| **Link clicked** | Firewall logs confirmed her machine attempted to connect to the malicious URL at 17:41:08, 81 seconds after arrival |
| **Connection blocked** | The firewall blocked the connection under the "Blocked Websites" rule before the destination was reached. No data transmitted |
| **Threat confirmed** | TryDetectThis confirmed the URL as MALICIOUS. The sender domain returned CLEAN but is a clear Amazon impersonation |
| **IOCs** | Sender `urgents@amazon[.]biz`, URL `hxxp://bit[.]ly/3sHkX3da12340`, Dest IP `67.199.248[.]11`, Source 10.20.2.17, Host win-3457 |

**Verdict: True Positive.** A spoofed Amazon email was delivered to Hannah Harris, who clicked the embedded malicious link within 81 seconds. The company's web filtering policy blocked the outbound connection before any communication with the malicious destination could be established. No data was exfiltrated, no payload was delivered, and no endpoint was compromised. The threat was fully contained by existing security controls.

## Skills demonstrated

- **Phishing email analysis.** Identified multiple indicators from the alert details before starting any investigation: spoofed sender domain, shortened URL, generic greeting, and urgency based social engineering
- **SIEM log analysis (Splunk).** Queried email and firewall logs across multiple search terms to build a complete timeline
- **Firewall log interpretation.** Correctly read action, source IP, destination IP, destination port, URL, and the rule that triggered the block
- **Threat intelligence.** Ran multiple IOC checks and understood why a CLEAN result for the sender domain did not clear the alert
- **IOC identification and documentation**
- **Alert classification.** Correctly identified a True Positive based on evidence
- **Incident report writing.** Completed a detailed report covering entities, timeline, classification rationale, remediation actions, and attack indicators
- **Analytical thinking under pressure.** Worked against the MTTR timer while making accurate, evidence based decisions

## What I learned

This alert taught me **the difference between a threat that was stopped and a threat that never existed.** These are two very different things.

With the previous alert, the email was legitimate. Nothing bad happened and nothing was ever going to happen. With this one, something bad absolutely happened. Hannah received a real phishing email and she clicked the link. The only reason this did not become a serious incident is that the firewall caught it in time. **That is not a false alarm. That is a near miss, and near misses deserve just as much documentation and follow up as actual incidents.**

A CLEAN result from threat intelligence does not mean something is safe. The sender domain came back CLEAN because it was likely newly registered and not yet reported. The real threat was in the URL. Always check all the indicators, not just one.

The firewall log tells you more than just whether traffic was blocked. It tells you the exact time, the source machine, the destination, and the rule that fired. Reading those details carefully is what let me confirm the click and exactly when it happened.

Speed matters, but accuracy matters more. The simulator tracks MTTR, so there is pressure to close alerts quickly. But closing an alert incorrectly wastes more time than taking an extra two minutes to verify findings properly.

Documenting your reasoning is as important as reaching the right verdict. A good case report shows not just what you decided but why. Anyone reading it should be able to follow every step of the thinking from the first alert detail to the final classification.
