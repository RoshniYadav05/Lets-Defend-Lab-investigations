# 🛡️ SOC Investigation: Possible IDOR Attack Detected

> **Platform:** Let'sDefend
> **Alert:** SOC169 – Possible IDOR Attack Detected
> **Event ID:** 119
> **Severity:** Medium
> **Alert Type:** Web Attack
> **MITRE ATT&CK:** T1190 – Exploit Public-Facing Application
> **Verdict:** True Positive
> **Attack Successful:** Yes
> **Tier 2 Escalation:** Yes

---

## 📌 Alert Information

| Field              | Details                                |
| ------------------ | -------------------------------------- |
| **Event ID**       | 119                                    |
| **Event Time**     | 2022-02-28 22:48:05 +03:00             |
| **Rule**           | SOC169 – Possible IDOR Attack Detected |
| **Role**           | Security Analyst                       |
| **Alert Type**     | Web Attack                             |
| **Severity**       | Medium                                 |
| **MITRE ATT&CK**   | T1190                                  |
| **Hostname**       | WebServer005                           |
| **Source IP**      | `134.209.118.137`                      |
| **Destination IP** | `172.16.17.15`                         |
| **HTTP Method**    | POST                                   |
| **Requested URL**  | `https://172.16.17.15/get_user_info/`  |
| **Device Action**  | Allowed                                |
| **Trigger Reason** | Consecutive requests to the same page  |

### Alert Evidence

![Alert Details](screenshots/01-alert-details.png)

---

## 📋 Incident Details

The alert was generated because an external source repeatedly accessed the `/get_user_info/` endpoint using consecutive requests.

![Incident Details](screenshots/02-incident-details.png)

The destination host was identified as:

* **Hostname:** `WebServer005`
* **IP:** `172.16.17.15`
* **OS:** Windows Server 2019
* **Domain:** `letsdefend.local`
* **Primary User:** `webadmin35`

![Endpoint Information](screenshots/07-endpoint-information.png)

---

# 🔎 Investigation Process

## 1. Reviewed the Alert

The initial alert showed an external source IP:

`134.209.118.137`

communicating with the internal web server:

`172.16.17.15`

The requested endpoint was:

`/get_user_info/`

The request method was `POST`, and the alert was triggered by consecutive requests to the same page.

![Alert Details](screenshots/01-alert-details.png)

---

## 2. Reviewed Network/Firewall Logs

The source IP was observed making repeated connections to the destination server over HTTPS/port `443`.

Observed source ports included:

* `49271`
* `43261`
* `47274`
* `48523`
* `49211`

Destination:

`172.16.17.15:443`

![Log Management](screenshots/03-log-management.png)

This established that the source IP was actively communicating with the internal web server.

---

# 🌐 3. Source IP Investigation

### Source IP

`134.209.118.137`

### WHOIS Investigation

WHOIS information showed:

* **Organization:** DigitalOcean, LLC
* **NetRange:** `134.209.0.0 – 134.209.255.255`
* **CIDR:** `134.209.0.0/16`
* **NetName:** `DIGITALOCEAN-134-209-0-0`
* **NetType:** Direct Allocation

![WHOIS Lookup](screenshots/10-whois-ip-address.png)

### ASN Investigation

The IP was associated with:

* **ASN:** `AS14061`
* **ASN Name:** `DIGITALOCEAN-ASN`
* **Organization:** DigitalOcean, LLC

![ASN Lookup](screenshots/11-asn-lookup-source-ip.png)

### IP Intelligence

Further enrichment classified the IP as:

* **Company:** DigitalOcean, LLC
* **ASN:** AS14061
* **AS Type:** Hosting
* **Infrastructure:** Cloud / Hosting
* **Hosted Domains:** 0

![IP Intelligence](screenshots/12-ipinfo-source-ip.png)

### Source IP Conclusion

`134.209.118.137` is a public IP associated with **DigitalOcean cloud/hosting infrastructure**.

The available public intelligence does **not conclusively establish whether this specific IP was a static/reserved address or a dynamically assigned pool address**.

---

# 🧪 4. VirusTotal Investigation

The source IP was checked against VirusTotal.

The available analysis showed:

> **0 / 91 security vendors flagged the IP as malicious.**

![Source IP VirusTotal](screenshots/04-source-ip-virustotal.png)

### Interpretation

The absence of threat-intelligence detections does **not** mean the activity is benign.

The maliciousness of this incident is determined primarily from the **observed web request behavior**, rather than reputation alone.

---

## 5. Destination IP Investigation

The destination IP was:

`172.16.17.15`

This is a **private/internal IPv4 address**.

VirusTotal identified it as a private IP and did not show malicious vendor detections.

![Destination IP VirusTotal](screenshots/08-destination-ip-virustota.png)

The destination was also confirmed as the internal web server:

`WebServer005`

---

# 🔬 6. Web Request Analysis

The most important part of the investigation was reviewing the raw web logs.

The same endpoint was repeatedly accessed:

```text
/get_user_info/
```

The attacker manipulated the `user_id` parameter.

---

## Request 1

```text
POST /get_user_info/
user_id=1
HTTP Response Status: 200
HTTP Response Size: 188 bytes
```

![Raw Log 1](screenshots/13-rawlog1.png)

---

## Request 2

```text
POST /get_user_info/
user_id=2
HTTP Response Status: 200
HTTP Response Size: 253 bytes
```

![Raw Log 2](screenshots/14-rawlog2.png)

---

## Request 3

```text
POST /get_user_info/
user_id=3
HTTP Response Status: 200
HTTP Response Size: 351 bytes
```

![Raw Log 3](screenshots/15-rawlog3.png)

---

## Request 4

```text
POST /get_user_info/
user_id=4
HTTP Response Status: 200
HTTP Response Size: 158 bytes
```

![Raw Log 4](screenshots/16-rawlog4.png)

---

## Request 5

```text
POST /get_user_info/
user_id=5
HTTP Response Status: 200
HTTP Response Size: 267 bytes
```

![Raw Log 5](screenshots/17-rawlog5.png)

---

# 🚨 7. IDOR Analysis

The request pattern showed sequential manipulation of the `user_id` parameter:

```text
user_id=1
      ↓
user_id=2
      ↓
user_id=3
      ↓
user_id=4
      ↓
user_id=5
```

This is consistent with **user ID enumeration**.

The endpoint `/get_user_info/` is specifically related to retrieving user information, making manipulation of the `user_id` parameter security-relevant.

### Key Evidence

| Indicator      | Observation                   |
| -------------- | ----------------------------- |
| Endpoint       | `/get_user_info/`             |
| Method         | POST                          |
| Parameter      | `user_id`                     |
| IDs tested     | 1, 2, 3, 4, 5                 |
| HTTP Status    | 200 for all requests          |
| Response Sizes | 188, 253, 351, 158, 267 bytes |
| Requests       | Permitted                     |
| Source         | External cloud-hosting IP     |

The **varying HTTP response sizes**, combined with successful `HTTP 200` responses for sequential user IDs, indicate that different user records were likely returned.

This is consistent with a successful **Insecure Direct Object Reference (IDOR)** attack.

---

# 🧩 8. Endpoint Verification

The targeted system was identified as:

| Field         | Value               |
| ------------- | ------------------- |
| Hostname      | `WebServer005`      |
| Domain        | `letsdefend.local`  |
| IP Address    | `172.16.17.15`      |
| OS            | Windows Server 2019 |
| Primary User  | `webadmin35`        |
| Client/Server | Server              |

![Endpoint Information](screenshots/07-endpoint-information.png)

The endpoint was shown as **host contained** in the investigation interface.

---

# 🦠 9. Additional File/URL Intelligence

Additional investigation artifacts were reviewed through VirusTotal, including URL and file/hash information available within the investigation environment.

![URL VirusTotal](screenshots/05-url-virustotal.png)

![Hashed File VirusTotal](screenshots/06-hashed-files-virustotal.png)

These reputation results were treated as supporting evidence only. The primary evidence for this incident remained the web request behavior and HTTP responses.

---

# 🔗 Indicators of Compromise (IoCs)

### Source IP

```text
134.209.118.137
```

**Provider:** DigitalOcean, LLC
**ASN:** AS14061
**Type:** Cloud / Hosting

### Destination IP

```text
172.16.17.15
```

**Hostname:** WebServer005
**Type:** Private/Internal IP

### Target Endpoint

```text
https://172.16.17.15/get_user_info/
```

### Observed Parameters

```text
user_id=1
user_id=2
user_id=3
user_id=4
user_id=5
```

---

# 🎯 MITRE ATT&CK

### T1190 – Exploit Public-Facing Application

The alert was mapped to:

**T1190 – Exploit Public-Facing Application**

The attacker interacted with a web-facing application endpoint and manipulated an object reference (`user_id`) to access different user records.

---

# 🛡️ Security Controls / Recommended Mitigations

To prevent similar IDOR attacks:

1. **Implement server-side authorization checks** for every object request.
2. Verify that the authenticated user has permission to access the requested `user_id`.
3. Avoid relying solely on client-controlled object identifiers.
4. Use unpredictable identifiers such as properly implemented UUIDs where appropriate.
5. Monitor for sequential object enumeration.
6. Alert on repeated requests where object IDs change rapidly.
7. Log authentication context together with object access.
8. Return appropriate `401/403` responses for unauthorized access attempts.
9. Apply rate limiting to sensitive API endpoints.
10. Conduct regular access-control testing on web applications.

---

# 📝 Analyst Note

Investigated the alert and identified sequential requests to `/get_user_info/` with changing `user_id` values (1–5) from external IP `134.209.118.137`. All requests returned HTTP 200 with varying response sizes, indicating successful access to different user records. The source IP is associated with DigitalOcean hosting infrastructure. The activity is consistent with a successful IDOR attack. Escalating to Tier 2 for further investigation.

![Analyst Note](screenshots/09-analyst-note.png)

---

# ✅ Investigation Conclusion

The investigation identified **malicious IDOR activity** targeting the `/get_user_info/` endpoint on `WebServer005`.

The attacker:

* Originated from external IP `134.209.118.137`
* Used DigitalOcean cloud/hosting infrastructure
* Repeatedly accessed `/get_user_info/`
* Sequentially manipulated the `user_id` parameter
* Tested user IDs `1–5`
* Received `HTTP 200` responses
* Received varying response sizes for each user ID
* Successfully accessed different user records

### Final Classification

| Investigation Result      | Verdict                    |
| ------------------------- | -------------------------- |
| **Traffic**               | Malicious                  |
| **Attack Type**           | IDOR                       |
| **Attack Successful**     | Yes                        |
| **True Positive**         | Yes                        |
| **Tier 2 Escalation**     | Yes                        |
| **Source Infrastructure** | DigitalOcean Cloud/Hosting |
| **Affected Host**         | WebServer005               |

The incident should be **escalated to Tier 2** for further investigation, impact assessment, and remediation of the application's access-control vulnerability.

---

## 📚 Key Learning

This investigation helped reinforce the importance of:

* Understanding IDOR and broken access control
* Analyzing HTTP request parameters
* Identifying user ID enumeration
* Using HTTP response codes and response sizes as investigation evidence
* Performing IP ownership and ASN enrichment
* Distinguishing IP reputation from actual malicious behavior
* Correlating firewall logs with web application logs
* Determining whether a web attack was successful
* Documenting evidence for SOC escalation

---

## ⚠️ Disclaimer

This investigation was performed in an authorized **Let'sDefend training environment** for cybersecurity learning and SOC analyst practice. All analysis and techniques were conducted for educational purposes.
