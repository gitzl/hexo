---
title: git命令
---

>生成SSH

```bash
git init
# 初始化本地git版本库（创建新仓库）

git config --global user.name "xxx"
# 配置用户名

git config --global user.email "xxx@xxx.com"
# 配置邮件

git config --list
#查看当前配置列表

ssh-keygen -t rsa -C "xxx@xxx.com"
#敲三次回车，在~/.ssh/目录下生产公钥和私钥文件

git clone <url>
# clone远程仓库

```
> 修改、提交、删除

```bash
git add index.php
# 添加index.php文件到缓存区
git add .
# 添加所有改动过的文件到缓存区
git add --all
# 添加所有文件到缓存区

git commit
# 提交缓存区内的文件(回车后需要键入描述:wq保存退出)
git commit -m "描述"
# 提交缓存区内的文件,并提供描述

git commit -am '描述'
# 将add和commit合为一步
git commit --amend -m 'xxx'
# 合并最后一次提交(用于反复修改)

git rm index.php
# 删除index.php文件
git rm --cached index.php
# 将index.php文件移出缓存区,但不删除( -r * 递归目录)
git rm -f 1.html
# 将缓存区中的1.html文件移出并删除
```
> git 回退到指定版本

```bash
git reset --hard  xxx(历史版本号)
git log 查看版本
git push --force  强制推送

```

> 查看

```bash
git status
# 查看当前版本状态（是否修改）

git diff
# 查看所有添加到缓存区的变更(工作区与版本库的区别)
git diff index.php
# 查看工作区文件和库文件区别
git diff --cached
# 查看所有已添加到缓存区,但还未commit的变更(缓存区与版本库的区别)

git log
# 查看提交历史
git log --oneline
# 以简短的方式查看提交日志
git log -3
# 查看最近3次的提交日志
git reflog
# 行为日志,显示所有提交,回滚等..
git ls-files
# 显示缓存区的所有文件

```
> 分支操作

```bash
git pull origin master
# 获取远程分支master并merge到当前分支

git branch
# 显示本地分支
git branch -r  #查看远程分支

git push orgin <branch-name> #推送本地分支到远端

git checkout <branch-name> #切换分支

git branch -a
# 显示所有分支
git checkout 分支名/标签名
# 切换到指定分支或标签

git branch 分支名
# 新建分支
git branch -d 分支名
# 删除本地分支 -D 强制删除

```
> cherry-pick

``` bash
场景：将master 提交中的ab73c32 提取出到分支：1.6.0-release中

git checkout 1.6.0-release 
git chrry-pick ab73c32 
git push orgin 1.6.0-release 

```
> merge 合并代码

```bash
场景：将1.6.0-release代码合并到master中
git checkout master
git merge 1.6.0-release
git status #检测冲突
git push origin master
```
> 合并commit

```
git rebase -i HEAD~3 
pick xxx
s    yyy
s    yyy

#如果合并有冲突，解决冲突  
git add .
git rebase continue

```

> 打包

```
git archive --output xxx/yyy.tar.gz(输出的 目录) v1.4.9(版本)

```
> 远程操作

```bash
git remote add <remote> <url>
# 添加远程版本库
git remote -v
# 查看远程版本库信息
git remote show <remote>
# 查看指定远程版本库信息
git remote remove <remote>
# 删除远程remote链接
git remote rename <old> <new>
# 重命名远程链接名

git pull <remote> <branch>
# 下载代码及快速合并

git push <remote> <branch>
# 上传代码及快速合并

git merge origin master
# 将本地的远端库合并

git fetch origin
# 将远端库获取本地但不合并
```
