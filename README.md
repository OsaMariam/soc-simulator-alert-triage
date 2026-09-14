# SOC Simulator: Alert Triage Series

**TryHackMe SOC Simulator | Tier 1 analyst at a simulated media company, The Try Daily | June 2026**

Three alerts worked end to end from a live queue: triage, investigation in Splunk, threat intelligence validation, classification, escalation decision, and a written case report for each.

---

## Why these three together

They are not three copies of the same investigation. Put side by side they show the judgement calls that actually separate a Tier 1 analyst from someone who can only spot that an email looks fake.

| | [Alert 8815](alert-8815-phishing-email.md) | [Alert 8816](alert-8816-blocked-url.md) | [Alert 8817](alert-8817-allowed-malicious-login.md) |
| --- | --- | --- | --- |
| **Alert type** | Inbound phishing email | Firewall block on blacklisted URL | Inbound phishing email |
| **Severity as raised** | Medium | High | Medium |
| **Lure** | Fake Amazon delivery failure | (same URL as 8815) | Fake Microsoft sign-in alert |
| **Did the user click?** | Yes, after 81 seconds | Yes, after 74 seconds | Yes, after 69 seconds |
| **Firewall outcome** | **Blocked** | **Blocked** | **Allowed** |
| **Verdict** | True Positive | True Positive | True Positive |
| **Escalated?** | See note below | Yes, to Level 2 | Yes, possible account compromise |

**The pair that matters is 8816 and 8817.** Same kind of attack, same speed of click, completely different outcome. In 8816 the firewall blocked the connection because the URL was already blacklisted. In 8817 it allowed the connection because the lookalike domain was new and not on any list, so the user's browser reached a credential harvesting page.

Reading the word "allowed" instead of "blocked" in a firewall log is what turns a delivered phishing email into a possible account compromise. That is the whole difference, and it decides how the alert is handled.

## What each one demonstrates

**8815, the fake Amazon email.** Full phishing analysis from the alert detail alone: spoofed `.biz` domain, shortened URL hiding the destination, generic greeting, 48 hour deadline. Then proving the click from firewall logs. Includes the lesson that a CLEAN threat intelligence result on the sender domain does not clear an alert, because freshly registered spoofed domains have not been reported yet. The malicious indicator was in the URL, not the domain.

**8816, the blocked connection.** Working the same incident from the firewall's side instead of the email's. Pivoting on a shortened link to connect a blocked connection back to the phishing email that delivered it. Also the reasoning for escalating a threat that was already stopped: a blacklist only covers known threats, so it will not catch the next new domain the same user clicks.

**8817, the allowed connection.** The most serious of the three. A lookalike Microsoft domain (`m1crosoftsupport[.]co`, using a 1 in place of the i) that was not on any blacklist, so the connection went through to a fake login page. Includes spotting a decoy IP planted in the email body to be checked and come back clean, and checking the domain, the full URL and the actual server IP separately rather than assuming one result covers all three.

## Tools used across all three

- **Splunk** for email logs, firewall logs and web traffic
- **TryDetectThis** for URL, domain and IP reputation checks
- TryHackMe SOC Simulator Analyst VM, alert queue and case report system

## A note on consistency

In 8815 I recorded that escalation was not required, because the firewall blocked the connection before any impact. In 8816 I escalated a block on similar reasoning. Those two decisions sit differently and I would now escalate both, for the reason I gave in 8816: a blocked attempt still tells you a user clicked, and the blacklist that saved them this time will not cover the next unknown domain.

Leaving that visible rather than tidying it away, because changing your mind with a stated reason is part of the work.

## Skills across the series

Alert triage and prioritisation, phishing email analysis, SIEM log analysis with Splunk, pivoting across email and firewall data sources, distinguishing allowed from blocked connections, URL and domain and IP reputation checks, IOC identification and defanging, true positive versus false positive classification, Level 1 versus Level 2 escalation decisions, recognising a possible account compromise, avoiding planted false leads, and incident report writing under an MTTR timer.
