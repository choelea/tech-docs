## 帮助命令

```
git help command // eg: git commit help
```
windows 打开默认的浏览器显示帮助内容， mac直接显示

## 配置

```
git config --global setting value
示例：git config --global user.name "Your Name"
示例：git config --global user.email "you@someplace.com"
git config --global --list // 列出全局配置项
```
配置内容保存在当前**用户**目录下的.gitconfig文件中

## 本地命令
### 初始化
方式一：
```
cd projects/
git init git-demo  // projects下面创建文件夹 git-demo, 并初始化； 初始化其实就是在文件夹下面创建了相关内容存放在.git 隐藏文件夹下面
```
方式二：
```
cd projects/
mkdir website
cd website/
git init // 初始化
```
方式三：
大多数的方式，我们从clone一个git 库开始的。
```
git clone 'url'
```

### 查看状态 -> 添加新文件 -> 提交修改 

```
git status // Shows which files have been modified in the working directory vs Git's staging area.
git add file-name  // Adds the new or newly modified file-name to Git's staging area (index).
```
```
git commit -m "A really good commit message" // Commits all files currently in Git's staging area.
```
> 上面的命令只有所有的文件都在staging area在有效。  `git commit -am "A really good commit message"` 可以省掉git add这步，不过新文件必须先add下。

```
git add . // Add all new and newly modified files.
git reset HEAD file-name // Unstage the specified file from stage area. 修改的内容还在
git checkout -- file-name // 回滚本次修改
```
### 检查修改内容
```
git diff // 查看unstage状态下的文件的修改内容，staged的无法查看
```
### 合并到上次提交
```
git commit --amend // 将当前的staged的修改合并到上次commit，并打开编辑器修改commit
git commit --amend -m "New commit message" // 将当前的staged的修改合并到上次commit，并实用新的Message
```
> 使用Interactive Rebasing/squash也可以达到合并的效果，区别就是一个是事先（commit 前）就合并，一个是事后（commit 后）合并。

### 切换分支
> 切换分支前必须保证工作空间是干净的。（没有未提交的修改和新增）

```
git checkout branchename // 切换到branchname分支
```

### 日志

```
git log
git log --oneline --graph --decorate --color
```
### 移除文件
方式一：完全通过git命令
```
git rm debug.log  // remove and stage the change
git commit -m 'remove file debug.log'
```
方式二：
非git 命令删除文件后，运行下面的命令
```
git add -u // git 2.0 以前的版本  stage删除的change
git add file-name // git 2.0 后也可以通过这个命令达到上面的命令的效果
git commit -m 'commit message'
```
### 移动文件

```
git mv index.html web/  // 移动index.html 到web文件夹内。 命令完成后直接进入staging 状态
git commit -m 'move index.html into web folder'
```
### ignore 文件
编辑 .gitignore 文件

## SSH 命令
windows cmd并没有自带ssh命令，我们可以通过git bash命令窗来运行这些命令。
假定在当前用户的目录下：

```
cd .ssh
ssh-keygen -t rsa -C "your email" // 生成SSH Key， 将id_rsa.pub公钥配置到github/bitbucket 等服务器上
ssh -T git@github.com // 验证SSH 配置成功
```

## Git Remote 相关命令
关联一个远程的Repo。 （针对前两种初始化方式，一般情况用不上）
```
git remote add remote-name remote-repository-location // 示例: git remote add origin git@github.com:choelea/keycloak-demo.git
git push -u remote-name branch-name // 示例: git push -u origin master;  The -u parameter is needed the first time you push a branch to the remote.
git remote -v // list the names of all the remote repositories
```
关联远程repo之前需要先在git服务器上创建对应的repo，如果采用的是github，在创建repository后，会有如下的提示：
![git reposority tips](http://img.blog.csdn.net/20170905111310596?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hvZWxlYQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


```
git pull origin master // 下载当前分支远程修改；每次push前都应该先pull
```

## Git Rebase
关于rebase和merge的区别，建议参考：[Merging vs Rebasing](https://www.atlassian.com/git/tutorials/merging-vs-rebasing)
> 一定要看看 'The Golden Rule of Rebasing'这部分.
rebase 和 merge都是应该发生在分支之间的事情，当然在同一个分支上有时也需要。（可能不是最佳实践，git的开发流程一般建议创建单独的feature分支来完成不同的story，避免出现多人在同一个分支上直接commit）
直接在当前分支做rebase

```
git fetch // 这一步必须
git rebase // 将未push的commits 放至remote所有commits之上
// 修复冲突 如果有冲突必须进行修复，完成后，注意提示。一般需要git add 命令来stage下 ，接着git rebase --continue 至到没有任何冲突
```

## Squash
通过Interactive Rebasing来完成当前分支的commits的squash

```
git rebase -i // 列出所有未push的commit，注意是倒序
```
根据提示编辑来达到squash的作用。
![这里写图片描述](http://img.blog.csdn.net/20170828172922198?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hvZWxlYQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
将第二个commit（435d22b）修改为:`pick 435d22b ...` 即将这个commit压缩至上面的commit，并放弃当前的commit message。
> 有些公司会很强调squash。 git估计本地多次提交防止丢失，所以git的commit有可能会很多；而svn的commit就意味着修改可以被其他用户拉取到， 所以svn的每一次commit都要保证系统可以运行，svn的commit会偏少。svn的代码更新时间取决于文件多少和大小；git的代码拉取时间取决于commit的多少。所以。。。是每次提交尽量合理依然很重要，squash/Ineractive Rebasing 很实用。

## Changing a remote's URL
repo换了名字，或者之前是https clone下来的，现在想换成ssh；这些情况都面临着修改远程的URL。
```
git remote set-url origin git@github.com:choelea/tech-docs.git
```
> https的URL一般来说push代码是需要用户明和密码；而ssh的不需要。