---
layout: post
title: Docker时代来了，你准备好了吗
published: true
---

刚刚过去的2014年或许是史上最热的一年，在这火热的年份里，Docker 也好似一支被点燃的火箭，掀起一股股热浪。Docker之所以如此受人瞩目，并不是因为它创造了多么神奇的技术，而是因为它重新定义了软件的交付方式，进而将改变传统“开发-测试-部署”的软件流程。尤其是在云计算和开源软件大行其道的今天，Docker的出现正好顺应了时代的发展，占尽了天时地利人和。

本文的重点不是讨论Docker的基本原理或实现机制（网上有太多的资料文档可以参考，比如这篇博客[《Docker，云时代的程序交付方式》](http://liubin.org/2014/08/11/docker-cloud-app-delivery-style/))，而是基于前一段时间的探索实践，分享一些我们对Docker的认识与思考，期待有更多的同行参与进来，一起推动Docker在国内的应用，享受到其发展的红利。因此，我们这里着眼于以下三个问题：为什么Docker有价值？什么是Docker思维？如何把Docker玩起来?

##为什么Docker有价值
为什么Docker这么有价值？在回答这个问题之前，我们先想一个相似的问题，为什么阿里巴巴这么有价值？

传统的商业存在两大弊端：一是买卖双方相对分散，加大了需求收集、营销、运输的成本；二是买卖双方信息不对称，对交易产生了抑制作用。

**电子商务的出现则将商品从卖方到买方的流通过程变得集中化、标准化、透明化**：

- 集中化：买家足不出户就能从各个商户集中购买商品，卖家根据买家需求集中配货、根据下单情况集中发货。
- 标准化：买家支付下单-卖家发货-快递包裹-物流运输-买家收货确认，整个流程都是标准化的。
- 透明化：买家可以在网上浏览商品评价、比较商品价格。买卖双方的信用等级也是完全透明的。

综上所述，可以说**阿里巴巴这样的电商带来的最大价值在于降低了商品的社会交易成本**。

现在我们把商业发展的逻辑搬到软件交付这里。传统的软件交付面临同传统商业类似的问题：

1. 软件发布分散化，使用者搜索和安装软件的成本较高。当然，我们有类似yum、brew这样的工具来集中处理软件安装与软件依赖问题。但是，不要忘记开源软件已经占据越来越重要的位置，它们的发展速度和协作方式使传统的工具很难跟上节奏。

2. 软件开发者和软件使用者双方信息不对称。尽管开发者对软件的代码编译、参数配置、运行环境等信息了如指掌，但使用者却很可能一知半解。因此，我们会经常看到使用者抱怨，明明按照用户手册一步步操作，却仍然跑不起来。这种情况也时常出现在软件流程中，开发将程序和文档交付给测试或运维，测试或运维却总是无法重现期望的运行状态，几经辗转发现是机器环境 、系统环境、软件依赖版本、参数配置等等原因导致。这样低效的软件流程将使持续集成与持续交付很难真正实施起来，仅仅流于形式。

**Docker的出现则将软件从开发方到使用方的交付过程变得集中化、标准化、透明化**：

- 集中化：软件使用者可以从Docker仓库找到琳琅满目的软件镜像，一个镜像包含商业软件或开源软件，也可以包含单个软件或任意的软件组合。
- 标准化：Docker镜像的“构建-发布-存储-下载-运行”是标准化的，统一通过Docker工具来执行，而且所有操作都可以移植到任意的机器或平台。
- 透明化：Docker镜像是自包含的，包括程序、软件依赖、参数配置等所有运行环境，使用者无需了解细节，只需运行同样的Docker命令就能达到与开发者同样的运行状态。

综上所述，可以说**Docker带来的最大价值在于降低了软件的交付成本**。
    
##什么是Docker思维
这年头流行思维主义，什么互联网思维、大数据思维格外赚人眼球。这里我们跟风一下，提出一个Docker思维，其实归纳起来就两句话：

1. **做为软件使用者，避免直接安装软件包，总是以Docker镜像形式获取软件、以Docker容器形式运行软件**。

2. **做为软件开发者，避免直接发布软件包，总是以Docker镜像形式发布到Docker仓库**。

怎样理解这两句话？下面通过一个简单的实践来说明。假定现在我们在开发一个Java程序，从编译到运行需要使用以下几个软件工具：1) git，源码版本控制; 2) maven，源码编译; 3) mysql，存储数据库。

遵循Docker思维，做为使用者，我们应该避免直接安装git、maven及mysql，而是通过Docker来获取。

首先，从github上获取源代码：
    
    docker run -it --rm -v "$(pwd)":/app switchback/git-debian git clone https://github.com/spring-by-example/spring-by-example.git /app
    
运行以上命令将在当前文件夹clone指定github地址的源代码。简单解析一下命令：
 
- switchback/git-debian，是git的Docker镜像URL。
- -it，指定以交互方式启动容器。
- --rm，指定命令结束自动删除容器。
- -v "$(pwd)":/app， 指定将宿主机的当前文件夹mount到容器里的/app文件夹。

然后，通过maven来编译源码：

    docker run -it --rm -v "$(pwd)":/app -w /app docker.cn/docker/maven mvn clean package

运行以上命令将在当前源码文件夹编译maven项目。简单解析一下命令：

- maven:3.2-jdk-7，是maven的Docker镜像URL。
- -w /app，指定容器的工作路径为"/app"。

最后，搭建mysql数据库：

    docker run -e MYSQL_ROOT_PASSWORD=mypassword -d -p 3306:3306 docker.cn/docker/mysql
    
运行以上命令将在本机启动一个mysql数据库。简单解析一下命令：

- docker.cn/docker/mysql，是mysql的Docker镜像URL。
- -e MYSQL_ROOT_PASSWORD=mypassword，-e选项用于向Docker容器里注入环境变量，这里通过MYSQL_ROOT_PASSWORD环境变量传递mysql的root密码。
- -d，指定后台方式启动容器
- -p 3306:3306，指定将容器的3306端口绑定到宿主机的3306端口。
    
可以看到，三条Docker命令就满足了我们对git/maven/mysql的使用需求，而且可以完全移植到有Docker环境的任意机器。那么，接下来的问题是，如何发布我们的Java程序？

遵循Docker思维，做为开发者，应该通过Docker来发布软件。

首先，编写一个Dockerfile来制作Docker镜像，它有点类似Makefile的作用：

    #每个Docker镜像需要基于某个基础镜像来构建
    #新镜像的构建操作会在基础镜像上叠加
    FROM docker.oa.com:8080/docker/java7
    #将源码编译后产生的jar文件拷贝到镜像里
    ADD target/myapp.jar /app/myapp.jar
    #指定容器的启动命令
    CMD java -jar /app/myapp.jar
    
保存以上的Dockerfile，运行以下命令制作镜像，指定镜像名为myapp，版本为1.0：

    docker build -t myapp:1.0 .

镜像制作完成后，可以提交到Docker仓库，软件发布就完成了:

    docker push myapp:1.0

可以看到，两条Docker命令加一个Dockerfile就满足了我们对软件的发布需求。更重要的是，使用者可以像我们之前使用git/maven/mysql那样，通过Docker获取并运行该软件。

##如何把Docker玩起来
君子动口又动手，才是好程序员。要想玩转Docker，首先得动手把Docker环境搭建起来。下面让我们花5分钟的时间在Windows上搭建Docker环境。

####方式一
最简单的方式是直接安装boot2docker，它是一个为体验Docker而打造的轻量级Linux发行版。在Windows上可以直接运行[boot2docker安装文件](https://github.com/boot2docker/windows-installer/releases/download/v1.4.1/docker-install.exe)，它默认会绑定安装VirtualBox以便在虚拟机中运行boot2docker。

boot2docker极其轻量，安装简单，但是它完全是运行在内存中的，这意味着你在系统中修改的配置或保存的文件，在重启后都会丢失。在某些情况下，可能你希望永久创建某个用户账号，或者永久保存某些文件（例如Dockerfile），这时候boot2docker就无法满足了，可以尝试方式二。

####方式二
通过vagrant安装coreos。coreos是另一个为Docker而打造的轻量级Linux发行版，但它的目标是大规模生产环境部署，而不仅仅是体验；vagrant是一个创建可移植的开发环境的工具。我们通过以下几步来安装coreos：
1. 安装[virtualbox](http://download.virtualbox.org/virtualbox/4.3.20/VirtualBox-4.3.20-96997-Win.exe)
2. 安装[vagrant](https://dl.bintray.com/mitchellh/vagrant/vagrant_1.7.1.msi)
3. 下载coreos的vagrant box——[coreos_production_vagrant.box](http://storage.core-os.net/coreos/amd64-usr/alpha/coreos_production_vagrant.box)
4. 运行以下命令添加vagrant box:

    vagrant.exe box add --name coreos coreos_production_vagrant.box

5\. cd到一个新的文件夹，运行以下命令启动coreos虚拟机：

    vagrant.exe init coreos
    vagrant.exe up
    
6\. 现在就可以ssh到coreos了：
    
    vagrant.exe ssh

7\. 当然也可以通过putty这样的工具来ssh到coreos：IP=127.0.0.1，Port=2222。

到目前为止，相信你已经能够在Windows上自由体验Docker了，不妨试试将自己的软件制作成镜像，发布到仓库，再让其他人通过Docker获取运行，你会发现原来软件交付真的是如此便捷！

接下来更进一步，你可以考虑将复杂一点的分布式应用迁移到Docker，这时候就需要搭建Docker集群，而更关键的问题是，如何做资源调度、扩缩容、服务发现、自动容错、集群监控。Docker自身目前还不能解决集群应用的问题（虽然已经在朝着这个方向努力），这需要在Docker之上构建集群管理系统，业界有名的开源项目包括Kubernetes和Flynn，具体介绍可参考笔者的文章[《Kubernetes初探：原理及实践应用》](http://www.csdn.net/article/2014-10-31/2822393)和[《Flynn初探：基于Docker的PaaS平台》](http://blog.csdn.net/zhangjun2915/article/details/41266133)。

##总结
经过本文的分享，希望大家对Docker有更了清晰的认识，并且能动手玩起来。有任何疑问或者想法，欢迎通过邮件zhangjun2915@163.com交流，让我们携手走进Docker时代，做新时代的弄潮儿。
