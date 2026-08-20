# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

收车请求返回成功后，行程和班次已经完成，但车辆仍处于出车状态且里程没有更新，后续排班无法使用该车。请修复收车事务中的跨实体状态一致性。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/t63-qa-05
- 仓库地址：https://github.com/zhanglei10281852-gif/t63-qa-05.git
- parent SHA：b946fc072aa5257f822de191aa632a37516dd4f2

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/t63-qa-05.git bug-repro
cd bug-repro
git checkout --detach b946fc072aa5257f822de191aa632a37516dd4f2
go test ./internal/httpapi -run TestCompleteDispatchFlowIsTransactionalAndIdempotent -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/httpapi -run TestCompleteDispatchFlowIsTransactionalAndIdempotent -count=1
--- FAIL: TestCompleteDispatchFlowIsTransactionalAndIdempotent (1.43s)
    server_test.go:266: vehicle not restored after return: map[capacity_kg:9000 created_at:2026-08-18T00:00:00Z depot_code:H-01 id:vehicle_000001 inspection_due_at:2027-08-18T00:00:00Z odometer_km:100 plate_number:沪A00301 status:on_duty updated_at:2026-08-18T00:00:00Z vehicle_type:compactor version:2]
FAIL
FAIL	sanitation-operations/internal/httpapi	1.434s
FAIL

```

stderr：

```text
warning: internal/httpapi/server_test.go has type 100755, expected 100644
warning: internal/httpapi/server_test.go has type 100755, expected 100644

```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/httpapi -run TestCompleteDispatchFlowIsTransactionalAndIdempotent -count=1
--- FAIL: TestCompleteDispatchFlowIsTransactionalAndIdempotent (1.54s)
    server_test.go:266: vehicle not restored after return: map[capacity_kg:9000 created_at:2026-08-18T00:00:00Z depot_code:H-01 id:vehicle_000001 inspection_due_at:2027-08-18T00:00:00Z odometer_km:100 plate_number:沪A00301 status:on_duty updated_at:2026-08-18T00:00:00Z vehicle_type:compactor version:2]
FAIL
FAIL	sanitation-operations/internal/httpapi	1.758s
FAIL

```

stderr：

```text
warning: internal/httpapi/server_test.go has type 100755, expected 100644
warning: internal/httpapi/server_test.go has type 100755, expected 100644

```

## 通过条件

在触发条件下，定向测试 TestCompleteDispatchFlowIsTransactionalAndIdempotent 应通过，相关包、全量测试、竞态测试和构建检查均通过；回退 gold 唯一修复后定向测试重新失败。
