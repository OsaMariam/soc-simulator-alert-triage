# Alert 8817: Phishing Email and an Allowed Connection to a Malicious Login Page

Investigating a phishing email that led to a successful, allowed connection to confirmed malicious infrastructure.

| Investigation at a glance | |
| --- | --- |
| Alert ID | 8817 |
| Alert rule | Inbound Email Containing Suspicious External Link |
| Severity | Medium (as raised) |
| Date | 23 June 2026 |
| Environment | TryHackMe SOC Simulator (Analyst VM) |
| Tools used | Splunk, TryDetectThis |
| Verdict | **True Positive, escalated to Level 2 (possible account compromise)** |

**[Read the full report with all 20 screenshots (PDF)](Alert-8817-Allowed-Login.pdf)** for the original document exactly as written, with every figure.

---

## Goal

A Medium severity phishing alert came in for an inbound email that contained a suspicious external link. The alert asked me to check the firewall and proxy logs to see whether any endpoint had actually tried to reach the link, and whether that connection was allowed or blocked.

My job was to triage the alert, confirm whether the email was a real phishing attempt, find out if the user clicked and what the firewall did about it, decide whether it could be closed at Level 1 or needed escalating, and write a clear case report for whoever picked it up next.

## Tools used

- **Splunk**, the SIEM I used to search the email and firewall logs
- **TryDetectThis**, a URL and IP reputation tool, used to confirm the domain, the URL, and the server IP
- TryHackMe SOC Simulator (Analyst VM), including the alert queue and case report system
- Email logs, to read the phishing message and find the link inside it
- Firewall logs, web browsing traffic, to see whether the connection was allowed or blocked

## What I did

Picked up Alert 8817 and read the email details, a Microsoft "unusual sign-in" message with a link, sent to c.allen. Searched the recipient in Splunk to pull the email and confirm what it contained. Identified the phishing signs: a lookalike sender domain, a fake login link, and fear based urgency.

Searched the source host (10.20.2.25) to see whether the machine connected to the link, and whether the firewall allowed or blocked it. Searched the malicious domain directly to confirm the connection and tie it back to the email. Ran the domain, the full URL, and the server IP through TryDetectThis to confirm the reputation.

Classified the alert as a True Positive and wrote the case report, treating it as a possible account compromise. Marked the alert for escalation to Level 2 and submitted it.

## Investigation summary

The email was a Microsoft "unusual sign-in" message sent to `c.allen@thetrydaily[.]thm` from `no-reply@m1crosoftsupport[.]co`. It claimed someone had signed into the user's Microsoft account from Lagos and pushed them to "Review Activity" through a link.

### Finding the email

I searched the recipient in Splunk. Two inbound emails came back. The first was the phishing message. The second was a separate, ordinary business email, a networking luncheon invitation from a different sender, which read as normal and was not related. So only one of the two was a threat.

The phishing email had the usual tells. The sender domain, `m1crosoftsupport[.]co`, is a lookalike of microsoft.com. **It uses the number 1 in place of the "i", and it ends in .co rather than .com.** The link pointed to a fake Microsoft sign-in page built to capture whatever the user typed. The message leaned on fear and urgency to push a fast click, and the real link was hidden behind the words "Review Activity."

One detail worth noting: the email also showed an IP address (`102.89.222[.]143`) as the supposed sign-in location. **That IP is part of the scare story, not the attacker's infrastructure**, so I left it out of the investigation and focused on the domain in the link.

### Checking what the host actually did

I searched the source host, 10.20.2.25 (Charlotte Allen's machine, win-3463). Seven events came back.

The important one was at 06:09:50: the host connected to the fake login page, and **the firewall allowed the connection under the "Allow-Internet" rule.**

This is the part that makes 8817 serious. The malicious domain was not on the company blacklist, so instead of being blocked it was simply waved through as ordinary web traffic. The email had arrived at 06:08:41, so the click came about 69 seconds later.

The rest of the host's activity was routine work: internal company sites, Facebook Ads Manager, Asana, and a Google search about competitor tools, all normal for someone in a web and marketing role.

Going through the full set mattered, because it confirmed the only dangerous connection in the host's traffic was the one to the malicious domain. Everything else was benign.

### Confirming through the domain

I searched the malicious domain directly. The results showed the same allowed connection alongside the original phishing email that delivered the link. That confirmed the chain from end to end: the email carried the link, and the host connected to it.

### Confirming the reputation

I checked three things to be thorough: the domain, the exact URL, and the server IP the host actually connected to.

All three came back MALICIOUS. **The third one matters most.** `45.148.10[.]131` is the address Charlotte's machine actually reached, and confirming it as malicious closes any gap in the evidence. It is a different IP from the one in the email body, which, as expected, was just the scare story lure.

### Documenting the case and making the call

I set the classification to True Positive, because a real phishing email had led to a successful connection to confirmed malicious infrastructure.

The key point for escalation is that this was not a blocked attempt like the earlier firewall alert. The connection was allowed, so the user's browser reached a credential harvesting page.

**I was careful with the wording.** The firewall logs prove the connection succeeded, but they cannot show whether she actually typed in her password, so I treated it as a *possible* account compromise rather than a confirmed one.

The remediation actions focused on protecting the account: reset the password and revoke sessions, enable MFA, review the Microsoft sign-in logs for logins from unfamiliar locations, isolate and scan the host, block the domain and IP everywhere, add the indicators to the blacklist, search the mail environment for other recipients, and follow up with the user on phishing awareness.

One thing I flagged in the report: **the alert came in as Medium severity, but a successful, allowed connection to a confirmed credential phishing page realistically carries more risk than that.** I noted that the severity felt understated given the connection went through, so whoever handles the escalation treats it with the right urgency.

## Results and findings

- Charlotte Allen received a phishing email impersonating Microsoft, designed to steal her account password
- She clicked the link about 69 seconds after it arrived, and the firewall allowed the connection because the malicious domain was not on the blacklist
- Her machine reached the fake login page
- TryDetectThis confirmed the domain, the URL, and the server IP as malicious
- The rest of the host's activity was routine and benign
- Classified as a True Positive and escalated to Level 2 as a possible account compromise

### Key indicators (defanged)

| Indicator | Detail |
| --- | --- |
| Malicious URL | `hxxps://m1crosoftsupport[.]co/login` |
| Malicious domain | `m1crosoftsupport[.]co`, confirmed malicious |
| Malicious server IP | `45.148.10[.]131` (TCP/443), confirmed malicious |
| Phishing sender | `no-reply@m1crosoftsupport[.]co` |
| Email subject | Unusual Sign-In Activity on Your Microsoft Account |
| Affected user | `c.allen@thetrydaily[.]thm` |
| Source host | win-3463 / 10.20.2.25 |
| Firewall rule applied | Allow-Internet (connection allowed) |
| Verdict | True Positive, escalated to Level 2 |

Indicators are defanged, which is standard practice in written reports so the links cannot be clicked by accident. The `102.89.222[.]143` address from the email body is deliberately excluded. It was a decoy used in the lure, not real attacker infrastructure.

## Skills demonstrated

- Alert triage and prioritisation
- Phishing email analysis: lookalike domains, fake login pages, social engineering
- SIEM log analysis with Splunk
- Pivoting across email and firewall logs
- Distinguishing allowed versus blocked connections in firewall data
- URL, domain, and IP reputation checks
- Indicator of Compromise identification
- Incident classification, true positive versus false positive
- Escalation decision making and recognising a possible account compromise
- Avoiding false leads, specifically the decoy IP in the email body
- Incident documentation and case report writing

## What I learned

The main lesson from this one was the difference between a blocked attempt and an allowed one. A firewall block means the threat was stopped. An allowed connection to a malicious page means the user may have reached it and handed over their details, which is a completely different level of risk. **Reading the "allowed" in the firewall log is what turned this from a delivered phishing email into a possible account compromise.**

I also learned to watch for decoy details. The IP address in the email body was there to look scary and would have come back clean if I had checked it, so knowing to go after the domain in the link instead saved me from a wrong conclusion.

Checking the domain, the URL, and the server IP separately gave me a complete, confirmed picture rather than a partial one.

And writing the report reminded me to stay precise: say the connection succeeded, but do not claim the password was stolen when the logs cannot show that.
