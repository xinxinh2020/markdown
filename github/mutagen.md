如果需要验证信息，create 会提示用户输入。

每个 Session 都有 name，label，id

flush 可以手动触发一次同步，可以开启 no-watch 模式关闭自动同步，配合 watchman 等工具来手动控制同步进程

mutagen 会使用 scp 命令来把 agent 的二进制文件拷到远程系统。

全局配置的位置：~/.mutagen.yml