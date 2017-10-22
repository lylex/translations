# 代码审阅评论
-------------

原文：[Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments) 作者：Go team

这篇文章收集了Go代码review时候的常见的comments，因此一个单独的详细的解释就可以用速记来被作为参考。这只是常见错误做成的类似要洗的衣服的单子，不是风格指导。

你能把这个作为https://golang.org/doc/effective_go.html的补充。

*请在编辑这页请先讨论一下要改变的内容*，即便是最小的变化。很多人有很多不同的观点，而且这不是引发编辑战争的地方。

* [Gofmt](#Gofmt)
* Comment句子
* 上下文
* 复制
* Crypto Rand
* 申明空切片
* 文档评论
* 不要Panic
* 错误字符串
* 例子
* Goroutine生命周期
* 处理Error
* Imports
* Import Dot
* In-Band Errors
* Indent Error Flow
* 初始化
* Interfaces
* 行长度
* 混合大写
* Named Result Parameters
* Naked Returns
* Package Comments
* Package Names
* Pass Values
* Receiver Names
* Receiver Type
* Synchronous Functions
* Useful Test Failures
* Variable Names

## Gofmt

在你的代码上运行[gofmt](https://golang.org/cmd/gofmt/)来自动地修正大多数机械的格式问题。几乎所有的Go代码都使用`gofmt`。这篇文章的其余部分阐述了非机械格式点。

一个可选的方法就是使用[goimports](https://godoc.org/golang.org/x/tools/cmd/goimports)，一个`gofmt`的超集，它额外地作为必需品增加了（而且移除了）导入行。
