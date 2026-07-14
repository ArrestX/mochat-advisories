# MoChat — 上传目录作为静态资源根目录公开访问 (MC-VULN-016)

| Field | Value |
|-------|-------|
| **Vendor** | mochat-cloud |
| **Product** | MoChat |
| **Version** | master（Hyperf 2.2） |
| **Type** | Exposure of Sensitive Information to an Unauthorized Actor |
| **CWE** | CWE-548 |
| **Authentication** | None (read) |

## Summary

Swoole `document_root` points to `storage/upload/` with `enable_static_handler=true`.

## Root cause

`config/autoload/server.php` static handler config.

## Proof of Concept

```http
GET /static/2026/0622/2025/178213112371526a3929b3ae9de.svg HTTP/1.1
Host: 127.0.0.1:9501
Connection: close
```

**Expected:** HTTP 200 file body without authentication

## Impact

Amplifies upload/XSS impact; direct exfiltration of uploaded assets.

## Remediation

Serve uploads via Nginx with auth or separate non-executable domain.

## References

- Audit: `SECURITY_AUDIT_REPORT.md` §MC-VULN-016
- PoC: `POC_VERIFICATION_REPORT.md`
