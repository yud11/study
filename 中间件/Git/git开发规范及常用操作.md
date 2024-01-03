## Git开发规范

### GitFlow 工作流

![git流](https://i0.wp.com/lanziani.com/slides/gitflow/images/gitflow_1.png)

GitFlow通常包含五种类型的分支：Master分支、Develop分支、Feature分支、Release分支以及Hotfix分支。

- **Master分支**：主干分支，也是正式发布版本的分支，其包含可以部署到生产环境中的代码，通常情况下只允许其他分支将代码合入，不允许向Master分支直接提交代码（对应生产环境）。
- **Develop分支**：开发分支，用来集成测试最新合入的开发成果，包含要发布到下一个Release的代码（对应开发环境）。
- **Feature分支**：特性分支，通常从Develop分支拉出，每个新特性的开发对应一个特性分支，用于开发人员提交代码并进行自测。自测完成后，会将Feature分支的代码合并至Develop分支，进入下一个Release。
- **Release分支**：发布分支，发布新版本时，基于Develop分支创建，发布完成后，合并到Master和Develop分支（对应集成测试环境）。
- **Hot** **fix分支**：热修复分支，生产环境发现新Bug时创建的临时分支，问题验证通过后，合并到Master和Develop分支。

通常开发过程中新特性的开发过程如下：

从Develop分支拉取一条Feature分支，开发团队在Feature分支上进行新功能开发；开发完成后，将Feature分支合入到Develop分支，并进行开发环境的验证；开发环境验证完成，从Develop分支拉取一条Release分支，到测试环境进行SIT/UAT测试；测试无问题后，可将Develop分支合入Master分支，待发版时，直接将Master分支代码部署到生产环境。

## Git 开发常用操作及原理

### 两种常见的团队协作方式

#### 团队内部协作（clone-pull-push）

![image-20230719230607088](https://nie-note.oss-cn-nanjing.aliyuncs.com/typora/image-20230719230607088.png)



#### 跨团队协作（fork-pullRequest）

![image-20230719230646537](https://nie-note.oss-cn-nanjing.aliyuncs.com/typora/image-20230719230646537.png)

### 从零开始的项目开发

```shell
# 1> 初始化一个 git 仓库
git init 
# 2> 创建一个远程仓库
# 3> 将本地仓库与远程仓库相关联
git remote add <远程仓库别名> <远程仓库地址>
# 4> 本地生成密钥，使用rsa算法，指定大小4096 添加签名niemiao@gmail.com
ssh-keygen -t rsa -b 4096 -C "niemiao@gmail.com"
# 5> 将公钥添加到远程仓库
# 6> 追踪本地文件，查看状态
git add .
git status
# 7> 提交
git commit -m 'first commit'
# 8> 推送本地分支到远程仓库
git branch dev
git checkout dev
git push <远程仓库别名> <本地分支>:<远程分支>
# 9> 关联本地分支和远程分支
git branch --set-upstream-to <远程分支名称> 或者  git branch -u demo/test2
# 10> 解除本地分支与远程分支的关联
git branch --unset-upstream
# 10.1 查看本地分支与远程分支的关联关系
git branch -vv
# 11> 获取远程分支列表
git fetch 
# 12> 拉取远程分支到本地
git pull <远程仓库别名> 远程分支
```



### 几种误操作的处理方式

#### 1> 暂存状态的文件如何取消暂存

```shell
# 暂存文件
git add <filename>
# 取消暂存
git rm --cached <filename>
```

#### 2> 提交状态的文件如何取消（版本穿梭）

```shell
# 退回到上一个版本，但不取消暂存状态
git reset --soft HEAD~ 
# 退回到上一个版本，取消暂存状态
git reset --soft HEAD~ 
# 退回到指定版本
git reset --hard <版本 hash 值> 
# 回退至上 n+1 步，从 0 开始
git reset HEAD@{n}
```

#### 3> 远程分支同步覆盖本地分支

```shell
# 先拉取远程分支
git fetch origin
# 切换到待同步的分支
git checkout dev
# 覆盖分支
git reset --hard origin/dev
```

#### 4> 本地分支覆盖远程分支（禁止使用！）

```shell
git push -f origin dev
```

#### 5> push 操作如何回撤

##### 方式 1：reset（不推荐）

<blockquote alt="danger"><p>reset 操作本质上是通过移动 HEAD 指针的指向来回退版本，这会导致本地的版本低于远程分支的版本，如果修改后再次提交会报错，需要使用强制推送，本身是一个不安全的操作</p></blockquote>

##### 方式 2：revert （推荐）

```shell
git revert -n <版本 hash 值>
git push
```

<blockquote alt="success"><p>revert 操作是将指定版本的提交内容恢复成为改动前的内容，并将本次回撤作为一次改动提交，本质上是在现有版本的基础上进行版本升级，不需要使用强制推送，是一个相对安全的操作</p></blockquote>

#### 6> 单独 commit 提交 cherry-pick

> 单独提交是为了应对紧急 bug 修复而产生的，由于提交单位都是每一次 commit，所以 cherry-pick 多个提交的时候，一定要按照顺序提交，减少代码冲突的情况

![image-20230720112023551](https://nie-note.oss-cn-nanjing.aliyuncs.com/typora/image-20230720112023551.png)

##### Cherrypick 常见冲突场景

> git 是采用三路合并算法
>
> - 由于提交`3rd`是基于提交`2nd`创建的，因此`3rd`中保留了`2rd`中对文件的操作记录；
> - 如果直接将`3rd`拼接到`initial commit`后面，就会失去提交`2nd`的记录；
> - 由此提交`3rd`就不能通过提交`2nd`找到公共提交节点`init`，这就会导致合并失败

![image-20230720114030772](https://nie-note.oss-cn-nanjing.aliyuncs.com/typora/image-20230720114030772.png)

### merge 与 rebase

#### merge 与三路合并算法

##### 常见误区

* git merge 是用时间先后决定merge结果的，后面会覆盖前面的
* git merge 的时候如果相同文件的同一行不相同，就一定会发生冲突

##### 二路合并

> 当两个分支上同一个文件的同一行出现的差异，就会发生冲突

![image-20230720113024940](https://nie-note.oss-cn-nanjing.aliyuncs.com/typora/image-20230720113024940.png)

##### 三路合并

> 三路合并就是先找出一个基准，然后以基准为Base 进行合并，如果2个文件相对基准(base)都发生了改变 那git 就报冲突，然后让你人工决断。否则，git将取相对于基准(base)变化的那个为最终结果

![image-20230720113632444](https://nie-note.oss-cn-nanjing.aliyuncs.com/typora/image-20230720113632444.png)

![image-20230720113647433](https://nie-note.oss-cn-nanjing.aliyuncs.com/typora/image-20230720113647433.png)

#### rebase

#####  merge 原理图

![Dec-30-2020-merge-example](https://waynerv.com/posts/git-rebase-intro/Dec-30-2020-merge-example.gif)

#####  merge 提交历史

```shell
* 6fa5484 (HEAD -> master, feature) commit F
*   875906b Merge branch 'master' into feature
|\  
| | 5b05585 commit E
| | f5b0fc0 commit D
* * d017dff commit C
* * 9df916f commit B
|/  
* cb932a6 commit A
```

#####  rebase 原理图

![Dec-30-2020-rebase-example](https://waynerv.com/posts/git-rebase-intro/Dec-30-2020-rebase-example.gif)

##### rebase 提交历史

``` shell
* 74199ce (HEAD -> master, feature) commit F
* e7c7111 commit E
* d9623b0 commit D
* 73deeed commit C
* c50221f commit B
* ef13725 commit A
```

