# 1 Git安装

从官方下载安装包并进行安装

# 2 git命令

> Git的设置 --global参数为设置全局

```shell
git config --global user.name 'Tuffy' # 设置全局用户名
git config --global user.email 'yusheng831143@gmail.com' # 设置全局邮箱
git config --list # 查看所有设置
```

## 2.1 本地查询

### 2.1.1 log

```shell
git log # 查看历史所有版本信息 
git log -x # 查看最新的x个版本信息
git log --pretty=oneline # 查看历史所有版本信息，只包含版本号和记录描述
```



## 2.2 远程分支操作



### 2.2.1 pull

`pull`命令将远程分支拉取到本地时，**会**自动`merge`

```shell
git pull <远程主机名> <远程分支名>:<本地分支名>
```

### 2.2.2 fetch

`fetch`命令将远程分支获取到本地时，但是**不会**自动`merge`



## 2.3 分支操作

### 2.3.1 branch 

用于查看和切换分支

```shell
git branch <branch_name> # 创建分支
git branch -D <branch_name> # 删除分支
git branch -a / -v  # 查看本地与远程所有分支 / 查看分支详情（commit）
git checkout [-b] <branch_name> # [创建]并切换到指定分支
```

### 2.3.2 meger

`merge`命令合并指定分支到当前分支

```shell
git merge <other_name> # 合并other_name到当前分支
```

### 2.3.3 commit /  add 相关操作

```powershell
git add [./name] # 向缓存区提交文件
git commit -m 'message' # 从缓存区向本地仓库提交文件
git rm --cached  # 从缓存区删除使用 add 命令添加的文件


git push <远程主机名> <本地分支名>:<远程分支名> # 推送
git branch -al  # 查看本地与远程所有分支
git pull <远程主机名> <远程分支名>:<本地分支名> # 拉取


# reset --hard是回退版本命令，可以跟版本ID进行回退
git reset --hard # 丢弃工作区所有更改，修改保持与pull下来的版本一致

git commit --amend --reset-author # 设置了用户名后用于修正提交
```

### 2.3.4 舍弃更改

```shell
git reset HEAD # 丢弃暂存区更改到工作区
git checkout -- <文件名> # 丢弃工作区文件修改
```