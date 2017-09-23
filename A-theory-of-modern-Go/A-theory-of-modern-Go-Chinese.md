# 现代Go理论
-----------

原文：[A theory of modern Go](https://peter.bourgon.org/blog/2017/06/09/theory-of-modern-go.html) 作者：Peter Bourgon

**魔法是坏的；全局状态就是魔法 → 不该有包级别的变量，不该有init函数**

  Go唯一最好的属性就是，最基本的“没有魔法”。基本上没有什么例外，直来直去地阅读Go代码遇不到含糊的定义，依赖关系或者运行时环境。这使得Go相对地容易阅读，从而使得它相当地易于维护,这是产业化编程最好的优点了。

  但是魔法能够通过一些方法渗透进来。一个不幸的非常普遍的方法就是通过使用全局状态。包一级的全局对象能够编码对外部调用者不可见的状态和（或者）行为。对于这些全局变量调用的代码能够带来令人惊讶的边际效应，这破坏了读者理解和心理上为程序建模的能力。

  函数（包括方法）基本上是仅有的Go用来建立抽象的机制。考虑如下的函数定义。

    func NewObject(n int) (*Object, error)

  约定俗成地，我们期望具有NewXxx形式的函数是构造函数。我们看到了这个函数返回了一个指向一个结构的指针和一个error，这个期望就得到验证了。由此我们可以推断出，构造函数或许成功了，或许没有，如果它失败了，我们将会得到了一个error来告诉我们为什么。我们观察到，上面这么函数接受一个int型参数，我们假设以此来控制返回对象的某些表象或能力。大概会有一些限制在n上，如果不满足的话，会返回error。但是因为这个函数不接受其他的参数，所以我们期望它除了分配内存之外（期望能成功分配）应该没有其他的效果。

  通过单纯地阅读这个函数申明，我们能够做出所有的这些推断，并且心里能够构建出这个函数的模型。这个过程，从main函数的第一行代码开始，就重复地、递归地被应用着，这就是我们怎么阅读和理解程序的。

  现在, 考虑如果以下这个是这个函数的函数体。

    func NewObject(n int) (*Object, error) {
    	row := dbconn.QueryRow("SELECT ... FROM ... WHERE ...")
    	var id string
    	if err := row.Scan(&id); err != nil {
    		logger.Log("during row scan: %v", err)
    		id = "default"
    	}
    	resource, err := pool.Request(n)
    	if err != nil {
    		return nil, err
    	}
    	return &Object{
    		id:  id,
    		res: resource,
    	}, nil
    }

  这个函数引用了一个包的全局对象database/sql.Conn，用来针对某些非特定的数据库作查询；一个包的全局logger，用来输出任意形式的字符串到某些未知的位置；一个包的全局的某个对象池，用来请求某种类型的资源。所有这些操作都有检查函数的申明时不可见的边际效应。没有方法让一个调用者来预测任何一件这样的事情会发生，除非去读这些函数的代码或者去研究这些全局变量的定义。

  考虑另外一个可选的申明：

    func NewObject(db *sql.DB, pool *resource.Pool, n int, logger log.Logger) (*Object, error)

  通过把这些依赖作为参数放到申明中，我们能够让读者能够精确地为这个函数的作用范围和潜在的行为建模。调用者完全知道这个函数需要什么来完成它的工作，从而相应地提供给它们。

  如果为这个包设计公共的API，那么我们甚至能够朝着有帮助的方向再走远点：

    // RowQueryer models part of a database/sql.DB.
    type RowQueryer interface {
    	QueryRow(string, ...interface{}) *sql.Row
    }
    
    // Requestor models the requesting side of a resource.Pool.
    type Requestor interface {
    	Request(n int) (*resource.Value, error)
    }
    
    func NewObject(q RowQueryer, r Requestor, logger log.Logger) (*Object, error) {
    	// ...
    }

  通过把这些抽象的对象建模成interface，仅仅捕捉我们用到的方法，我们允许调用者用可选的实现来替换它们。这样可以减少包与包之间代码级的耦合，并且能够让我们在测试时mock出这些抽象的依赖。测试原来的带有包级的全局变量的那个版本的代码，需要枯燥的、容易出现错误的组件替换。

  如果我们所有的构造函数和方法都显式地处理它们的依赖，那么我们再也不需要任何全局变量的使用了。相反地，我们可以在我们的main函数中，构造出所有我们数据库的连接、我们的logger，我们的资源池，那么将来的阅读者就能够清楚地映射出一个组件图表。并且，我们能够非常明确地将这些依赖传递给使用他们的组件，从而我们可以去除这种全局变量的对于理解有害的魔法了。此外，可以观察到，如果我们不用全局变量的话，我们再也不要使用那些仅仅用来示例或者改变全局状态的init函数了。那么我们就会带着适当的怀疑去看待这些init方法了：这些代码是干嘛的，为什么不在main函数中，它属于哪里？

  去写没有全局状态的Go程序，不仅可行而且非常容易，并且极端新鲜。在我的经验里，这样的程序不仅没有观察到变慢，也没有比使用全局变量来缩短函数定义更加枯燥。相反地，当一个函数申明可靠地、完全地描述了函数体的行为范围的时候，我们有理由相信，重构和维护大量的代码将会是更加高效的。Go kit从一开始就已经用这种方式来写了，因为这样有着巨大的好处。

  由此，我们可以发展处一个Go的理论。基于Dave “Humbug” Cheney的话，我提出下面的纲领：

* 不用包级别的变量
* 不用init函数

  当然了，会有一些例外。但是，从这个规则开始，其他的实践自然地跟从了。
