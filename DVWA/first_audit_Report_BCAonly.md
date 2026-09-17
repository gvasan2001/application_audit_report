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

### 🔍 Endpoint 3: Directory Information Update Interface
*   **URL:** `http://localhost/DVWA/vulnerabilities/authbypass/`
*   **Payload Schema:**
    ```json
    { "id": 3, "first_name": "Hack", "surname": "Me1123" }
    ```
*   **Server Response:** `{"result":"ok"}`
*   **Recon Observation:** Allows updating a specific profile username from a list of accounts. The JSON payload operates without an active authentication token.


*   **Audit Purpose:** This endpoint is noted as a target surface for potential **Broken Access Control (BAC)** testing to check if an attacker can manipulate other records. **Vertical privilege escalation is not possible** here due to the singular account architecture.



### aduit reprot

### 🔍 Endpoint 1: Password Update Feature
*   **URL:** `http://localhost/DVWA/vulnerabilities/csrf/`
*   **Parameters:** `password_current`, `password_new`, `password_conf`, `Change`, `user_token`
*   **Recon Observation:** The application passes a `user_token` value alongside the password parameters. 
*   **Audit Purpose:** Future audits will test if this token can be removed, replayed, or if a generic **Broken Access Control (BAC)** scenario is possible.

---




## Endpoint 2: User Profile Search Interface
**URL:** http://localhost/DVWA/vulnerabilities/sqli/
**Mechanism Verified:** Session Validation & Authorization Enforcement 

### 🗒 Test 1: Session Deletion & Unauthenticated Access

* **Test Type:** Session Token Removal / Authorization Enforcement
* **Status:** ✅ Pass (Working as expected)

### Description

This test verifies whether the endpoint securely restricts access when the session identifier is completely removed from the HTTP request headers. 

### Steps to Reproduce

1. **Log in** to the application using valid user credentials.
2. Navigate to the **SQL Injection (SQLi)** vulnerability module interface.
3. Input a valid user ID into the search field and click **Submit**.
4. Observe the successful retrieval and display of the corresponding user profile data.
5. Intercept the outbound HTTP request using a proxy tool such as **Burp Suite**.
6. Completely delete the session identifiers (e.g., Cookie: PHPSESSID=...) from the HTTP request headers and forward the modified request.

### Expected & Observed Result

* **Observed Result:** The backend application rejects the modified request, returning an HTTP **302 Found** status code. The browser is instantly redirected to the main login portal.
* **Conclusion:** The backend server reliably validates session states and enforces access controls when authentication tokens are entirely missing.

### 🗒 Test 2: Cookie Replay After Session Termination

* **Test Type:** Session Management & Authorization Verification
* **Status:** ✅ Pass (Working as expected)

### Description

This test evaluates the risk of session replay attacks by checking if a previously used session token remains active in the backend environment after a formal user logout. 

### Steps to Reproduce

1. **Log in** to the application using valid user credentials.
2. Navigate to the **SQL Injection (SQLi)** module interface.
3. Open the browser's **Developer Tools** panel, navigate to the storage settings, and copy the active PHPSESSID cookie value.
4. Log out of the web application normally to terminate the session.
5. Capture a new request to the target endpoint using **Burp Suite**, then inject the copied, post-logout PHPSESSID value into the headers before forwarding the packet.

### Expected & Observed Result

* **Observed Result:** The application refuses to process the invalid session token. The server issues an HTTP **302 Found** status code, redirecting the unauthenticated request straight back to the login page.
* **Conclusion:** The application properly invalidates session identifiers server-side upon logout, nullifying the potential for token-replay exploits.

### 🗒 Test 3: HTTP Method Variation (GET to POST)

* **Test Type:** HTTP Method Tampering & Session Persistence
* **Status:** ✅ Pass (Working as expected)

### Description

This test confirms that session controls and validation filters remain actively enforced even if an attacker alters the structural HTTP request method from a standard GET to a POST. 

### Steps to Reproduce

1. **Log in** to the application using valid user credentials.
2. Navigate to the **SQL Injection (SQLi)** module interface.
3. Input a valid user ID, click **Submit**, and intercept the transaction using **Burp Suite**.
4. Use the proxy tool to structurally change the request method from a GET method to a POST method.
5. Terminate the active login session or manipulate the parameter tracking, and forward the request to test authentication consistency.

### Expected & Observed Result

* **Observed Result:** The backend infrastructure maintains its security boundaries across alternating request types. The server handles the method switch securely, returns an HTTP **302 Found** status code, and forces a redirection to the authentication page.
* **Conclusion:** Changing the structural HTTP method does not bypass the underlying session verification mechanisms. The backend successfully protects the endpoint uniformly.
