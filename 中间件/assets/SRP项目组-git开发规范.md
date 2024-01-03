# SRP项目组-git开发规范

## 分支定义

### master：

> 分支命名：master、master-huian、master-yuanxin 
>
> 由于 srp 项目存在多个客户特性分支的开发，所以单个 matser 分支的 git 模型不适用于 SRP 项目

* master 分支为标品主分支，属于上线分支，不允许在 master 分支上直接进行代码修改和提交，master 分支只允许 develop 分支、 hotfix 分支和 release 分支进行合并
* master-huian 为汇安定制化 master 分支，特性与默认 master 分支一致
* master-yuanxin 为圆信定制化 master 分支，特性与默认 master 分支一致
* <font color='red'>不允许直接修复bug</font>

### develop:

> 分支命名：develop、develop-huian、develop-yuanxin

* develop 为开发分支，始终保持最新完成以及bug修复后的代码
* develop-huian 为汇安定制化 develop 分支
* develop-yuanxin 为圆信定制化 develop 分支
* <font color='red'>bug修复</font>
  * 无正在开发 feature 或者 release 分支：直接修复
  * 无正在开发 feature 或者 release 分支但影响线上版本：走 hotfix 分支修复流程
  * 有正在开发 feature 或者 release 分支：走 feature 或 release 分支修复流程

### feature:

> 分支命名： feature/[特性]_[模块名称] 例 ：feature/huian-approval_center，标品不带特性前缀

* 开发新功能时，以相应特性的 develop 分支为基础创建 feature 分支
  * 标品需求：从 develop 分支创建 feature 分支，开发完成后合并到 develop、develop-huian、develop-yuanxin
  * 特性需求：从特性的 develop 分支（如develop-huian）上创建 feature 分支，开发完成后合并到对应特性 develop 分支上
  * 标品和特性：从特性的 develop 分支（如develop-huian）上创建 feature 分支，同时创建标品 feature 分支，记录标品功能清单和 commit，开发过程中持续 pick 标品功能到标品的 feature 分支上，开发完成后特性 feature 分支和标品 feature 分支合并到对应的 develop 分支上
* <font color='red'>bug修复</font>
  * 新功能bug：只在 feature 分支上直接修复，不影响其他分支
  * 历史遗留bug：在 develop 分支上进行修复，并 merge 到当前开发的各个 feature 分支上

### release:

> 分支命名：release/[特性]_[版本号] 例：release/huian_V1.9.1

- release 为预上线分支，阶段性功能开发完成，发布提测阶段，首先会将对应的 feature 分支合并到 develop 分支上，再从 develop 分支创建 release 分支
- 提测阶段产生的 bug 直接在 release 分支上进行修复，并记录 bug 修复提交清单，提测阶段结束后，release 分支需要合并到 develop 分支以及 matser 分支上
- <font color='red'>若提测阶段出现新需求开发，且出现bug</font>
  * 历史遗留 bug：需要根据记录的 bug 修复提交清单，同步 pick 到 develop 分支上，在一个合适的时间点，将 develop 分支合并到新功能 feature 分支上
  * 新功能 bug：只在当前 release 分支上进行修改，提测结束后 merge 分支到 develop 分支上
- 提测结束后，release 分支不允许再提交代码，release 分支需要合并到 master 分支以及 develop 分支上，以 master 分支为基准做上线准备

### hotfix：

> 分支命名：hotfix/[bug号]_[日期]，例 hotfix/SRP-3355_20221121

* hotfix 分支用来处理已经上线的系统产生的 bug，不适用与大范围的需求变更所产生的 bug（大范围的需求变更产生的 bug 需要按照新功能开发的流程来处理）
* <font color='red'>bug修复</font>：
  * 以 master 分支为基线，创建本地 hotfix 分支
  * 修复完成后，提交本地分支到远程 hotfix 分支
  * 在 bitbucket 上提交分支合并请求，hotfix 分支合并到 master 分支以及 develop 分支上

## commit 规范

> 基于 [Angular Git Commit Guidelines](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fangular%2Fangular.js%2Fblob%2Fmaster%2FDEVELOPERS.md%23-git-commit-guidelines) 规范，制定更加适合 SRP 项目的规范
>
> 具体格式为 [type]:[title]，例如：feat:新增审批中心XXX功能

### type

| 名称     | 说明                                   |
| -------- | -------------------------------------- |
| feat     | 添加新特性                             |
| fix      | 修复bug                                |
| test     | 增减测试范围代码                       |
| refactor | 优化、重构代码、增减依赖项、增减工具类 |
| docs     | 仅仅修改了文档                         |

