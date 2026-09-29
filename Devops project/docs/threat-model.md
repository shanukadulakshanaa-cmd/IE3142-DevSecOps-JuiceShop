# Section 2.2: Threat Modelling & Risk Assessment

## 1. System Architecture & Trust Boundaries Overview
Based on the provided OWASP Juice Shop system architecture diagram, the system consists of a client-side interface and containerized backend components running on a Docker Host[cite: 3, 5]:
- **Client Side / External:** User Web Browser issuing HTTP requests to `localhost:3000`[cite: 3, 5].
- **Containerized Application (Docker Host):** Single container hosting the Angular Frontend, Node.js / Express Backend API, and Application Data / Data Storage layer[cite: 3, 5].

### Identified Trust Boundaries:
1. **Trust Boundary 1 (External to Container):** Network boundary between the untrusted public Web Browser and the Angular Frontend running on port 3000[cite: 1, 3, 5].
2. **Trust Boundary 2 (Internal Frontend to API):** Boundary between the Angular Frontend and Node.js/Express Backend API handling API requests and responses[cite: 1, 3, 5].
3. **Trust Boundary 3 (Internal API to Persistence):** Boundary separating the Express API service from the Application Data Storage layer during read/write queries[cite: 1, 3, 5].

---

## 2. STRIDE Threat Identification (6 Key Threats)
Using the STRIDE framework, six threats were identified based on the architectural data flows[cite: 1, 5]:

| Threat ID | STRIDE Category | Threat Name | Description | Target Component |
| :--- | :--- | :--- | :--- | :--- |
| **TH-01** | **Spoofing (S)** | JWT Session Token Spoofing | An attacker steals or forges weak JWT tokens to impersonate legitimate users and bypass authentication[cite: 1]. | Express Backend API[cite: 1, 3, 5] |
| **TH-02** | **Tampering (T)** | SQL Injection via Login Form | Malicious SQL inputs injected through login forms modify query execution to tamper with database records[cite: 1]. | Application Data Storage[cite: 1, 3, 5] |
| **TH-03** | **Repudiation (R)** | Insufficient Security Logging | Absence of audit logs for critical API actions allows users to perform unauthorized transactions without accountability[cite: 1]. | Express Backend API[cite: 1, 3, 5] |
| **TH-04** | **Information Disclosure (I)** | Verbose Error & Stack Trace Leakage | Detailed 500 error responses disclose internal directory structures and framework stack traces[cite: 1]. | Angular Frontend & Express API[cite: 1, 3, 5] |
| **TH-05** | **Denial of Service (D)** | API Endpoint Rate Limit Exhaustion | Lack of rate limiting on login/registration endpoints allows automated bot flooding to crash the API container[cite: 1, 3, 5]. | Express Backend API[cite: 1, 3, 5] |
| **TH-06** | **Elevation of Privilege (E)** | Broken Object Level Authorization (BOLA) | Manipulating request payload parameters (e.g., `user_id` or `role`) escalates privileges to administrator level[cite: 1]. | Express Backend API[cite: 1, 3, 5] |

---

## 3. Risk Assessment & Rating Matrix
Each threat is rated using Likelihood and Impact metrics (Low / Medium / High)[cite: 1]:

| Threat ID | Threat Name | Likelihood | Impact | Overall Risk | Risk Justification |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **TH-01** | JWT Token Spoofing | Medium | High | **High** | Weak key signatures allow complete account takeover across all user sessions[cite: 1]. |
| **TH-02** | SQL Injection | High | High | **High** | Publicly accessible login input fields allow direct database dump or destruction[cite: 1]. |
| **TH-03** | Insufficient Logging | High | Low | **Medium** | Hinders post-incident forensics, making active attacks harder to trace[cite: 1]. |
| **TH-04** | Error Data Leakage | High | Low | **Medium** | Exposes system reconnaissance data without causing immediate direct system damage[cite: 1]. |
| **TH-05** | API DoS Exhaustion | Medium | Medium | **Medium** | Unrestricted requests can consume host resources, resulting in temporary service downtime[cite: 1]. |
| **TH-06** | Privilege Escalation (BOLA) | Low | High | **Medium** | Requires valid active user session, but successful exploitation yields full admin control[cite: 1]. |

---

## 4. Threat-to-Control Mapping
Corresponding security controls and their location within the codebase / deployment pipeline[cite: 1]:

| Threat ID | Threat Name | Applied Security Control | Control Location in Codebase / Pipeline |
| :--- | :--- | :--- | :--- |
| **TH-01** | JWT Token Spoofing | Strong JWT signing algorithms and environment secret storage[cite: 1, 2]. | Configuration: `config/jwt.js` & GitHub Secrets[cite: 1, 2] |
| **TH-02** | SQL Injection | Parameterized queries and ORM input validation[cite: 1]. | Backend Controller: `routes/login.js`[cite: 1] |
| **TH-03** | Insufficient Logging | Centralized audit logging middleware for state-changing requests[cite: 1]. | Middleware: `middleware/logger.js`[cite: 1] |
| **TH-04** | Error Data Leakage | Custom global error handler returning sanitized HTTP 500 responses[cite: 1]. | Application Core: `app.js` & SAST Pipeline Gate[cite: 1] |
| **TH-05** | API DoS Exhaustion | API Rate-limiting middleware (`express-rate-limit`)[cite: 1]. | Server Configuration: `server.js`[cite: 1] |
| **TH-06** | Privilege Escalation (BOLA) | Server-side Role-Based Access Control (RBAC) middleware verification[cite: 1]. | Middleware: `middleware/authmiddleware.js`[cite: 1] |