# API Penetration Testing Report — VAmPI (Vulnerable API)

**Prepared by:** Adeola Odunlade  
**Date:** November 21, 2025  
**Target Application:** VAmPI (Mock API via Postman)  
**Test Environment:** Windows 11 + Postman Mock Server + Burp Suite Community  
**Testing Type:** API Design Security Assessment & Mock Interaction Testing  

---

## Executive Summary

This report documents a security assessment conducted on the VAmPI API, a purposely vulnerable API provided in the form of an OpenAPI/Swagger specification.

---

## What I Learned

As part of the Week 1 Cybersafe API Security training, I completed the API Security Fundamentals course from APISec University, which covered:

- The importance of API security  
- The OWASP API Security Top 10  
- API threat modeling  
- The three pillars of API security: Governance, Monitoring, and Testing  
- The modern API application security technology landscape  

This assessment allowed me to apply those concepts in a hands-on environment. By analyzing the Swagger file, creating a Postman mock server, and testing traffic through Burp Suite, I was able to practically apply theoretical skills such as identifying design-layer vulnerabilities, evaluating access control gaps, and understanding how API weaknesses can be exploited. The project reinforced how API security principles are applied in real API testing workflows.

---

## Scenario

You have been brought in to review a company’s API prior to launch. Your responsibility is to analyze the provided API specification and identify potential security issues before the system is exposed to real users or connected to production databases.

---

## Primary Objectives

- Review the Swagger/OpenAPI file to understand the API’s structure, expected behavior, and security model  
- Identify three (3) security vulnerabilities focusing on:
  - Broken Object Level Authorization (BOLA)  
  - Excessive Data Exposure  
  - Lack of access controls on sensitive endpoints  
- Explain the risks using the OWASP API Security Top 10  
- Recommend realistic fixes or mitigations to improve the API’s security posture before deployment  

Testing was performed by importing the Swagger file into Postman, generating a mock server, and routing all traffic through Burp Suite to observe and validate the API’s behavior. Since the mock server reflects the specification’s documented responses, all findings are based strictly on design-level weaknesses present in the Swagger file.

These design flaws—particularly around authentication, authorization, and data exposure—would pose major security risks if implemented as-is. Addressing these issues during the design phase is crucial to ensuring a secure implementation ahead of the API’s launch.

---

## Scope

- Swagger/OpenAPI YAML file for VAmPI  
- Postman mock server generated from the Swagger file  
- API endpoints defined in the specification  
- Local traffic inspection using Burp Suite  

---

## Boundaries

- Only design-level issues were evaluated  
- Mock server responses were used strictly for verification  
- No destructive actions were performed  

---

## Methodology

The testing approach simulated an API review using safe tools.

### Tools Used

- **Postman** — Import Swagger, create mock server, issue requests  
- **Burp Suite Community** — Proxy traffic, capture HTTP history  
- **Swagger/OpenAPI Specification** — Primary evidence source  

---

## Process Overview

### Download & Review Swagger File

The provided YAML file was reviewed line-by-line, focusing on:

- Authentication requirements  
- Sensitive data fields  
- Path parameters  
- Security definitions  
- Response schemas  

### Import into Postman & Create Mock Server

- Swagger YAML imported into Postman  
- Postman generated all endpoints automatically  
- A mock server was created to emulate API behavior  
- An environment variable `{{base_url}}` was automatically generated  

### Configure Burp Suite

- Burp installed on Windows  
- Proxy listener enabled on `127.0.0.1:8080`  
- Intercept turned OFF  
- Postman configured to route requests through Burp proxy  

### Send Test Requests to Confirm Behavior

Requests such as:

GET /
GET /users/v1
GET /users/v1/_debug
GET /createdb
GET /users/v1/{username}

were sent through Postman, and responses validated through Burp’s HTTP History.

---

# Key Findings

---

## 1. Excessive Data Exposure

**Severity:** High  
**OWASP Mapping:** API3:2023 – Excessive Data Exposure  
**Endpoint:** `GET /users/v1/_debug`

### Description

The `/users/v1/_debug` endpoint returns complete user records, including:

- Plaintext password  
- Admin flag  
- Email address  
- Username  

This is visible directly in the Swagger YAML definitions.

### Business Impact

If implemented:

- Attackers could harvest all user passwords instantly  
- Admin accounts could be compromised  
- Identity theft, privilege escalation, and full system compromise could occur  

### Mitigation

- Remove debug endpoints from production  
- NEVER store or return plaintext passwords  
- Hash passwords using bcrypt or Argon2  
- Implement response filtering (return only required fields)  

---

## 2. Broken Object Level Authorization (BOLA)

**Severity:** High  
**OWASP Mapping:** API1:2023 – Broken Object Level Authorization  

### Affected Endpoints

- `GET /users/v1/{username}`  
- `PUT /users/v1/{username}/email`  
- `DELETE /users/v1/{username}`  

### Description

These endpoints allow direct access to user-specific data based solely on the username path parameter.

The Swagger file does not define authentication or authorization controls.

There is no:

security:

bearerAuth: []

Therefore, any user (or attacker) could potentially request:

GET /users/v1/alice
GET /users/v1/bob

and retrieve user information.

### Business Impact

- Unauthorized access to other users' data  
- Account manipulation  
- User data scraping  
- Full profile takeover in real implementations  

### Mitigation

- Require JWT authentication on all user-specific routes  
- Enforce user ownership checks (“user can only access their own resources”)  
- Restrict DELETE and modification operations to admins only  

---

## 3. Lack of Access Control on Sensitive Endpoints

**Severity:** High  
**OWASP Mapping:** API2:2023 – Broken Authentication / Missing Access Control  

### Affected Endpoints

- `/createdb`  
- `/users/v1`  
- `/users/v1/_debug`  

### Description

The Swagger file defines no authentication requirements for endpoints that perform sensitive operations, including:

- Database initialization  
- Debug data retrieval  
- User listing  

The absence of access control results in sensitive functionality being available to the public.

### Business Impact

If implemented:

- Anyone could wipe or repopulate the database  
- Anyone could retrieve full user lists  
- Attackers could enumerate users  

### Mitigation

- Protect all sensitive endpoints with JWT-based authentication  
- Restrict database-reset endpoints to development environments  
- Implement RBAC (Role-Based Access Control)  
- Explicitly define required permissions in Swagger/OpenAPI  

---

## Recommendations

1. Remove debug endpoints from production environments.  
2. Never expose sensitive fields such as passwords.  
3. Enforce strict object-level authorization on user routes.  
4. Implement JWT authentication globally.  
5. Apply Role-Based Access Control (RBAC) for administrative actions.  
6. Define explicit `security:` blocks in the Swagger specification.  
7. Ensure highly sensitive endpoints are never publicly exposed.

---

## Blockers and Challenges

During initial testing on Kali Linux, Postman required account login and experienced severe lag and freezing during basic actions such as importing the Swagger file and navigating collections.  

To resolve the issue, testing was moved to Windows, where Postman ran smoothly and allowed uninterrupted assessment.

---

## Conclusion

The VAmPI API Swagger specification contains deliberately vulnerable design elements for educational purposes. However, the issues identified closely resemble real-world API security failures, including missing or weak access controls, inadequate object-level authorization, and unnecessary exposure of sensitive data.

Such vulnerabilities can lead to unauthorized account access, data breaches, and full system compromise in production environments.

Addressing these weaknesses at the API design stage—through strong authentication, least-privilege authorization, and secure data handling practices—will significantly improve the API’s security posture prior to deployment.

---
