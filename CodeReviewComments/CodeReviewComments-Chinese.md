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

## 复制

为了避免意想不到的别名，从其他的包中复制一个struct时候要小心点。
例如，bytes.Buffer类型包含了一个`[]byte`切片，以及一个这个切片可能指向的byte的小数组，作为对小字符串的一个优化。
如果你复制了一个`Buffer`，这个复制中的切片可能还是原来那个数组的别名，这可能会给后续的方法调用带来意向不到的效果。

通常说来，如果一个类型的方法关联着它的指针类型`*T`的时候,那就不要复制这个`T`类型的值。

## 申明空切片

申明一个空切片的时候，

`var t []string`

优于

`t := []string{}`

前者申明了一个nil切片值，而后者是一个非空的但是是零长度的。
他们功能上相同——他们的`len`和`cap`都是零——但是nil切片是比较倾向的一种风格。

注意，只有在非常少的情况下，非空零长的切片才是较好的，比如在编码JSON对象的时候（一个`nil`切片被编成`null`，而`[]string{}`被编成JSON数组`[]`）。

在设计interface的时候，避免区别对待一个nil切片和一个非空零长切片，因为他们会导致微妙的编程错误。

预知更多关于Go中的nil的讨论，请看Francesc Campoy的演讲《[Understanding Nil](https://www.youtube.com/watch?v=ynoY2xz-F8s)》

## Crypto Rand

不要使用`math/rand`来生成秘钥，哪怕是用完就抛弃的也不要用。不适用种子，这个生成器是完全可预测的。用了`time.Nanoseconds()`作为种子，也仅仅是一些熵而已。反而，使用`crypto/rand`的Reader，并且，如果你需要文本的话，打印成十六进制或者base64：

```
import (
    "crypto/rand"
    // "encoding/base64"
    // "encoding/hex"
    "fmt"
)

func Key() string {
    buf := make([]byte, 16)
    _, err := rand.Read(buf)
    if err != nil {
        panic(err)  // out of randomness, should never happen
    }
    return fmt.Sprintf("%x", buf)
    // or hex.EncodeToString(buf)
    // or base64.StdEncoding.EncodeToString(buf)
}
```

## 文档评论

所有顶级、导出的名字，都应当有文档评论，不是那么细碎的未导出类型以及函数申明也该有。看https://golang.org/doc/effective_go.html#commentary去了解更加详细的评论惯例。

## 不要Panic

看https://golang.org/doc/effective_go.html#errors。不要用panic来作为平常错误的处理。使用error和多返回值。

## 错误字符串

错误字符串不要用大写（除非以本就如此的名词或者缩略词开头）或者以标点符号结尾，因为他们一般是跟在其他的上下文后面被打印出来的。也就是，用`fmt.Errorf("something bad")`而不用`fmt.Errorf("Something bad")`，那么`log.Printf("Reading %s: %v", filename, err)`就格式化成不带大写字母在句子中的格式了。这不适用于logging，因为它是隐式地面向行并且不被组合在其他的消息中。

## 例子

当增加一个新的包的时候，包含一些预期的用法：一个可以运行的例子，或者一个简单的简单的测试来演示一个完整的调用序列。

读读[testable Example() functions](https://blog.golang.org/examples)。

## Goroutine生命周期

当你生产goroutine的时候，什么时候退出——或者是否退出——要弄清楚点。

Goroutines能够以阻塞channel收发的方式泄漏：垃圾回收者不会回收一个goroutine，即使被它阻塞的channel不可达。

甚至当它们没有泄漏，在它们没有了用的时候把它们留在内存里依然能够导致一些微妙且难以诊断出的错误。
发送关闭channel时的panic。
“在结果没用的的时候”修改还在使用的输入仍然能够导致数据竞争。
而且，吧把goroutine留在内存里任意长的时间还会导致不可预测的内存使用。

尽量让并发的代码足够的简洁，这样goroutine的生命周期就会明显。
如果那是不可行的，就要用文档说明什么时候以及为什么这些goroutine退出。

## 处理error

看https://golang.org/doc/effective_go.html#errors。不要用`_`变量来放弃error。如果一个函数返回了一个error，检查它从而确保这个函数成功了。处理这个error，返回它，或者在一个真正超出异常的情境panic。

## Import

避免重命名import除非为了避免名字冲突；好的包名应当不需要重命名。遇到了这样的冲突的是的话，比较好的做法是重命名最本地化的或者项目相关的import。

Import一个分组的形式组织，有空行来隔开它们。标准库的包始终在第一个分组中。

package main

```
import (
	"fmt"
	"hash/adler32"
	"os"

	"appengine/foo"
	"appengine/user"

	"code.google.com/p/x/y"
	"github.com/foo/bar"
)
```

[goimports](https://godoc.org/golang.org/x/tools/cmd/goimports)会为你做这些。

## Import点

这种`.`import形式在一些测试中是非常有用的，因为循环依赖，导致一些包没法被测到：


```
package foo_test

import (
	"bar/testutil" // also imports "foo"
	. "foo"
)
```

在这个例子中，测试文件不可以在foo包中因为它使用了bar/testutil，而这个包导入了foo。因此我们使用'import .'形式来让这个文件假装成为foo包的一部分尽管实际不是的。出了这种情况，不要在你的程序中使用import.。它使你的程序很难懂，因为它不是很清楚的，比如一个类似Quux使一个在这个当前包中的顶层的标示符还是一个导入的包中的，很不清楚。

## In-Band Errors

在C和类似的语言中，函数返回-1或者null这样的值给error或者结果缺失，是比较常见的一种做法：

```
// Lookup returns the value for key or "" if there is no mapping for key.
func Lookup(key string) string

// Failing to check a for an in-band error value can lead to bugs:
Parse(Lookup(key))  // returns "parse failure for value" instead of "no value for key"
```

Go对于多返回值的支持提供了一个较好的解决方案。
与其要求客户端检查一个in-band错误的值，一个函数应当返回一个额外的值来表示其他的返回值是不是有效。
这个返回值可以是一个error，或者在不必要解释的时候用一个布尔值也行。
它必须是最后一个返回值。

```
// Lookup returns the value for key or ok=false if there is no mapping for key.
func Lookup(key string) (value string, ok bool)
```

这样可以让调用者避免错误地使用结果：

```
Parse(Lookup(key))  // compile-time error
```

并能够激励出更加鲁棒和易读的代码：

```
value, ok := Lookup(key)
if !ok  {
    return fmt.Errorf("no value for %q", key)
}
return Parse(value)
```

这个规则适用于导出的函数，但对未导出的函数也依然适用。

像nil，“”和-1这样的返回值也是可以的，只要它们对于函数是一个有效的结果，那就是，调用者不需要跟其他值区别开来处理它们就行。

一些标准库函数，比如那些在"strings"包中的，返回in-band错误值。
这以程序猿的更多的努力为代价，换来了对于字符串操纵代码的极大简化。
总体说来，go代码应当返回额外的值给error。

## 缩短error流
