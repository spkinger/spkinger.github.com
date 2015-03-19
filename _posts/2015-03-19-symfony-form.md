---
layout: page
title: "Symfony 笔记3--表单"
description: "Symfony的表单部分"
category: "symfony"

---
{% include JB/setup %}

#9.表单生成

####(1).基本的生成

	#控制器中
	public function indexAction($name, Request $request)
    {
        $user = new User();
        $form = $this->createFormBuilder($user)
            ->add('email','email',array('label'=>'邮箱'))
            ->add('password')
            ->add('age')
            ->add('提交','submit')
            ->getForm();

        $form->handleRequest($request);

        if($form->isValid()){
            $em = $this->getDoctrine()->getManager();
            $em->persist($user);
            $em->flush();
        }

        return array('content'=>$name, 'form'=>$form->createView());
	}
	#视图中
	{{ form(form) }}

####(2).FORM模板的修改

	{ { form(form) } }可扩展为
	
	{ { form_start(form) } }
	{ { form_widget(form) } }
	{ { form_end(form) } }
	
	{ { form_start(form) } }
	<!--这里有多个-->
	{ { form_label(form.email) } }
	{ { form_errors(form.email) } }
	{ { form_widget(form.email) } }
	
	{ { form_end(form) } }

####(3).选项模板的修改
	
	#可修改下面这种每个单元
	{ % block email_widget -% }
		{ % set type = type|default('email') -% }
		{ { block('form_widget_simple') } }
	{ % endblock email_widget % }

	#对模板的修改可以放到view/form/my_form_widget.html.twig然后通过下面代码引入
	{ % form_theme from ['xxxBundle:Form:my_form_widget.html.twig'] % }