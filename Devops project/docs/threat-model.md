# Section 2.2: Threat Modelling and Risk Assessment

**Student Name / ID:** M D M H Dissanayaka (IT24101909)  
**Module:** IE3142 - DevOps Security  
**Target Application:** OWASP Juice Shop (Node.js / Express / Angular / SQLite Stack)  

---

## 1. Executive Overview & Architecture Assumptions

This threat model evaluates the core architecture of **OWASP Juice Shop** using the **STRIDE** methodology. The system consists of three primary layers:
1. **Frontend Layer:** Single Page Application (SPA) built with Angular.
2. **Backend API Layer:** RESTful API services built with Node.js and Express.
3. **Database Layer:** SQLite database managed via Sequelize ORM.

---

## 2. 3x3 Risk Assessment Matrix

The risk levels are calculated by evaluating the **Likelihood** (probability of occurrence) against the **Impact** (business/security damage):

| Likelihood \ Impact | Low Impact | Medium Impact | High Impact |
| :--- | :--- | :--- | :--- |
| **High Likelihood** | Medium Risk | High Risk | **Critical Risk** |
| **Medium Likelihood** | Low Risk | Medium Risk | **High Risk** |
| **Low Likelihood** | Low Risk | Low Risk | **Medium Risk** |

---

## 3. Identified STRIDE Threats & Risk Analysis

### Threat 1: Session Token Manipulation & Spoofing (Spoofing)
- **STRIDE Category:** Spoofing
- **Description:** An attacker crafts or modifies JWT session tokens or forges headers to impersonate legitimate users or admin accounts.
- **Likelihood:** Medium | **Impact:** High | **Risk Rating:** **High Risk**
- **Justification:** Without strict cryptographic JWT signature verification and cookie flags, attackers can hijack user sessions and bypass authentication.

### Threat 2: SQL Injection via Search & Auth Endpoints (Tampering)
- **STRIDE Category:** Tampering
- **Description:** Unsanitized user inputs in search queries or login fields allow attackers to execute arbitrary SQL commands, altering or leaking database records.
- **Likelihood:** High | **Impact:** High | **Risk Rating:** **Critical Risk**
- **Justification:** SQL Injection allows complete data tampering, unauthorized authentication bypass, and potential data destruction across the database.

### Threat 3: Plaintext Credentials & Secret Leakage (Information Disclosure)
- **STRIDE Category:** Information Disclosure
- **Description:** API keys, database connection strings, or JWT secret keys committed directly to source code repositories or exposed via unhandled error logs.
- **Likelihood:** Medium | **Impact:** High | **Risk Rating:** **High Risk**
- **Justification:** Exposed secret keys allow external attackers to mint valid JWTs or access backend cloud resources directly.

### Threat 4: Broken Object Level Authorization / BOLA (Elevation of Privilege)
- **STRIDE Category:** Elevation of Privilege
- **Description:** A standard logged-in user modifies object IDs in API requests (e.g., password reset or profile updates) to alter other users' data or gain admin privileges.
- **Likelihood:** High | **Impact:** High | **Risk Rating:** **Critical Risk**
- **Justification:** Missing route-level access control allows low-privileged users to perform administrative actions across the application.

---

## 4. Threat-to-Control Mapping Table (OWASP Juice Shop Stack)

| Threat ID | STRIDE Category | Identified Threat Description | Risk Rating | Specific Mitigating Security Control | Codebase / Pipeline Location |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **T-01** | **Spoofing** | Session token spoofing and JWT manipulation | High Risk | Cryptographic JWT verification and token sanitization middleware | `lib/insecurity.js` & `routes/login.ts` |
| **T-02** | **Tampering** | SQL Injection in Search and Authentication APIs | Critical Risk | Parameterized queries using Sequelize ORM & strict input validation | `routes/search.ts` & `routes/login.ts` |
| **T-03** | **Information Disclosure** | Hardcoded API keys, JWT secrets, or credentials leakage | High Risk | GitHub Encrypted Secrets, environment variables, & automated Gitleaks scan | `.github/workflows/main.yml` & `config/default.yml` |
| **T-04** | **Elevation of Privilege** | Broken Access Control / BOLA on user endpoints | Critical Risk | Enforced Role-Based Access Control (RBAC) middleware checks | `routes/changePassword.ts` & `lib/insecurity.js` |