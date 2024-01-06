# Git基本操作

**团队使用git协作**

暂定流程，每个人在本地develop上修改（修改时可以创建feature本地分支），提交合并到远程develop分支，管理员 merge 开发分支到 master分支。

master分支:  保持与用户使用的发布版本一致

develop分支:  开发的分支

feature分支:  基于develop创建出的新的 修改分支

release分支：  版本分支

git init：创建本地仓库

**先拉取版本库：**

git clone

**创建自己的分支：**

git branch <branch_name>

**切换到自己的分支：**

git checkout <branch_name>

**提交修改：**

git add .

git commit

**提交分支到远程库：**

git push origin <branch_name>

**合并分支并推送主分支到远程库**

首先从远程库更新主分支

git checkout master

git pull origin master   

将dev分支合并到master上

git merge dev

然后提交到主分支

git push origin master

**如果主分支版本更新**

git checkout <dev>

git merge master       // 将主分支合并到其他分支

**本地项目关联远程库**

origin为本地库的名称，使用不同的名称可以关联不同的远程库

git remote add origin [git@github.co](mailto:git@github.com)m:zhayujie/sharding    // 关联某个远程库

git remote rm origin                     // 删除某个关联

git remote -v                        // 查看所有关联

**合并分支问题: 合并时只有修改了同一文件的同一个地方才会出现冲突，需要手动解决**

git pull == git fetch + git merge  ： 会自动将远程的分支合并到本地，建议分开操作

所以多人开发一个dev分支时，比较好的做法是：

- (推荐): 修改时创建一个feature/bug分支，从远程pull下dev分支(此时一定不会冲突)，然后合并到dev分支（解决冲突），将dev分支push到远程，删除 feature分支。

- 在dev分支上开发， 使用get fetch 获取远程dev分支(此时产生一个 FETCH_HEAD)分支，然后在dev上merge  FETCH_HEAD，解决冲突后提交dev到远程，删除FETCH_HEAD.

git status 查看仓库状态 可以本地仓库是否被修改和具体被修改的文件

git diff 查看被修改的内容

# Git问题

## 本地与github建立链接

先clone代码，再建立连接

git clone <代码路径>

自从 21 年 8 月 13 后不再支持用户名密码的方式验证了，需要创建个人访问令牌

git remote set-url origin https://<your_token>@[github.com/](http://github.com/)<USERNAME>/<REPO>.git

git remote set-url origin https://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@[github.com/](http://github.com/)code-salon/ai-chat-service.git

## 删除github上面的.idea文件

本地不删除.idea文件，并且将github上面的.idea文件删除的操作步骤如下

	1. git rm -r --cached .idea 删除git cache里面的文件
	2. git commit -m " 删除多余文件夹 .idea"
	3. git push -u origin master push到github上



# Git命令

## git常用的六个命令

1、push命令；2、pull命令；3、commit命令；4、add命令；5、checkout命令；6、fetch/clone命令。

Git有很多命令，如果想要完全记住是很难的，一般来说我们只要记住下图中的六个命令即可。

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2022/08/10/kuangstudy29be9135-2680-4491-8808-e35cc865d293.png)
Git 命令清单及个别专用名词的译名如下：

Workspace：工作区

Index / Stage：暂存区

Repository：仓库区（或本地仓库）

Remote：远程仓库

## 一、新建代码库

> 在当前目录新建一个Git代码库

$ git init

> 新建一个目录，将其初始化为Git代码库

$ git init [project-name]

> 下载一个项目和它的整个代码历史

$ git clone [url]

## 二、配置

Git的设置文件为.gitconfig，它可以在用户主目录下（全局配置），也可以在项目目录下（项目配置）。

> 显示当前的Git配置

$ git config —list

> 编辑Git配置文件

$ git config -e [—global]

> 设置提交代码时的用户信息

$ git config [—global] user.name “[name]”

$ git config [—global] user.email “[email address]”

## 三、增加/删除文件

> 添加指定文件到暂存区

$ git add [file1] [file2] …

> 添加指定目录到暂存区，包括子目录

$ git add [dir]

> 添加当前目录的所有文件到暂存区

$ git add .

> 添加每个变化前，都会要求确认
>
> 对于同一个文件的多处变化，可以实现分次提交

$ git add -p

> 删除工作区文件，并且将这次删除放入暂存区

$ git rm [file1] [file2] …

> 停止追踪指定文件，但该文件会保留在工作区

$ git rm —cached [file]

> 改名文件，并且将这个改名放入暂存区

$ git mv [file-original] [file-renamed]

## 四、代码提交

> 提交暂存区到仓库区

$ git commit -m [message]

> 提交暂存区的指定文件到仓库区

$ git commit [file1] [file2] … -m [message]

> 提交工作区自上次commit之后的变化，直接到仓库区

$ git commit -a

> 提交时显示所有diff信息

$ git commit -v

> 使用一次新的commit，替代上一次提交
>
> 如果代码没有任何新变化，则用来改写上一次commit的提交信息

$ git commit —amend -m [message]

> 重做上一次commit，并包括指定文件的新变化

$ git commit —amend [file1] [file2] …

## 五、分支

> 列出所有本地分支

$ git branch

> 列出所有远程分支

$ git branch -r

> 列出所有本地分支和远程分支

$ git branch -a

> 新建一个分支，但依然停留在当前分支

$ git branch [branch-name]

> 新建一个分支，并切换到该分支

$ git checkout -b [branch]

> 新建一个分支，指向指定commit

$ git branch [branch] [commit]

> 新建一个分支，与指定的远程分支建立追踪关系

$ git branch —track [branch] [remote-branch]

> 切换到指定分支，并更新工作区

$ git checkout [branch-name]

> 切换到上一个分支

$ git checkout -

> 建立追踪关系，在现有分支与指定的远程分支之间

$ git branch —set-upstream [branch] [remote-branch]

> 合并指定分支到当前分支

$ git merge [branch]

> 选择一个commit，合并进当前分支

$ git cherry-pick [commit]

> 删除分支

$ git branch -d [branch-name]

> 删除远程分支

$ git push origin —delete [branch-name]

$ git branch -dr [remote/branch]

## 六、标签

> 列出所有tag

$ git tag

> 新建一个tag在当前commit

$ git tag [tag]

> 新建一个tag在指定commit

$ git tag [tag] [commit]

> 删除本地tag

$ git tag -d [tag]

> 删除远程tag

$ git push origin :refs/tags/[tagName]

> 查看tag信息

$ git show [tag]

> 提交指定tag

$ git push [remote] [tag]

> 提交所有tag

$ git push [remote] —tags

> 新建一个分支，指向某个tag

$ git checkout -b [branch] [tag]

## 七、查看信息

> 显示有变更的文件

$ git status

> 显示当前分支的版本历史

$ git log

> 显示commit历史，以及每次commit发生变更的文件

$ git log —stat

> 搜索提交历史，根据关键词

$ git log -S [keyword]

> 显示某个commit之后的所有变动，每个commit占据一行

$ git log [tag] HEAD —pretty=format:%s

> 显示某个commit之后的所有变动，其”提交说明”必须符合搜索条件

$ git log [tag] HEAD —grep feature

> 显示某个文件的版本历史，包括文件改名

$ git log —follow [file]

$ git whatchanged [file]

> 显示指定文件相关的每一次diff

$ git log -p [file]

> 显示过去5次提交

$ git log -5 —pretty —oneline

> 显示所有提交过的用户，按提交次数排序

$ git shortlog -sn

> 显示指定文件是什么人在什么时间修改过

$ git blame [file]

> 显示暂存区和工作区的差异

$ git diff

> 显示暂存区和上一个commit的差异

$ git diff —cached [file]

> 显示工作区与当前分支最新commit之间的差异

$ git diff HEAD

> 显示两次提交之间的差异

$ git diff [first-branch]…[second-branch]

> 显示今天你写了多少行代码

$ git diff —shortstat “@{0 day ago}”

> 显示某次提交的元数据和内容变化

$ git show [commit]

> 显示某次提交发生变化的文件

$ git show —name-only [commit]

> 显示某次提交时，某个文件的内容

$ git show [commit]:[filename]

> 显示当前分支的最近几次提交

$ git reflog

## 八、远程同步

> 下载远程仓库的所有变动

$ git fetch [remote]

> 显示所有远程仓库

$ git remote -v

> 显示某个远程仓库的信息

$ git remote show [remote]

> 增加一个新的远程仓库，并命名

$ git remote add [shortname] [url]

> 取回远程仓库的变化，并与本地分支合并

$ git pull [remote] [branch]

> 上传本地指定分支到远程仓库

$ git push [remote] [branch]

> 强行推送当前分支到远程仓库，即使有冲突

$ git push [remote] —force

> 推送所有分支到远程仓库

$ git push [remote] —all

## 九、撤销

> 恢复暂存区的指定文件到工作区

$ git checkout [file]

> 恢复某个commit的指定文件到暂存区和工作区

$ git checkout [commit] [file]

> 恢复暂存区的所有文件到工作区

$ git checkout .

> 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变

$ git reset [file]

> 重置暂存区与工作区，与上一次commit保持一致

$ git reset —hard

> 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变

$ git reset [commit]

> 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致

$ git reset —hard [commit]

> 重置当前HEAD为指定commit，但保持暂存区和工作区不变

$ git reset —keep [commit]

> 新建一个commit，用来撤销指定commit
>
> 后者的所有变化都将被前者抵消，并且应用到当前分支

$ git revert [commit]

> 暂时将未提交的变化移除，稍后再移入

$ git stash

$ git stash pop

## 十、其他

> 生成一个可供发布的压缩包

$ git archive

# Github相关博客

Git Flow

http://www.cnblogs.com/cnblogsfans/p/5075073.html

idea中Git分支相关使用

https://blog.csdn.net/andyzhaojianhui/article/details/78037738

https://www.cnblogs.com/vcmq/p/9966644.html



