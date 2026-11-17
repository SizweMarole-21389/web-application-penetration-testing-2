[← Back to overview](../README.md)

# Finding 14: Broken Access Control: Voucher PDF Without Authentication

**Severity:** Medium
**CWE:** CWE-306
**OWASP Top 10 (2021):** A01:2021

## Description

The PDF document generated for a purchased gift voucher can be downloaded by anyone who knows or guesses its code, with no login check of any kind standing in the way. The endpoint is `GET /api/vouchers/<code>/pdf`.

This is **CWE-306 (Missing Authentication for Critical Function)**, under **OWASP A01:2021-Broken Access Control**.

## Reproduction Steps

1. Buy a voucher and download its PDF once through the application, so the code is known and the genuine request is captured.

2. Resend the identical request with the `Authorization` header and every cookie removed entirely.

3. The response is still `200 OK`, and the genuine PDF document, complete with the voucher's code and monetary value, is returned in full, with no credentials of any kind required.

## Evidence


## Business Impact

This finding scores Medium on Confidentiality: the voucher document, and the monetary value it represents, is exposed to anyone who can enumerate or guess a code, undermining the confidentiality of the original purchaser's information, such as their name and the voucher's value, and the integrity of the voucher scheme as a control against a document a legitimate purchaser reasonably expects only they can retrieve.


## Remediation

Require authentication on this endpoint and confirm the requesting user genuinely owns the voucher in question before serving the document at all, rather than treating knowledge of the code alone as sufficient proof of ownership.
