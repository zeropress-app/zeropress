# Security Policy

This document describes the security policies, threat assumptions, and disclosure process for the ZeroPress project.

ZeroPress is a Cloudflare-based CMS designed with a focus on minimal attack surface, standards-based cryptography, and clear trust boundaries.

---

## Supported Versions

Only the **latest minor release of the current major version** is supported with security updates.

Older versions may continue to function but will not receive security fixes.

| Version        | Supported |
| -------------- | --------- |
| Current (main) | ✅ Yes    |
| Older releases | ❌ No     |

---

## Reporting a Vulnerability

If you discover a security vulnerability, please report it responsibly.

### How to Report

- **Do not** open a public GitHub issue for security vulnerabilities.
- Send a detailed report via one of the following channels:
  - GitHub Security Advisories (preferred)
  - Direct contact with the project maintainer

A good report includes:
- A clear description of the issue
- Steps to reproduce (if applicable)
- Potential impact
- Any relevant logs, screenshots, or proof-of-concept code

### Response Expectations

- You can expect an initial response within **7 days**.
- If the issue is accepted, we will work on a fix and coordinate a responsible disclosure.
- If the issue is declined, we will provide a brief explanation.

We appreciate responsible disclosure and community efforts to keep ZeroPress secure.

---

## Threat Model (Password & Email)

This section outlines the primary threat assumptions related to authentication, passwords, and email-based security flows.

### Assets

The following assets are considered security-sensitive:

- User account credentials
- Password hashes
- Email addresses
- Password reset tokens
- Email verification tokens

### Threat Assumptions

ZeroPress assumes the following realistic threat scenarios:

1. **Database Compromise**  
   An attacker may gain read or write access to application data stores.

2. **Network-Level Attacks**  
   Attackers may observe network traffic but cannot break modern TLS encryption.

3. **Brute-Force & Credential Stuffing**  
   Attackers may attempt online password guessing or reuse leaked credentials.

4. **Email Abuse**  
   Attackers may attempt password reset flooding or email enumeration.

5. **Token Leakage**  
   Reset or verification links may be exposed via logs, browser history, or forwarding.

### Out of Scope

ZeroPress does not attempt to protect against:

- Compromise of the user’s email inbox
- Malware or keyloggers on the user’s device
- Phishing attacks outside the ZeroPress domain
- Platform-level isolation failures

---

## Authentication Assumptions

This section documents explicit trust boundaries and assumptions in ZeroPress’s authentication model.

### Trust Boundaries

Authentication spans the following trust domains:

1. User environment (browser, device, network)
2. ZeroPress application (Cloudflare Workers, application logic, KV/D1)
3. Third-party services (email delivery providers, DNS, TLS infrastructure)

### Assumptions

- All authentication traffic is protected by HTTPS.
- TLS termination is handled by Cloudflare.
- Email is treated as a proof-of-control channel, not a secure storage medium.
- Possession of a valid email inbox implies authorization to verify an account or reset a password.
- Password reset and verification links are bearer tokens.
- Cloudflare Workers provide adequate runtime isolation between tenants.

### Explicit Non-Assumptions

ZeroPress does not assume:

- The user’s device is secure
- The user’s email inbox cannot be compromised
- Email links remain private once delivered

---

## Password Handling

ZeroPress never stores plaintext passwords.

Passwords are hashed using a standards-based key derivation function designed for password storage.

### Password Requirements
- Minimum length: 15 characters
- Passphrases with spaces are allowed and encouraged
- Maximum length: 256 characters (server-side enforced for stability)
- Passwords are checked against a blocklist that includes:
  - known breached passwords
  - commonly used passwords
  - context-specific values such as the service name, username, email address, and obvious derivatives
- Password managers, paste, and browser autofill are allowed
- Passwords are not silently truncated before hashing

ZeroPress does not require composition rules such as mandatory uppercase letters, numbers, or symbols. Users are encouraged to choose long, unique passphrases instead.

### Hashing Algorithm

- **Algorithm**: PBKDF2 (HMAC-SHA256)
- **Iterations**: 100,000
- **Salt**: Cryptographically secure random salt (unique per password). 16 bytes
- **Derived key length**: 256 bits
- **Storage format**: $pbkdf2-sha256$100000$<salt>$<hash>

PBKDF2-HMAC-SHA256 is used because it is supported in the Cloudflare Workers runtime and remains acceptable for compatibility-sensitive deployments.

Hash metadata (algorithm, hash function, iteration count, salt) is stored alongside the derived hash to allow future upgrades.

The current PBKDF2 baseline is a compatibility constraint. The Cloudflare Workers runtime currently imposes limits that constrain higher PBKDF2 work factors inside the Worker itself. For this reason, ZeroPress treats password strength as a layered control that combines:

- strong minimum password length
- breached-password and context-aware blocklist checks
- per-account and per-IP online attack throttling
- single-use recovery tokens
- transparent rehash on successful login when stronger parameters become available

### Preferred Authenticators

ZeroPress prefers phishing-resistant authenticators such as passkeys (WebAuthn) when available.

Passwords remain supported for compatibility, but they are treated as a higher-risk authenticator than passkeys or multi-factor authentication. Enabling MFA or passkeys does not relax password storage requirements, but it does materially reduce risk from credential stuffing, password reuse, and phishing.

---

## Password Reset Security

Password resets are implemented using time-limited, single-use tokens.

### Token Properties

- High-entropy, unguessable values
- Stored only in hashed form
- Automatically expire via TTL
- Manually invalidated immediately after successful use

### Reset Flow

1. User requests a password reset
2. A reset token is generated and stored temporarily
3. A reset link is sent via email
4. The token is validated when a new password is submitted
5. The token is immediately invalidated upon success

Tokens are never reused, and expired or consumed tokens cannot be replayed.

---

## Email Verification

Email ownership is verified using a time-limited verification token.

- Tokens are single-use
- Tokens automatically expire if unused
- Tokens are invalidated immediately after successful verification

---

## Email Delivery

ZeroPress does not operate its own SMTP servers.

Transactional emails (such as password reset and email verification messages) are delivered via third-party email APIs that provide authentication (SPF/DKIM), rate limiting, and abuse prevention.

The email delivery layer is intentionally pluggable to avoid long-term vendor lock-in.

---

## Rate Limiting & Abuse Prevention

To reduce abuse and account enumeration:

- Failed password authentication attempts are rate-limited per account and per IP address
- Progressive backoff begins after 5 failed password attempts within 15 minutes
- A password authenticator may be temporarily locked for 30 minutes after 10 consecutive failed attempts
- No password authenticator is permitted to exceed 100 consecutive failed attempts without requiring an explicit recovery or rebind flow
- Password reset and verification requests are rate-limited
- Password reset issuance is limited per IP address and per email address
- Email verification issuance is limited per IP address and per email address
- Excessive or abusive requests may be silently rejected

These thresholds are baseline defaults and may be tightened over time in response to abuse patterns or updated standards.

---

## Data Minimization

ZeroPress follows a data minimization principle:

- Only required data is stored
- Temporary security tokens are automatically purged
- Sensitive values are never logged

---

## Security Updates

Cryptographic parameters (such as iteration counts) may be updated over time to reflect evolving best practices.

Existing password hashes may be transparently upgraded upon successful authentication.

---

## Responsible Disclosure

We value responsible disclosure and collaboration with the security community.

If you believe you have found a security issue, please report it privately and allow time for investigation and remediation before public disclosure.

