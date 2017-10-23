# 代码审阅评论
-------------

原文：[Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments) 作者：Go team

这篇文章收集了Go代码review时候的常见的comments，因此一个单独的详细的解释就可以用速记来被作为参考。这只是常见错误做成的类似要洗的衣服的单子，不是风格指导。

你能把这个作为https://golang.org/doc/effective_go.html的补充。

*请在编辑这页请先讨论一下要改变的内容*，即便是最小的变化。很多人有很多不同的观点，而且这不是引发编辑战争的地方。

* [Gofmt](#Gofmt)
* [Comment句子](Comment句子)
* [上下文](上下文)
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

## Comment句子

看https://golang.org/doc/effective_go.html#commentary。评论文本申明应该是完整的句子，即使那看起来有点冗余。这种方法能够使它们在被提取成godoc文本的时候，可以很好地格式化。评论应当以正在描述的东西开头，并以句号结尾。

    // Request represents a request to run a command.
    type Request struct { ...
    
    // Encode writes the JSON encoding of req to w.
    func Encode(w io.Writer, req *Request) { ...

诸如此类。

## 上下文

context.Context类型的值承载着API和进程边界间的安全证书、跟踪信息、截止时间以及取消信号。
Go程序显式地在从进入的RPC和HTTP请求到出去的请求构成的整个函数调用链条间传递着Context。
大多数使用Context的函数应当以第一个参数的形式来接纳它。

    func F(ctx context.Context, /* other arguments */) {}

不是专门为了处理请求的函数可以使用context.Background()，但是在传递Context时候加上err，即使你认为你不需要。默认的情况是传一个Context；仅仅在你有一个好的理由能回答为什么其他备选方法是错误的时候，才直接使用context.Background()。

不要把Context作为成员加到一个struct类型中去；相反地，那个struct类型的某个方法需要一个Context往下传的话，就为它增加一个ctx参数。
一个例外就是有些特别的函数，那些函数的签名必须要满足一个标准库interface或者第三方库。

不要创造自定义的Context类型或者在函数签名中使用interface而不是Context。

如果你有应用参数需要传递的话，就把它放到参数中、在接受者中、在全局变量中，或者它真的属于那--Context的值中。

Context是不可改变的，你想把同样的ctx传递到多个共享着同样的截止时间、取消信号、证书以及母跟踪等等的调用中，是可以的。
