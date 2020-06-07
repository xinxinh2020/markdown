参考资料：https://git-scm.com/book/en/v2/Getting-Started-What-is-Git%3F



其他的版本控制一般把数据和它们的修改保存为文件（delta-based，有点像增量备份）。git则把版本当做快照看待。

git管理的文件都是记录checksum值的，所以一旦文件内容发生了变化，git一定会知道。git数据库中保存的不是文件的名字，而是