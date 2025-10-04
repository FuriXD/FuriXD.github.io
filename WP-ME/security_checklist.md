# Security & Hardening Checklist

## Authentication & Authorization

- [x] **Google OAuth Integration** (@firebase_auth or @auth0)
  - Implement OAuth 2.0 flow with PKCE
  - Validate ID tokens server-side
  - Store minimal user data from OAuth providers
  
- [x] **Phone OTP Verification** (@twilio)
  - Rate limit OTP requests: 3 per phone per hour
  - OTP expiration: 5 minutes
  - Hash OTP codes before storage
  - Implement attempt limits (max 5 attempts)
  
- [x] **Session Management**
  - JWT tokens with 1-hour expiration
  - Refresh tokens with 30-day expiration, stored securely
  - HttpOnly, Secure, SameSite=Strict cookie flags
  - Implement token rotation on refresh
  - Store session metadata (IP, user-agent) for anomaly detection
  
- [x] **Multi-Factor Authentication (MFA)**
  - Enforce MFA for admin accounts
  - Optional MFA for user accounts
  - TOTP-based (RFC 6238) using @google-authenticator or similar

## OWASP Top 10 Mitigations

### A01: Broken Access Control
- [x] Implement role-based access control (RBAC)
- [x] Validate user ownership before returning itinerary data
- [x] Use UUIDs for resource IDs (not sequential integers)
- [x] Implement least-privilege principle for all API endpoints

### A02: Cryptographic Failures
- [x] Enforce HTTPS everywhere (HSTS with max-age=31536000)
- [x] TLS 1.3 minimum
- [x] Encrypt sensitive data at rest (AES-256)
- [x] Use bcrypt (cost factor 12) for password hashing
- [x] Never log passwords, tokens, or sensitive PII

### A03: Injection
- [x] Use parameterized queries for all database operations
- [x] Validate and sanitize all user inputs
- [x] Use ORM with prepared statements (Prisma, TypeORM, SQLAlchemy)
- [x] Implement input validation on both client and server
- [x] Escape HTML output to prevent XSS

### A04: Insecure Design
- [x] Implement rate limiting on authentication endpoints (5 attempts/15 min)
- [x] Rate limit itinerary creation (10 per hour per user)
- [x] Implement CAPTCHA on signup (@google-recaptcha)
- [x] Design with security requirements from start

### A05: Security Misconfiguration
- [x] Remove default credentials
- [x] Disable directory listings
- [x] Configure strict CSP headers (see below)
- [x] Keep all dependencies updated (automated via Dependabot)
- [x] Minimize attack surface (disable unused services)

### A06: Vulnerable Components
- [x] Automated dependency scanning (Snyk, GitHub Dependabot)
- [x] Pin dependency versions in package.json/requirements.txt
- [x] Regularly update critical dependencies
- [x] Use Software Composition Analysis (SCA) tools

### A07: Identification & Authentication Failures
- [x] Implement account lockout after failed attempts
- [x] Use strong password requirements (min 12 chars, complexity)
- [x] Prevent credential stuffing with rate limits
- [x] Implement secure password reset flow with expiring tokens

### A08: Software & Data Integrity Failures
- [x] Use subresource integrity (SRI) for CDN resources
- [x] Sign CI/CD artifacts
- [x] Implement code signing for deployments
- [x] Verify third-party library integrity

### A09: Security Logging & Monitoring Failures
- [x] Log all authentication attempts (success/failure)
- [x] Log authorization failures
- [x] Monitor for unusual patterns (CloudWatch, DataDog)
- [x] Alert on critical security events
- [x] Retain logs for 90+ days

### A10: Server-Side Request Forgery (SSRF)
- [x] Validate and sanitize URLs before fetching
- [x] Use allowlists for external API calls
- [x] Disable URL redirects in HTTP clients
- [x] Network segmentation for internal services

## HTTP Security Headers

```
Content-Security-Policy: default-src 'self'; script-src 'self' https://api.mapbox.com https://www.google.com/recaptcha/; style-src 'self' 'unsafe-inline' https://api.mapbox.com; img-src 'self' data: https:; font-src 'self' data:; connect-src 'self' https://api.travelplanner.com https://api.mapbox.com; frame-src 'self' https://www.google.com/recaptcha/; object-src 'none'; base-uri 'self'; form-action 'self'; frame-ancestors 'none'; upgrade-insecure-requests;

Strict-Transport-Security: max-age=31536000; includeSubDomains; preload

X-Frame-Options: DENY

X-Content-Type-Options: nosniff

X-XSS-Protection: 1; mode=block

Referrer-Policy: strict-origin-when-cross-origin

Permissions-Policy: geolocation=(), microphone=(), camera=(), payment=(self)
```

## Rate Limiting

| Endpoint | Limit | Window |
|----------|-------|--------|
| POST /auth/login | 5 requests | 15 minutes per IP |
| POST /auth/phone/send | 3 requests | 1 hour per phone |
| POST /auth/phone/verify | 5 requests | 15 minutes per session |
| POST /itineraries | 10 requests | 1 hour per user |
| GET /itineraries/:id | 100 requests | 15 minutes per user |
| GET /map/geojson | 50 requests | 15 minutes per user |

Implementation: Use @upstash/ratelimit with Redis or in-memory store

## Input Validation

- [x] Validate all inputs against schema (Zod, Joi, Yup)
- [x] Sanitize HTML inputs (DOMPurify)
- [x] Validate email format (RFC 5322)
- [x] Validate phone numbers (E.164 format)
- [x] Limit string lengths (max 500 chars for most fields)
- [x] Validate dates (ISO 8601 format)
- [x] Whitelist allowed characters for search queries
- [x] Validate numeric ranges (budget: 0-1000000, days: 1-30)

## Cloud Storage Security (@aws_s3, @gcp_storage, @azure_blob)

### AWS S3
- [x] Enable bucket encryption at rest (AES-256 or KMS)
- [x] Block public access by default
- [x] Use signed URLs for sharing (expiration: 1 hour)
- [x] Enable versioning for critical data
- [x] Configure lifecycle policies (delete after 90 days for temp files)
- [x] Enable access logging to separate bucket
- [x] Use IAM roles with least privilege
- [x] Enable MFA Delete for production buckets
- [x] Use VPC endpoints to avoid internet exposure

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyInsecureTransport",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": "arn:aws:s3:::travel-itineraries/*",
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    }
  ]
}
```

### GCP Cloud Storage
- [x] Enable uniform bucket-level access
- [x] Use customer-managed encryption keys (CMEK)
- [x] Configure retention policies
- [x] Enable audit logging
- [x] Use service accounts with minimal scopes

### Azure Blob Storage
- [x] Enable encryption at rest
- [x] Use Shared Access Signatures (SAS) with minimal permissions
- [x] Enable soft delete (7-day retention)
- [x] Configure firewall rules

## Database Security

- [x] Encrypt connections (SSL/TLS)
- [x] Encrypt data at rest
- [x] Use read replicas for analytics queries
- [x] Implement automated backups (daily, 30-day retention)
- [x] Test backup restoration quarterly
- [x] Use connection pooling (max 20 connections)
- [x] Implement query timeouts (30 seconds)
- [x] Audit database access logs
- [x] Rotate database credentials quarterly
- [x] Use database parameter groups to enforce security settings

## API Security

- [x] Implement API versioning (/v1, /v2)
- [x] Use API keys for third-party integrations
- [x] Rotate API keys every 90 days
- [x] Implement request signing for sensitive operations
- [x] Validate Content-Type headers
- [x] Limit request payload size (max 1MB for POST bodies)
- [x] Implement CORS with strict origin whitelist
- [x] Use GraphQL query complexity analysis (if applicable)

## Third-Party Service Security

### @mapbox
- [x] Restrict API keys by domain/IP
- [x] Use separate keys for dev/staging/prod
- [x] Monitor usage for anomalies
- [x] Set spending limits

### @stripe (if used for payments)
- [x] Use Stripe.js for PCI compliance
- [x] Never store card details
- [x] Use Stripe webhooks with signature verification
- [x] Test in Stripe test mode before prod
- [x] Enable Radar for fraud detection

### @twilio
- [x] Implement phone number verification
- [x] Rate limit SMS sends
- [x] Monitor for toll fraud
- [x] Use geographic permissions

### @google OAuth
- [x] Validate ID tokens server-side
- [x] Verify token audience and issuer
- [x] Check token expiration
- [x] Use state parameter to prevent CSRF

## Secure CI/CD

- [x] Store secrets in environment variables (never in code)
- [x] Use secret management (AWS Secrets Manager, HashiCorp Vault)
- [x] Implement branch protection rules
- [x] Require code reviews for main branch
- [x] Run automated security scans on PRs (Snyk, SonarQube)
- [x] Use separate environments (dev/staging/prod)
- [x] Implement blue-green deployments
- [x] Automated rollback on deployment failures
- [x] Use Docker image scanning (Trivy, Clair)
- [x] Sign container images

## Monitoring & Incident Response

- [x] Set up real-time security alerts
- [x] Monitor for anomalous traffic patterns
- [x] Implement automated threat response
- [x] Create incident response playbook
- [x] Conduct security drills quarterly
- [x] Maintain security contacts list
- [x] Document breach notification procedures

## Compliance

- [x] **GDPR** (if serving EU users)
  - Implement data export functionality
  - Allow account deletion
  - Maintain data processing agreements
  - Implement cookie consent
  
- [x] **CCPA** (if serving California users)
  - Implement "Do Not Sell" option
  - Provide data deletion
  
- [x] **PCI DSS** (if handling payments)
  - Use Stripe or similar PCI-compliant processor
  - Never store CVV
  - Implement network segmentation

## Security Testing

- [x] Automated security tests in CI/CD
- [x] Penetration testing annually
- [x] Bug bounty program (HackerOne, Bugcrowd)
- [x] Regular vulnerability assessments
- [x] OWASP ZAP automated scans
- [x] Static Application Security Testing (SAST)
- [x] Dynamic Application Security Testing (DAST)

## Encryption Standards

- [x] Use AES-256-GCM for data at rest
- [x] Use TLS 1.3 for data in transit
- [x] Implement key rotation (90 days)
- [x] Store encryption keys in HSM or KMS
- [x] Never hardcode secrets or keys

## User Privacy

- [x] Minimize data collection (privacy by design)
- [x] Anonymize analytics data
- [x] Implement data retention policies
- [x] Provide privacy policy and terms
- [x] Obtain explicit consent for tracking
- [x] Honor Do Not Track (DNT) headers
- [x] Implement right to be forgotten

## Backup & Disaster Recovery

- [x] Automated daily backups
- [x] Store backups in different region
- [x] Encrypt backup data
- [x] Test restoration monthly
- [x] Document recovery procedures (RTO: 4 hours, RPO: 1 hour)
- [x] Maintain offsite backup copies

## Network Security

- [x] Use VPC with private subnets
- [x] Implement network segmentation
- [x] Use security groups with minimal ports
- [x] Enable DDoS protection (AWS Shield, Cloudflare)
- [x] Use Web Application Firewall (WAF)
- [x] Implement IP whitelisting for admin access
- [x] Use VPN for internal tool access

## Dependency Management

```bash
# Node.js
npm audit fix
npm outdated

# Python
pip-audit
safety check

# Regular updates
npm update
pip install --upgrade -r requirements.txt
```

## Security Contacts

- Security Team: security@travelplanner.com
- Bug Bounty: https://hackerone.com/travelplanner
- Responsible Disclosure: 90-day disclosure policy

## Automated Security Checks

```yaml
# GitHub Actions example
name: Security Scan
on: [push, pull_request]
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run Snyk
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      - name: Run OWASP ZAP
        uses: zaproxy/action-baseline@v0.7.0
```

---

**Review Frequency**: Quarterly security audits  
**Last Updated**: October 4, 2025  
**Next Review**: January 4, 2026
