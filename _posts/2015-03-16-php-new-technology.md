---
layout: page
title: "PHP的设计模式"
description: "PHP的设计模式、命名空间、自动载入"
category: "php"

---
{% include JB/setup %}

#1.Autoload

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

#2.spl库（PHP标准库）

####(1).栈（数据结构）--先进后出

	$stack = new SplStack();
	$stack->push("data1");//进
	echo $stack->pop();//出

####(2).队列（数据结构）--先进先出

	$queue = new SplQueue();
	$queue->enqueue("data1");//进
	echo $queue->dequeue();//出

####(3).堆

	$heap = new SplMinHeap();//小顶堆
	$heap->insert("data1");//存入
	echo $heap->extract();//取出

####(4).固定长度数组

	$array = new AplFixedArray(10);
	$array[0] = 1;
	$array[9] = 10;
	var_dump($array);

#3.链式操作的实现

	$db->where()->order()->limit();
	
	实现方式是在db类的function的结尾处
	return $this;

#4.PHP的魔术方法

####(1).__set __get

	#对不存在的对象属性访问进行控制
	function __set($key, $value){
	
	}
	function __get($key){
	
	}

####(2).__call __callStatic

	#对不存在的对象方法进行控制
	function __call($func, $param){
	
	}
	
	#对不存在的类的静态方法进行控制
	static function __callStatic($func, $param){
	
	}

####(3). __toString

	#需要对象转为字符串时会执行此方法
	class Obj{
		function __toString(){
			return __CLASS__;
		}
	}
	$obj = new Obj();
	echo $obj;

####(4). __invoke

	#当对象被当做方法执行时会调用
	class Obj{
		function __invoke($param){
			var_dump($param)
			return "invoke";
		}
	}
	$obj = new Obj();
	$obj("xxx");

#5.设计模式

####(1).工厂模式--辅助生成对象

用处是可以统一控制对象的生成，方便预置参数等、方便类名修改

	namespace Xxx;
	class Factory{
		static function createDatabase(){
			$db = new Database();
			//中间可以加入一些预操作的代码
			return $db;
		}
	}
	
	$db = Xxx\Factory::createDatabase();

####(2).单例模式--控制对象仅被创建一次

	class Database{
	
		static $db;
		
		//用private可方式对象直接在外部被实例化
		private function __construct(){
		
		}
		
		//用于实例化本类
		static function getInstance(){
			if(self::$db){
				return self::$db;
			}
			
			self::$db = new self();
			return self::$db;
		}
	}

####(3).注册表模式--全局共享和交换对象

用处是声明一次对象后，以后直接通过get方法来存取
	
	namespace Xxx;
	
	class Register{
	
		protected static $objects;
		
		static function set($alias, $object){
			self::$objects[$alias] = $object;
		}
		
		static function get($alias){
			if ( isset(self::$objects[$alias]) ) {  
				return self::$objects[$alias];
			}  
        	return null; 
		}
		
		static function _unset($alias){
			unset(self::$objects[$alias]);
		}
	}
	
	namespace Xxx;
	
	class Factory{
		static function createDatabase(){
			$db = Database::getInstance();
			Register::set('db1', $db);
			return $db;
		}
	}
	
	#第一次调用
	$db = Xxx\Factory::createDatabase();
	#第二次调用
	$db = Xxx\Register::get('db1);

####(4).适配器模式--将不同的函数封装成统一的API

	#首先创建一个接口类，里面定义几个需要实现的方法；真正使用的类要继承自这个类并实现里面几个必须方法
	#如下接口
	interface IDatabase{
		function connect($host, $user, $passwd, $dbname);
		function query($sql);
		function close();
	}
	#实现的类
	class MySQL implement IDatabase{
	
		protected $conn;
		
		function connetc($host, $user, $passwd, $dbname){
			$conn = mysql_connect($host, $user, $passwd);
			mysql_select_db($dbname, $connect);
			$this->conn = $conn;
		}
		
		function query($sql){
			return mysql_query($sql, $this->conn);
		}
		
		function close(){
			mysql_close($this->conn);
		}
	}

####(5).策略模式

	#通过传入接口不同的实现，使用统一的调用模式，得到不同的结果
	#方便实现的扩展，和功能的解耦，功能只需要在实现类里面实现修改即可
	
	#例子
	class Show{
		/**
		 * @var \Xxx\UserStrategy
		 */
		 protected $strategy;
	 
		 function index(){
	 		$this->stragegy->showAd();
		 }
	 
		 function setStrategy(\Xxx\UserStrategy $strategy){
	 		$this->strategy = $stragegy;
		 }
	 }
	 interface UserStrategy{
	 	function showAd();
	 }
	 class FemaleUserStragegy implements UserStrategy{
	 	function showAd(){}
	 }
	 
	 $show = new Show();
	 $show->setStrategy(new FemaleUserStragegy());
	 $show->index();
	