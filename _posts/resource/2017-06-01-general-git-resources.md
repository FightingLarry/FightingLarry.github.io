---
layout: post
title: Git 常用资源
category: 资源
tags: Git
keywords: Git
---

## Git协同开发时使用习惯

要在使用git的过程中，有个好的使用习惯，能在不知不觉中给我们来带便利。
要点：保证每次开发的提交，都是基于服务器最新代码节点的提交。
步骤：
- 我们现在使用的开发分支是ChaosKnight-1.5.2，这个分支也是和远程服务器关联的。
- git建分支很方便，我们本地开发的时候，应该基于ChaosKnight-1.5.2分支拉取自己的分支（如：dev_1.5.2，名字随便取)。使用的命令：git branch dev_1.5.2
- 切换到dev_1.5.2分支，切换前确保当前分支没有提交的项。切换命令：git checkout dev_1.5.2
- 到了dev_1.5.2分支，可以随便提交，无论多次git commit -m "msg" 都行。
- 开发完后，想提交，先确保dev_1.5.2分支没有提交的项。切换到ChaosKnight-1.5.2分支。
- 到ChaosKnight-1.5.2分支后，git pull orign ChaosKnight-1.5.2 更新代码。
- 切换到dev_1.5.2分支，执行git rebase ChaosKnight-1.5.2，让dev_1.5.2基于最新代码改动的开发。
- 切换到ChaosKnight-1.5.2分支，再执行：git merge --squash dev_1.5.2命令，会发现开发的代码全部合入到ChaosKnight-1.5.2本地分支，此时本地先git commit 提交代码。
- git push orign ChaosKnight-1.5.2把本地提交push到服务器。

这么做能省略不必要的提交清单，让提交历史更好看，更有目的性，回溯起来更好定位问题。




## 库管理

### 克隆库

```bash
git clone https://github.com/php/php-src.git
git clone --depth=1 https://github.com/php/php-src.git # 只抓取最近的一次 commit
```



## 历史管理

### 合并两个commit

```bash
合并两个commit
git log:
Commit A
Commit B
Commit C
Commit D
Commit O

比如要合并 A B C D 为一个commit
git rebase -i Commit O
在里面把 B C D 前面设s
退出rebase, 编辑commit

这个时候就成了
Commit X
Commit O

然后git push -f origin 远程分支
```

### 还原到某个commitId

```bash
#还原到某个commitId
git reset --hard {commitId......}
git push -f {branch} 
```

### 查看历史

```bash
git log --pretty=oneline filename # 一行显示
git show xxxx # 查看某次修改
```

### tag功能

```bash    
git tag # 显示所有标签
git tag -l 'v1.4.2.*' # 显示 1.4.2 开头标签
git tag v1.3 # 简单打标签   
git tag -a v1.2 9fceb02 # 后期加注标签
git tag -a v1.4 -m 'my version 1.4' # 增加标签并注释， -a 为 annotated 缩写

#为特定的commit打tag
git tag -a mie_updatesdk_2.1.7 -m "mie_updatesdk_2.1.7" 593141949b79831389951797b123c132bafa0d4c
#删除tag
git tag -d mie_updatesdk_2.1.8
#删除远程TAG
git push origin :refs/tags/mie_updatesdk_2.1.8

git show v1.4 # 看某一标签详情
git push origin v1.5 # 分享某个标签
git push origin --tags # 分享所有标签
```

### 回滚操作

```bash
git reset 9fceb02 # 保留修改
git reset 9fceb02 --hard # 删除之后的修改
```

### 取消文件的修改

```bash
git checkout -- a.php #  取消单个文件
git checkout -- # 取消所有文件的修改
```

### 删除文件

```bash
git rm a.php  # 直接删除文件
git rm --cached a.php # 删除文件暂存状态
```

### 移动文件

```bash
git mv a.php ./test/a.php
```

### 查看文件修改

```bash
git diff          # 查看未暂存的文件更新 
git diff --cached # 查看已暂存文件的更新 
```

### 暂存和恢复当前staging

```bash
git stash # 暂存当前分支的修改
git stash apply # 恢复最近一次暂存
git stash list # 查看暂存内容
git stash apply stash@{2} # 指定恢复某次暂存内容
git stash drop stash@{0} # 删除某次暂存内容
```

### 修改 commit 历史纪录

```bash
git rebase -i 0580eab8
```



## 分支管理（branch）

### 创建分支

```bash
git branch develop # 只创建分支
git checkout -b master develop # 创建并切换到 develop 分支
```

### 合并分支

```bash
git checkout master # 切换到 master 分支
git merge --no-ff develop # 把 develop 合并到 master 分支，no-ff 选项的作用是保留原分支记录
git rebase develop # rebase 当前分支到 develop
git branch -d develop # 删除 develop 分支
```

### 克隆远程分支

```bash
git branch -r # 显示所有分支，包含远程分支
git checkout origin/android
```

### 删除分支

```bash
#删除远程：
git push origin :Beastmaster-1.5.1
#删除本地
git branch -D Beastmaster-1.5.1
```



### 修复develop上的合并错误

1. 将merge前的commit创建一个分之，保留merge后代码
2. 将develop `reset --force`到merge前，然后`push --force`
3. 在分支中rebase develop
4. 将分支push到服务器上重新merge

### 强制更新到远程分支最新版本

```bash
git reset --hard origin/master
git submodule update --remote -f
```

## Submodule使用

### 克隆带submodule的库

```bash
git clone --recursive https://github.com/chaconinc/MainProject
```

### clone主库后再去clone submodule

```bash
git clone https://github.com/chaconinc/MainProject
git submodule init
git submodule update
```

## Git设置

Git的全局设置在`~/.gitconfig`中，单独设置在`project/.git/config`下。

忽略设置全局在`~/.gitignore_global`中，单独设置在`project/.gitignore`下。

### .gitconfig（设置快捷键）

```bash
[user]
    name = Larry
    email = 511573633@qq.com

[credential]
    helper = store

[alias]
    co = checkout
    ci = commit
    st = status
    br = branch
    hist = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr)%Creset %Cred(%an)%Creset' --abbrev-commit --date=relative
    type = cat-file -t
    dump = cat-file -p
    me = merge --squash
```

### 设置 commit 的用户和邮箱

```bash
git config user.name "xx"
git config user.email "xx@xx.com"
```

或者直接修改config文件

```bash
[user]
    name = xxx
    email = xxx@xxx.com
```

### 查看设置项

```bash
git config --list
```

### 设置git终端颜色

```bash
git config --global color.diff auto
git config --global color.status auto
git config --global color.branch auto
```

### 代理

```bash
#添加
git config --global http.proxy http://107.151.136.194:80
git config --global https.proxy https://127.0.0.1:1080

#取消代理
git config --global --unset http.proxy
git config --global --unset https.proxy
```
### 统计某个人修改代码行数

``` bash
#比如DemonHunter的代码行数
git log --author="DemonHunter" --since="2016-3-1" --before="2016-4-1" --pretty=tformat: --numstat | gawk '{ add += 1 ; subs += 2 ; loc += 1 - 2 } END { printf "added lines: %s removed lines : %s total lines: %s\n",add,subs,loc }' -
```





