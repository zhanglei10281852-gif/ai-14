# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

ML 工程师查看一条推理运行的详情时，运行主体和 readiness 都正常，readiness 也显示已经绑定输入，可响应中的 `items` 一直为空。请修好运行详情与快照的关联：要返回本次运行的输入，同时不能夹带同工作空间的其他快照。修复过程中不要碰测试文件或断言，不能跳过现有用例，也不能放宽它的隔离检查。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/ai-14
- 仓库地址：https://github.com/zhanglei10281852-gif/ai-14.git
- parent SHA：390384b3d995e70bbad1ce51b822f4c5ad86f622

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/ai-14.git bug-repro
cd bug-repro
git checkout --detach 390384b3d995e70bbad1ce51b822f4c5ad86f622
go test ./internal/service -run ^TestInferenceRunDetailLoadsOnlyItsOwnSnapshots$ -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/service -run ^TestInferenceRunDetailLoadsOnlyItsOwnSnapshots$ -count=1
--- FAIL: TestInferenceRunDetailLoadsOnlyItsOwnSnapshots (0.49s)
    annotation_repo_behavior_test.go:130: detail snapshots = []
FAIL
FAIL	github.com/zhanglei10281852-gif/ai/internal/service	0.498s
FAIL

```

stderr：

```text
(empty)
```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/service -run ^TestInferenceRunDetailLoadsOnlyItsOwnSnapshots$ -count=1
--- FAIL: TestInferenceRunDetailLoadsOnlyItsOwnSnapshots (1.22s)
    annotation_repo_behavior_test.go:130: detail snapshots = []
FAIL
FAIL	github.com/zhanglei10281852-gif/ai/internal/service	1.401s
FAIL

```

stderr：

```text
(empty)
```

## 通过条件

推理运行详情的 items 必须返回该运行实际绑定的输入快照，并严格排除同工作空间内未绑定到本次运行的其他快照；运行主体和 readiness 的既有内容保持正确。定向隔离用例与相关详情查询回归须通过，不得修改测试或弱化关联范围断言。
