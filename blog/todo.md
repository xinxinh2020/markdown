

Cobra 各种 flags 的作用



### Persistent Flags

对该命令及其子命令都有效，全局 flags 可以通过在 root 命令定义 Persistent Flags 实现。

```go
rootCmd.PersistentFlags().BoolVarP(&Verbose, "verbose", "v", false, "verbose output")
```



### Local Flags

仅对于当前命令有效：

```shell
localCmd.Flags().StringVarP(&Source, "source", "s", "", "Source directory to read from")
```

默认情况 cobra 只会解析当前命令的 local flags，对当前命令的父命令的 loacl flags 是不解析的，可以通过`Command.TraverseChildren`开启：

```go
command := cobra.Command{
  Use: "print [OPTIONS] [COMMANDS]",
  TraverseChildren: true,
}
```

