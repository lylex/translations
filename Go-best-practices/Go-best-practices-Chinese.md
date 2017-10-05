# Go最佳实践
-----------

原文：[Go best practices](https://peter.bourgon.org/go-best-practices-2016/) 作者：Peter Bourgon

_这篇文章本来是在2016年伦敦QConf上的一个讲话_

在2014年的时候，我在GopherCon做了一个名为《产品环境的最佳实践》的开幕式演讲。我们在声云曾是Go的最早的使用者，并且从那时候起就一直在写Go、运行Go以及近两年的时间从一个产品到另外一个地在产品环境维护Go。我们学了一些东西，我在努力提炼和传授这些学到的东西。

从那时候起，我就在Go全职工作了，然后再声云的活动和基础架构团队，现在在Weaveworks，在做Weave Scope和Weave Mesh。我也在努力地做Go kit，那是一个为了微服务使用的开源套件。在这段时间里，我一直活跃在Go社区，在碰头会和正式会议上见了很多来自欧洲和美国的开发者，并收集了他们成功和失败的故事。

在Go的定于2015年11月的6周年发行版发布之际，我回想起第一次谈话。这些好的实践中的哪些，经受住了时间的考验？哪些已经成了过时的或者适得其反的？有没有一些新的实践显露出来？3月里，我有了在伦敦QCon做一个演讲的机会，在那我审阅了从2014年开始Go的好的实践，并看了在2016年中Go进化了哪些。这是这次谈话的的主要内容。

我已经高亮了重要的东西，来作为可以链接的Top Tips。

> **Top Tip — 利用Top Tip来提升你的Go游戏的等级**

并增加了一个内容的快速表......

1. 开发环境
1. 代码库结构
1. 格式化和样式
1. 配置
1. 程序设计
1. Logging和instrumentation
1. 测试
1. 依赖管理
1. 构建和部署
1. 结论

## 开发环境

Go有着围绕着GOPATH的开发环境的约定俗成。在2014年，我曾强烈提倡单一的全局GOPATH。我现在的立场已经软化很多了。我还是认为那是最好的点子，所有的其他的做法效果上等价，但是取决于你的项目或团队，此外也就合情合理了。

如果你或者你的团队主要产出二进制文件，那么使用一个项目一个GOPATH你或许能发现很多优势。有一个新的工具，[gb](https://getgb.io/)，来自于Dave Cheney及其贡献者，它在这个使用场景中替代了标准的go工具。许多人正在说使用这个的成功案例。

一些Go开发者使用一个双入口的GOPATH，例如，$HOME/go/external:$HOME/go/internal。go tool总是知道怎么处理这样的事情：go get将会把代码拉到第一个入口地址中，因此，如果你需要严格区分第三方代码和内部代码的话，这将会有用。

我注意到一些开发者忘记做的一件事：把GOPATH/bin放到你的PATH中去。放进去会让你非常容易地运行你通过go get得到的二进制文件，并让构建代码的go install机制（倾向于这个）更好地工作。没有理由不这么做。

> **Top Tip -- 把$GOPATH/bin放到$PATH中，这样安装的二进制文件就能轻易地访问**

至于编辑器和IDE，已经有了长足的进步了。如果你是vim党，日子已经不能更好过了：归功于不知疲倦也非常有才华的Fatih Arslan，[vim-go](https://github.com/fatih/vim-go)插件是在一个绝对超常的状态，同类中最好。我不熟悉emacs，但是Dominik Honnef的[go-mode.el](https://github.com/dominikh/go-mode.el)仍然是一个魔法师般的存在。

再有就是，很多人还在使用[Sublime Text](https://www.sublimetext.com/)+[GoSublime](https://github.com/DisposaBoy/GoSublime)，并取得了成功。并且，很难打败它的速度。但是，近来更多注意力都关注在了那些电子动力（Electron-powered）的编辑器上了。[Atom](https://atom.io/)+[go-plus](https://atom.io/packages/go-plus)有很多的粉丝，特别是那些不得不频繁切换到JavaScript的开发者。黑马是[Visual Studio Code](https://code.visualstudio.com/) + [vscode-go](https://github.com/Microsoft/vscode-go)，它比Sublime Text慢点，但比Atom快点，并且对于那些对我很重要的特性有默认支持，比如说点击到定义。在被Thomas Adam推荐后，我已经日常中用它一年了。很好玩。

在全IDE方面，倾心打造的[LiteIDE](https://github.com/visualfc/liteide)已经收到了定期的更新，当然有一定份额的粉丝。并且[IntelliJ Go插件](https://github.com/go-lang-plugin-org/go-lang-idea-plugin)也始终在提高。

## 代码库结构

_**更新**：Ben Johnson写了一个很出色的文章，名为[《Standard Package Layout》](https://medium.com/@benbjohnson/standard-package-layout-7cdbc8391fc1)，为典型的商业的行（line-of-business）的软件。_


_**更新**：Tim Hockin的[《go-build-template》](https://github.com/thockin/go-build-template)，稍微改编了些，已经被证明是一个更加通用的模型。我自从它的原始发布以来，就已经采纳这段的内容_

我们有过很长的时间来使我们的项目成熟，很多种模式也随之出现了。我不相信存在最好的一个独一无二的代码结构，然而我相信有一个好的通用的模型来适合很多类型的项目。这对那些既提供二进制文件也提供库、或者结合Go代码跟其他非Go代码的项目非常有用。

基本的想法就是有两个最顶层的目录，pkg和cmd。在pkg下面，为你的每个库创建目录。在cmd下面，为你的每个二进制创建目录。所有的你的Go代码应该只在这两个地方种的一个中。

    github.com/peterbourgon/foo/
      circle.yml
      Dockerfile
      cmd/
        foosrv/
          main.go
        foocli/
          main.go
      pkg/
        fs/
          fs.go
          fs_test.go
          mock.go
          mock_test.go
        merge/
          merge.go
          merge_test.go
        api/
          api.go
          api_test.go

所有你的存档都是可以go get到的。这个路径可能稍微长了点，但是这是跟其他的Go开发人员相类似的命名方法。并且你和那些非Go的东西会有空间和分隔。例如，Javascript可以在一个client或者ui的子目录中。Dockerfiles、持续集成配置文件或者其他构建辅助文件可以在项目根下面或者在一个build的子目录中。并且，像Kubernetes manifests这样的运行时配置也可以有一个home目录。

> **Top Tip -- 把库文件放在pkg/子目录下，把二进制文件放在cmd/子目录下**

当然，你将仍然使用完整合格的import路径。也就是，在cmd/foosrv中的main.go应该import "github.com/peterbourgon/foo/pkg/fs"。并且，当心下游用户会包含vendor目录的分支。

> **Top Tip -- 一直使用完整import路径。不要使用相对路径**

这点结构使得我们在更加广阔的生态中玩得更好，也希望能够持续地确保我们的代码更好用。

## 格式化和样式

事情大体上已经保持一样了。这是Go做的非常正确的一个地方，我也十分感激社区的共识和这个语言的稳定。

这篇[《Go Code Review Comments》](https://github.com/golang/go/wiki/CodeReviewComments)是很赞的，应该成为代码审阅强制标准的最小集。当命名时遇到争论和不一致的时候，Andrew Gerrand的[《idiomatic naming conventions》](https://talks.golang.org/2014/names.slide)就是一个不错的指导方针集合。

> **Top Tip -- 参考Andrew Gerrand的命名规范**

在工具方面，事情应该做得更好点了。你应当配置你的编辑器来在保存时候运用gofmt，或者更好的[goimports](https://github.com/bradfitz/goimports)。（在这点上，我希望那不是在任何方面有争议的）。go vet工具基本上没有误报，因此你应当考虑把它变成你代码提交前钩子的一部分。并且，看看这个出色的用来扫描问题的[gometalinter](https://github.com/alecthomas/gometalinter)。这个可能会误报，因此[制定你自己的约定](https://github.com/weaveworks/mesh/blob/master/lint)某种程度上不是一个坏点子。


## 配置

配置是处于运行时环境和进程之间的的表面区域。它应该是明显的而且是有较好的文档的。我仍然使用并推荐flag包，但是这会我承认，我希望它不要那么难懂。我希望它有个标准，getopts型并且有长短参数语法，而且我希望它的用法说明更加简洁。

[12-factor apps](http://12factor.net/)鼓励你使用环境变量来进行配置，我觉得那挺好的，每个提供出来的参数都也被定义成了一个标记。明确性也是重要的：改变一个应用的运行时环境应当已能够被发现和有文档说明的方式发生。

我在2014年曾经说过了，但是应为它太重要所以我还想说一遍：[在main函数里定义和解析标记](https://robots.thoughtbot.com/where-to-define-command-line-flags-in-go)。只有main函数才有权力来决定对于用户可见的标记。如果你的库的代码需要参数化它的行为，那些参数应当是类型构造函数的一部分。把配置放到包一级之后有着方便的假象，但这样划不来：这样破坏代码模块性的行为，让开发者或者后续维护者很难理解依赖关系，也使得写出独立、并行的测试变得困难。

> **Top Tip -- 只有main函数有权利决定哪些标记对用户可用**

我觉得这里有一个机会，让一个组织很好的、结合了以上所有特性的flags包从社区中脱颖而出。或许它已经存在了，如果有的话，请让我知道。我当然要用下它。

## 程序设计

在这个谈话中，我用了配置来作为起跳点，来探讨程序设计的其他问题。（我在2014年的谈话中，并没有涵盖这一点。）首先，我们看一看构造函数。如果我们合理地参数化所有的依赖，我们的构造函数会相当的大。

    foo, err := newFoo(
        *fooKey,
        bar,
        100 * time.Millisecond,
        nil,
    )
    if err != nil {
        log.Fatal(err)
    }
    defer foo.close()

有时候，这样的构造函数最好是用一个config对象：一个构造函数的struct参数，这个struct的结构化的对象带有_可选_的参数。让我们假设fooKey是一个required的参数，其他的都是合理的默认值或者是可选的。通常，我看到一般的项目都用一种零碎的方式来构造config对象。

    // Don't do this.
    cfg := fooConfig{}
    cfg.Bar = bar
    cfg.Period = 100 * time.Millisecond
    cfg.Output = nil
    
    foo, err := newFoo(*fooKey, cfg)
    if err != nil {
        log.Fatal(err)
    }
    defer foo.close()

但是，利用所谓的struct初始化语法来一次性地、在一条语句里构造对象是非常好的。

    // This is better.
    cfg := fooConfig{
        Bar:    bar,
        Period: 100 * time.Millisecond,
        Output: nil,
    }
    
    foo, err := newFoo(*fooKey, cfg)
    if err != nil {
        log.Fatal(err)
    }
    defer foo.close()

当这个对象是一个中间状态、无效状态的时候，没有一条语句出现。并且，所有的field都被很好的分割和缩进，构造出这个fooConfig定义。

注意到我们构造了这个cfg对象，并很快就使用它。这种情况下，我们可以节省另外一个中间状态的等级以及另外一行代码，这可以通过直接把struct的定义直接放到newFoo构造函数的行内来达成。

    // This is even better.
    foo, err := newFoo(*fooKey, fooConfig{
        Bar:    bar,
        Period: 100 * time.Millisecond,
        Output: nil,
    })
    if err != nil {
        log.Fatal(err)
    }
    defer foo.close()

太好了。

> **Top Tip -- 使用struct连续的初始化方法，来避免无效的中间状态。必要时候使用行内stuct初始化**

让我们回到合理默认值的话题。观察到Output参数是可以用nil值的。看在参数的份子上，我们假设它是一个io.Writer。如果我们不做特别的事情的话，当我们想把它用在foo对象里时候，我们将不得不一开始就要做nil检测。

    func (f *foo) process() {
        if f.Output != nil {
            fmt.Fprintf(f.Output, "start\n")
        }
        // ...
    }

那不够好，它必须更安全、更好，能够不必检查它的存在性就可以使用它。

    func (f *foo) process() {
         fmt.Fprintf(f.Output, "start\n")
         // ...
    }

那么我们应当在这里提供一个有用的默认值。用interface类型，一个好的方式就是传递一个参数，它提供了interface的无需操作的实现。那就变成了标准库ioutil包用了一个无需操作的io.Writer，叫做ioutil.Discard。

> **Top Tip -- 使用默认的no-op实现，来避免nil检测**

我们能够把这个传递到fooConfig结构体中，但那仍然是脆弱的。如果调用者在调用的地方忘记了这么做，我们最终还是得到一个nil值。那么，相反地，我们能够在这个构造函数中做一些安全措施。

    func newFoo(..., cfg fooConfig) *foo {
        if cfg.Output == nil {
            cfg.Output = ioutil.Discard
        }
        // ...
    }

这仅仅是一个Go俗语的应用，_让零值有用_。我们让这个参数的零值（nil）来产生一个好的默认行为（no-op）。

> **Top Tip -- 让零值有用，特别是在config对象里**

让我们再来看看构造函数。参数fooKey、bar、period和output都是依赖。foo对象依赖他们中的每一个，从而使得启动和运行成功。如果说我从在野外写Go代码和每天日常看Go代码的过去6年经历里学到一课的话，那就是：**让依赖显式化**。

> **Top Tip -- 让依赖显式化！**

我相信，一个难以置信的维护负担、困惑、bug和没有还清的技术债都可以追溯到含糊不清的或者不显式的依赖。考虑这个foo类型的一个方法。

    func (f *foo) process() {
        fmt.Fprintf(f.Output, "start\n")
        result := f.Bar.compute()
        log.Printf("bar: %v", result) // Whoops!
        // ...
    }

fmt.Printf是自包含的并且没有影响或者依赖全局状态；从功能角度讲，它有着某种像是指示透明层的东西。因此它不是依赖。很明显地，f.Bar是依赖。并且，有趣的是，log.Printf作用在一个包全局logger对象上，它仅仅是被免费的函数Printf给掩盖了。因此，它也是一个依赖。

我们怎么处理依赖呢？**我们把它们显式化**。因为这个处理函数把打印到一个log作为它的一项工作，或者是这个函数，或者是foo对象自己需要一个logger对象来作为它的依赖。例如，log.Printf应该成为f.Logger.Printf。

    func (f *foo) process() {
        fmt.Fprintf(f.Output, "start\n")
        result := f.Bar.compute()
        f.Logger.Printf("bar: %v", result) // Better.
        // ...
    }

我们习惯于认为这样的工作，例如写log，是附带产物。因此，我们非常高兴地去依赖helpers，比如说包全局logger，来减少表面的依赖。但是logging，就像是instrumentation一样，经常是对于运维至关重要。并且，把依赖藏在包全局的作用域里这种的事能够并且也会反过来咬我们一口，比如说某些东西表面上看像是logger啦，或者其他的，更重要的、域相关的组件我们还没有去初始化的。用严格一点来节省你将来的痛苦：把一切的依赖都显示化。

> **Top Tip -- logger是依赖，就像其他组件、数据库句柄或者命令行标记等的引用一样**

当然了，我们应当为我们的logger弄一个合情合理的默认值。

    func newFoo(..., cfg fooConfig) *foo {
        // ...
        if cfg.Logger == nil {
            cfg.Logger = log.New(ioutil.Discard, ...)
        }
        // ...
    }

_更新：想了解这个的具体细节和魔法方面的命题，看2017年六月的博客[theory of modern Go](https://peter.bourgon.org/blog/2017/06/09/theory-of-modern-go.html)_

## Logging和instrumentation
