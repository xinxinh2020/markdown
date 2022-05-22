

[TOC]

## some points

- cue 的重点在数据校验，而 jsonnet 的重点在 data templating (boilerplate removal)
- cue 为了降低复杂性，使用了强类型，放弃了一些灵活性



## Tutorals

### Types and Values

- """ : 多行文本
- "#"(相关数量的 # 对) ：Raw Strings
- close() : closed struct
- 以 # 或 _# 开头：定义一个 [definition](https://cuelang.org/docs/tutorials/tour/types/defs/)，Structs defined by definitions are implicitly closed.
- "\_|\_": 空对象，或错误
- let: 定义一个别名，别名只能在 struct 中使用
- 以下划线(_)开头并且没有在括号里的字段是隐藏字段，不会出现在输出中
- \ ( xxx ) : interpolation

#### Type Hie rarchy

Incomplete values 不能被当作 JSON

