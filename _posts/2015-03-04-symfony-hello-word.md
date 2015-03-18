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

#2.实际项目流程

####(1).从头创建一个新项目（参考上面）

	sudo curl -LsS http://symfony.com/installer -o /usr/local/bin/symfony
	sudo chmod a+x /usr/local/bin/symfony
	
	symfony new my_project
	
	sudo chmod -R 777 app/logs
	sudo chmod -R 777 app/cache

	app/console generate:bundle --namespace=Xxx/XxxBundle --format=yml
	
	php app/console cache:clear --env=prod
	php app/console cache:clear --env=dev
	
	#然后设置Apache、nginx的域名指向到（项目根目录/web）下
	然后使用如下测试访问
	http://域名/hello/jobeet or http://域名/app_dev.php/hello/jobeet

####(2).数据库的生成

[Doctrine2表对应关系设置文档](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/association-mapping.html)

	#编辑app/config/parameters.yml(数据库配置)
	#如果没有预先创建数据库，执行如下创建
	app/console doctrine:database:create
	
	#添加数据库结构文件
	创建目录src/xxxBundle/Resources/config/doctrine
	在下面创建如Category.orm.yml的文件
	
	#生成entity文件(model文件)
	app/console doctrine:generate:entities XxxXxxBundle
	
	#生成数据库的表
	app/console doctrine:schema:update --force

####(3).生成测试数据
[DoctrineFixturesBundle文档](http://symfony.com/doc/current/bundles/DoctrineFixturesBundle/index.html)

	#安装DoctrineFixturesBundle
	
	1.编辑composer.json
	{
		"require": {
        	"doctrine/doctrine-fixtures-bundle": "2.2.*"
	    }
	}
	
	composer update
	
	编辑app/AppKernel.php，在测试环境注册这个bundle，添加如下语句
	$bundles[] = new Doctrine\Bundle\FixturesBundle\DoctrineFixturesBundle();
	
	2.创建文件夹src/Xxx/XxxBundle/DataFixtures/ORM
	例：mkdir -p src/Spk/WebBundle/DataFixtures/ORM
	在此文件夹下添加LoadCategoryData.php等数据生成文件(具体格式参照文档)
	执行命令生成数据
	app/console doctrine:fixtures:load

####(4).生成具备CRUD功能的控制器和表单

	解释是使用entity SpkWebBundle:Job，路由前缀是job，添加“write”方法（包括new, update, and delete方法），格式使用yml
	app/console doctrine:generate:crud --entity=SpkWebBundle:Job --route-prefix=job --with-write --format=yml
	
	然后可访问如下这些等等
	http://yymm.com/app_dev.php/job
	http://yymm.com/app_dev.php/job/new
	http://yymm.com/app_dev.php/job/1/show
	