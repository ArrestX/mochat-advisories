# MoChat Security Advisories

Public advisory / exploit write-ups for VulDB CVE submissions (MC-VULN-* series).

## 漏洞列表

| ID | 文件 | 严重度 | 需登录 |
|----|------|--------|--------|
| MC-VULN-001 | [MC-VULN-001-password-reset-idor.md](./MC-VULN-001-password-reset-idor.md) | Critical | Any authenticated dashboard user (no RBAC) |
| MC-VULN-002 | [MC-VULN-002-default-jwt-secret-forge.md](./MC-VULN-002-default-jwt-secret-forge.md) | Critical | None (forge token offline / via helper script) |
| MC-VULN-003 | [MC-VULN-003-unauth-txt-verify-upload.md](./MC-VULN-003-unauth-txt-verify-upload.md) | High | None |
| MC-VULN-004 | [MC-VULN-004-status-update-idor.md](./MC-VULN-004-status-update-idor.md) | High | Any authenticated dashboard user |
| MC-VULN-005 | [MC-VULN-005-unauth-wx-jssdk-config.md](./MC-VULN-005-unauth-wx-jssdk-config.md) | High | None |
| MC-VULN-006 | [MC-VULN-006-work-fission-h5-unauth.md](./MC-VULN-006-work-fission-h5-unauth.md) | High | None |
| MC-VULN-007 | [MC-VULN-007-file-download-ssrf.md](./MC-VULN-007-file-download-ssrf.md) | High | Authenticated admin (store fission/welcome with http URL) |
| MC-VULN-009 | [MC-VULN-009-rbac-missing-sensitive-routes.md](./MC-VULN-009-rbac-missing-sensitive-routes.md) | High | Any authenticated user |
| MC-VULN-010 | [MC-VULN-010-svg-stored-xss-upload.md](./MC-VULN-010-svg-stored-xss-upload.md) | Medium | Authenticated dashboard user |
| MC-VULN-011 | [MC-VULN-011-sidebar-upload-path-control.md](./MC-VULN-011-sidebar-upload-path-control.md) | Medium | Sidebar JWT |
| MC-VULN-016 | [MC-VULN-016-public-static-upload-root.md](./MC-VULN-016-public-static-upload-root.md) | Medium | None (read) |

## Push to GitHub

```bash
cd mochat
git add SECURITY_AUDIT_REPORT.md POC_VERIFICATION_REPORT.md advisories/
git commit -m "Add security advisories"
git push
```

## Auto-fill poc_verify queue

```bash
cd ../poc_verify
python sync_*_queue.py
python vuldb_submit.py prepare
```
