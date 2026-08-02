---
{"dg-publish":true,"permalink":"/jr-penetration-tester/recruit2/","tags":["#professionalreport","#vulnerabilities","ssrf","openredirect","lfi","hardcodedcredentials","sqlinjection","sqli"],"dg-note-properties":{"tags":["#professionalreport","#vulnerabilities","ssrf","openredirect","lfi","hardcodedcredentials","sqlinjection","sqli"]}}
---


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



```smart-table
{"columns":[{"id":"c1-v2bet","name":"Vulnerability","type":"text"},{"id":"c2-luorc","name":"Severity","type":"text","options":[{"label":"Todo","color":"gray"},{"label":"In progress","color":"blue"},{"label":"Done","color":"green"}]},{"id":"c3-k4emn","name":"Impact","type":"text"}],"rows":[{"id":"r6-viglh","cells":{"c1-v2bet":"Open Redirect","c2-luorc":"Low","c3-k4emn":"Bypass SSRF Allowlist"}},{"id":"r7-4bh6s","cells":{"c1-v2bet":"LFI / Arbitrary File Read","c2-luorc":"High","c3-k4emn":"Disclosure of application source code"}}],"sort":null,"filters":{},"showFilters":false}
```

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
**Impact**
**Evidence:**
**Remediation**
### Finding 2 Server-side Request Forgery ~ SSRF

**Severity:** High
**CWE**: CWE-918
**CVSS:** 
**Description**
**Impact**
**Evidence:**
**Remediation**
### Finding 3 Arbitrary Local File Read

**Severity:** 
**CWE**: 
**CVSS:** 
**Description**
**Impact**
**Evidence:**
**Remediation**

### Finding 4 Hardcoded Credentials

**Severity:** 
**CWE**: 
**CVSS:** 
**Description**
**Impact**
**Evidence:**
**Remediation**

### Finding 4 SQL Injection

**Severity:** 
**CWE**: 
**CVSS:** 
**Description**
**Impact**
**Evidence:**
**Remediation**


----

## Attack Narrative

### Open Redirect discovered & read config.php
![open-redirect-success 1.png](/img/user/open-redirect-success%201.png)

### SQL Injection discovered

![sql-injection-confirmed 2.png](/img/user/sql-injection-confirmed%202.png)

----

## Lessons Learned

This assessment demonstrates how individually low- and medium-severity vulnerabilities can be chained together into a complete compromise of the application.