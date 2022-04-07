git add .  （注意后边有个  . ）   把修改的部分全部选中，你也可以不用  .  ，修改那个选中哪个，一般都是全部选中的
git commit -m"提交时提交的备注"   这一步是把刚才选中的东西提交到暂存区域，暂存区域是在你本地，并不在远程
git push  解决完冲突之后，就需要把你的暂存区域代码提交到远程仓库了。解决冲突之后运行这一步即可，然后远程仓库就可以看到你的代码修改了
git checkout -b local_branch_name origin/remote_branch_name // 在本地基于远程的 remote_barnch_name 新建一个 local_branch_name 分支
git push origin local_branch_name // 把本地分支推到远程

git branch --set-upstream-to=origin/local_branch_name // 绑定远程分支

git pull --prune // 拉取所有远程分支的改动，包括新创建的分支 --prune 刷新本地分支缓存，会清除掉本地已经删除的分支名称

git checkout local_branch_name // 直接切换分支

git branch --set-upstream-to=origin/local_branch_name // 分支绑定 git status 常看仓库的状态，

git diff 常看git中文件的修改和改动 git init 初始化仓库，这个一般只在刚开始使用

git clone 仓库地址 本地目录 把远程仓库的代码克隆到本地 git add 把修改的文件添加到本地暂存区，

git commit -m ‘提交信息’ 把本地暂存区的代码提交到本地版本仓库

git push origin 本地分支:远程分支 把本地分支推送到远程

git branch 查看本地分支列表

git branch -r 查看远程分支列表

git branch 分支名 源分支名 创建一个新分支默认是当前分支

git checkout 分支名 切换分支

git checkout -b 分支名 创建并且切换一个分支

git fetch 更新远程仓库分支信息到本地

git merge 分支名 合并指定分支到当前分支

git pull 更新分支信息并合并当前分支的源分支到当前分支

git log 查看分支的提交日志记录

gitk --all 查看提交信息