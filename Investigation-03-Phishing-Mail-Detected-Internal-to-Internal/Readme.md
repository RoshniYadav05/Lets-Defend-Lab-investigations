# Investigation-03: Phishing Mail Detected - Internal to Internal

## Overview

This repository documents the investigation of a **SOC120 - Phishing Mail Detected - Internal to Internal** alert completed on the LetsDefend SOC platform.

The investigation focused on validating whether an internal email communication was malicious or a legitimate business email by analyzing the email content, endpoint activity, threat intelligence, and log monitoring.

---

## Alert Information

| Field | Value |
|-------|--------|
| Rule | SOC120 - Phishing Mail Detected - Internal to Internal |
| Severity | Medium |
| Event ID | 52 |
| Event Type | Exchange |
| SMTP IP | 172.16.20.3 |
| Sender | john@letsdefend.io |
| Recipient | susie@letsdefend.io |
| Subject | Meeting |
| Device Action | Allowed |

---

## Investigation Summary

The email metadata and content were reviewed to identify any phishing indicators.

The email contained only a meeting request between two internal users and did not contain any malicious URLs or file attachments.

Further investigation was performed using:

- Email Security
- Endpoint Security
- Threat Intelligence
- Log Monitoring

No malicious indicators or suspicious user activity were identified during the investigation.

---

## Conclusion

The email was determined to be a legitimate internal communication.

**Alert Classification:** False Positive

---

## Skills Demonstrated

- Email Security Investigation
- Email Header Analysis
- Endpoint Investigation
- Threat Intelligence Verification
- Log Monitoring
- SOC Investigation Workflow
- Incident Documentation

---

## Platform

LetsDefend.io
