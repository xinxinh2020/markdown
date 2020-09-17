



面试官您好，我叫黄鑫鑫，我是2017年毕业于中山大学数据科学与计算机学院，毕业后我在平安科技云平台事业部工作了两年左右的时间，在平安科技，我负责的主要是CloudStack源码的研究和基于CloudStack开源版本进行定制化的功能开发。CloudStack是一个类似于OpenStack的开源项目，我花了半年多的时间，把CloudStack的核心源码看了差不多，之后就根据我们的需求对CloudStack的源码进行修改，主要有3方面的修改：1是对我们线上发现的bug但是开源版本还没有解决的Bug进行修复、2是把CloudStack集成到我们整个云的架构里面，因为我们还有很多自己开发的系统，就需要把CloudStack集成到这些系统里面，相当于CloudStack原先是一个完整的系统，现在它在我们整个云的架构里它就只是一个核心的组件，这其中就会有一些适配工作，比如我们有自研的网络控制器，就需要把CloudStack原有的网络实现替换成我们自研的方案，以及和我们云门户的对接等等、第3个方面就是进行一些定制化的功能开发，加入一些CloudStack原本不支持的功能比如增量快照功能和跨集群迁移功能等等。出了CloudStack之外，我还负责过了另外一个项目是我们的运维自动化系统的开发，以前我们的运维工作大部分都是通过人工登到服务器上去执行命令或者脚本的方式完成的，我自己也有感受，因为我也会承担一部分运维工作，虽然我主要是负责开发的工作，但是像我们搞基础设施的，很难把开发和运维完全分得很清楚，所以我自己也有切身的体会，就是这样效率很低下，并且存在配置不规范，容易误操作等问题，于是我们做了个运维自动化系统作为我们运维工作的统一入口，底层使用SaltStack和Ansible这些常用的自动化工具去执行远程的服务器操作，便于运维的管理。在负责这个项目的过程中，我有一些技术之外的体会，这个系统主要由我负责，加上一个新人和几个外包同事，在这个过程中我会体会到了一个团队在沟通和协作方面的重要性，我个人对技术还是比较执着的，并且乐于分享，所以在遇到问题的时候，我总是很有耐心地让和大家一起分析问题的原因并寻找解决方案，有些同事对底层服务器的知识了解不够，比如外包的同事，他们可能之前就是纯粹做Java开发或者做前端，他们对底层服务器，包括Linux,虚拟化等方面都不是很了解，我也尽可能地让大家把底层的原理弄清楚，因为很多时候，大家可能就是习惯于你给他提个需求，它就把它完成，就是你叫他做什么，他就做什么，很多时候他们也不知道为什么这么做，很多时候他们都不知道自己在做什么，但是我觉得这不是很好的方式，所有我也有一直在帮大家把底层的原理弄清楚，这样在写代码的过程中就不仅仅是别人叫你做什么，你就做什么，而是会带入自己的一些思考，这也是我喜欢的方式。所以在完成这个项目的过程中，我们不仅是完成了一个项目，而且也让大家的技术积累有了很大的提升，并且大家乐于互相分享，从而让我们的团队形成比较好的合力，因此也取得了比较好的效果，而且对我们后续的工作也有很深远的影响，对我们效率提高也很有帮助，

在19年7月的时候，我离职来到现在所在的国家超级计算广州中心，原因是当时我有读博的打算，和超算这边的老师联系好，先过来这边做一些项目熟悉研究课题，然后一边进行读博的申请，后来因为导师的分配的问题，还有一部分原因我感觉国内的科研氛围不是很好，而国外现在环境又比较复杂，所以放弃了读博的打算。

然后我在超算这边做的主要是一个SaaS平台的开发。这个平台就是提供一些SaaS应用，比如Tenorflow，Spark，MongoDB之类的应用，让用户可以一键提交运行，我们后台就会把这些应用通过k8s的api接口调度到k8s集群上去运行。这个系统的是用Golang实现的，我们使用的是golang实现的一个k8s 客户端client-go去掉k8s的接口，来实现把我们平台上的应用调度到k8s集群上去运行。

然后对底层基础设施方面的技术，比如k8s和容器方面也了解的比较多一点，云计算相关的技术也都比较熟。

我看到这个岗位是专有云相关的业务，专有云和公有云是共用基础设施的吗

负责云专有客户的交付和升级方案和实施指导和赋能：

其实我们之前在平安做的也是类似于专有云的业务，私有云，公有云


