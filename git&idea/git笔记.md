# Git基本操作

**团队使用git协作**

暂定流程，每个人在本地develop上修改（修改时可以创建feature本地分支），提交合并到远程develop分支，管理员 merge 开发分支到 master分支。

master分支:  保持与用户使用的发布版本一致

develop分支:  开发的分支

feature分支:  基于develop创建出的新的 修改分支

release分支：  版本分支

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



# Git问题

## 本地与github建立链接

先clone代码，再建立连接

git clone <代码路径>

自从 21 年 8 月 13 后不再支持用户名密码的方式验证了，需要创建个人访问令牌

git remote set-url origin https://<your_token>@[github.com/](http://github.com/)<USERNAME>/<REPO>.git

## 删除github上面的.idea文件

本地不删除.idea文件，并且将github上面的.idea文件删除的操作步骤如下

	1. git rm -r --cached .idea 删除git cache里面的文件
	2. git commit -m " 删除多余文件夹 .idea"
	3. git push -u origin master push到github上



# Github相关博客

Git Flow

http://www.cnblogs.com/cnblogsfans/p/5075073.html

idea中Git分支相关使用

https://blog.csdn.net/andyzhaojianhui/article/details/78037738

https://www.cnblogs.com/vcmq/p/9966644.html



