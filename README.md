# Web Application Penetration Testing 2 - Imgurr Image Marketplace

A **private** write-up of a **black-box web application penetration test** against Imgurr, an image
marketplace application provided as a lab. Testing started from an
unauthenticated visitor and a self-registered user, and reached remote code execution, administrator
takeover, arbitrary file read and token theft.

> ⚠️ **Private repository.** Proof markers are redacted in the text and hidden in the screenshots. Screenshots are
> **not** blurred, so do not make this repository public without sanitising them first.
> Testing was authorised and carried out in a lab. Nothing here should be used against a live system.

**Assessor:** Sizwe Marole &nbsp;|&nbsp; **Assessment type:** Black-box web application penetration test
**Dates:** 10 September - 6 October 2026 &nbsp;|&nbsp; **Report version:** 3.5

---

## Executive Summary

Imgurr is an image marketplace application that was assessed through manual web application security testing of the exposed application functionality. A total of 28 distinct findings were identified: 10 High, 7 Medium, 10 Low, and 1 Informational. The overall security posture is high risk because several weaknesses can be chained from an unauthenticated starting point to achieve serious outcomes, including remote code execution in the application server context, unauthorised administrative actions, account compromise, arbitrary file read, and exposure of authentication tokens.

The highest priorities are to eliminate the unsafe file-handling and operating-system command-execution paths, enforce centralised deny-by-default authentication and authorisation across protected routes, and immediately rotate the weak administrator credential and JWT signing secret. The remaining High findings should then be remediated and re-tested before the Medium and Low findings are addressed according to dependency and business impact. Until these issues are corrected, an equivalent production deployment should not be used to process real customer information, credentials, payment-card data, or monetary transactions.

---

## 📋 Findings at a Glance

| # | Finding | Severity | CWE | OWASP 2021 |
|---|---------|----------|-----|------------|
| 1 | [Cleartext Transport (No TLS)](findings/01-cleartext-transport.md) | **Medium** | CWE-319 | A02:2021 |
| 2 | [Information Disclosure (Server Version Banner)](findings/02-information-disclosure.md) | **Informational** | CWE-200 | A05:2021 |
| 3 | [Missing Clickjacking Protection](findings/03-clickjacking.md) | **Low** | CWE-1021 | A05:2021 |
| 4 | [Permissive Cross-Origin Resource Sharing](findings/04-permissive-cors.md) | **Low** | CWE-942 | A05:2021 |
| 5 | [User Enumeration via Registration](findings/05-user-enumeration.md) | **Low** | CWE-203 | A07:2021 |
| 6 | [Weak Password Policy](findings/06-weak-password-policy.md) | **Low** | CWE-521 | A07:2021 |
| 7 | [Reflected Cross-Site Scripting (Image Search)](findings/07-reflected-xss.md) | **Medium** | CWE-79 | A03:2021 |
| 8 | [Open Redirect and DOM-Based XSS via Unsafe Navigation](findings/08-open-redirect-dom-xss.md) | **Medium** | CWE-601 | A01:2021 |
| 9 | [Pagination Limit Bypass](findings/09-pagination-limit-bypass.md) | **Low** | CWE-770, CWE-20 | A04:2021 |
| 10 | [Stored Cross-Site Scripting (Comments)](findings/10-stored-xss.md) | **High** | CWE-79 | A03:2021 |
| 11 | [Client-Side Template Injection (Profile Bio)](findings/11-client-side-template-injection.md) | **Medium** | CWE-1336 | A03:2021 |
| 12 | [Sockpuppet Vote Manipulation via Unrestricted Registration](findings/12-sockpuppet-votes.md) | **Low** | CWE-799, CWE-840 | A04:2021 |
| 13 | [Business Logic Flaw: Negative Voucher Amount](findings/13-negative-voucher-logic.md) | **Medium** | CWE-840, CWE-20 | A04:2021 |
| 14 | [Broken Access Control: Voucher PDF Without Authentication](findings/14-voucher-pdf-no-auth.md) | **Medium** | CWE-306 | A01:2021 |
| 15 | [Race Condition: Voucher Double-Redeem](findings/15-voucher-race-condition.md) | **Medium** | CWE-362 | A04:2021 |
| 16 | [Seller Approval Endpoint Requires No Authentication](findings/16-seller-approval-no-auth.md) | **High** | CWE-306 | A01:2021 |
| 17 | [Unrestricted File Upload leading to Remote Code Execution](findings/17-file-upload-rce.md) | **High** | CWE-434, CWE-94 | A04:2021 |
| 18 | [Broken Access Control: Purchase Verification Without Authentication](findings/18-purchase-verify-no-auth.md) | **Low** | CWE-306, CWE-639 | A01:2021 |
| 19 | [Path Traversal / Arbitrary File Read](findings/19-path-traversal-lfi.md) | **High** | CWE-22 | A01:2021 |
| 20 | [Weak JWT Signing Secret](findings/20-weak-jwt-secret.md) | **High** | CWE-798, CWE-321 | A02:2021 |
| 21 | [Privilege Escalation via Forged JWT Claim After Signing-Key Compromise](findings/21-forged-jwt-privesc.md) | **High** | CWE-863 | A01:2021 |
| 22 | [No Session or Token Invalidation](findings/22-no-token-invalidation.md) | **Low** | CWE-613 | A07:2021 |
| 23 | [Weak Administrator Password](findings/23-weak-admin-password.md) | **High** | CWE-521 | A07:2021 |
| 24 | [Admin Dashboard Accessible With No Authentication](findings/24-admin-dashboard-no-auth.md) | **High** | CWE-306 | A01:2021 |
| 25 | [CSRF Protection Checks the Cookie But Never Validates the Token](findings/25-csrf-token-not-validated.md) | **Low** | CWE-352 | A01:2021 |
| 26 | [Account Takeover via Administrative Password Reset](findings/26-admin-account-takeover.md) | **High** | CWE-620 | A01:2021 |
| 27 | [Excessive Number of Administrator Accounts](findings/27-excessive-admin-accounts.md) | **Low** | n/a | A01:2021 |
| 28 | [OS Command Injection in the Admin Ping Tool](findings/28-command-injection-ping.md) | **High** | CWE-78 | A03:2021 |

**Result:** 28 findings - 10 High, 7 Medium, 10 Low, 1 Informational.

Click any finding for its full write-up: description, reproduction steps, evidence, business impact and remediation.

---

## Methodology

Testing was manual and evidence-driven. The application's front-end code and exposed functionality were reviewed first to map reachable functions, endpoints, parameters, and application flows. Testing was then performed systematically across the identified attack surface, with particular attention to authentication, authorisation, input handling, business logic, session management, file handling, command execution, and security configuration.

Burp Suite was the primary assessment tool and was used to capture, modify, replay, and compare HTTP requests and responses. Suspected vulnerabilities were manually validated through reproducible application behaviour, server responses, and successful exploitation where applicable.

Testing also considered the relationship between individual weaknesses. Where vulnerabilities could be combined, the resulting attack path was assessed to determine whether an unauthenticated or low-privileged starting point could lead to greater access, compromise, or impact elsewhere in the application or its supporting server-side environment. Server-side effects were assessed where they could be demonstrated through the application during testing.

Where testing required repeated or concurrent requests that could not be reliably performed through Burp Suite's GUI, short Python scripts using the requests library were used. Independent sessions were used where required to accurately represent separate client interactions. jwt.io was used to inspect and re-sign JWTs after recovery of the signing secret, while hashcat was used for offline recovery of the JWT signing secret.

The assessment was limited to the Imgurr web application, its exposed functionality, and the application and server-side behaviour that could be demonstrated through that functionality. Evidence was collected from observed requests and responses, application behaviour, server responses, and reproducible exploitation.

Findings were classified using the OWASP Top 10:2021 framework to maintain consistent categorisation throughout the report. Severity was assessed using CVSS v3.1, considering attack vector, attack complexity, privileges required, user interaction, scope, and potential impact to confidentiality, integrity, and availability.

## Risk Rating Definitions

| Rating | Definition |
| --- | --- |
| **High** | Potential for an attacker to control, alter, or delete electronic assets. Unauthorised access, data capture, defacement. |
| **Medium** | Combined with other factors, potential for asset compromise. Conditional exploitation, configuration-dependent. |
| **Low** | Likelihood or impact of exploitation is extremely low. Weak ciphers, outdated protocols. |
| **Informational** | Cannot be exploited directly but not aligned with best practice. Information disclosure, version leakage. |

---

## Conclusion

The assessment identified 28 vulnerabilities and security weaknesses across authentication, authorisation, input handling, session management, business logic, transport security, and security configuration: 10 High, 7 Medium, 10 Low, and 1 Informational. The application has a high-risk security posture in its current state. The most serious demonstrated outcomes include remote code execution in the application server context, privilege escalation, administrative account compromise, arbitrary file read, and stored cross-site scripting capable of exposing authentication tokens.

Remediation should begin with the High-rated findings and the attack chains that enable server-side execution or privileged access. The administrator password and JWT signing secret should be rotated immediately, vulnerable command and upload functionality should be redesigned to remove execution paths, and protected routes should inherit a common deny-by-default authentication and authorisation policy. After remediation, each affected function should be re-tested using the reproduction conditions documented in this report before the finding is considered closed.
