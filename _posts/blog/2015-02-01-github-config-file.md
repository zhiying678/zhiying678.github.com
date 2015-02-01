---
layout: post
title: git config文件配置
description: 使用git config 文件连接多个git 仓库
category: blog
---
###### February 1, 2015 5:31 PM

额，之前弄过，又忘了，好记性不如烂笔头,记录一下吧。
 git bash之前的配置就略过了。
- - - 
- 首先，参照[github上的教程](https://help.github.com/articles/generating-ssh-keys/)使用****ssh-keygen****生成sshkey。我把名字改为了*id_rsa_github*
- 登陆github账号，添加公钥。
- 切换到用户home目录，在config文件中添加密钥信息
```bash
cd ~/.ssh
touch config
vi config
```
在config文件中添加：
```bash
Host "mygithub"
HostName "github.com"
User "git"
IdentityFile "~/.ssh/id_rsa_github"
```
- git bash中测试,会输出连接成功信息。
```bash
$ ssh -v git@mygithub
Hi xxxxxxxx! You've successfully authenticated, but GitHub does not provide sh
ell access.
```

####注意事项
1. config文件中不支持当前路径的相对路径，****IdentityFile "./id_rsa_github"****这样是不对的，应该这样****IdentityFile "~/.ssh/id_rsa_github"****。
2. git clone时，应当把网站(如github.com)改为对应的Host，
```bash
 git clone git@mygithub:username/path.to.git.repository
```
3. 可以在用户Home目录下，将*.ssh*文件夹打包备份，
```bash
cd ~
tar -zcvf mykey.tar.gz .ssh
```
换台电脑，
```bash
tar -zxvf mykey.tar.gz -C ~/
```
或将上述过程写到脚本文件
```bash
sh a.sh
```
4. TortoiseGit使用puttygen生成的密钥格式跟ssh-keygen不兼容，需要改一下再用。

以下内容是测试markdown的标记的
- [ ] 任务1
- [x] 完成了的任务


什么是`inline code;`啊








[zhiying678]:    http://blog.houmingjiang.cn  "zhiying678"
