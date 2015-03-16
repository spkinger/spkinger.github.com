---
layout: page
title: "PHP的设计模式"
description: "PHP的设计模式、命名空间、自动载入"
category: "php"

---
{% include JB/setup %}

####(1).php类的自动载入

	#自动载入方法注册
	spl_autoload_register('autoload1');
	#自动载入方法
	function autoload1($class){
		require __DIR__."/".$class.".php";
	}
	#然后就可以这样调用函数了
	Test1::test();
	Test2::test();

####(2).PSR-0规范的框架
	
	1.全部使用命名空间，文件名与类名一致且首字母大写
	2.所有PHP文件必须自动载入，不能使用include、requeire
	3.单一入口文件
	
	#例子
	index.php [在根目录]
	define('BASEDIR', __DIR__);
	include BASEDIR.'/Lib/Loader.php
	spl_autoload_register('\\Lib\\Loader::autoload');
	Lib::Test::test();
	App\Controller\Home\Index::test();
	
	Loader.php [在Lib/Loader.php]
	class Loader{
		static function autoload($class){
			require BASEDIR.'/'.str_replace('\\', '/', $class).'.php';
		}
	}