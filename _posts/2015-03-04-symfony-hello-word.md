---
layout: page
title: "Hello Word"
description: "Symfony's hello world"
category: "symfony"

---
{% include JB/setup %}

##1.Hello Word

####(1).php-cli运行网站

	#在网站根目录下执行，-vvv代表将请求信息都打印出来
	php app/console server:run -vvv
	
####(2).创建一个新Bundle

	app/console generate:bundle
	然后输入Scourgen/SppBundle，得到的是在src目录下创建一个Scourgen目录，再在其下创建一个SppBundle目录防止项目
	#测试
	编辑Controller下的DefaultController；hello改成hh
	php app/console server:run -vvv启动服务
	访问http://127.0.0.1:8000/app_dev.php/hh/abc
	或者http://127.0.0.1:8000/hh/abc
	
