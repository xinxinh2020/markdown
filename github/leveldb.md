[TOC]

## 一些常识

- leveldb 只允许打开一个 db 连接，如果打开连接后没有 close 掉，其他打开 db 连接的尝试都会失败，报 `resource temporarily unavailable`

- 进程退出后，进程持有的 db 连接锁会自动释放掉

- 数据库连接支持使用 `ReadOnly` 打开，该模式下的访问性能会快很多：

  ```go
  db, err := leveldb.OpenFile("/tmp/tmp/db", &opt.Options{
  		ReadOnly: true,
  })
  ```

- 使用 `ReadOnly` 模式打开时，需要保证该数据库已经存在，否则会报错 `stat /tmp/tmp/db: no such file or directory`

- 如果打开一个数据库后，将该数据库目录删除，此时程序执行 Put 方法向数据库中写入数据，程序不会报错，但也不会将 db 目录创建出来

- 每次打开一个 rw 的数据库，会创建一个 log 文件，之后的写操作都写在这个 log 文件上，并不会每次写都创建一个 log 文件

## opt.Option 参数

- ErrorIfMissing: 设为 true 时，打开的数据库文件如果不存在，则保持

## 原理

### MemTable

https://cloud.tencent.com/developer/article/1625049

- 最多会维护两个 MemTable: mem_ 和 imm_。mem_可以读写， imm_ 只读
- leveldb 最新写入的数据都会保存到 mem_ 中，的大小超过 write_buffer_size 时，LevelDB 就会将其切换成 imm_，并生成新的 mem_
-  LevelDB 的后台线程会将 imm_ compact 成 SSTable 保存在磁盘上。
- 如果前台的写入速度很快，有可能出现 mem_ 的大小已经超过 write_buffer_size，但是前一个 imm_ 还没有被 compact 到磁盘上，无法切换 MemTable，此时就会出现 [stall write（阻塞写请求）](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fgoogle%2Fleveldb%2Fblob%2F1.22%2Fdb%2Fdb_impl.cc%23L1336)

### Log

这里的 log 是指 [Write Ahead Log](https://links.jianshu.com/go?to=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FWrite-ahead_logging)。前面说了，LevelDB 写入的数据会先保存到 MemTable。为了防止宕机导致数据丢失，在将数据写入 MemTable 之前，会先将数据持久化到 log 文件中

### SSTable

https://cloud.tencent.com/developer/article/1625051

- SSTable 全称 Sorted String Table，顾名思义，里面的 key-value 都是有序保存的
- 除了两个 MemTable，LevelDB 中的大部分数据是以 SSTable 的形式保存在外存上
- SSTable 由 compaction 生成：Minor Compaction 和 Major Compaction
- Minor Compaction：一个 MemTable 直接 dump 成 level-0 的一个 SSTable
- Major Compaction：多个 SSTable 进行合并、重整，生成 1～多个 SSTable。

### Manifest

- 内容上，Manifest 文件保存了整个 LevelDB 实例的元数据，比如：每一层有哪些 SSTable
- 格式上，Manifest 文件其实就是一个 [log 文件](https://links.jianshu.com/go?to=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FRy7FRClFz8XRut_fSYlX2g)，一个 log record 就是一个 [VersionEdit](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fgoogle%2Fleveldb%2Fblob%2F1.22%2Fdb%2Fversion_edit.h%23L29)
- 

