---
layout: page
title: "Hello Word By Symfony"
description: "Hello Word By Symfony"
category: "symfony"

---
{% include JB/setup %}

##1.Hello Word

####(0).symony的安装

	首先要保证php-cli可用，没有请自行安装（symfony要求php在5.0版本以上
	#安装composer包依赖管理工具，然后就可以使用composer这个命令了
	curl -sS https://getcomposer.org/installer | php
	mv composer.phar /usr/local/bin/composer
	#安装symfony
	#我安装的是2.6版本，然后安装到当前目录的symfony下
	composer create-project symfony/framework-standard-edition ./symfony

####(1).php-cli运行网站

	#在网站根目录下执行，-vvv代表将请求信息都打印出来（此处根目录是symfony里面）
	php app/console server:run -vvv
	
####(2).创建一个新Bundle

	app/console generate:bundle
	然后输入Scourgen/SppBundle，得到的是在src目录下创建一个Scourgen目录，再在其下创建一个SppBundle目录防止项目
	#测试
	编辑Controller下的DefaultController；hello改成hh
	php app/console server:run -vvv启动服务
	访问http://127.0.0.1:8000/app_dev.php/hh/abc
	或者http://127.0.0.1:8000/hh/abc
	
