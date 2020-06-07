# 分支模型和发布路径

## 目标

针对产品迭代诉求，设计一种合理的代码发布路径和集成方式。

## 产品迭代诉求

* 支持多条线并行，如：核心功能重构、固定双周迭代发布等等。
* 同一功能支持多人协作开发。
* 已经完成开发的功能可以选择性发布。

## 详细设计

**功能开发**

![分支模型 - 发布路径（功能开发）](./assets/img/branching-model-01.png)

**hotfix**

![分支模型 - 发布路径（hotfix）](./assets/img/branching-model-02.png)

### 分支介绍

| 分支 | 描述 |
| :-- | :-- |
| `feature/${featureName}` | 功能分支，用于整合参与功能开发的各个工程师的代码，`featureName` 可以描述独立发布的大功能，也可以是某个具体的发布日期。 |
| `develop/${featureName}/${fe}` | 基于 `feature/${featureName}` 的 `开发者` 分支，建议将每天新增的分支内容提 Merge Request 到 `feature/${featureName}` 分支，高频提交可以减少解决冲突和 CR 的工作量。 |
| `test/${featureName}` | 功能对应的测试分支，也可以同时整合多个 `feature/${featureName}` 分支进行测试，视具体测试计划而定。 |
| `develop/hotfix` | 专门用于 bug 修复的开发分支。 |
| `test/hotfix` | 专门用于 bug 修复的测试分支。 |
| `release` | 发布分支，始终体现为已完成测试的最新发布代码。 |
| `master` | 稳定分支，`release` 分支发布后线上回归没问题，将 release 分支合入 master 分支。 |

### 测试发布

上面提到的 `测试分支`、`发布分支`，并非直接拿某个分支去测试或者发布，而是通过 CI 基于某个分支来自动编译生成要测试、发布的资源。

| 触发 CI 的动作 | 描述 |
| :-- | :-- |
| `test/${featureName}`、`test/hotfix` 分支更新 | 生成测试包：代码检查、单元测试、编译打包，始终只拿最新版的包去测试。 |
| `release` 分支更新 | 生成发布包：代码检查、单元测试、编译打包，始终只拿最新版本的包去发布。 |

**对系统的要求**

* CI 系统产出的包有分支和版本标识。
* 发布系统有自动回滚功能。

### 改进空间

* 发布和测试的包最好是同一个，拿完成测试的包去发布，test 分支和 release 分支虽然代码一致，但也可能因外部依赖的差异导致产出不一样。
* 整体流程的管控，如：CI 失败或测试不通过可以阻断 test 分支向 release 分支的入库流程。

### 其他

**保持关键分支 commit 的干净**

* 给 GitLab 项目启用 `Fast-forward merge`，默认情况下 Merge Request 入库时会产生一条额外的 commit，启用 `Fast-forward merge` 后将不会产生。
* 提 Merge Request 时选择 `Squash and merge` 选项，可以将多条 commit 合并成一条。

> 皮成，2020.05.23