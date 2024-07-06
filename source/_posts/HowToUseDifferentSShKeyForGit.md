---
title: 如何在Git中使用不同的SSH密钥
date: 2024-07-06 18:41:08
tags: git
---

介绍如何解拥有多个不同账户的情况下使用ssh密钥使用Git

<!-- more -->



## 简介

通常情况下一般我们只是用单一的ssh密钥进行Git仓库的代码推送，但是某些i情况下会使用多个账户进行多个项目的开发，这个时候就需要配置正确的SSH密钥，否则将无权限进行推送等。


## 创建一个新的密钥
通过命令行工具可以轻松生成一个新的 SSH 密钥, 将下方邮箱替换为自己的

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

输入命令后，会询问你想将文件存储在哪里, 他会有一个默认值，不同的系统路径不一样，如果没有其他要求直接回车，或者可以自己输入路径和名称然后进行回车

```bash
Enter a file in which to save the key (/home/YOU/.ssh/id_ALGORITHM):
```
接下来会重复让你输入密码,密码是用来保护SSH密钥的，一般不需要直接回车即可。
```bash
> Enter passphrase (empty for no passphrase):
> Enter same passphrase again:
```
## 将SSH密钥添加到GitHub

https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account

这里GitHub提供了一个详细文档。


## 修改 SSH

下面基于Linux演示，Windows用户可以使用 [WSL](https://learn.microsoft.com/en-us/windows/wsl/install) 


```bash
# use vim

vim ~/.ssh/config

# use vscode

code ~/.ssh/config
```

打开文件后一般是空的，现在我们添加配置文件


```bash
Host *
  IgnoreUnknown UseKeychain
# 这个用作为默认
Host github.com
  Hostname github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519

Host github.com-forWork #这里可以根据自己需要自己起名字
  Hostname github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_forWork  #这里选择你需要使用的私钥

```

上面我们添加了两个配置，第一个id_ed25519作为默认，会应用于每一个GitHub仓库。接下来展示如何使用第二个配置。

## 使用第二个配置

首先我们克隆一个仓库, 一般使用如下格式命令进行克隆

```bash
git clone git@github.com:user/repo.git

```

但是这里我们使用修改过的命令进行克隆

```bash
git clone git@github.com-forWork:user/repo.git

```
这里将github.com 修改成为 github.com-forWork, 这里的 github.com-forWork 是上面 config 里面定义的，根据自己的定义修改这个命令


## 修改项目配置

如果直接推送，使用的还是默认SSH密钥，所以这里还需要修改一下当前项目远程地址


直接使用命令进行修改

```bash
git remote set-url origin git@github.com-forWork:user/repo.git

```

就是将这里的推送地址修改为我们在config中定义的。

另外一种是使用编辑器修改项目的配置

```bash
git config --global core.editor "code --wait"
```
使用这条命令可以将Git使用的编辑器修改为 vscode 当然这条命令是可选的

```bash
git config -e 
```

这里会调用编辑器打开配置文件，将配置文件中的

```bash
[remote "origin"]
	url = git@github.com-forWork:user/repo.git  #将这里的url修改一下
```

之后就可以正常的进行推送了
