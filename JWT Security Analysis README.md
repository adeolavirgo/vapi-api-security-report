# JWT Security Analysis Report

**Performed by:** Adeola Odunlade  
**Date:** 28/11/2025  

---

## Executive Summary

This report documents a full security analysis of a JSON Web Token (JWT) provided as part of the Week 2 lab assignment.

The assessment included:

- Manual decoding
- Automated inspection
- Token generation
- Token forgery
- Server interaction
- Traffic interception with Burp Suite
- Exploitation of insecure JWT implementations

To validate discovered weaknesses, I built a controlled environment consisting of two Flask servers:

1. A properly secured implementation  
2. A deliberately vulnerable implementation  

By interacting with both, I demonstrated how insecure JWT handling enables privilege escalation and full administrative compromise.

---

## Scope of Work

The scope of the assessment included:

- The JWT provided in the assignment prompt
- Local Flask servers (`app_secure.py` and `app_vulnerable.py`)
- Tools used:
  - jwt.io
  - Burp Suite
  - PyJWT
  - jwttool
  - Base64 utilities
- Locally generated tokens and scripts

All testing was performed within the `~/jwt_lab` environment.

---

# Methodology

The methodology followed a structured penetration testing approach focused on JWT weaknesses.

## 1. Manual Token Inspection

The provided JWT was analyzed using:

- jwt.io
- Base64 decoding
- Python (PyJWT)

The decoded header revealed:

```json
{
  "alg": "none",
  "typ": "JWT"
}
This immediately indicated that:

The token had no cryptographic signature

It required no secret key

It was fully forgeable

The decoded payload showed:
{
  "user": "admin",
  "exp": 1600000000,
  "role": "admin",
  "valid": false
}
Critical Observations

"alg": "none" removes signature protection entirely

"role": "admin" grants privilege via plain JSON

"valid": false can be manipulated

"exp": 1600000000 was already expired (2020 timestamp)

This confirmed the token was insecure and exploitable.
2. Controlled Lab Deployment

Two Flask servers were launched:

Secure Server — Enforces signature verification and expiration checks

Vulnerable Server — Accepts unsigned tokens and ignores expiration

This allowed direct behavioral comparison.
3. Token Generation Analysis

Using:
python3 tokens.py | tee tokens_output.txt
Three tokens were generated:

VALID_HS256 — Properly signed

EXPIRED_HS256 — Signed but expired

ALG_NONE — Unsigned token

Findings

The secure server accepted only the valid HS256 token.

The secure server rejected expired tokens.

The secure server rejected unsigned tokens.

The vulnerable server accepted all of them.

This clearly demonstrated the importance of signature validation.
Forged Token Creation

To simulate attacker behavior, I executed:
python3 make_modified_none.py > modified.txt
The forged token payload became:
{
  "user": "admin",
  "role": "admin",
  "valid": true,
  "exp": 1600000000
}
Why This Matters

"valid": true bypassed application logic.

"role": "admin" granted elevated access.

"alg": "none" meant no signature verification occurred.

The token required:

No secret key

No cryptographic signing

No advanced tooling

Just JSON modification + base64url encoding.
Successful Exploitation

Submitting the forged token to the vulnerable server:
curl -i -H "Authorization: Bearer $TOKEN" http://127.0.0.1:5002/admin
Response:
200 OK
Welcome admin (VULNERABLE)
This confirmed full authentication and authorization bypass.
Burp Suite Interception & Manipulation

Traffic was routed through Burp Suite:

Proxy configured

Authorization header intercepted

Forged JWT injected

Burp confirmed a 200 OK response from the vulnerable server.

This proved the exploit works through real HTTP flows, not just command-line tools.
Full Security Impact

The assessment demonstrated that:

Disabling signature verification completely destroys JWT security

"alg": "none" must never be allowed

Expiration checks must be enforced

Privilege claims in payloads cannot be trusted

Simple JSON editing can lead to full admin compromise

The secure server correctly rejected:

Forged tokens

Unsigned tokens

Expired tokens

The vulnerable server accepted them all.
Recommendations

Enforce mandatory signature verification.

Reject all tokens using "alg": "none".

Validate all critical claims:

exp

nbf

iat

iss

aud

Do not trust privilege claims inside JWT payloads.

Use strong signing keys (HS256 or RS256).

Use short token lifetimes.

Log and monitor invalid token attempts.
Conclusion

This lab demonstrates the catastrophic impact of misconfigured JWT validation.

The forged administrative token required:

No secret key

No cryptographic operations

No advanced tools

Just modifying JSON and re-encoding it.

To prevent compromise, servers must enforce:

Strict algorithm whitelisting

Mandatory signature verification

Expiration validation

Complete rejection of "alg": "none"

Failure to do so results in total system compromise.
