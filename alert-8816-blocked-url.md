# Alert 8816: Blocked Connection to a Malicious Phishing URL

Alert triage, log analysis, and escalation in a simulated SOC environment.

| Investigation at a glance | |
| --- | --- |
| Alert ID | 8816 |
| Alert rule | Access to Blacklisted External URL Blocked by Firewall |
| Severity | High |
| Date | 23 June 2026 |
| Environment | TryHackMe SOC Simulator (Analyst VM) |
| Tools used | Splunk, TryDetectThis |
| Verdict | **True Positive, escalated to Level 2** |

**[Read the full report with all 15 screenshots (PDF)](Alert-8816-Blocked-URL.pdf)** for the original document exactly as written, with every figure.

---

## Goal

A firewall alert fired in the SOC when a workstation on the internal network tried to reach a URL that was on the company blacklist. The connection was blocked, but **a blocked attempt still needs looking into.**

My job was to triage the alert, work out whether it was a real threat or a false alarm, find out what the host was doing and where the link came from, decide whether it could be closed at Level 1 or needed escalating, and write a clear case report that anyone picking it up later could follow.

## Tools used

- **Splunk**, the SIEM I used to search and read the firewall and email logs
- **TryDetectThis**, a URL and IP reputation tool, used to check the destination address
- TryHackMe SOC Simulator (Analyst VM), including the alert queue and the case report system
- Firewall logs, web browsing traffic that showed the blocked connection
- Email logs, used to trace the phishing message that carried the link

## What I did

Picked up Alert 8816 from the queue and read through the alert details. Noted the key fields: the source host, the destination IP, the URL, the port, and the firewall rule that blocked the traffic.

Searched the source host in Splunk to see what else that machine had been doing around the same time. Searched the destination IP to check whether the host actually connected or the attempt was stopped. Searched the shortened URL in Splunk, which linked the firewall event back to a phishing email sent to the same user a minute earlier.

Ran the destination IP through TryDetectThis to check its reputation. Classified the alert as a True Positive and wrote the case report. Marked the alert for escalation to Level 2 and submitted it.

## Investigation summary

The alert was High severity, raised on 23 June 2026 at around 06:10. The firewall had blocked an outbound connection: action "blocked", source internal host 10.20.2.17 on port 34257, destination `67.199.248[.]11` on port 80, URL `hxxp://bit[.]ly/3sHkX3da12340`, application web browsing, protocol TCP, rule "Blocked Websites".

So the connection had already been stopped. But a machine on the network had tried to reach a blacklisted address, and I wanted to know why.

### Checking the source host

I searched the source host in Splunk. The search returned the blocked connection at 06:07:37.

The same host also had an ordinary allowed connection a couple of minutes earlier, a normal Google search about setting up a payroll system. That looked like routine work browsing, which told me the machine was not generally reaching out to bad sites. The blocked connection was the one event that stood out.

### Checking the destination IP

Next I searched the destination IP to confirm whether the host had actually reached it. Only one event came back. It was the same blocked connection, stopped under the "Blocked Websites" rule.

**Because there was no successful session to that IP anywhere in the logs, I could be confident the firewall had done its job** and nothing got through to that address.

### Tracing the URL back to its source

I then searched the shortened URL itself to see where it had come from. This time two events came back.

The second event was an inbound email, and it was the source of the link. The message went to `h.harris@thetrydaily[.]thm` from `urgents@amazon[.]biz`, with the subject "Your Amazon Package Couldn't Be Delivered - Action Required." It told the user their package could not be delivered because of an incomplete address and asked them to confirm their shipping details by clicking the link, with a warning that the package would be returned within 48 hours.

That is a clear phishing email. The sender is pretending to be Amazon but using a `.biz` domain, the greeting is a generic "Dear Customer", it pushes a deadline to make the user act fast, and the real destination is hidden behind a shortened link.

The email arrived at 06:06:23 and the firewall blocked the connection at 06:07:37, **about 74 seconds apart.** That short gap strongly suggests the user opened the email and clicked the link almost straight away.

### Confirming the reputation of the destination

To be sure the link was not just suspicious looking but actually dangerous, I opened TryDetectThis and ran a check on the destination. The result came back as MALICIOUS.

### Documenting the case and making the call

I set the incident classification to True Positive, because the evidence showed a real attempt to reach confirmed malicious infrastructure rather than a false alarm.

I explained why I classified it as a True Positive and why it needed escalating: **the firewall blocked this attempt, but the blacklist only covers known threats, so it will not catch a new malicious domain the user might click next.** The same URL and host were also tied to earlier phishing activity against this user, so the threat was not a one off.

I then listed the recommended remediation actions: investigating the workstation, checking the endpoint logs, confirming no other machines tried the same link, keeping the URL and IP blocked, searching the mail environment for similar messages, and following up with the user on phishing awareness.

**I kept the wording of the report careful on one point.** The firewall log proves the host attempted the connection and that it was blocked, so I wrote it that way rather than stating outright that the user definitely clicked, even though the 74 second timing points strongly towards a click. It is a small thing, but I wanted the report to say exactly what the evidence supports.

## Results and findings

The investigation confirmed this was a genuine threat, not a false positive.

- A workstation used by Hannah Harris tried to reach a malicious URL, and the firewall blocked it under the "Blocked Websites" rule
- The link came from a phishing email impersonating Amazon, sent to the same user about a minute before the connection attempt
- TryDetectThis confirmed the destination IP as malicious
- There was no successful connection to the malicious IP in the logs, the block held
- Classified as a True Positive and escalated to Level 2 so the endpoint could be checked and the wider follow up handled

### Key indicators (defanged)

| Indicator | Detail |
| --- | --- |
| Malicious URL | `hxxp://bit[.]ly/3sHkX3da12340` |
| Destination IP | `67.199.248[.]11` (TCP/80), confirmed malicious |
| Source host | 10.20.2.17 |
| Affected user | `h.harris@thetrydaily[.]thm` |
| Phishing sender | `urgents@amazon[.]biz` |
| Email subject | Your Amazon Package Couldn't Be Delivered - Action Required |
| Firewall rule | Blocked Websites |
| Verdict | True Positive, escalated to Level 2 |

## Skills demonstrated

- Alert triage and prioritisation
- SIEM log analysis with Splunk
- Phishing email analysis: sender, content, and intent
- Pivoting across multiple data sources, firewall and email logs
- URL and IP reputation checks
- Indicator of Compromise identification
- Incident classification, true positive versus false positive
- Escalation decision making, Level 1 versus Level 2
- Incident documentation and case report writing
- Evidence based reasoning and attention to detail

## What I learned

The biggest thing was **how much a single shortened link can tie separate events together.** Searching the bit.ly URL is what connected the firewall block to the phishing email and turned a "blocked connection" into a clear phishing story.

I also learned not to treat a blocked connection as the end of the job. The firewall stopped this attempt, but the user behaviour and the chance that something else slipped past still matter, which is why it was worth escalating.

Checking the host's other activity, rather than just the one flagged event, helped me judge how serious it really was. And writing the report taught me to be careful with my words, to say what the logs actually prove and not more than that.
