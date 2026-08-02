---
{"dg-publish":true,"permalink":"/jr-penetration-tester/recruit2/","tags":["#professionalreport","#vulnerabilities","ssrf","openredirect","lfi","hardcodedcredentials","sqlinjection","sqli"],"created":"2026-08-02T10:20:08.614+02:00","updated":"2026-08-02T15:40:11.934+02:00","dg-note-properties":{"tags":["#professionalreport","#vulnerabilities","ssrf","openredirect","lfi","hardcodedcredentials","sqlinjection","sqli"]}}
---

![images.jpeg](/img/user/images.jpeg)

## Executive Summary

During the assessment of the Recruit web application, multiple vulnerabilities were chained together to obtain administrative access. An Open Redirect enabled Server-Side Request Forgery, which resulted in arbitrary file disclosure and exposure of hardcoded HR credentials. After authenticating as an HR user, an authenticated SQL Injection vulnerability allowed extraction of administrator credentials from the backend database.

----


| Step | Vulnerability               | Severity | Impact                                |
| ---- | --------------------------- | -------- | ------------------------------------- |
| 1    | Open Redirect               | Low      | Bypass SSRF Allowlist                 |
| 2    | SSRF                        | High     | Internal resource access              |
| 3    | LFI / Arbitrary File Read   | High     | Disclosure of application source code |
| 4    | Hardcoded credentials       | Medium   | HR account compromise                 |
| 5    | SQL Injection               | Critical | Database enumeration & extraction     |
| 6    | Admin credential disclosure | Critical | Admin account compromised             |


## Vulnerability Classification

To categorize weaknesses, this report utilizes the industry-standard **CWE (Common Weakness Enumeration)** taxonomy. Severity is calculated using the **CVSS (Common Vulnerability Scoring System)** to produce a numerical score reflecting the principle characteristics and impact of each finding

----
## Scope

**Target:** Recruit from Jr. Penetration Tester Path
**Environment:** 
1. Linux
2. Apache
3. PHP
4. MySQL
**Methodology**:
Black hat

----

## Technical Findings

### Finding 1 ~ Open Redirect

**Severity:** Low
**CWE**: CWE-601
**CVSS:** 3.1
**Description**
The application contained an Open Redirect vulnerability that allowed user-controlled redirection to arbitrary destinations.
**Impact**
An attacker can abuse the redirect functionality to redirect users to arbitrary locations. In this assessment, the vulnerability enabled the SSRF protection to be bypassed, ultimately contributing to arbitrary file disclosure.
**Evidence:**
See "Cyber Kill Chain" Section.
**Remediation**
Validate redirect destinations using an allowlist of trusted hosts or replace user-controlled redirects with server-side mappings.

----
### Finding 2 Server-side Request Forgery ~ SSRF

**Severity:** High
**CWE**: CWE-918
**CVSS:** 
**Description**
After bypassing the allowlist, the vulnerable server followed the redirect and accessed internal resources on behalf of the attacker.
**Impact**
**Evidence:**
**Remediation**

----
### Finding 3 Arbitrary Local File Read

**Severity:** 
**CWE**: 
**CVSS:** 
**Description**
The SSRF primitive was abused to retrieve the PHP source code of internal files, disclosing hardcoded HR credentials.
**Impact**
**Evidence:**
**Remediation**

----
### Finding 4 Hardcoded Credentials

**Severity:** 
**CWE**: 
**CVSS:** 
**Description**
The recovered credentials were used to authenticate as an HR user, exposing additional application functionality.
**Impact**
**Evidence:**
**Remediation**

----
### Finding 4 SQL Injection

**Severity:** 
**CWE**: 
**CVSS:** 
**Description**
The candidate search functionality was vulnerable to SQL injection, allowing extraction of administrator credentials from the backend database.
**Impact**
**Evidence:**
**Remediation**

----

## Cyber Kill Chain

#### Step 1 – Open Redirect

The application accepted a user-controlled redirect destination without sufficient validation. This allowed an attacker to bypass the SSRF allowlist by first requesting an approved URL that redirected to an arbitrary destination.

---

#### Step 2 – SSRF

After bypassing the allowlist, the vulnerable server followed the redirect and accessed internal resources on behalf of the attacker.

---

#### Step 3 – Arbitrary File Read

The SSRF primitive was abused to retrieve the PHP source code of internal files, disclosing hardcoded HR credentials.

![open-redirect-success 1.png](/img/user/open-redirect-success%201.png)

---

#### Step 4 – HR Authentication

The recovered credentials were used to authenticate as an HR user, exposing additional application functionality.

![hr-access 1.png](/img/user/hr-access%201.png)

---

#### Step 5 – SQL Injection

The candidate search functionality was vulnerable to SQL injection, allowing extraction of administrator credentials from the backend database.

![[sql-injection-confirmed 2.png\|sql-injection-confirmed 2.png]]

---

#### Step 6 – Administrative Access

The recovered administrator credentials were used to authenticate as an administrator, resulting in full compromise of the application.



----

## Lessons Learned

This assessment demonstrates how individually low- and medium-severity vulnerabilities can be chained together into a complete compromise of the application.