---
layout: page
title: "使用github的第一步github的SSH key设置"
description: "创建 SSH key 并启用，建立与 github 的初步连接"
category: "git"

---
{% include JB/setup %}

##1.创建 SSH key

	#以下内容适用于linux、mac系统
	
	#切换到ssh的存储目录
	cd ~/.ssh
	#创建一个 ssh key，这个邮箱可以随便填。。
	ssh-keygen -t rsa -C "your_email@example.com"
	
	#然后会出现以下提示
	#输入 ssh key 存储的文件名，直接回车文件名就会使用默认的 id_rsa
	Enter file in which to save the key (/Users/you/.ssh/id_rsa): 
	#要求输入两次密码，有时为了方便可以留空，直接回车
	Enter passphrase (empty for no passphrase):
	Enter same passphrase again:	
	#以下是生成后的提示
	Your identification has been saved in /Users/you/.ssh/id_rsa.
	Your public key has been saved in /Users/you/.ssh/id_rsa.pub.
	
##2.启用刚刚生成的 SSH key （关键的一步）
	
	#一、永久添加
	添加一个配置文件
	vim ~/.ssh/config
	添加如下内容
	Host github.com
	  User git
	  Hostname github.com
	  PreferredAuthentications publickey
	  IdentityFile ~/.ssh/id_rsa
	这样就能一直使用这个key了，也可以继续添加，进行多域名管理
	解释：
	Host 是请求的服务器的代号
	Hostname是服务器地址
	#二、临时添加
	#id_rsa是刚刚生成的 ssh key 的文件名
	ssh-add ~/.ssh/id_rsa
	#这里如果报错 Could not open a connection to your authentication agent
	#就在ssh-add前执行这个命令(解释是Note that this will start the agent for msysgit Bash on Windows.)
	eval `ssh-agent -s`
	
##3.将 SSH key 的公钥内容放入github的key库中
	
	#mac可以用如下命令
	pbcopy < ~/.ssh/id_rsa.pub
	#linux的cat下吧。。然后手动复制
	cat ~/.ssh/id_rsa.pub
	
	#然后进入下面地址点击 Add SSH key
	https://github.com/settings/ssh
	#title随便填
	#key放入刚刚复制的内容
	#点Add key保存
	
##4.测试是否能顺利通信
	
	ssh -T git@github.com
	#然后可能会有个提示，问是否continue connecting，选yes
	#成功的话会有下面的提示
	Hi username! You've successfully authenticated, butGitHub does not
	#坎塞~
