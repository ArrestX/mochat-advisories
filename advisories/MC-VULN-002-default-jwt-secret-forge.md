# MoChat — 默认 JWT 密钥可伪造身份令牌 (MC-VULN-002)

| Field | Value |
|-------|-------|
| **Vendor** | mochat-cloud |
| **Product** | MoChat |
| **Version** | master（Hyperf 2.2） |
| **Type** | Use of Hard-coded Password |
| **CWE** | CWE-798 / CWE-287 |
| **Authentication** | None (forge token offline / via helper script) |

## Summary

`.env.example` ships `SIMPLE_JWT_SECRET=3S6ybWbSy&23fFeq8`. MoChat uses qbhy/simple-jwt with PasswordHashEncrypter (bcrypt of md5(header.payload+secret)). Attacker with the default secret can mint valid dashboard JWT for any `uid`.

## Root cause

Hardcoded secrets in `.env.example` / `auth.php` sidebar fallback; weak deployment hygiene.

## Proof of Concept

```
# Forge token (run on host with PHP or poc_verify/mochat_jwt_forge.py)
# php /tmp/forge_jwt.php  → paste token below

GET /dashboard/user/loginShow HTTP/1.1
Host: 127.0.0.1:9501
Authorization: Bearer FORGED_JWT_HERE
Connection: close
```

**Expected:** HTTP 200 with `userId` matching forged `uid` payload (verified uid=1 on live env)

## Impact

Unauthenticated full dashboard API access when default secret is unchanged.

## Remediation

Generate random secrets at install; remove code defaults; rotate on first boot.

## References

- Audit: `SECURITY_AUDIT_REPORT.md` §MC-VULN-002
- PoC: `POC_VERIFICATION_REPORT.md`
