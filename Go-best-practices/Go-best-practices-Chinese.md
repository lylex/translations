# Go最佳实践
-----------

原文：[Go best practices](https://peter.bourgon.org/go-best-practices-2016/) 作者：Peter Bourgon

_这篇文章本来是在2016年伦敦QConf上的一个讲话_

在2014年的时候，我在GopherCon做了一个名为《产品环境的最佳实践》的开幕式演讲。我们在声云曾是Go的最早的使用者，并且从那时候起就一直在写Go、运行Go以及近两年的时间从一个产品到另外一个地在产品环境维护Go。我们学了一些东西，我在努力提炼和传授这些学到的东西。

从那时候起，我就在Go全职工作了。

Since then, I’ve continued working in Go full-time, later on the activities and infrastructure teams at SoundCloud, and now at Weaveworks, on Weave Scope and Weave Mesh. I’ve also been working hard on Go kit, an open-source toolkit for microservices. And all the while, I’ve been active in the Go community, meeting lots of developers at meetups and conferences throughout Europe and the US, and collecting their stories—both successes and failures.

With the 6th anniversary of Go’s release in November of 2015, I thought back to that first talk. Which of those best practices have stood the test of time? Which have become outmoded or counterproductive? Are there any new practices that have emerged? In March, I had the opportunity to give a talk at QCon London where I reviewed the best practices from 2014 and took a look at how Go has evolved in 2016. Here’s the meat of that talk.

I’ve highlighted the key takeaways as linkable Top Tips.
