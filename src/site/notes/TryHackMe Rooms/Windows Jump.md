---
{"dg-publish":true,"permalink":"/try-hack-me-rooms/windows-jump/","tags":["#professionalreport","#vulnerabilities","ethicalhacking","hacking","owasp","owasptop10","pentesting","pentestmethodology","reportwriting","infosec"],"noteIcon":"","created":"2026-08-02T18:46:22.586+02:00","updated":"2026-08-15T21:04:22.118+02:00","dg-note-properties":{"tags":["#professionalreport","#vulnerabilities","ethicalhacking","hacking","owasp","owasptop10","pentesting","pentestmethodology","reportwriting","infosec"]}}
---

# Windows Jump

## Executive Summary

insert summary here

----


## Vulnerability Classification

To categorize weaknesses, this report utilizes the industry-standard **CWE (Common Weakness Enumeration)** taxonomy. Severity is calculated using the **CVSS (Common Vulnerability Scoring System)** to produce a numerical score reflecting the principle characteristics and impact of each finding

----
## Scope

**Target:** Windows Jump - Medium Level
**Environment:** 
1. Microsoft Windows Server 2019 

**Services to Target**

| Port                              | Service                       | Version                     |
| --------------------------------- | ----------------------------- | --------------------------- |
| 135                               | MSRPC                         | Microsoft Windows RPC       |
| 139                               | NetBIOS Session Service       | Microsoft Windows NetBIOS   |
| 3389                              | Remote Desktop Protocol       | Microsoft Terminal Services |
| 5985                              | WinRM                         | Microsoft HTTPAPI 2.0       |
| 47001                             | WinRM Management              | Microsoft HTTPAPI 2.0       |
| 7680                              | Windows Delivery Optimization | Unknown                     |
| 49665, 49666, 49669, 49671, 49674 | MSRPC                         | Dynamic RPC                 |
| 49664, 49667, 49670               | Unknown                       | Dynamic RPC endpoints       |

**Assessment Type**:
Black hat

----

## Technical Findings

### Finding 1 ~ 

**Severity:** 
**CWE**: 
**CVSS:** 
**Description**
**Impact**
**Evidence:**
**Remediation**
### Finding 2 

**Severity:** 
**CWE**: 
**CVSS:** 
**Description**
**Impact**
**Evidence:**
**Remediation**
### Finding 3 

**Severity:** 
**CWE**: 
**CVSS:** 
**Description**
**Impact**
**Evidence:**
**Remediation**

### Finding 4

**Severity:** 
**CWE**: 
**CVSS:** 
**Description**
**Impact**
**Evidence:**
**Remediation**

### Finding 4 

**Severity:** 
**CWE**: 
**CVSS:** 
**Description**
**Impact**
**Evidence:**
**Remediation**


----

## Attack Chain

| Step | Vulnerability               | Severity | Impact                                |
| ---- | --------------------------- | -------- | ------------------------------------- |
| 1    | Open Redirect               | Low      | Bypass SSRF Allowlist                 |
| 2    | SSRF                        | High     | Internal resource access              |
| 3    | LFI / Arbitrary File Read   | High     | Disclosure of application source code |
| 4    | Hardcoded credentials       | Medium   | HR account compromise                 |
| 5    | SQL Injection               | Critical | Database enumeration & extraction     |
| 6    | Admin credential disclosure | Critical | Admin account compromised             |


#### 1. Open Redirect
The application accepted a user-controlled redirect destination without sufficient validation. This allowed an attacker to bypass the SSRF allowlist by first requesting an approved URL that redirected to an arbitrary destination.

#### 2. SSRF

----

## Lessons Learned



