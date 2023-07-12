## 常用git命令

### 基础命令

``` bash
git status # 查看文件状态
git diff # 显示已暂存文件和已修改文件的区别
git clone 仓库地址 # 克隆仓库代码到本地
git checkout 分支名 # 检出已存在分支
git checkout -b 分支名 master #基于master分支新建分支
git fetch --all # 同步远程仓库的最新提交到本地仓库，不会改变当前工作区内容， --all是所有分支，不加则只同步当前分支
git commit -m '本次提交注释,描述主要变更内容' # 提交代码到本地仓库
git push origin dev # push本地新commit到远程仓库的dev分支
git branch -d feature-xx #删除本地的feature分支
git branch -a # 查看所有分支
git branch --set-upstream-to=remotes/origin/feature-oldInterface-improve origin/feature-oldInterface-improve # 设置本地分支跟踪远程分支
git remote add 仓库名 仓库地址 # 添加远程仓库
git remote -v # 查看远程仓库地址及协议
git remote rm 仓库名 # 删除远程仓库的关联，不会删除远程仓库本身，如果没有设置过仓库名默认是origin
git stash save 暂存说明 #暂存本地更改，不commit，用于临时存储本地更改
git stash # 暂存本地更改，不写暂存说明，git会生成暂存说明
git stash list # 列出暂存列表
git stash apply stash@{0} # 恢复 index 0 的暂存文件 也可以使用 --index
git stash pop # 恢复最后一次暂存的文件
git stash clear # 删除所有暂存历史记录
git rebase 属于高阶内容，可以修改提交历史，提交日志，有兴趣的自己了解
git pull origin dev # 更新本地dev 分支


```

### 检出远程分支到本地分支操作

```bash
git checkout -b dev #新建分支
git pull origin dev # 更新本地分支
git branch --set-upstream-to=remotes/origin/dev dev  # 设置dev分支跟踪远程dev分支
```

###  远程仓库设置

```bash
git remote 查看所有远程仓库
git remote xxx 查看指定远程仓库地址
git remote set-url origin xxxx 修改远程仓库地址
git remote rm origin 移除远程仓库
git remote add origin xxx 添加远程仓库地址
```

### 回滚git，并强制更新分支（操作慎重，会覆盖分支内容，仅允许单人单分支情况下操作）

```bash
1.找到回滚分支记录位置，右键点击“Reset Current Branch to Here...”
2.模式选择“Hard”
3.推送本地分支至远程分支，push的时候选择“Force Push”

Note : git  reset 是用来撤回代码的，本质就是修改当前的HEAD指针的指向（移动当前HEAD以及它所指向的branch）。 
  git撤回代码有三种方式，分别是--hard、--soft、--mixed。使用方式是在命令后面加上相应的HEAD或者commit  ID。如（git reset  --hard  <commit  ID>）。三种方式的区别总结如下。
  git reset  --hard  <commit ID >后，本地电脑上的工作区（working  tree ）内容、暂存区（index/stage）内容以及Repository的内容，都会恢复到<commit ID>那个节点的代码，并且之前<commit  ID>后面的所有<commit ID>都会被删除，所以，带上--hard参数后，所有未add的修改（也就是working tree ）的修改、暂存区的修改（已经add过）以及commit到Repository的修改都会被删除，在很明确地知道某些修改的不需要的情况下，就可以使用--hard参数。
  git reset --soft的话，工作区（working tree ）和暂存区（index/stage）的内容都不会被修改，不过--soft后所带来的差异（就是目标commit  ID）以后，所带来的所有的修改，会被保存到暂存区，也就是相当于git add 后但是没commit前，并且--soft后，目标commit  ID后面的所有commit ID也会消失，其实是任意一种方式都会这样，因为reset操作，本来就是要撤销提交的，那么目标commit  ID后面的commit ID肯定是要被删除的，要不就不叫撤销。
  git  reset --mixed 的话，只会保留工作区（working  tree） 的修改，暂存区以及--mixed带来的差异，会被全部放到工作区中，也就是相当于没有git  add之前的状态。如果这时候不想保留修改内容，可以直接git  checkout  <file>即可。
```
