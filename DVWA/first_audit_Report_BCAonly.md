### Audit Scope & Rules of Engagement

**Target:** `http://localhost/DVWA/security.php` (DVWA — local instance, Security Level: Low)

**Testing Perspective:** Grey-box (authenticated). Tester has both admin-level and standard-user credentials to assess horizontal and vertical access control boundaries.

**In-Scope:**

* Broken Access Control (BAC)

**Out of Scope:**

* Other client-side and server-side vulnerability classes
* Denial-of-Service (DoS/DDoS) testing
* Server/infrastructure misconfiguration testing

**Rules of Engagement:**

1. No modification of DVWA source code.
2. No destructive actions, such as dropping tables or deleting data.
3. Self-assessment using admin-level and standard-user access.
4. Testing window is unrestricted (local lab).

**Notes:**

1. Vulnerabilities outside the defined scope will be documented as **Observed but Out of Scope** and not pursued further.
2. Any real or sensitive data encountered will be reported immediately and will not be copied, retained, or exfiltrated.




## RECON

### 🔍 Endpoint 1: Password Update Feature
*   **URL:** `http://localhost/DVWA/vulnerabilities/csrf/`
*   **Parameters:** `password_current`, `password_new`, `password_conf`, `Change`, `user_token`
*   **Recon Observation:** The application passes a `user_token` value alongside the password parameters. 
*   **Audit Purpose:** Future audits will test if this token can be removed, replayed, or if a generic **Broken Access Control (BAC)** scenario is possible.

---

### 🔍 Endpoint 2: User Profile Search Interface
*   **URL:** `http://localhost/DVWA/vulnerabilities/sqli/`
*   **Recon Observation:** One input field field that accepts numbers to display user profiles. Entering letters results in a completely blank screen with no data.
*   **Audit Purpose:** Target surface mapped for checking input types and field handling. No exploits were attempted.

---

### 🔍 Endpoint 3: Boolean Identity Validation Portal
*   **URL:** `http://localhost/DVWA/vulnerabilities/sqli_blind/`
*   **Recon Observation:** Input validation endpoint. Submitting `1` returns `"User ID exists in the database."` Submitting `"DfaadF"` returns `"User ID is MISSING from the database."`
*   **Audit Purpose:** Mapped baseline true/false text logic patterns to understand the target application's standard lookup behaviors.

---

### 🔍 Endpoint 4: Directory Information Update Interface
*   **URL:** `http://localhost/DVWA/vulnerabilities/authbypass/`
*   **Payload Schema:**
    ```json
    { "id": 3, "first_name": "Hack", "surname": "Me1123" }
    ```
*   **Server Response:** `{"result":"ok"}`
*   **Recon Observation:** Allows updating a specific profile username from a list of accounts. The JSON payload operates without an active authentication token.


*   **Audit Purpose:** This endpoint is noted as a target surface for potential **Broken Access Control (BAC)** testing to check if an attacker can manipulate other records. **Vertical privilege escalation is not possible** here due to the singular account architecture.


