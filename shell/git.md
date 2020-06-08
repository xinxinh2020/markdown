



```shell
git init  # 将当前目录加入git管理,会在当前目录创建一个.git目录,但此时该目录下所有文件都是untraced状态的
git add . # 添加所有修改到暂存区(staging area)
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

git branch # 查看分支
git branch -va # 查看所有分支，包括远程分支
git branch -d [branchName] # 删除已merge分支
git branch -D [branchName] # 强制删除分支
git branch -m oldName  newName # 重命名本地分支

git reset HEAD filename # 把暂存区的文件移除到工作区
git reset --soft HEAD^  # 撤回刚刚的commit，代码仍然保留
git reset --hard HEAD~1 # 同上
git reset --hard origin/dev # 强制用远程分支替换本地分支，代码不保留

git pull origin # 从远端拉取代码
git push origin [branch] --force # 强制用本地分支覆盖远程分支

git checkout -b login remotes/origin/login # checkout远程分支，并在本地重命名为login，并切换到本地login分钟

```

