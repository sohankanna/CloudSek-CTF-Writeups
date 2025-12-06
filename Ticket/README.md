# Ticket  - Writeup

**Category:** Web / Mobile / OSINT  
**Technique:** OSINT (BeVigil) & JWT Signature Forgery  

---

## 1. Challenge Description
The challenge presents a scenario involving unusual activity in a customer portal. We are provided with a specific Android package name (`com.strikebank.netbanking`) and a hint pointing towards **BeVigil** (a mobile security search engine).

**Objective:** Investigate the mobile app information, find a way into the associated web portal, and escalate privileges to retrieve the hidden flag. <br>

<img width="476" height="556" alt="Screenshot 2025-12-06 102105" src="https://github.com/user-attachments/assets/dcf5e1c9-54a1-4cb1-aae7-5e1956df0172" />


---

## 2. OSINT & Leak Discovery
Instead of manually downloading and decompiling an APK file (which can be time-consuming), I leveraged **Open Source Intelligence (OSINT)**. The challenge description explicitly mentioned **BeVigil**, so I searched for the package name `com.strikebank.netbanking` on their platform.

This search returned a generated security report for the application. Upon analyzing the report, specifically under the **"Strings" / "Secrets"** section (sourced from `resources/res/values/strings.xml`), I discovered sensitive hardcoded information.

<img width="1525" height="443" alt="Screenshot 2025-12-06 102338" src="https://github.com/user-attachments/assets/9969e969-12a7-4616-a4e6-e84f3ead377e" />


**Findings from the Report:**
*   **Target URL:** `http://15.206.47.5.nip.io:8443/login.php`
*   **Internal Username:** `tuhin1729`
*   **Internal Password:** `123456`
*   **Encoded Secret:** `c3RyIWszyjRua0AxMDA5JXN1cDNyIXMzY3IzNw==`

---

## 3. Access & Protocol Analysis
Using the credentials found in the mobile app report (`tuhin1729` / `123456`), I navigated to the discovered URL and successfully logged in.

<img width="1384" height="588" alt="Screenshot 2025-12-06 102332" src="https://github.com/user-attachments/assets/ae0f9a26-11c1-4329-8569-2ac04b63e854" />


The dashboard indicated that I was logged in as a standard user (`tuhin1729`). To capture the flag, I needed **Admin** privileges.

I inspected the browser's storage and network requests. The application was using a **JSON Web Token (JWT)** stored in a cookie/local storage to handle session authentication. This token contained my user identity and role.

---

## 4. Exploitation: JWT Forgery
To escalate privileges, I needed to modify the JWT. However, JWTs are signed. If I simply changed the payload, the signature would become invalid, and the server would reject the token. To forge a valid signature, I needed the **Secret Key**.

### Step A: Decoding the Secret
I revisited the `Encoded Secret` found in the BeVigil report. The string ended with `==`, suggesting Base64 encoding.

*   **Input:** `c3RyIWszyjRua0AxMDA5JXN1cDNyIXMzY3IzNw==`
*   **Decoded Output:** `str!k3b4nk@1009%sup3r!s3cr37`

### Step B: Modifying the Payload
I took the original valid token assigned to `tuhin1729` and decoded it using [jwt.io](https://jwt.io).

**Original Payload:**
```json
{
  "username": "tuhin1729",
  "exp": 1764997256
}
```

I modified the claims to impersonate an administrator. I added a `role` field (a common convention in Access Control) and changed the username.

**Forged Payload:**
```json
{
  "username": "admin",
  "role": "admin",
  "exp": 1999999999
}
```

### Step C: Signing the Token
Using the decoded secret key (`str!k3b4nk@1009%sup3r!s3cr37`) and the **HS256** algorithm, I generated a new valid signature for my malicious payload.

<img width="1496" height="686" alt="Screenshot 2025-12-06 110706" src="https://github.com/user-attachments/assets/86ec3be3-2fd9-4a40-9d1c-5fe9549fa190" />


---

## 5. Result
I captured the request to the dashboard, replaced the session cookie with my forged Admin JWT, and forwarded the request.

The server validated the signature using the secret key (which matched), processed the `admin` role in the payload, and granted access to the administrative dashboard containing the flag.
<img width="1406" height="697" alt="Screenshot 2025-12-06 110248" src="https://github.com/user-attachments/assets/8a1ad0e5-7148-492d-998f-21d7b825d5c3" />


**Flag:** `CL0UdSEk_ReSeArCH_tEaM_CTF_2025{ccf62117a030691b1ac7013fca4fb685}`
