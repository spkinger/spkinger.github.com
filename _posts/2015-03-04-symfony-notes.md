---
layout: page
title: "Symfony Notes"
description: "Symfony Notes"
category: "symfony"

---
{% include JB/setup %}

##1.视图

####(1).模板分隔符

	注：{和后面的{、%、#之间没有空格
	{ {。。。} } #输出变量
	{ %。。。% } #条件语句
	{ #。。。# } #注释语句，不会在html中显示

####(2).特定语句

	{ % extends "AcmeDemoBundle::layout.html.twig" % } #继承模板
	——————————————————————————————————————
	{ % block content % } #父模板中的格式
    { % endblock % }
    
    { % block content % } #子模板中的格式
    	<h1>Hello { { name } }!</h1>
	{ % endblock % }
	——————————————————————————————————————
	{ % include "AcmeDemoBundle:Demo:embedded.html.twig" % } #包含同级模板语句
	——————————————————————————————————————
	{ % render "AcmeDemoBundle:Demo:fancy" with { 'name': name, 'color': 'green' } % } #输出Demo类下fancy控制器的内容
	——————————————————————————————————————
	<a href="{ { path('_demo_hello', { 'name': 'Thomas' }) } }">Greet Thomas!</a> #创建链接
	——————————————————————————————————————
	<link href="{ { asset('css/blog.css') } }" rel="stylesheet" type="text/css" /> #引入图片、js、css

	<img src="{ { asset('images/logo.png') } }" />

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

<h4><a id="get_request">(3).获取请求信息</a></h4>

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

<h4><a href="#get_request">(1).request</a></h4>

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

##6.服务
	
	#列出所有服务
	app/console container:debug
	
##7.视图

####(1).资源文件配置
	
	#<1>.css、js等资源文件所在的目录是在scr/bundle名/Resource/public下面
	#为了使资源文件能够访问，需要给资源文件作一个软连接到web/bundles/xxx目录下面
	php app/console assets:install web --symlink --relative
	
	#然后在视图文件中css文件的路径就像如下书写方式
	<link rel="stylesheet" href="{ { asset('bundles/spkingerweb/css/bootstrap.css') } }">
	
=================

	#<2>.另一种资源加载的方法,这样会把目录下面的内容全部加载
	#注意：这种写法有可能会报一个错，就是提示你SpkingerWebBundle未绑定，这样修改
	------------------------------------------------------------
	编辑 app/config/config.yml
	在assetic下的bundles中添加SpkingerWebBundle，如下
	assetic:
		bundles: [ SpkingerWebBundle ]
	------------------------------------------------------------
	{ % block my_css % }
        { % stylesheets '@SpkingerWebBundle/Resources/public/css' % }
        <link rel="stylesheet" href="{ { asset_url } }" />
        { % endstylesheets % }
	{ % endblock % }
	{ % block my_js % }
	    { % javascript '@SpkingerWebBundle/Resources/public/js' % }
        <script type="text/javascript" src="{ { asset_url } }"></script>
        { % endjavascript % }
	{ % endblock % }
	
=================

	#<3>.其他资源加载方式
	#以coffeescript举例(这里需要安装coffeescript)
	先修改app/config/config.yml
	在assetic下的filters中添加
	filters:
		coffee:
			bin: /usr/local/bin/coffee
			node: /usr/bin/node
	视图中加载方式如下
	{ % javascripts
        '@SpkingerWebBundle/Resources/public/js/*.coffee' fileter="coffee" % }
        <script src="{ { asset_url } }"></script>
    { % endjavascripts % }
    
=================

	#<4>.js压缩
	需要先安装gnlifyjs
	npm install uglify-js
	ln -s 执行文件路径 /usr/local/bin/uglifyjs
	然后在script中添加filter="uglifyjs2"
	{ % javascripts '@SpkingerWebBundle/Resources/public/js/vendor/*' filter="uglifyjs2" % }
    	<script type="text/javascript" src="{ { asset_url } }"></script>
    { % endjavascripts % }

=================

	#<5>.版本控制
	修改app/config/config.yml
	找到framework->templating添加
	assets_version: 5
	assets_version_format: %%s?version=%%s

=================

	#<6>.生产环境中生成压缩混淆后文件的命令
	php app/console assetic:dump --env=prod --no-debug
	#启用多线程生成
	先修改composer.json中"symfony/assetic-bundle"为2.4及以上
	"symfony/assetic-bundle": "2.6.1",
	添加
	"kriswallsmith/spork": "dev-master",
	然后在生成的时候加--forks=线程数
	php app/console assetic:dump --env=prod --no-debug --forks=3
	
####(2).block的使用

	#为了将layout文件可以拆分模块，并在子类视图中复写，可以使用block函数，如下
	#子类视图中也用如下格式来替换
	{ % block header % }
	内容
	{ % endblock % }
	
	#继承父类的内容(一般不需要，因为不写这个模块的情况下会自动继承)
	{ % block header % }
		{ { parent() } }
	{ % endblock % }
	
####(3).内容的输出
	
	#控制器中
	return array('content'=>"内容");
	#视图中
	{ { content } }
	#使用函数
	{ { content|upper } }