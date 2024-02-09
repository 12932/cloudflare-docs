---
title: Recommended HTTP policies
pcx_content_type: learning-unit
weight: 4
layout: learning-unit
---

Add the following recommended HTTP policies.

## 1. All-HTTP-Application-InspectBypass

Bypass HTTP inspection for applications which use embedded certificates. This will help avoid any certificate pinning errors that may arise from an initial rollout.

| Selector    | Operator | Value            | Action         |
| ----------- | -------- | ---------------- | -------------- |
| Application | in       | _Do Not Inspect_ | Do Not Inspect |

## 2. Android-HTTP-Application-InspectionBypass

Bypass HTTPs inspection for the Android applications (such as Google Drive) use certificate pinning, which is incompatible with Gateway inspection.

| Selector                     | Operator | Value                | Logic | Action         |
| ---------------------------- | -------- | -------------------- | ----- | -------------- |
| Application                  | in       | _Google Drive_       | And   | Do Not Inspect |
| Passed Device Posture Checks | in       | _OS Version Android_ |       |                |

## 3. All-HTTP-Domain-Inspection-Bypass

Bypass HTTP inspection for a custom list of domains that were identified to have issues with the TLS Inspection.

| Selector | Operator | Value                                 | Logic | Action         |
| -------- | -------- | ------------------------------------- | ----- | -------------- |
| Domain   | in list  | <DomainInspectionBypass>              | Or    | Do Not Inspect |
| Domain   | in list  | <Corporate Domains \| Trusted Domain> |       |                |

## 4. All-HTTP-SecurityRisks-Blocklist

Block known threats such as Command & Control, Botnet and Malware based on Cloudflare's threat intelligence.

| Selector       | Operator | Value                | Action |
| -------------- | -------- | -------------------- | ------ |
| Security Risks | in       | _All Security Risks_ | Block  |

## 5. All-HTTP-ContentCategories-Blocklist

Although these categories are not always a security threat it's convenient to Block or Isolate them to minimize the risk your organization be exposed to Security Threats. Block content categories which go against your organization's acceptable use policy.

Initially, Allow action will help to track the policy matching, and identify potential false positives. Finally blocking these categories, allowlisting the Trusted Domains on the 'Trusted Domain' List used on the Rule 1.

| Selector           | Operator | Value                                                                                 | Action                      |
| ------------------ | -------- | ------------------------------------------------------------------------------------- | --------------------------- |
| Content Categories | in       | _Questionable Content_, _Security Risks_, _Miscellaneous_, _Adult Themes_, _Gambling_ | <Allow \| Inspect \| Block> |

## 6. All-HTTP-DomainHost-Blocklist

Block specific Domains or Hosts that are known to be malicious or pose a threat to your organization. This policy is usually implemented by creating custom blocklists or by using blocklists provided by threat intelligence partners or regional Computer Emergency and Response Teams (CERTs). Ideally Incident Response Teams can feed this List with API automation.

| Selector | Operator      | Value             | Logic | Action |
| -------- | ------------- | ----------------- | ----- | ------ |
| Domain   | in list       | <DomainBlocklist> | Or    | Block  |
| Host     | in list       | <HostBlocklist>   | Or    |        |
| Host     | matches regex | `.*example\.com`  |       |        |

## 7. All-HTTP-Application-Blocklist

Block unauthorized applications to limit their users' access to certain web-based tools and minimize the risk of Shadow IT. For example, the following policy blocks AI assistants

| Selector    | Operator | Value             | Action |
| ----------- | -------- | ----------------- | ------ |
| Application | in       | _ChatGPT_, _Bard_ | Block  |

## 8. PrivilegedUsers-HTTP-Any-Isolate

Isolate all the traffic for the Privileged Users and other Users that regularly have access to critical systems or they execute actions like Threat Analysis, Malware Testing, etc.

Security Teams usually need to perform Threat Analysis or Malware Testing that could lead them to trigger some Malware Connection.

Likewise for Privileged users that could be target of an attacker to gain access to critical systems.

| Selector         | Operator | Value                               | Action  |
| ---------------- | -------- | ----------------------------------- | ------- |
| User Group Names | in       | <Privileged Users \| Security Team> | Isolate |

## 9. All-HTTP-Domain-Isolate

Isolate High Risk or a Custom List of Domains to avoid data exfiltration or malware infection. Ideally Incident Response Teams can feed this List with API automation.

| Selector           | Operator | Value                              | Logic | Action  |
| ------------------ | -------- | ---------------------------------- | ----- | ------- |
| Content Categories | in       | _New Domain_, _Newly Seen Domains_ | Or    | Isolate |
| Domain             | in list  | DomainIsolation                    |       |         |
