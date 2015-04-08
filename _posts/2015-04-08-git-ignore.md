---
layout: page
title: "git忽略未提交文件和忽略已提交文件"
description: "git忽略文件的两种方法"
category: "git"
---
{% include JB/setup %}

##1.忽略未提交文件（Unstracked的文件）

    #在需要的目录下建立.gitignore文件
    vim .gitignore
    #里面添加要忽略的文件路径,如下
    xx/*
    *.cc
    #文件路径
    web
     |--.gitignore
     |--xx/
     |--yy.cc
     |--zz.cc

##2.忽略已提交的文件

    #忽略单个文件
    git update-index --assume-unchanged yy.cc
    #忽略文件夹（末尾不要加*）
    git update-index --assume-unchanged xx/
    #解除忽略
    git update-index --no-assume-unchanged xx/
    #查看忽略的文件
    git  ls-files -v
