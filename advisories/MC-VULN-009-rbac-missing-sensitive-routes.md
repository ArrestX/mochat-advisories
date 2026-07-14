# MoChat — 敏感后台接口缺失 RBAC 权限校验 (MC-VULN-009)

| Field | Value |
|-------|-------|
| **Vendor** | mochat-cloud |
| **Product** | MoChat |
| **Version** | master（Hyperf 2.2） |
| **Type** | Improper Authorization |
| **CWE** | CWE-862 |
| **Authentication** | Any authenticated user |

## Summary

Routes like `PUT /dashboard/workEmployee/synEmployee` only use DashboardAuthMiddleware. Low-priv user receives HTTP 200 (live verified).

## Root cause

Opt-in RBAC model; no default deny.

## Proof of Concept

```http
PUT /dashboard/workEmployee/synEmployee HTTP/1.1
Host: 127.0.0.1:9501
Authorization: Bearer LOW_PRIV_TOKEN
Content-Type: application/json
Connection: close

{}
```

**Expected:** HTTP 200 (not 403) for non-privileged JWT

## Impact

Employee sync, corp bind, password reset, etc. without menu permission.

## Remediation

Global PermissionMiddleware on `/dashboard/*` except explicit public list.

## References

- Audit: `SECURITY_AUDIT_REPORT.md` §MC-VULN-009
- PoC: `POC_VERIFICATION_REPORT.md`
