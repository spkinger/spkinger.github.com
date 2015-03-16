---
layout: page
title: "Symfony 笔记2--数据库操作"
description: "Symfony使用的数据库框架Doctrine"
category: "symfony"

---
{% include JB/setup %}

#8.数据库设置

####(1).生成set、get方法和repository
	
	#需要先在src/Spkinger/WebBundle/Entity下建立User.php文件
	#内容如下
	<?php

	namespace Spkinger\WebBundle\Entity;

	use Doctrine\ORM\Mapping as ORM;

	/**
	 * @ORM\Entity(repositoryClass="UserRepository")
	 * @ORM\Table(name="user")
	 */
	class user{

        /**
         * @ORM\Id
         * @ORM\Column(type="integer")
         * @ORM\GeneratedValue(strategy="AUTO")
         */
        protected $id;

        /**
         * @ORM\Column(type="string")
         */
        protected $username;

        /**
         * @ORM\Column(type="string")
         */
        protected $password;
    }
	#生成(更新)命令
	app/console generate:doctrine:entities SpkingerWebBundle
	
####(2).数据库连接配置

	#数据库配置文件
	编辑app/config/parameters.yml
	
####(3).库和表的建立
	
	#创建数据库(会创建上面配置文件中的数据库)
	app/console doctrine:database:create
	
	#在数据库中建表并更新表结构
	app/console doctrine:schema:update (这条不会有实际操作)
	app/console doctrine:schema:update --force (强制执行更新表结构)
	app/console doctrine:schema:update --dump-sql (仅打印要操作的sql)
	
####(4).表之间一对一的关系建立

	#编辑两个表对应的src/Spkinger/WebBundle/Entity下的php文件
	
	#引入两个文件
	use Doctrine\ORM\Mapping\OneToOne;
	use Doctrine\ORM\Mapping\JoinColumn;
	
	#分别添加关联对方的类属性
	#如下
	#这里，
	targetEntity="user"是指关联user；
	inversedBy="profile"指双向关联到profile自己这里（不写就是单向关联）；
	JoinColumn(name="user_id",referencedColumnName="id")是指在本表建立一个user_id字段，并关联到对方表的id字段
	class profile{
	
		...
	
		/**
		 * @OneToOne(targetEntity="user",inversedBy="profile")
		 * @JoinColumn(name="user_id",referencedColumnName="id")
		 */
		private $user;
	}
	
	class user{
		
		...
		
		/**
		 * OneToOne(targetEntity="profile",mappedBy="user")
		 */
		 private $profile;
	}
	
	#先执行如下代码，更新Entity下的php控制文件
	app/console generate:doctrine:entities SpkingerWebBundle
	
	#再执行
	app/console doctrine:schema:update --force (强制执行更新表结构)
	
#9.数据库操作

####(1).简单的插入

	$user = new User();
	$user->setAge(19);
	$user->setEmail("xxx@xx.com");
	$user->setPassword('xxx');
	
	$em = $this->getDoctrine()->getManager();
	$em->persist($user);
	$em->flush();

####(2).关联的数据插入
	
	use Spkinger\WebBundle\Entity\UserRepository;
	use Spkinger\WebBundle\Entity\Book;

	$em = $this->getDoctrine()->getManager();
	$user = $em->getRepository("SpkingerWebBundle:User")-findOneBy(array('id'=>1));

	$book1 = new Book();
	$book1->setTitle("book1")->setPrice(10)->addUser($user);
	
####(3).关联数据的查询

	/** @var $user \Spkinger\WebBundle\Entity\User */
	$user = $em->getRepository("SpkingerWebBundle:User")-findOneBy(array('id'=>1));

	foreach($user->getBooks() as $book){
		echo $book->getTitle();
	}

####(4).更新数据库

	$em = $this->getDoctrine()->getManager();
	$user = $em->getRepository("SpkingerWebBundle:User")-findOneBy(array('id'=>1));
	$user->setAge(20);
	$em->persist($user);
	$em->flush();

####(5).删除数据

	$em = $this->getDoctrine()->getManager();
	$user = $em->getRepository("SpkingerWebBundle:User")-findOneBy(array('id'=>1));
	$em->remove($user);
	$em->flush();

####(6).数据查询1--直接获取单条数据、根据id绑定数据库

	/**
	 * @Route("/book/show/{id}")
	 * @ParamConverter("book", class="SpkingerWebBundle:Book")
	 */
	public function showBookAction(Book $book){
		return new Response($book->getTitle());
	}

####(7).数据查询2-find的使用

	/** @var $books \Spkinger\WebBundle\Entity\Book */
	$books = $em->getRepository("SpkingerWebBundle:Book")
		->findBy(
				array("title"=>"book1"),
				array("price"=>"DESC")
		);
	#或者
	$books = $em->getRepository("SpkingerWebBundle:Book")
		->findByTitle("book1");

	echo $books[0]->getTitle();

####(8).使用自定义方法查询

	表对应的repository文件中添加
	public function getAllValidByTitle($title){
		return $this->findBy(array("title"=>$title));
	}
	
	控制器里面可以这样调用
	$books = $em->getRepository("SpkingerWebBundle:Book")
		->getAllValidByTitle("book1");

####(9).自动更新创建和编辑时间

	#表对应的entry类外注释增加 @ORM\HasLifecycleCallbacks();
	如下
	/**
 	 * @ORM\Entity(repositoryClass="BookRepository")
 	 * @ORM\Table(name="book")
 	 * @ORM\HasLifecycleCallbacks();
 	 */
 	 
 	 #添加字段
 	 /**
      * @ORM\Column(type="datetime")
      */
     protected $createAt;

     /**
      * @ORM\Column(type="datetime")
      */
     protected $updateAt;
     
     #增加方法，在persist（创建）之前执行，在update（更新）之前执行
     /**
      * @ORM\PrePersist()
      */
     public function PrePersist(){
         if($this->getCreateAt() == null){
             $this->setCreateAt(new \DateTime('now'));
         }

         $this->setUpdateAt(new \DateTime('now'));
     }
     /**
      * @ORM\PreUpdate()
      */
     public function PreUpdate(){
         $this->setUpdateAt(new \DateTime('now'));
     }
     
     #注意更新表结构和set、个头方法
     app/console doctrine:schema:update --force
     app/console generate:doctrine:entities SpkingerWebBundle

####(10).直接执行sql语句

	注意这里返回的是mysql原生的结果集(数组)
	$this->get('database_connection')->fetchAll("SELECT * FROM book WHERE book.id = 1");
	
####(11).手动控制事务

	(1)使用try{}catch{}
	$em->getConnection()->beginTransaction();
	try{
		$book2 = $em->getRepository("SpkingerWebBundle:Book")
				->findOneBy(array("title"=>"book1"));
		$book2->setPrice(11);
		$em->persist($book2);
		$em->getConnection()->commit();
	}catch(Exception $e){
		$em->getConnection()->rollback();
	}
	
	(2)使用框架自身方法
	$em->transactional(function($em){
		$book2 = $em->getRepository("SpkingerWebBundle:Book")
			->findOneBy(array("title"=>"book1"));
		$book2->setPrice(11);
		$em->persist($book2);
	});

####(12).执行sql返回（symfony可用的）对象

	#注意：这样搜索结果不全时，使用对象调用不存在的set、get会报错
	$sql = "SELECT
			partial b.{id,title}
			partial u.{id,email}
			FROM SpkingerWebBundle:Book b join b.users u"

####(13).打印sql结果

	$query = $this->getDoctrine()
			->getManager()
			->createQuery("SELECT u FROM SpkingerWebBundle:User u");
	Debug::dump($query->getResult());
	exit();