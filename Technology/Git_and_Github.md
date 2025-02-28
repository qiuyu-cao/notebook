# Git and GitHub



Git工作流和核心原理 | GitHub基本操作 | VS Code里使用Git和关联GitHub。[B 站：技术蛋老师](https://www.bilibili.com/video/BV1r3411F7kn/?vd_source=ddca02f5ce8a93f7668a9c6f429a73d1)

![image-20250228111119443](img\Git_and_GitHub\image-20250228111119443.png)

## Git

Git 官方参考文档：https://git-scm.com/docs

### Git 备忘单

#### 配置工具

```bash
# 对 commit 操作设置关联的用户名
$ git config --global user.name "[name]"
# 对 commit 操作设置关联的邮箱地址
$ git config --global user.email "[email address]"
```

### 分支

```bash
# 查看当前分支
$ git status
# 创建一个新分支
$ git branch [branch-name]
# 重命名当前分支
$ git branch -m [new-branch-name]
# 切换到指定分支并更新工作目录
$ git switch -c [branch]
# 将指定分支合并到当前分支，分支可以有好几层，通过“/”进行分割
# 不同分支下的分支名字可以重复
# 如：git merge feature/login
$ git merge [branch]
# 查看分支，支持通配符
$ git branch --list [partner]
# 删除指定分支
$ git branch -d [branch]
```

#### 进行更改

```bash
# 列出当前分支的版本历史
$ git log
# 列出文件的历史版本，包括重命名
$ git log --fllow [file]
# 展示两个分支之间的内容差异
$ git diff [first-branch]...[second-branch]
# 展示 commit 信息
$ git show [commit]
# 将文件提交到暂存区，文件会从 untracked 状态转变为 unstage 状态
$ git add [file]
# 将文件快照永久记录在版本历史中，ustage 状态的文件变为 stage
$ git commit -m "[discriptive message]"
# 同时运行 add 和 commit 的操作，但是会忽略 untracked 的文件
$ git commit -a -m "[discriptive message]"
```

#### 重新提交

```bash
# 保持本地文件不变，撤销所有 commit 后的提交描述
$ git reset [commit]
# 回溯到历史 commit 版本，并且删除所有之后的版本。“谨慎操作”
$ git reset --hard [commit]
```

#### 创建仓库

```bash
# 将当前文件夹初始为一个 Git 仓库
$ git init
# 下载一个远程仓库，包括文件、分支和提交（commmit）
git clone [url]
```

#### .gitignore 文件

```
# gitignore template for InforCRM (formerly SalesLogix)
# website: https://www.infor.com/product-summary/cx/infor-crm/
#
# Recommended: VisualStudio.gitignore

# Ignore model files that are auto-generated
ModelIndex.xml
ExportedFiles.xml

# Ignore deployment files
[Mm]odel/[Dd]eployment

# Force include portal SupportFiles
!Model/Portal/*/SupportFiles/[Bb]in/
!Model/Portal/PortalTemplates/*/SupportFiles/[Bb]in
```

#### 同步更改

```bash
# 在创建好本地仓库之后，跟远程仓库进行交互
# 下载远程仓库的历史分支

```

### GitHub Action

在创建好的 `Git` 仓库下的 `.github/workflow` 中，可以使用 `YAML` 格式的文件定义工作流。将本地仓库上传到 GitHub 后，GitHub 服务器会自动运行其中的工作量。这些工作流的运行情况可以在 GitHub 上登录自己的账号 -> 打开对应的仓库 -> 点击 `Actions` 进行查看。

定义工作流的语法可以参考[官网教程](https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsenv)。实例：

```yaml
on:
  workflow_dispatch:

jobs:
  pass-secrets-to-workflow:
    uses: ./.github/workflows/called-workflow.yml
    secrets: inherit
```

## 与 GitHub 交互

### 通过 SSH 与 GitHub 交互

##### 配置密钥对

```bash
# 在本地生成 SSH 密钥对
$ ssh-keygen -t <ed25519> -C <annotation> -f <key-path>
# 在 GitHub 登录自己的账号，点击 `头像->Settings->SSH and GPG keys->New SSH key`
# 将 <key-path>.pub 文件的信息复制并填写到 GitHub
# 只有进行过密钥对配置之后才可以通过 ssh 连接 clone 一个仓库
```

##### 配置 `~/.ssh/config` 文件

`~/.ssh/config`  文件的主要功能时告诉 `ssh` 通过哪个密钥文件连接哪个服务端。

```
IdentityFile <key-path>
```

##### 与 GitHub 仓库建立连接

```bash
# 在本地初始化一个仓库
$ git init
# 在当前仓库添加一个远程仓库
$ git remote add <name> <git@github.com:...>
# 常看当前仓库的远程仓库信息
$ git remote -v
# 重命名一个远程仓库
$ git remote rename <old-name> <new-name>
# 修改远程仓库的 URL
$ git remote set-url <name> <url>
# 查看远程仓库的分支信息
$ git remote show <name>
```

##### 与远程仓库交换文件

```bash
# 将远程仓库拉取到本地工作区
# 由于 GitHub 默认的主分支名为 'main'，git init 创建的主分支名为 'maste'
# 用 git pull <remote-name> 的时候会报错现在有两种解决方法
# 方法 1：将本地主分支名更换为 'main'
git branch -m main
git pull <remote-name>
# 方法 2：指定将远程 'main' 分支与本地 'master' 分支进行连接
git pull <remote-name> master:main
# 完成 add 和 commit 后可以将本地仓库推送到远程仓库
git push -u <remote-name> <local-branch>:<remote-branch>
```



```bash
# 在 GitHub 上注册账户之后，创建一个仓库，并且 clone 到本地
git clone [url]
# 
```

