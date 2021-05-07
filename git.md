---
title: GIT
date: 2021-05-07 16:12:47
tags: GIT
categories: GIT
---
Git是一款免费、开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。
Git是一个开源的分布式版本控制系统，可以有效、高速的处理从很小到非常大的项目版本管理。Git 是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。官网参考[https://git-scm.com/](https://git-scm.com/)
<!-- more -->
#### 一、介绍
##### 1、Git特点
**优点：**
适合分布式开发，强调个体；
公共服务器压力和数据量都不会太大；
速度快、灵活；
任意两个开发者之间可以很容易的解决冲突；
离线工作。
**缺点：**
代码保密性差，一旦开发者把整个库克隆下来就可以完全公开所有代码和版本信息；
权限控制不友好；如果需要对开发者限制各种权限的建议使用SVN。
##### Git工作流程
**一般工作流程如下：**
1. 从远程仓库中克隆 Git 资源作为本地仓库；
2. 从本地仓库中checkout代码然后进行代码修改；
3. 在提交本地仓库前先将代码提交到暂存区；
4. 提交修改，提交到本地仓库；本地仓库中保存修改的各个历史版本；
5. 在需要和团队成员共享代码时，可以将修改代码push到远程仓库。
##### Git的几个核心概念
**Workspace：** 工作区，存放项目代码的地方
**Index / Stage：** 暂存区，用于临时存放你的改动，事实上它只是一个文件，保存即将提交到文件列表信息
**Repository：** 仓库区（或版本库），就是安全存放数据的位置，这里面有你提交到所有版本的数据。其中HEAD指向最新放入仓库的版本
**Remote：** 远程仓库，托管代码的服务器，可以简单的认为是你项目组中的一台电脑用于远程数据交换
#### 二、安装/命令
##### 1、简易安装
~~~kotlin
#yum search git
#yum install git
~~~
##### 2、安装指定版本
~~~kotlin

~~~
##### 3、连接
~~~kotlin
git config --global user.name "zhonghuameimei" #配置账号
git config --global user.eamil "email.com" #配置邮件
ssh-keygen -C 'email.com' -t rsa #生成ssh key
cd ~/.ssh #进入此文件夹
cat id_rsa.pub #查看ssh key并在github设置ssh key
ssh -T git@github.com #验证是否连接成功
~~~
##### 4、使用
~~~kotlin
git init directory #初始化指定目录为本地仓库
git remote add origin git@github.com:账户/training.git #指定远程仓库
git add file #添加文件到暂缓区，告诉git冲突已解决
git add . 全部文件添加暂缓区
git commit -m "注释" #提交文件到本地仓库
git push origin master #提交代码到远程仓库
~~~
##### 5、其他命令
~~~kotlin
git clone git@github.com:账户/training.git #下载远程项目
git status #查看仓库当前的状态，显示有变更的文件
git diff #比较文件的不同，即暂存区和工作区的差异
git reset #回退版本
git remote #远程仓库操作
git fetch #从远程获取代码库
git pull #下载远程代码并合并
git push #上传远程代码并合并
~~~
##### 5、分支命令
~~~kotlin
git branch branchname #创建分支
git checkout branchname #切换分支
git checkout -b branchname #创建分支并切换
git branch -d branchname #删除分支
git meger branchname #合并分支到当前分支
~~~
#### 三、整合hexo
##### 1、所需环境
git、node.js
##### 2、安装
~~~kotlin
npm install -g hexo-cli #一键安装hexo
cd /opt/hexo; hexo init #进入文件夹并初始化，作为hexo的本地仓库
hexo g #根据文件的md文件生成html文件
hexo s #启动hexo服务
hexo d #提交本地仓库到远程仓库（git）
~~~
##### 3、hexo介绍
中文文档[https://hexo.io/zh-cn/docs/](https://hexo.io/zh-cn/docs/)


