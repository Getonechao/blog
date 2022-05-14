+++
title= "Git常用命令"
description= "Git常用命令"
date= 2022-05-14T20:30:47+08:00
author= "somebody"
draft= false
image= "" 
math= true
categories= [
    "app"
]

tags=  [
    " git"
]

+++

# git常用指令

# git 流程图



![img](images/git.jpg)

## git指令



#### 1 分支操作

~~~shell
1查看  
    //查看本地所有分支 
    git branch 

    //查看远程所有分支
    git branch -r 

    //查看本地和远程的所有分支
    git branch -a 

2新建
    //新建分支
    git branch 本地分支名

    //新建一个本地分支并切换到该分支
    git checkout -b 本地分支名

3删除
    //删除本地分支
    git branch -d <branchname> 

    //删除远程分支
 	git push origin :XXX 

4重命名
    //重命名本地分支
    git branch -m <oldbranch> <newbranch> 

5关联远程
    //本地分支与远程分支建立关联
    git branch -u origin/分支名 
    git branch --set-upstream-to=origin/sit-basic-v1.0.1

    //撤销本地分支与远程分支的关系
    git branch --unset-upstream

    //查看本地分支与远程分支的映射关系
    git branch -vv
    
6合并分支
	//切换到master分支
    git  checkout master
    //将develop分支合并到master分支
    git  merge develop
    

~~~

#### 2 fetch+merge操作

~~~shell
	//拉取远程分支
    git fetch origin
    git fetch origin master

    //冲突查看
    git log -p FETCH_HEAD

    //将拉取下来的最新内容合并到当前所在的分支中
    git merge FETCH_HEAD   
    git merge [远程主机名]/[branch] --allow-unrelated-histories
~~~

#### 3 push操作

~~~shell
	git push <远程主机名> <远程分支名>:<本地分支名>
~~~

#### 4 remote操作

~~~shell
1.查看当前仓库
	git remote -v
	
2.新增远程仓库
	git remote add [name] [url]
	
3.删除远程仓库
	git remote remove [name]

4.修改远端仓库地址
	git remote set-url [have_a_name] [url]

5.修改远端仓库名字
	git remote rename <old_name> <new_name>
	
6.同步本地仓库与远程仓库的分支
	//场景：有些分支在远程其实早就被删除了，但是在你本地依然可以看见这些被删除的分支
	git remote prune [远程仓库名]
~~~

