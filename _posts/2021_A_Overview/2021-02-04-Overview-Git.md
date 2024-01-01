---
layout: post
title: "Git概要"
date: 2021-02-04
tag: Overview
---   

- [参考：玩转Git三剑客](https://time.geekbang.org/course/intro/100021601)



## 目录
* [补充：名词解释](#content0)
* [补充：好用的命令](#content01)
* [1、Git介绍](#content1)
* [2、Git基础](#content2)
* [3、Git基础（二）](#content3)
* [4、Git基础（三）](#content4)
* [5、Git基础（四）](#content5)
* [6、Git独自使用](#content6)
* [7、Git独自使用（二）](#content7)
* [8、Git独自使用（三）](#content8)
* [9、分支操作](#content9)
* [10、Git与GitHub简单同步](#content10)
* [11、Git多人协作](#content11)
* [12、Git集成使用禁忌](#content12)
* [13、分支开发工作流](#content13)


<!-- ************************************************ -->
## <a id="content0">补充：名词解释</a>

```shell
# 仓库
reponsitory    

# 分层的
hierarchical  

# 平铺的
flat            
```

```shell
# 刚创建未被管理   
？ 

# add添加,进行了add操作                       
a 

# delete 删除          
d  

# modify 修改          
m

# conflict冲突          
c 

# ignore忽略          
i

# 详细 --verbose
-v

# 终止
--abort

# 具有分叉分支需要指定如何解决：git pull --no-rebase; 或者 git pull --rebase;
You have divergent branches and need to specify how to reconcile them.

# 获取和更新状态
fetch and refresh status

# 此操作不能撤销
This operation cannot be undone. 

# 存储库是最新的。
The repository is up to date.
```

```shell
#  检出
check out  

# 提交 
commit   

# 丢弃
discard 

# 抓取
fetch  

# 推送远端
push   

# 拉取远端
pull             

# 合并
merge

# 变基
rebase
```


<!-- ************************************************ -->
## <a id="content01">补充：好用的命令</a>

查看某次提交的修改内容
```shell
# 通过commit的hash来查看
git show <commit-hash>

# 查看某次提交某个文件的修改
git show <commit-hash> <file>

```

<!-- ************************************************ -->
## <a id="content1">1、Git介绍</a>

git的基本配置查看   

```shell
# 帮助查看 
git help 
# 查看所有的帮助
git help -a


# git 版本查看
git --version 

# git 配置查看
git config --list
```


git的用户相关查看和配置   
```shell
# 当前仓库用户名查看和设置
git config user.name
git config user.name 'your_name'

# 当前仓库用户邮件查看和设置
git config user.email
git config user.email 'your_email@domain.com'


# 三个配置选项，--local 为默认选项
--local 
--global 
--system
```


<!-- ************************************************ -->
## <a id="content2">2、Git基础</a>

#### **一、git的三个区**  

<img src="/images/Git/git2_0.png" alt="img">

M（绿色）：文件内容有更新，执行add，暂存区     
A（绿色）：首次添加，执行add,暂存区      

M（红色）：文件内容有更新，未执行add，工作区           
？？(红色)：首次添加，未执行add，工作区         


#### **二、查看仓库状态**        
```shell
git status 

git status -s 
```

#### **三、文件操作**    
```shell
# 加入暂存区
git add .
git add filename


# 从暂存区移除
git restore --staged .
git restore --stated filename


# 从工作区移除(disgard)
git restore .
git restore filename
```



<!-- ************************************************ -->
## <a id="content3">3、Git基础（二）</a>

查看提交记录    
```shell
git log 

git log --oneline -n 3

git log --oneline --graph -n 3

# 显示内容更改
git log -p

# 过滤提交者
git log --author=xxxx
```

<!-- ************************************************ -->
## <a id="content4">4、Git基础（三）</a>

#### **一、.git目录下的两个文件**   

HEAD文件：存储了当前的工作分支   

config文件：仓库的配置信息


#### **二、.git目录下的两个子目录**   

**refs目录**  
refs/heads:本地分支          
refs/remotes:远程分支        
refs/tags:标签    


**objects目录**    

存放对象：commit/tree/blob  


#### **三、git对象**      
一次提交对应一个commit对象A     
commit对象A下有一个tree对象B     
tree对象B下面可以包含blob对象或tree对象        
blob对象指向的是文件，tree对象指向的是文件夹     

<img src="/images/Git/git4_1.png" alt="img">

<span style="font-weight:bold;color:red;">一次commit是对当时文件的一个快照</span>

```shell
# 查看对象的类型
git cat-file -t commitHash

# 查看对象的内容
git cat-file -p commitHash
```



<!-- ************************************************ -->
## <a id="content5">5、Git基础（四）</a>

#### **一、分离头指针**   
当我们需要尝试做一些东西的时候，可以使用分离头指针，   
如果最终发现结果不理想可以直接切换到其它分支丢弃提交    
如果最终发现结果是我们想要的可以创建一个新的分支来保存我们的提交    

```shell
# 分离头指针
git checkout commitHash

# 在分离头指针状态下创建一个新的分支来保存我们的提交 
git switch -c <new-branch-name>  
```


#### **二、进一步理解HEAD和branch**   
HEAD不仅可以指向某个分支，还可以指向某次具体的commit(分离头指针状态),     
但当HEAD指向某个分支时，也是指向这个分支的最后一次commit    

HEAD的用法
```shell
git diff HEAD^ HEAD
```


<!-- ************************************************ -->
## <a id="content6">6、Git独自使用</a>

**以下是针对未共享的提交，仅做了解**    
修改commit的message     
整理commit的message,比如多次提交整理成一次        



<!-- ************************************************ -->
## <a id="content7">7、Git独自使用（二）</a>

#### **一、比较差异**    

```shell
# 工作区与暂存区差异
git diff

# 暂存区与HEAD差异
git diff --staged


# 工作区与当前HEAD的差异
git diff HEAD


# 执行了git add . 命令之后，在执行 git diff 会发现输出未空
# xy：执行add后，修改内容同时属于暂存区和工作区，所以 git diff 没有差异
# xy：这个时候执行 git diff HEAD 显示的内容和执行 git diff --staged 显示的内容一致，也能佐证这一点
```

#### **二、其它比较方法**   
```shell
# 对比两个分支的不同
git diff temp master

# 对比两次commit的不同:hash2 与 hash1 做了哪些修改
git diff hash1 hash2

# 查看具体文件的不同
git diff hash1 hash2 -- index.html
```


#### **三、恢复暂存区和工作区**       
```shell
# 恢复暂存区
git restore --staged <file>

# 恢复工作区
git restore <file>
```

#### **四、重定向**     
```shell
# 重置HEAD指向
git reset HEAD --hard
```


<!-- ************************************************ -->
## <a id="content8">8、Git独自使用（三）</a>

#### **一、贮藏的使用**   

**1、贮藏的查看**      
```shell
# 查看下stash列表
git stash list

# 查看索引为1的贮藏的具体内容
git stash show -p 1
```

**2、添加和使用贮藏**      
```shell
# 贮藏当前内容,并清空暂存区,工作区
git stash push -m "忽略文件"


# 应用某一次贮藏
git stash apply --index 1
```

#### **二、忽略文件.gitignore**      
.gitignore文件与.git文件夹平级    
加入.gitignore的文件或文件夹应在未追踪时加入     
否则应该先删除 - commit - 重新添加文件 - 加入gitignore文件     


#### **三、远程仓库**   

#### **1、查看**   
```shell
# 显示详细信息
git remote -v    

# 显示具体仓库信息：origin
git remote show origin
```

#### **2、添加和删除**      
```shell
# 查看远程仓库
git remote -v

# 添加远程仓库
git remote add origin https://xxx

# 移除远程仓库
git remote rm origin
```


<!-- ************************************************ -->
## <a id="content9">9、分支操作</a>

#### **一、分支的查看**     
```shell
# 查看本地分支
git branch
# 查看本地分支的详细信息
git branch -v 


# 查看本地和远程分支
git branch -a
# 查看本地和远程分支的详细信息
git branch -av
```


#### **二、分支的切换**      
```shell
git checkout branchName 
```


#### **三、分支的创建**  
```shell
# 创建分支
git branch branchName
# 创建并切换分支
git checkout -b branchName
```


#### **四、分支的删除**  
```shell
# 删除分支
git branch -d branchName

# 强制删除分支
git branch -D branchName
```


#### **五、补充：git pull**    
```shell
# <remote> 是远程仓库的名称，通常是 origin。
# <branch> 是要拉取更新的远程分支的名称。
git pull <remote> <branch>

# 如果你已经设置了分支的上游分支（upstream），可以简化为：
git pull

# 拉取并变基（rebase）：
git pull --rebase

# 拉取并合并(merge):
git pull --no-rebase

# 只拉取不合并：
git pull --no-merge
```

#### **六、补充：git push**   

**1、git push**    
```shell
# 将本地的 master 分支推送到远程仓库 origin 的 develop 分支。
# origin 是远程仓库的名称，通常用于指代远程仓库的地址。
# master 是本地仓库的分支名，表示要推送的本地分支。
# develop 是远程仓库的分支名，表示要将本地分支推送到的目标远程分支。
git push origin master:develop


# 将本地的 master 分支的提交推送到远程仓库 origin 的同名分支
# git push origin master:master
git push origin master


# origin 是远程仓库的名称，它是在本地仓库中配置的远程仓库的别名。
# HEAD 是指代当前所在的本地分支的引用。
# refs/for/master 是指代远程仓库中的一个特定分支的引用格式。
git push origin HEAD:refs/for/master
```

**2、追踪**    
```shell
# <branch-name> 是你要推送的本地分支的名称。
# -u 或 --set-upstream 选项用于建立远程分支与本地分支的关联。
# 执行这个命令后，Git 将会建立本地分支与远程分支的关联关系，并将远程分支设为默认上游分支。
# 这样，之后的 git pull 和 git push 就可以省略远程分支的名称，Git 将自动使用建立的关联关系。
git push -u origin <branch-name>
```

**3、refs/for/master 这种分支名与普通的分支名有几个区别：**      
**审查流程：**refs/for/master 分支名通常在代码审查工作流程中使用，用于提交代码进行审查。它与普通分支名的区别在于，提交到这种特殊分支上的代码会触发代码审查工具的相应流程，例如自动分析、评审和讨论等。

**命名空间：**refs/for/master 使用了 Git 引用（references）的命名空间 refs，它是用于管理分支、标签和其他引用的命名空间。这种命名空间让 Git 能够更好地组织和管理引用，以便进行更精确的控制和操作。

**引用格式：**refs/for/master 是一种特定的引用格式，其中 for 表示用于提交代码进行审查。这种引用格式可以被代码审查工具识别，并触发相应的审查流程。普通分支名通常只是简单的分支名称，不包含特殊的引用格式。

总的来说，refs/for/master 分支名是一种特殊用途的引用格式，用于代码审查流程和提交流程控制。它与普通分支名在使用场景和命名空间上有所区别，并具有特定的引用格式

#### **七、Merge**    
分支合并有两种场景：fast-forward 和 diverged     
```shell
# 将dev分支合并到当前分支
git merge dev

# 合并远程分支到当前分支
git merge origin/dev
```


#### **八、Rebase**    

**1、rebase的使用**   

```shell
# 检出master分支,并且重放到dev分支
git checkout master

# 将当前分支的修改重放到目标分支dev
git rebase dev
```

```shell
git pull origin master
# 相当于下面的操作
git fetch origin master
git merge origin temp

git rebase origin master
# 相当于下面的操作
git fetch origin master
git rebase origin master
```

**2、重要**    
变基的作用是使提交历史更整洁      
变基操作的实质是丢弃一些现有的提交，然后相应地新建一些内容一样但实际上不同的提交。      
<span style="color:red;font-weight:bold;">总的原则是，只对尚未推送或分享给别人的本地修改执行变基操作清理历史， 从不对已推送至别处的提交执行变基操作</span>


#### **七、cherry-pick**    
```shell
# 检出master分支,并将某一次提交捡到master分支
git checkout master
git cherry-pick e17811c
```


<!-- ************************************************ -->
## <a id="content10">10、Git与GitHub简单同步</a>
```
创建github账号
配置SSH
创建github仓库
关联github仓库
```

<!-- ************************************************ -->
## <a id="content11">11、Git多人协作</a>
```
多人协作
多人协作2
```


<!-- ************************************************ -->
## <a id="content12">12、Git集成使用禁忌</a>
```
禁止使用git push -f
    危害
        如果不是fast-forward，Git是禁止我们向远程分支push的
        这是Git的一种保护机制
        git push -f 会强制push到远程分支

    危害2
        比如我们在本地 git reset 很好之前 --hard
        git push -f 我们将永久丢失我们的代码

```

```
禁止向集成分支执行变更历史的操作
```


<!-- ************************************************ -->
## <a id="content13">13、分支开发工作流</a>
```
长期分支
主题分支
```



----------
>  行者常至，为者常成！



