参考资料：https://git-scm.com/book/en/v2/Getting-Started-What-is-Git%3F



其他的版本控制一般把数据和它们的修改保存为文件（delta-based，有点像增量备份）。git则把版本当做快照看待，每一次commit都类似于给项目打了个快照。

git管理的文件都是记录checksum值的，所以一旦文件内容发生了变化，git一定会知道。git数据库中保存的不是文件的名字，而是checksum.

commit操作会把数据存储到本地数据库中。数据库中的数据是被压缩过的。提交会永久地把快照存储到git目录中。

和其他版本控制工具不同，git鼓励多创建分支和进行merge操作。分支在git中只是一个包含40个字符的sha-1 checksum的文件，所以创建一个分支只需要写41个字符(包含一个换行符)到一个文件中即可。而其他版本控制工具大多需要拷贝项目的所有文件到另外一个目录中。

Fast-Forward：当前分支只是简单地比要合并的分支落后几个提交时，git合并的时候只需要把HEAD指针向前移动几个commit即可，因此叫Fast-Forward。

remote/xxx是Remote-tracking分支，这些分支追踪的是remote仓库上的分支，这些分支的指针精确地追踪remote分支上的状态，无法通过普通操作移动它们的指向，只有fetch/pull/push等和remote仓库通信的操作才可以同步它们的指向。



## 三种状态

1. working tree是项目一个版本的检出(checkout)，这个版本中的文件会从.git的数据库中解压出来使用和修改。
2. staging area是一个文件，通常是在git目录下，它存放了下次提交的信息。
3. .git目录(Repository)：git目录存放着项目的元数据和对象数据库。clone命令拷的就是这个目录。

