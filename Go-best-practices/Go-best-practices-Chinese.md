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








