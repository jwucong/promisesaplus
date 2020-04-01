
# git
> [官方文档](https://git-scm.com/docs)

**命令列表:**

- [git init](#git-init)
- [git clone](#git-clone)
- [git add](#git-add)
- [git mv](#git-mv)
- [git rm](#git-rm)
- [git checkout](#git-checkout)
- [git commit](#git-commit)
- [git reset](#git-reset)
- [git revert](#git-revert)
- [git branch](#git-branch)
- [git tag](#git-tag)
- [git merge](#git-merge)
- [git rebase](#git-rebase)
- [git cherry-pick](#git-cherry-pick)
- [git stash](#git-stash)
- [git remote](#git-remote)
- [git fetch](#git-fetch)
- [git pull](#git-pull)
- [git push](#git-push)
- [git status](#git-status)
- [git log](#git-log)
- [git reflog](#git-reflog)
- [git show](#git-show)


### 0. 基本概念
> 图片来源: [阮一峰的网络日志 - 常用 Git 命令清单](https://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)

![git概念图](git.png)


**上图的几个名词解释:**

- Remote: 远端仓库
- Repository: 本地仓库
- Index: 索引，通常叫暂存区(stage)
- Workspace: 本地工作区，也叫工作目录(working tree)

### 1. 项目创建

#### git init

新建一个 git 仓库，可以指定一个文件夹名称。

```
git init [directory]
```

#### git clone

将一个已存在的 git 仓库下载到本地，可以指定一个文件夹名称。

```bash
git clone url [directory]
```


### 2. 文件操作(建立git快照)

#### git add

将 working 中的改动添加到 index 中，pathspec 是一个路径匹配模式，被这个模式命中的文件都会被添加。  

```bash
git add [options] [pathspec]
```

- 仅添加有更新的文件(不会添加新增的文件)：git add -u   
- 添加所有文件: git add -A   
- 添加指定文件: git add README.md   
- 添加一堆文件: git add a.md b.md c.md   
- 添加.md后缀名的文件: git add *.md


#### git mv

移动、重命名文件(夹)
   
```bash
git mv <from> <to>
```



#### git rm
   
删除文件(夹)

```bash
# -f: 强制删除
# -r: 递归删除
git rm [options] [pathspec]
```
   

#### git checkout

checkout除了切换分支功能外，也可以用于恢复文件，也就是使用目标commit下的文件直接替换当前文件，这是个危险的操作。
没有git add的改动如果执行了此操作，**耶稣也留不住它，我说的**！

> 网上说已经git add的改动可以通过git fsck --lost-found找回，但是我试了两下也还是无法找回，可能是运气不好  
> idea家的IDE自带的Local History功能应该可以抢救一下

```
# 不指定commitId则使用最新的那次commit
# git checkout -- README.md
git checkout [commitId] [--] [pathspec]
```


#### git commit

把改动记录到本地git仓库中，commit操作如果不加任何参数，会自动打开一个编辑器，让你写备注，通常是vi(m)编辑器。
commit操作有一个配置参数 --amend 可以修改上一次commit的备注。
   
```bash
git commit [options] [pathspec]
```


#### git reset

可以用来撤销git add 和git commit，此操作通常会配合以下三个配置选项：  

- --soft: 保留 index, 保留 workspace，
- --mixed: 撤销 index，保留 workspace，此配置为默认配置  
- --hard: 撤销 index，撤销 workspace

撤销git add比较简单，就是把文件状态从index退回到workspace

```
git reset --mixed [pathspec]  # 默认就是 --mixed 
```

撤销git commit，即版本回退

```bash
git reset [options] [commitId]
```

 
将当前的HEAD移动到commitId这个节点上，此命令的最终结果是，节点往后退，节点内容也往后退。

   
例如,当前的最新节点为G，想要退回到节点C(假设通过git log查到commitId为c0)，则:  
   
```bash
# 回退前节点图: A --- B --- C --- E --- F --- G (HEAD)
   
git reset c0
   
# 回退后节点图: A --- B --- C (HEAD)
```
   
通过上图可以看出，reset在回退时，删除了 E、F、G这三个节点，在git log中查询不到了(可以通过git reflog查到)，
同时，如果节点G已经push到远端，回退后造成了远端提交超前本地提交，再push时需要-f强制推送；
另外，因为--hard配置直接擦除修改的特性；所以，我们认为reset是一个危险的操作，
一般情况下，建议只在非公共分支或者需要回退的节点没有push到远端时才进行reset操作，
不建议在公共分支上使用此命令，除非公共分支上出现了一些"见不得人"的记录，
如果需要回退公共分支上的代码，推荐使用下面的git revert命令


#### git revert
> 使用此命令后会自动打开编辑器(通常是vi(m)编辑器，也可能是其他)让你去写此次 revert 操作的原因。

```bash
git revert [options] [commitId]
```
   
git revert也是一个回退操作，与reset不同的是，revert将使用一次新的commit来回退到指定的节点上，
此命令的最终结果是，节点往前推，节点内容往后退。
值得注意的是，此命令使用的commitId和reset使用的commitId有所不同，
这里需要的commitId是你需要回退到的目标节点的靠近 HEAD 那一侧的那个邻居节点的commitId(问号脸，哈？)
   
例如,当前的最新节点为G，想要回退到节点C，那么应该用E的commitId(假设为e0),则:
    
> 节点C的邻居是B和E，靠近 HEAD 那一侧的是E
   
```bash
# 回退前节点图: A --- B --- C --- E --- F --- G (HEAD)
   
git revert e0
   
# 回退后节点图: A --- B --- C --- E --- F --- G --- H (HEAD)
```

从上图看，revert操作产生了H这一个新的commit，它的commitId和C的不同，但是内容和C的一样，
所以说该命令的最终结果是：节点往前推，节点内容往后退。这就跟"这场战争使我们经济倒退30年"这句话是一样的，
时间不断的往前，但是在这个时间下的生活状态却跟30年前一样。这就是revert和reset最大的区别。
   
由于git revert操作完整的保留了commit历史，所以一般建议用于公共分支的代码回退。


### 3. 分支、标签管理

#### git branch

查看、切换、新建、删除、追踪分支

```
# 查看本地分支，-r: 查看远端分支；-a: 查看所有分支
git branch [options]

# 切换分支
git checkout <branchName>

# 新建分支但不切换过去
git branch <branchName>

# 新建分支并立即切换过去
git checkout -b <branchName>

# 删除本地分支，-f: 强制删除；-r: 解除与远程的追踪关系
git branch -d <branchName>

# 删除远程分支
git push <remoteName> -d <branchName>

# 本地分支与远端分支建立追踪关系，这样在使用git pull、git push时可以省略<remoteBranch>
git branch --set-upstream <localBranch> <remoteBranch>
```


#### git tag

为某次commit打上标签(取别名)，方便记忆，用法和上面的git branch差不多。tag有两种，一种是带注释的，一种是不带注释的。

```
# 查看所有标签
git tag

# 新建一个不带注释的tag，如果不指定commitId，则打在最新的commit上
# git tag v2.1.4
git tag <tagName> [commitId]

# 新建一个带注释的tag，如果不指定commitId，则打在最新的commit上
# git tag -a v2.1.4 -m "新增根据手机壳颜色自动切换主题的功能"
git tag -a <tagName> -m <remarks> [commitId]

# 删除本地tag
# git tag -d v2.1.4
git tag -d <tagName>

# 推送到远端
# git push origin v2.1.4
git push <remoteName> <tagName>

# 删除远端上的tag
# git push -d origin v2.1.4
git push -d <remoteName> <tagName>
```


#### git merge

合并分支，有时候在合并的时候会产生冲突(CONFLICT)，需要手动解决冲突；因此，通常建议在进行合并操作之前就先把冲突解决掉

```
git merge <branchName>
```

举个例子，赵四需要开发可以切换APP主题的功能，与是他从dev分支的C点切出一个新的分支dev-zhaosi，
很快，通宵两天他就搞定了，然后他把dev-zhaosi分支合入dev分支，
因为C点之后，dev上没有任何推进，所以此次合并没有任何冲突，很稳(图1):

```
              E --- F --- G     (dev-zhaosi)
             /             \
A --- B --- C ------------- H   (dev)
```


#### git rebase

基于指定分支，重新提交本分支上的提交。该操作会改变commit节点的顺序和commitId，是一个危险的命令，因此，**不要在公共分支上进行rebase操作**。
在熟悉该命令的前提下，建议在个人开发分支上进行该操作。

此命令的好处是在自己的开发分支解决冲突，不会把他人的提交当成自己的提交，不会引入额外的分支交汇，保持了git节点的单线性。

```
git rebase [options] <branchName>
```

<details>
<summary>rebase详细说明</summary>

rebase从字面意思上看就是再次base，既然是再次，说明在此之前，它曾经base过；
怎么理解，看图：

rebase前: 

```
              E --- F --- G      (dev-zhaosi)
             /                 
A --- B --- C --- H --- I --- J  (dev)
```

rebase后(默认使用pick): 

```
                                E' --- F' --- G'  (dev-zhaosi)
                               /                 
A --- B --- C --- H --- I --- J                   (dev)
```

rebase后(使用squash): 

```
                                K    (dev-zhaosi)
                               /                 
A --- B --- C --- H --- I --- J      (dev)
```


从上图看，base发生在C节点处；rebase在节点J处；
另外，你可能会疑问，在使用默认的pick的情况下，E、F、G在rebase之后变成了E'、F'、G'，是笔误吗？
不是，请再读一遍这句话："基于指定分支，**重新提交**本分支上的提交"。
重新提交(原文是Reapply commits)使得E、F、G的commitId变了。
使用squash的情况变化就更大了，三次提交直接就合并成了一次。

该操作如果遇到冲突，会进入一个临时分支中解决冲突，这样的好处是可以随时中断此操作而不会污染原分支。
在实际操作的过程中，极大概率会出现冲突需要你手动解决，
而且还有可能多次解决同一个冲突(可以使用squash解决)。

这是一个使用rebase squash模式的演示:

首先执行：

```shell
git checkout dev-zhaosi

git rebase -i dev
```

进入vim编辑器交互界面: 

```shell
  1 pick ed1450f t1
  2 pick 96293c4 t2
  3 pick 5bd4872 t3
  4 
  5 # Rebase 05034c8..5bd4872 onto 05034c8 (3 commands)
  6 #
  7 # Commands:
  8 # p, pick <commit> = use commit
  9 # r, reword <commit> = use commit, but edit the commit message
 10 # e, edit <commit> = use commit, but stop for amending
 11 # s, squash <commit> = use commit, but meld into previous commit
 12 # f, fixup <commit> = like "squash", but discard this commit's log message
 13 # x, exec <command> = run command (the rest of the line) using shell
 14 # b, break = stop here (continue rebase later with 'git rebase --continue')
 15 # d, drop <commit> = remove commit
 16 # l, label <label> = label current HEAD with a name
 17 # t, reset <label> = reset HEAD to a label
 18 # m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
 19 # .       create a merge commit using the original merge commit's
 20 # .       message (or the oneline, if no original merge commit was
 21 # .       specified). Use -c <commit> to reword the commit message.
 22 #
 23 # These lines can be re-ordered; they are executed from top to bottom.
 24 #
 25 # If you remove a line here THAT COMMIT WILL BE LOST.
 26 #
 27 # However, if you remove everything, the rebase will be aborted.
 28 #
 29 # Note that empty commits are commented out
```

可以看到1、2、3行就是dev-zhaosi上的提交，默认使用pick模式，5 - 29行都是注释教你怎么用rebase命令，
由说明可知，想用squash模式就需要把2、3行开头的pick改为s,所以我们去编辑一下，改为: 

> 为什么第一行不改为s,因为必须有一次pick作为squash的合入承载，你也可以试一下把第一行改为s，然后等报错查看报错原因。

```shell
  1 pick ed1450f t1
  2 s 96293c4 t2
  3 s 5bd4872 t3
```

改成上面的样子之后，保存并退出vim编辑器，然后，如果有冲突，就会自动进入一个临时分支中要求你解决冲突，
解决完了以后，执行 git add -u，然后再执行 git rebase --continue 继续 rebase 过程直到完成。
如果中途不想 rebase 了，执行 git rebase --abort 即可终止 rebase 操作。

</details>

<details>
<summary>merge与rebase的区别</summary>

假设: 项目组规定，dev分支作为公共的开发分支，各个开发人员在dev的基础上切出自己的开发分支，
当自己负责的功能完成后再合入公共的dev分支。

赵四在git merge章节中有非常出彩的merge操作，没有任何冲突，而且他开发的APP主题切换功能很受欢迎。

一周后...

产品希望开发一个能够根据手机壳颜色自动切换APP主题的功能，
众望所归，该功能由赵四负责开发，由于中途和产品打了一架，当他想要提交代码的时候发现其他同事已经在dev上推了好几次，
然后分支变成了这个鬼样子：

```
              E --- F --- G      (dev-zhaosi)
             /                 
A --- B --- C --- H --- I --- J  (dev)
```

那么问题来了：~~这一架赵四请了几天病假~~ 赵四如何保证dev-zhaosi合入dev的时候不产生冲突?

赵四: 这还用想？用merge先把dev合入dev-zhaosi(J --> G)，过程中解决了引入H、I、J造成的冲突，
然后再把dev-zhaosi合入dev(G --> K)，完美:

> dev-zhaosi上直接pull dev也是一样的，因为pull会自动merge

```
              E --- F ---------- G      (dev-zhaosi)
             /                 /   \
A --- B --- C --- H --- I --- J --- K   (dev)
```

这么一顿猛如虎的操作后，我们发现引入了J - G这一次不必要的交汇，于是: 

赵四仔细想了想，我只是想避免dev-zhaosi合入dev出现冲突而已啊，
可以在dev-zhaosi上rebase dev, 然后在dev上merge dev-zhaosi

在dev-zhaosi上rebase dev后，过程中解决了引入H、I、J造成的冲突，

```
                                E' --- F' --- G'  (dev-zhaosi)
                               /                 
A --- B --- C --- H --- I --- J                   (dev)
```

然后在dev上merge dev-zhaosi: 


```
A --- B --- C --- H --- I --- J --- E' --- F' --- G' (dev)
```

这时候，由于dev-zhaosi和dev没有差异，所以节点图从头到尾都是一条线，没有支线。
如果有代码review机制的话，这样很方便review。

</details>




#### git cherry-pick

从一个分支上挑选一个或一堆commit应用到另一个分支上。
要注意commit时间，end_commit的时间必须晚于start_commit。

例如，赵四在dev上pick dev-zhaosi上的提交
```
git checkout dev

# pick一次commit
git cherry-pick <commitId>

# pick从(start, end] 不包含start
git cherry-pick <start_commit>..<end_commit>

# pick从[start, end] 包含start
git cherry-pick <start_commit>^..<end_commit>
```


#### git stash

本地记录栈，通常用于储存正在改的东西，以便获得一个干净的workspace。

```
# 查看栈
git stash list

# 把本地改动存入栈中， -m可以写备注
# 如果有新增文件之类的，需要先执行git add，推荐有没有都执行一遍更香
# 低版本可能不支持push，而是save
git stash push [options] [pathspec]

# 应用某次stash记录, stash@{1}是通过git stash list查到的id
git stash apply stash@{1}

# 删除某次stash记录, stash@{1}是通过git stash list查到的id
git stash drop stash@{1}

# 应用并删除栈顶记录
git stash pop

# 清除记录栈
git stash clear
```


### 4. 本地与远端的同步(拉取和推送)


#### git remote

远端仓库管理

```shell
# 查看
git remote

# 新增
# git remote add origin xxxx
git remote add <name> <url>

# 删除
git remote remove <name>

# 查看某个remote的url
git remote get-url <name>

# 更新某个remote的url
git remote set-url <name> <new_url> [old_url]
```


#### git fetch

把远端拉取到本地，但不自动merge

```
git fetch [options]
```


#### git pull 

把远端拉取到本地，并自动merge

```
git pull <remote_name> <branch_name>

# 如拉取远端的dev分支
git pull origin dev

# 上面这个操作等于下面这两个操作
git fetch origin dev
git merge origin/dev
```


#### git push 

把本地推送到远端

```
git push <remote_name> <branch_name>
```


### 5. 查看

#### git status

查看workspace中文件的改动状态

```bash
git status
```



#### git log

查看提交记录, log 的 options 参数比较多，很灵活，列一些常用的

```
git log [options] [--] [path​]

# 查看README.md的提交历史
git log -p -- README.md

# 显示差异
git log -p  

# 每次提交显示在一行
git log --oneline  

# 显示某个作者的提交
git log --author=zhaosi

# 显示提交者的提交，
# author是具体修改代码的人
# committer是执行git commit这个操作的人
# 通常，我们自己改代码，自己提交，所以author和committer都是自己
git log --committer=zhaosi

# 关键词查询，列出所有提交的改动代码中出现searchString的commit
git log -S <searchString>

# 自定义输出格式,formatter可以使用替换字符
git log --pretty=format:"formatter"

git log --pretty=format:"commitId: %H%n提交人: %cn%n提交日期:%cd%n提交信息: %s%n" --date=format:"%Y-%m-%d %H:%M:%S"
```

formatter中的一些常用替换字符:

| 替换字符 | 表示 |
| :---: | :--- |
| %n | 换行 |
| %% | % |
| %H | 完整的commit hash值，即文中提到的commitId |
| %h | 短的commit hash值 |
| %an | 作者名 |
| %ae | 作者邮箱 |
| %ad | 作者修改日期,值又可以使用--date="dateFormatter"进行格式化日期 |
| %cn | 提交者名 |
| %ce | 提交者邮箱 |
| %cd | 提交日期,值又可以使用--date="dateFormatter"进行格式化日期 |
| %s | 提交说明 |


dateFormatter中的一些常用替换字符:

| 替换字符 | 表示 |
| :---: | :--- |
| %Y | 年 |
| %m | 月 |
| %d | 日 |
| %H | 时 |
| %M | 分 |
| %S | 秒 |


#### git reflog 

这里记录了几乎所有的git操作，包括那些被git reset删除的commit也都记录有

```shell
git reflog [options]
```

#### git show

用于查看各种类型的对象,用法和log差不多

```shell
git show [options] [object]
```

例如，使用git fsck --lost-found找到一些dangling blob，然后用git show来查看这些dangling blob的内容

```shell
git fsck --lost-found

# 得到
# dangling blob 42bc81e1bfadd5b1f787d4d75d43a918c495215f
# dangling blob 4df0c6a02c9732155e85aec37f6be0f367a2119b
# dangling commit 50d48098dc1c50797a04138c0e6decb59b499130
# dangling commit 5b0c93bc47099a69bcbf347d446cc0a38f92c174

# 查看
git show 42bc81e1bfadd5b1f787d4d75d43a918c495215f
```
