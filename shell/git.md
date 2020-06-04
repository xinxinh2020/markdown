



```shell
git add . # 添加所有修改
git branch # 查看分支
git branch -va # 查看所有分支，包括远程分支
git branch -d [branchName] # 删除已merge分支
git branch -D [branchName] # 强制删除分支
git branch -m oldName  newName # 重命名本地分支

git reset --soft HEAD^  # 撤回刚刚的commit，代码仍然保留
git reset --hard HEAD~1 # 同上
git reset --hard origin/dev # 强制用远程分支替换本地分支，代码不保留

git pull origin # 从远端拉取代码
git push origin [branch] --force # 强制用本地分支覆盖远程分支

git checkout -b login remotes/origin/login # checkout远程分支，并在本地重命名为login，并切换到本地login分钟

```

