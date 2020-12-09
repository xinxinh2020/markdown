



```shell
git init  # 将当前目录加入git管理,会在当前目录创建一个.git目录,但此时该目录下所有文件都是untraced状态的
git add . # 添加所有修改(包括untraced文件)到暂存区(staging area)
git clone https://github.com/libgit2/libgit2 # 拉取代码仓库到本地
git status -s # 查看状态简洁信息
git diff # 比较工作区和暂存区的区别
git diff --staged # 比较暂存区和本地仓库的区别

git commit -a # 包括了'git add .'操作
git commit --amend # 修改上一个commit，和当前暂存区的内容一起合并成一个commit提交，如果从上个commit之后没有修改，则仅改变commit的描述

git rm  # 把文件删除并移出git管理
git rm --cached # 把文件移出git管理，但不删除文件
git mv a.txt b.txt # 把a.txt重命名为b.txt

git log -p -2 # 查看最近2个commit的修改详情(git diff)
git log --stat # 包含commit时输出的文件修改概览
git log --all # 显示所有分支的log

git remote # 查看项目的远程仓库名
git remote -v # 查看远程仓库及对应的地址
git remote show <remote> # 查看远程仓库详情，可以查看git pull命令会自动合并本地哪些分支，git push会自动push哪些分支，也可以看到远程哪些分支是你本地没有的，哪些分支是你本地有但远程以及把它们删掉的了
git remote rename origin origin1 # 修改远程仓库在本地显示的名字
git remote remove <remote> # 移除某个远程仓库
git remote add <name> <url> # 添加远程仓库

git branch # 查看分支
git branch -a # 查看所有分支，包括远程分支
git branch -v # 查看分支及其对应最新的commit
git branch --merged # 查看已经被合并到当前分支的分支,这些分支可以删除掉
git branch --no-merged # 查看还没有合并到当前分支的分支 
git branch -d <branchName> # 删除已merge分支
git branch -D <branchName> # 强制删除分支
git branch -m oldName  newName # 重命名本地分支

git fetch # 把远程仓库的数据拉取到本地仓库中(不合并)
git pull # 拉取远程仓库代码并合并到当前分支(需要当前分支设置了追踪某个远程分支，clone命令会自动设置master分支的追踪)
git push <remote> <branch> # 推送本地分支到远端

git stash # 存储工作区修改
git stash pop # 将第一个stash应用到工作区，并从栈顶删除
git stash apply # 将第一个stash应用到工作区，并且不删除栈中该stash
git stash drop # 从栈中删除stash，不应用到工作区

git reset HEAD filename # 把暂存区的文件移除到工作区
git checkout -- filename # 撤销工作区的修改(危险操作)

    # 撤回刚刚的commit，代码仍然保留
git reset --hard HEAD~1 # 撤回刚刚的commit，代码不保留
git reset --hard origin/dev # 强制用远程分支替换本地分支，代码不保留

git pull origin # 从远端拉取代码
git push origin [branch] --force # 强制用本地分支覆盖远程分支

git checkout -b login remotes/origin/login # checkout远程分支，并在本地重命名为login，并切换到本地login分钟

```

