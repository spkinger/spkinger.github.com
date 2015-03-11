---
layout: page
title: "Symfony 笔记2--数据库操作"
description: "Symfony使用的数据库框架Doctrine"
category: "symfony"

---
{% include JB/setup %}

#8.数据库

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