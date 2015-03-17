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
	
####(00).symfony新的安装方式

	#linux和mac os
	sudo curl -LsS http://symfony.com/installer -o /usr/local/bin/symfony
	sudo chmod a+x /usr/local/bin/symfony
	#windows
	php -r "readfile('http://symfony.com/installer');" > symfony
	
	#然后执行如下即可（需要翻山越岭。。）
	symfony new my_project
	
	#将域名指向网站后可能出现报错，提示需要更改logs和cache文件夹的权限，直接更改所有者为apache(推荐线上)，本地权限改为777
	sudo chmod -R 777 app/logs
	sudo chmod -R 777 app/cache

####(1).php-cli运行网站

	#在网站根目录下执行，-vvv代表将请求信息都打印出来（此处根目录是symfony里面）
	php app/console server:run -vvv
	
####(2).创建一个新Bundle

	app/console generate:bundle
	然后输入Scourgen/SppBundle，得到的是在src目录下创建一个Scourgen目录，再在其下创建一个SppBundle目录防止项目
	#生成bundle后需要清理一下缓存
	php app/console cache:clear --env=prod
	php app/console cache:clear --env=dev
	#测试
	编辑Controller下的DefaultController；hello改成hh
	php app/console server:run -vvv启动服务
	访问http://127.0.0.1:8000/app_dev.php/hh/abc
	或者http://127.0.0.1:8000/hh/abc

####(3).如果建立bundle时选择的是annoation后期转yml方法

	#entity
	php app/console doctrine:mapping:convert --namespace="xxxBundle\Entity\Blog" 
yml src/xxxBundle/Resources/config/doctrine

	#router
	app/console router:convert yml destination-dir/
	
	#如下命令可查看现有的路由
	app/console router:debug
	app/console router:dump-apache（任选一个）
	
