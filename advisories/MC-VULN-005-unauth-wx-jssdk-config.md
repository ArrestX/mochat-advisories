# MoChat — 未授权企微 JSSDK 配置接口 (MC-VULN-005)

| Field | Value |
|-------|-------|
| **Vendor** | mochat-cloud |
| **Product** | MoChat |
| **Version** | master（Hyperf 2.2） |
| **Type** | Missing Authentication for Critical Function |
| **CWE** | CWE-306 / CWE-200 |
| **Authentication** | None |

## Summary

`GET /sidebar/wxJsSdk/config` has no middleware. Unauthenticated callers reach corp/agent secret loading and WeCom API calls (live: 422/500 business errors, never 401).

## Root cause

`common/Action/Sidebar/WxJSSDK/Config.php` missing `@Middleware`.

## Proof of Concept

```http
GET /sidebar/wxJsSdk/config?corpId=1&agentId=1 HTTP/1.1
Host: 127.0.0.1:9501
Connection: close
```

**Expected:** HTTP 200/422/500 without Authorization — not 401

## Impact

Corp secret abuse; in dev/test `jsapi_ticket` may leak in JSON.

## Remediation

Require SidebarAuthMiddleware or signed H5 token; rate-limit corpId enumeration.

## References

- Audit: `SECURITY_AUDIT_REPORT.md` §MC-VULN-005
- PoC: `POC_VERIFICATION_REPORT.md`
