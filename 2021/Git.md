### 零散的点
1. Git中文乱码
	- 可以使用Git Bash,不会乱码
    - 修改配置,百度下
2. 引用
	- 引用就是指向某个commit的指针,或指向某个指针的指针
3. HEAD
	- 是1个引用
	- 永远指向当下的位置
	- HEAD通过分支指向了commit
      ```
      //HEAD指向了main分支,main分支指向了commit(df1bcaba8c485ea7eeb2458002dd2559614293ff)
      //引用origin/main 指向了commit(df1bcaba8c485ea7eeb2458002dd2559614293ff)
      //引用origin/HEAD 指向了commit(df1bcaba8c485ea7eeb2458002dd2559614293ff)
      commit df1bcaba8c485ea7eeb2458002dd2559614293ff (HEAD -> main, origin/main, origin/HEAD)
      Author: HuanHaiLiuXin <472868476@qq.com>
      Date:   Sun Oct 4 10:30:10 2020 +0800

          老子也想看
      ```
4. 分支
	- 创建分支: git branch feature1
      ```
      //HEAD依然指向 main分支
      //feature1 也指向了 commit(df1bcaba8c485ea7eeb2458002dd2559614293ff)
      commit df1bcaba8c485ea7eeb2458002dd2559614293ff (HEAD -> main, origin/main, origin/HEAD, feature1)
      Author: HuanHaiLiuXin <472868476@qq.com>
      Date:   Sun Oct 4 10:30:10 2020 +0800

          老子也想看
      ```
	- 切换到新分支 feature1
	- 切换分支,只是将HEAD从1个分支指向另1个分支
      ```
      //HEAD指向了feature1分支, feature1分支指向了commit df1bcaba8c485ea7eeb2458002dd2559614293ff
      //HEAD的指向,从main分支换到了feature1分支
      commit df1bcaba8c485ea7eeb2458002dd2559614293ff (HEAD -> feature1, origin/main, origin/HEAD, main)
      Author: HuanHaiLiuXin <472868476@qq.com>
      Date:   Sun Oct 4 10:30:10 2020 +0800

          老子也想看
      ```
5. git pull
	- origin/-- 分支
      - origin是远程仓库的名称
      - origin/-- 分支是远程仓库分支的本地镜像
      - **origin/-- 分支不是在本地直接操作的,一般在push/pull/fetch成功后自动更新**
      - **origin/HEAD: 这是一个永远跟随 origin/main 的引用，它最大的作用是用来标记默认分支**
      ```
      //origin/HEAD 永远跟随 origin/main 证明:
      1:上一次push成功后
      //HEAD -> main, origin/main, origin/HEAD 都指向同一个commit
      commit 66ad5140520b814935bfd5ba7e99e67023414331 (HEAD -> main, origin/main, origin/HEAD)     Author: HuanHaiLiuXin <472868476@qq.com>
      Date:   Sun Oct 4 13:53:57 2020 +0800

          房租也高

      2:本地修改,提交一次commit后
      //HEAD -> main 指向最新提交
      //origin/main, origin/HEAD还是指向之前的commit
      commit 6a6af9d246ae0530f7db4c5a3b104c27e48dc287 (HEAD -> main)
      Author: HuanHaiLiuXin <472868476@qq.com>
      Date:   Sun Oct 4 14:48:32 2020 +0800

          测试origin/HEAD

      commit 66ad5140520b814935bfd5ba7e99e67023414331 (origin/main, origin/HEAD)
      Author: HuanHaiLiuXin <472868476@qq.com>
      Date:   Sun Oct 4 13:53:57 2020 +0800

          房租也高
      3:再次push成功后
      //HEAD -> main, origin/main, origin/HEAD 都指向最新push成功的commit
      commit 6a6af9d246ae0530f7db4c5a3b104c27e48dc287 (HEAD -> main, origin/main, origin/HEAD)     Author: HuanHaiLiuXin <472868476@qq.com>                                                     Date:   Sun Oct 4 14:48:32 2020 +0800

          测试origin/HEAD                       
      ```
  	- git pull 示意图
  	![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d118e360ab5b463f907e00431e4d97b2~tplv-k3u1fbpfcp-zoom-1.image)
6. git push
	- git push 示意图
	![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ebacc2c4317a4000bb09a30ae5077e42~tplv-k3u1fbpfcp-zoom-1.image)
	- git push成功后,本地仓库中的origin/HEAD的指向不一定会变化.
	- 只有push的是远程仓库的默认分支/main分支, origin/HEAD的指向才会变化.
7. commit
	- 本质上是当下提交距离上一次commit的改动
8. git add
	- 将指定文件,或全部变动文件放到暂存区
    - 暂存区:待commit的内容暂时存放的地方
9. 分支
	- branch本质上是一个引用,即指向某个commit的指针
    - branch只和它当下指向哪个commit有关
    - 默认分支,仓库的默认分支是master,main,默认分支可以修改
		- git clone拉取仓库,默认checkout的就是默认分支
		- 执行push命令,远程仓库的HEAD永远跟随默认分支,而不是和本地仓库HEAD同步,即只有push默认分支到远程仓库,远程仓库的HEAD才会移动
10. git merge
	- merge就是合并,用于将当前commit和指定commit进行合并,创建为1个新的commit
    - 当下位于main分支,将feature1的变动合并到main分支上: git merge feature1
	![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/812884fe5c034c1f8a18523a1932c0b7~tplv-k3u1fbpfcp-zoom-1.image)
	- 1.首先本地仓库将feature1的更改merge到main分支,然后push到远程仓库,push成功的示意图
	![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a7f85f53dda846f49729333543a0354b~tplv-k3u1fbpfcp-zoom-1.image)
	- 2.若main分支执行merge后,远程main分支已经有新的commit,则push会失败,这种情况,需要重新pull,并再次push.
      ```
      git checkout main
      git pull origin main
      git merge feature1
      //首次push失败,因为git merge后远程main分支提交了新的commit
      git push origin main
      //需要再次pull
      git pull origin main
      //然后再次push
      git push origin main
      ```
	![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/84331ed0bc3944f380c90df3a8fabd19~tplv-k3u1fbpfcp-zoom-1.image)
11. 删除本地分支
	- git branch -d 分支名称
12. 删除远程分支
13. git rebase
	![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/74ffcb42ce1e488dbd1a41aa27e942dc~tplv-k3u1fbpfcp-zoom-1.image)
	- git rebase -i 其他分支名称
14. git reset
	![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/af626bfb46014612bdfd358d3d16d33e~tplv-k3u1fbpfcp-zoom-1.image)
	- git reset --hard  会将当前分支指向对应的commit,并丢弃本地修改
15. git add -i 同一文件的多出修改，多次提交
```
git add -i
patch
指定文件序号，回车
y+回车：选用当前块
n+回车：不用当前块
s+回车：把当前块做自动切分后再重新询问
```
16. tag
	- tag一般用于标记版本号
	- 创建tag
		- git tag tag1
	- 显示所有tag
		- git tag
	- 显示指定tag
		- git show tag1
	- tag不能从本地改变位置，也不能被HEAD指向，同1个commit可以打多个tag
	- 删除本地标签
		- git tag -d tag1
	- 删除远程仓库标签
		- git push origin --delete tag1
17. git reflog 或 git reflog 分支名称
	- 用于查看HEAD 或 分支的移动历史，从而找到特定的commit
18. git checkout
	- git checkout 分支名称
    - git checkout commit
    - git checkout tag名称
19. git cherry-pick commitA commitX commitN
	- 一般用于将指定分支的指定几个commit应用到当前分支,并移动HEAD到commitN
20. 
