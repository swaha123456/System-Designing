### Step 10: Security

Security infrastructure within a distributed system is engineered to enforce three core pillars: protecting data integrity, rigorously verifying user identities (**Authentication**), and granularly controlling what those identities are permitted to execute (**Authorization**).

---

#### 1. Tokens for Auth

Traditional monolithic authentication architectures relied heavily on stateful server-side sessions stored directly in local application memory or centralized session databases. In highly distributed, multi-region microservices, this pattern creates critical storage and performance bottlenecks. Modern distributed systems universally pivot toward **Stateless Token-Based Authentication**.

- **JSON Web Tokens (JWTs):** Upon a successful user login event, the authentication service constructs a stateless token payload containing essential user metadata (such as an encrypted user ID, permission roles, and an explicit expiration timestamp). This payload is digitally signed using a secure private cryptographic key before being transmitted back to the client.
- **Stateless Validation:** For all subsequent API operations, the client app attaches this cryptographically signed string inside the standard HTTP `Authorization: Bearer <token>` header interface. Because any microservice across the cluster can mathematically verify the token's validity using a matching public key or shared secret, **validation requires zero network database lookups**, allowing the system to scale smoothly.

---

#### 2. SSO & OAuth

These centralized identity protocols eliminate the risky and redundant practice of forcing users to generate and manage separate, isolated username and password credentials for every software service they use.

- **SSO (Single Sign-On):** An enterprise-grade authentication scheme that permits a user to log in once to a single identity provider and seamlessly gain access to multiple independent, disconnected software applications (e.g., logging into your company dashboard to automatically unlock your corporate Gmail, HR dashboard, and Slack workspaces with one secure login step).
- **OAuth (Open Authorization):** An industry-standard framework explicitly designed for safe authorization delegation. It permits third-party applications to securely obtain restricted access tokens to web resources on behalf of a user without ever exposing the user's primary credentials.
  > 🔑 **Analogy:** Think of OAuth like a valet key for a high-end car. The valet key grants the parking attendant limited access to drive the vehicle and park it, but it purposefully locks them out of opening the glove compartment or accessing the trunk. It gives restricted access without handing over your master key.

---

#### 3. Access Control Lists & Rule Engines

Once an infrastructure accurately determines identity context (**Authentication**), the system must evaluate granular permission rules to authorize or deny specific operations (**Authorization**).

- **Access Control Lists (ACL):** A localized register mapped directly to an individual data object or file system resource. It maintains an explicit list detailing exactly which users or process tokens hold read, write, or execute permissions against that specific file.
- **Role-Based Access Control (RBAC):** An optimized authorization standard where system permissions are bound to generic functional organizational roles (e.g., `Admin`, `Editor`, `Compliance_Officer`, `Viewer`) rather than individual human records. Managing users becomes as simple as mapping a person to a pre-configured role group.
- **Rule Engines:** Complex dynamic evaluation engines designed to parse high-velocity data payloads against complex business rules to block malicious or anomalous activity on the fly:
  ```
  IF Transaction_Amount > $10,000
  AND Outbound_Country != Client_Home_Country
  THEN Intercept_Transaction() -> Request_MFA()
  ```

---

#### 4. Encryption

Encryption uses advanced mathematical algorithms to scramble plain text into completely unreadable ciphertext. To guarantee comprehensive data defense, it must be persistently enforced in two distinct operational environments:

| Encryption State | Core Objective                                                              | Primary Protocol / Mechanism                                          | Threat Mitigated                                                                           |
| :--------------- | :-------------------------------------------------------------------------- | :-------------------------------------------------------------------- | :----------------------------------------------------------------------------------------- |
| **In Transit**   | Secures moving data payloads as they travel across network links.           | **HTTPS / TLS** (Transport Layer Security)                            | Prevents packet sniffing, man-in-the-middle exploits, and data leaks on public networks.   |
| **At Rest**      | Secures stationary data residing physically on persistent hardware storage. | **AES-256 Block Ciphers**, Database Transparent Data Encryption (TDE) | Prevents data loss if a bad actor physically steals server hard drives from a data center. |
