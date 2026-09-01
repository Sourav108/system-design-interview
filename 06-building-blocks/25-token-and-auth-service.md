# Building Block 25: Token & Authentication Service (OAuth2 & JWT)

> **Module**: `06-building-blocks`
> **Reusability Category**: Ingress / Storage / Messaging / Coordination / Reliability / Telemetry

---

## 1. Problem
Monolithic server-side session stores do not scale across global multi-datacenter microservices. Querying a central database on every HTTP request to verify user credentials creates a massive single point of failure.

## 2. Requirements
- **Functional Requirements**: Provide robust, highly available, low-latency primitives for client and microservice workloads.
- **Non-Functional Requirements**:
  - **Availability**: $99.999\%$ uptime SLA (Five Nines).
  - **Latency**: $\text{p}99 < 5\text{ms}$ in-memory / $\text{p}99 < 50\text{ms}$ network hop.
  - **Scalability**: Linearly scale to millions of concurrent requests.

## 3. Why It Exists
Stateless token-based authentication (JSON Web Tokens / PASETO) allows microservices to verify user identity and permissions cryptographically using public keys without querying a central database.

## 4. Mental Model
A passport issued by a federal government containing an official cryptographic watermark; border guards at any airport verify authenticity instantly without calling the capital.

## 5. Core Concepts
OAuth 2.0 Framework, OpenID Connect (OIDC), JWT Structure (Header, Payload, Signature), Asymmetric Signing (RS256 / Ed25519), PASETO, Access Tokens vs Refresh Tokens, Token Revocation List (Blacklist).

## 6. Architecture
```mermaid
flowchart TD
    Client[Client App] -->|1. POST /login {user, pass}| AuthSvc[Auth Service (Identity Provider)]
    AuthSvc -->|2. Verify Credentials| UserDB[(User Database)]
    AuthSvc -->|3. Signs JWT with Private Key: RS256| Signer[Asymmetric Crypto Signer]
    AuthSvc -- 4. Returns: Access Token (15m) + Refresh Token (7d) --> Client

    Client -->|5. GET /api/orders (Authorization: Bearer JWT)| OrderSvc[Order Microservice]
    OrderSvc -->|6. Cryptographically Validates Signature via Public Key (0 DB Queries! ✅)| FastAuth[Local JWT Verification]
```

## 7. Request/Data Flow
1. Client logs in with credentials. 2. Auth service authenticates user. 3. Generates short-lived Access Token (15m, signed with RS256 private key) + long-lived Refresh Token (7d, stored in DB). 4. Client sends Access Token in `Authorization: Bearer` header. 5. Microservices verify signature locally using Auth Service's cached public key (JWKS).

## 8. Data Model
JWT Payload: `sub (user_id)`, `roles (ARRAY)`, `iss (auth.example.com)`, `iat (INT64)`, `exp (INT64)`. Refresh Token: `token_hash`, `user_id`, `expires_at`, `revoked (BOOL)`.

## 9. API Design
OIDC Endpoints: `POST /oauth/token`, `GET /.well-known/jwks.json` (JSON Web Key Set), `POST /oauth/revoke`.

## 10. Algorithms
RSA-SHA256 (RS256) / Ed25519 asymmetric cryptographic signatures, Refresh Token Rotation algorithm.

## 11. Scaling
Authentication verification scales infinitely across microservices because verification requires zero network I/O; Auth Service scales on Redis/Postgres.

## 12. Partitioning
Refresh tokens partitioned by `hash(user_id)` in database.

## 13. Replication
Auth server database replicated multi-AZ with read replicas.

## 14. Consistency
Cryptographic verification is immediate; revoked token blacklists synchronized via distributed Redis cache with short TTL.

## 15. Failure Scenarios
Compromised private signing key (requires JWKS key rotation), Stolen access token before expiration.

## 16. Recovery
Short access token lifespan (15 minutes); automated JWKS key rotation; Redis token revocation blacklist for emergency logout.

## 17. Observability
Token Issuance Rate, JWT Verification Latency (p99 < 0.1ms), Token Validation Failures, Refresh Token Churn.

## 18. Security
Never store sensitive data (passwords, PII) in unencrypted JWT claims; use `HttpOnly`, `Secure`, `SameSite=Strict` cookies for web storage.

## 19. Performance
Local in-memory caching of public keys (JWKS) allows microservices to verify millions of requests/sec with zero network calls.

## 20. Trade-offs
Stateless JWT (Zero DB queries, cannot be revoked instantly without blacklist) vs Stateful Sessions (Instant revocation, database query overhead).

## 21. When to Use
Microservice API security, mobile/web user authentication, third-party developer API access (OAuth2 scopes).

## 22. When NOT to Use
Low-scale monoliths where simple secure server-side sessions in Redis suffice.

## 23. Implementation Strategy
Build an OAuth2.0 Authorization Server in Java using Spring Security 6 with RSA asymmetric key generation and JWT token filters.

## 24. Practical Exercise
Create a Spring Boot resource server verifying JWT tokens against a public JWKS endpoint, benchmark signature verification across 100,000 requests.

## 25. Interview Questions
1. Explain the internal structure of a JSON Web Token (Header, Payload, Signature). 2. How does asymmetric RS256 signing allow microservices to verify tokens without calling the Auth service? 3. How do you implement instant token revocation with stateless JWTs?

## 26. Common Mistakes
Storing JWT access tokens in browser `localStorage`, making them completely vulnerable to Cross-Site Scripting (XSS) token theft.

## 27. Quick Revision
Token Auth = Asymmetric RS256 signature -> Microservices verify locally via Public JWKS -> 0 DB lookups -> Short 15m expiry.

## 28. Related Building Blocks
`BB-19` (API Gateway), `BB-10` (Cache), `BB-22` (Config Service)

## 29. Related Case Studies
All Case Studies (`CS-01` to `CS-20`)
