---
layout: page
title: "Symfony Notes"
description: "Symfony Notes"
---
{% include JB/setup %}

##1.视图

####(1).模板分隔符

	{{。。。}} #输出变量
	{#。。。#} #注释语句，不会在html中显示
	{ %。。。% } #条件语句

####(2).特定语句

	注意下面{和%之间要去掉空格
	{ % extends "AcmeDemoBundle::layout.html.twig" % } #继承模板
	——————————————————————————————————————
	{ % block content % } #父模板中的格式
    { % endblock %}
    
    { % block content % } #子模板中的格式
    	<h1>Hello {{ name }}!</h1>
	{ % endblock % }
	——————————————————————————————————————
	{ % include "AcmeDemoBundle:Demo:embedded.html.twig" % } #包含同级模板语句
	——————————————————————————————————————
	{ % render "AcmeDemoBundle:Demo:fancy" with { 'name': name, 'color': 'green' } % } #输出Demo类下fancy控制器的内容
	——————————————————————————————————————
	<a href="{{ path('_demo_hello', { 'name': 'Thomas' }) }}">Greet Thomas!</a> #创建链接
	——————————————————————————————————————
	<link href="{{ asset('css/blog.css') }}" rel="stylesheet" type="text/css" /> #引入图片、js、css

	<img src="{{ asset('images/logo.png') }}" />

##2.控制器

####(1).返回格式

	// src/Acme/DemoBundle/Controller/DemoController.php
	use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
	use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;

	/**
 	* @Route("/hello/{name}.{_format}", defaults={"_format"="html"}, requirements={"_format"="html|xml|json"}, name="_demo_hello")
 	* @Template()
 	* 请求链接/demo/hello/Fabien.xml
 	* _format指输出格式，这里默认html，可选项是html|xml|json
 	*/
	public function helloAction($name)
	{
	    return array('name' => $name);
	}

####(2).跳转

	return $this->redirect($this->generateUrl('_demo_hello', array('name' => 'Lucas')));
	# _demo_hello是在Router中设置的name，后面的是参数，直接进行跳转
	# generateUrl()与Twig模板里的path()函数是同一个
	——————————————————————————————————————
	$response = $this->forward('AcmeDemoBundle:Hello:fancy', array('name' => $name, 'color' => 'green'));
	# 产生一个子请求，返回respose对象
	# do something with the response or return it directly

<a id="get_request"><h4>(3).获取请求信息</h4></a>

	$request = $this->getRequest();
	$request->isXmlHttpRequest(); // 是否是Ajax请求？
	$request->query->get('page'); // 获取一个 $_GET 参数
	$request->request->get('page'); // 获取一个 $_POST 参数
	{{ app.request.query.get('page') }}
	{{ app.request.parameter('page') }}
	#注：2.4之后获取Request方式如下
	use Symfony\Component\HttpFoundation\Request
	public function indexAction($id, Request $request)
	{
		$name = $request->getSession()->get("c");
	}
	#或者
	$request = $this->container->get("request");
	#或者
	$request = $this->get("request");

####(4).设置session

	$session = $request->getSession();
	$session->set('foo', 'bar');
	$foo = $session->get('foo');
	$session->setLocale('fr');// 设置用户的本地化选项
	$session->setFlash('notice', 'Congratulations, your action succeeded!'); # 为下一个请求设置一个消息（控制器里）
	{{ app.session.flash('notice') }} # 模板里显示

####(5).安全机制

	# app/config/security.yml 文件的设置
	——————————————————————————————————————
	use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
	use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;
	use JMS\SecurityExtraBundle\Annotation\Secure;
	/**
	 * @Route("/hello/admin/{name}", name="_demo_secured_hello_admin")
	 * @Secure(roles="ROLE_ADMIN")
	 * @Template()
	 * 规定只有ROLE_ADMIN才能访问否则403
	 */
	public function helloAdminAction($name)
	{
    	return array('name' => $name);
	}

####(6).缓存

	use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
	use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;
	use Sensio\Bundle\FrameworkExtraBundle\Configuration\Cache;

	/**
	 * @Route("/hello/{name}", name="_demo_hello")
	 * @Template()
	 * @Cache(maxage="86400")
 	 */
	public function helloAction($name)
	{
	    return array('name' => $name);
	}

####(7).临时

	#给项目在对外的web目录创建映射
	app/console assets:install --symlink --relative
	
##3.Hello Word

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
	
##4.路由

####(1).注意

	当路由条数比较多时，将比较常访问的放到前面有助于提高它们的访问速度。
	
####(2).路由种类

	<1>annoation
	<2>router.yml
	
####(3).annoation示例

	<1>
	/**
	 * @Route("/page/{page_name}", defaults={"page_num":1}, requirements={"page_num"="\d+"})
	 * @Method("POST")
	 * @Template()
	 * 只匹配page_num是数字的url,且默认是1，方法是post
	 * 需要use Route Method等方法
	 */
	 
	 <2>
	 /**
	 * @Route("/page")
	 * 写在类的上面
	 */
	 /**
	 * @Route("/index")
	 * @Template()
	 * 写在方法上面
	 * 最后访问地址是/page/index
	 */
	 
	 <3>
	 /**
	  * @Route("/post/{id}")
	  * @paramConverter("post", class="ScourgenWebBundle:Post")
	  * @Template()
	  */
	  public function indexAction(Post $post)
	  {
	  	//直接获取某个数据库中的信息
	  	$title = $post->getTitle();
	  }
	 
####(4).打印、查找现有路由规则

	#打印规则
	app/console router:debug
	#匹配规则
	app/console router:match /page/index
	
##5.请求

<a href="#get_request"><h4>(1).request</h4></a>

####(2).response

	#返回一个json对象（格式化好的）
	return new JsonResponse(array("a"=>4));
	#返回一个应答对象
	return new Response("123456");
	#返回一个跳转
	return new RedirectResponse("http://www.xxx.com");
	
####(3).session

	#设置、获取session的值
	$request->getSession()->set("c", 1234);
	$request->getSession()->get("c");
	
	#设置、获取临时session的值
	$session = $request->getSession();
	$session->getFlashBag()->add('notice', 'Profile updated');
	foreach ($session->getFlashBag()->get('notice', array()) as $message) {
    	echo $message;
    }
