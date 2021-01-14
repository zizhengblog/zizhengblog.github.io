---
layout: post
title: "9、Git与GitHub简单同步"
date: 2019-01-09
tag: Git
---   

- [参考：玩转Git三剑客](https://time.geekbang.org/course/intro/100021601)



## 目录
* [创建github账号](#content1)
* [配置SSH](#content2)
* [创建github仓库](#content3)
* [关联github仓库](#content4)



<!-- ************************************************ -->
## <a id="content1"></a>创建github账号

这个就不多说了

<!-- ************************************************ -->
## <a id="content2"></a>配置SSH

[参考这个链接：https://docs.github.com/cn/free-pro-team@latest/github/authenticating-to-github/connecting-to-github-with-ssh](https://docs.github.com/cn/free-pro-team@latest/github/authenticating-to-github/connecting-to-github-with-ssh)

<!-- ************************************************ -->
## <a id="content3"></a>创建github仓库

这个也不多说了

<!-- ************************************************ -->
## <a id="content4"></a>关联github仓库


1、指令

```
git remote add
```

2、演示

```
#先查看下当前的remote状态
bogon:gitLearning LC$ git remote -v
zhineng	file:///Users/LC/Desktop/remoteGits/zhineng.git (fetch)
zhineng	file:///Users/LC/Desktop/remoteGits/zhineng.git (push)

#添加远程仓库
#去github复制链接，通过SSH的方式添加
bogon:gitLearning LC$ git remote add github git@github.com:JiangHuHiKe/gitLearning.git

#添加完成后看下状态
bogon:gitLearning LC$ git remote -v
github	git@github.com:JiangHuHiKe/gitLearning.git (fetch)
github	git@github.com:JiangHuHiKe/gitLearning.git (push)
zhineng	file:///Users/LC/Desktop/remoteGits/zhineng.git (fetch)
zhineng	file:///Users/LC/Desktop/remoteGits/zhineng.git (push)


#看下本地分支情况
bogon:gitLearning LC$ git branch
  LC
  main
* master
  temp
bogon:gitLearning LC$ 


#我们将所有的分支push到github
bogon:gitLearning LC$ git push github --all
Warning: Permanently added the RSA host key for IP address '13.229.188.59' to the list of known hosts.
To github.com:JiangHuHiKe/gitLearning.git
 ! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'git@github.com:JiangHuHiKe/gitLearning.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
bogon:gitLearning LC$ 
```
因为本地的main分支没有基于远端的main分支做变更，所以不允许push
通过gik看下现在的分支状态,main分支没有推送成功

<img src="/images/Git/git9_0.png" alt="img">

根据提示我们先把远端的main分支fetch到本地

```
bogon:gitLearning LC$ git fetch github main
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), done.
From github.com:JiangHuHiKe/gitLearning
 * branch            main       -> FETCH_HEAD
 * [new branch]      main       -> github/main
bogon:gitLearning LC$ 
```

查看下分支的状态
```
bogon:gitLearning LC$ git branch -av
  LC                    2e7d469 Readd readme file
* main                  2e7d469 Readd readme file
  master                d984cc2 Add first commond
  temp                  2e7d469 Readd readme file
  remotes/github/LC     2e7d469 Readd readme file
  remotes/github/main   73a1073 Initial commit
  remotes/github/master d984cc2 Add first commond
  remotes/github/temp   2e7d469 Readd readme file
  remotes/zhineng/LC    2e7d469 Readd readme file
```

<img src="/images/Git/git9_1.png" alt="img">

我们通过merge的方式将本地main和github/main分支合并
```
bogon:gitLearning LC$ git merge  --allow-unrelated-histories github/main
```

<img src="/images/Git/git9_2.png" alt="img">

此时将main分支推送到github
```
bogon:gitLearning LC$ git push github main
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 4 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (2/2), 303 bytes | 303.00 KiB/s, done.
Total 2 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To github.com:JiangHuHiKe/gitLearning.git
   73a1073..73eb75f  main -> main
bogon:gitLearning LC$ 
```
<img src="/images/Git/git9_3.png" alt="img">

----------
>  行者常至，为者常成！



