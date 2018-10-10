![](https://mmbiz.qpic.cn/mmbiz_gif/ia1Z7HH4plnCxdmpGxsqGY9DdFDbqL5pYibO4gWauzIcpicwLdtMZ2QFhMh1U1iaIhVGVRwTevzbN3K8z5RDiar6FSg/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)



  

在当今云计算领域，“容器技术”已经从三四年前的炒作期正式进入了产业落地期，而Kubernetes作为容器平台的标准已经得到了广泛应用。

  

Kubernetes从2014年6月由Google宣布开源，到2015年7月发布第一个正式版本1.0并进入CNCF基金会，再到2018年3月从CNCF基金会正式毕业，版本更替到1.10。短短三年间，Kubernetes含着Google的金钥匙，顶着Borg的光环迅速成为容器编排领域的标准，是开源历史上发展最快的项目之一。

  

截止3月份，Kubernetes项目在总体贡献方面位于GitHub第9位，作者/问题排在第2位，仅次于Linux项目。从CNCF基金会今年3月份发布的报告，71％的财富100强企业使用容器，超过50％的财富100强企业使用Kubernetes作为容器业务流程平台。

![](https://mmbiz.qpic.cn/mmbiz_png/ia1Z7HH4plnCxdmpGxsqGY9DdFDbqL5pYibFacnjPktdSy7wNQqPcSibGxicoZ8HtG0FWh1QYqF7Id03jkS7DJGojQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

企业在思考并实践落地Kubernetes的过程中，通常需要面对多个问题，比如：  

●  Kubernetes对我们有什么好处？能够解决当前的什么问题？

● 优先在哪些业务场景、流程环节使用Kubernetes？

●  现有基础设施能否平滑切换到Kubernetes？

● 现有基础设施上托管的业务能否平滑切换到Kubernetes？

●   企业人员（开发、测试、运维、设计等）能否快速适应Kubernetes？是否会带来较大的转型挑战？…………

如果我们对这些疑问进行归类，可以聚焦到以下三个基础问题上来。

  

## 1. 需不需要Kubernetes？
Kubernetes是一个“容器编排平台”，即：容器化业务的管理平台。因此，需不需要Kubernetes通常与业务需不需要做容器化改造在一起考虑。与基于“服务器+Linux+软件包”的传统非容器化业务相比，核心差异点主要有两处：

●   业务交付件是容器化交付件；

●   业务运行时环境是容器化运行时；

  

而需不需要Kubernetes完全取决于这两点是否能够带来收益。下表简单描述了一些关键收益，以及同时引入的问题。

  

Kubernetes是一个“容器编排平台”，即：容器化业务的管理平台。因此，需不需要Kubernetes通常与业务需不需要做容器化改造在一起考虑。与基于“服务器+Linux+软件包”的传统非容器化业务相比，核心差异点主要有两处：

●   业务交付件是容器化交付件；

●   业务运行时环境是容器化运行时；

  

而需不需要Kubernetes完全取决于这两点是否能够带来收益。下表简单描述了一些关键收益，以及同时引入的问题。

![](http://oldboy-study.oss-cn-qingdao.aliyuncs.com/%E5%8C%BA%E5%88%AB.png?Expires=1539171501&OSSAccessKeyId=TMP.AQGiQgPZ8kMPp57gw664qmAfysvyQLIYMd8gUhR_t9HGgxTBuiJXTph35nkjADAtAhRUUtRMr1Abd4Eq9bd9wLsSJTgRKwIVAK4uelMnNR9hoOdKGmRxmyosJaCH&Signature=zTDSL1uEFnkbS5ngMf9cXh%2B9VHw%3D)

从企业的角度来看，容器化改造对于关键的业务交付效率、基础设施资源利用率普遍会带来很好的收益，尤其是对交付效率和资源成本更为关注的轻资产型业务，这也是为何容器技术得到广泛关注与应用的主要原因。而相对而言容器化改造所带来的问题则可以通过引入一些工具与服务进行解决，比如自动化镜像构建工具、具有高速传输与高容量存储能力的镜像仓库、容器化环境与业务的监控运维工具、高性能与配置自动化的容器网络与存储管理服务等。

  

## 2. 如何切换到Kubernetes？


一旦确定要使用Kubernetes，那对于企业业务而言，就需要考虑业务研发流程、基础设施资源如何切换到Kubernetes。这里，通常可以分为三个场景考虑：

●   业务交付流程如何切换到Kubernetes

●   业务运维流程如何切换到Kubernetes

●   业务运行负载如何迁移到Kubernetes

  

下表描述了典型的处理方式：

  
![](https://oldboy-study.oss-cn-qingdao.aliyuncs.com/QQ%E5%9B%BE%E7%89%8720181010184002.png?Expires=1539171663&OSSAccessKeyId=TMP.AQGiQgPZ8kMPp57gw664qmAfysvyQLIYMd8gUhR_t9HGgxTBuiJXTph35nkjADAtAhRUUtRMr1Abd4Eq9bd9wLsSJTgRKwIVAK4uelMnNR9hoOdKGmRxmyosJaCH&Signature=C4HwcIBCEzAWit5n3UTld5yaOIE%3D)

在上述过程中可以看到，在企业落地Kubernetes过程中，除了Kubernetes平台本身的搭建，围绕着Kubernetes生态的一些工具与服务也非常重要，包括面向容器化业务的CI/CD工具链、容器化环境与业务的监控运维、应用发布与交付工具等。在不具备相关容器化平台完整能力的情况下，落地Kubernetes并不能够达成提升业务交付效率的目的。

  

## 3. 如何适应Kubernetes？



完成面向容器的交付、运维与迁移流程改造只是做好了实践Kubernetes的基础条件准备工作，对企业而言最重要的还是业务如何在新的平台之上运转的更高效、更可靠、更安全。“容器”相比于传统基础设施而言，更适合作为当前企业的新一代应用设施的原因主要包括三点：

●   “容器”将应用与基础设施紧密的联系在了一起，改变了传统的应用与设施管理相分离的思维与运作模式，而这一点恰恰与DevOps理念是完美契合的。

●   “容器”带来了更高的弹性，表现在更精细的资源分配与调度、更快速的弹性扩缩容，以及对底层资源更高的抽象，更适合以“弹性”为核心理念的云计算。

●   “容器”是应用微服务理念的最佳实践。微服务化架构更多是从应用开发角度来思考，而容器更偏向于应用发布与运维，这两者的结合真正能打通应用交付全流程。

  

上述三点在Kubernetes均得到完整的体现。Kubernetes通过CNI、CSI、Device Plugin等插件与网络、存储、GPU等基础设施资源整合在一起，通过上层统一的调度系统完成面向应用的实时、弹性资源分配，并且通过Autoscaler能够完成基于策略的自动扩缩容；而Service、Workload、ConfigMap等对象以及DNS、Ingress/Service controller等系统级组件原生支持了服务发现、配置、路由、负载均衡等微服务模型及框架能力，并且通过Kubernetes生态中的Istio项目能够提供更为完整的ServiceMesh微服务治理能力。

  

因此，企业业务如何适应Kubernetes的过程，也同时是如何落地DevOps、微服务、弹性基础设施理念的过程。这里面，不仅仅包含如何做业务的改造来适应容器化运行环境与容器化交付流程，更多的应该是思考如何更好地基于Kubernetes的各种最佳实践来优化业务本身，设计更适合的业务架构，优化业务交付流程中各个环节，做到更好的Cloud-Native与Kubernetes-Native。

  

## 4. 华为云帮助企业落地Kubernetes


作为Kubernetes 最早的采用者之一，华为自2013年起在内部多个产品落地Kubernetes，在这个过程中，围绕着本文上述的三个基本性问题，以及规模化生产环境落地场景，华为发现并解决了一些功能缺失、系统级高可用、可扩展性挑战等问题，并积极回馈给了Kubernetes社区。基于这些场景的落地经验，以及广泛的社区核心特性贡献，华为也顺利成为Kubernetes社区技术监管委员会成员，以及CNCF基金会TOC成员。

  

一方面基于内部实践的思考，另一方面基于外部各类客户场景的落地经验总结，华为云围绕着上述三个基础问题，面向企业用户提供了全栈Kubernetes服务，以期能够帮助企业快速落地Kubernetes，助力企业Cloud-Native战略实施。

![](https://oldboy-study.oss-cn-qingdao.aliyuncs.com/QQ%E5%9B%BE%E7%89%8720181010184757.png?Expires=1539172114&OSSAccessKeyId=TMP.AQGiQgPZ8kMPp57gw664qmAfysvyQLIYMd8gUhR_t9HGgxTBuiJXTph35nkjADAtAhRUUtRMr1Abd4Eq9bd9wLsSJTgRKwIVAK4uelMnNR9hoOdKGmRxmyosJaCH&Signature=QvkgGnH3ACyAc%2B7bvOGwZ%2FyjHJo%3D)
华为云提供的Kubernetes全栈服务主要包括：

  

**容器化基础设施**

华为云提供了通过CNCF官方认证的两种Kubernetes服务供用户选择，包括云容器引擎（CCE）与云容器实例（CCI）。CCE是用户专属Kubernetes服务，用户可以控制整个Kubernetes集群，同时管理基础设施资源与运行在Kubernetes上的容器化业务；而CCI是Serverless Kubernetes服务，用户只需要管理运行在Kubernetes上的容器化业务，无需感知Kubernetes集群，而交由华为云自动管理，进一步降低Kubernetes落地门槛。

  

**容器化交付流程**

华为云容器镜像服务（SWR）提供了高性能、高容量、高安全的企业级私有镜像仓库，并提供了镜像构建与发布流水线ContainerOps支持业务自动化交付，同时，ContainerOps能够支持企业现有工具的接入，最大程度减小对现有企业交付流程的冲击，辅助企业业务平滑迁移。

  

华为云应用编排服务（AOS）提供了自动化云设施管理工具，企业可以通过预置的模板自动化完成容器化的开发、测试、生产环境准备，以及日常配置与变更工作，将企业从繁杂的基础设施管理工作中解放出来，聚焦到业务本身。对于业务较复杂的场景，AOS还能够将Kubernetes上运行的各种工作负载、各类资源对象进行整合管理，并提供完善的版本与生命周期管理机制，便于企业以更完整的业务为对象进行日常管理。

  

**容器化运维流程**

华为云提供了应用运维管理（AOM）与应用性能管理（APM）服务辅助容器化业务运维，包括丰富的各类运维工具，除了基础的监控、日志与告警，进一步面向故障定位与分析场景提供了应用全局性能拓扑展示与调用链跟踪等高级特性，使得运维人员能够及时了解应用健康状态并进行相关处理。

  

**容器化架构转型**

华为云云容器引擎（CCE）与微服务引擎（CSE）提供了Kubernetes生态的Istio以及Apache ServiceComb两种微服务框架供企业实施微服务架构转型。对于Java企业级应用，CSE基于ServiceComb提供了具备升降级、容错、熔断等完整服务治理能力的微服务框架，并兼容Spring Cloud、Dubbo等开源接口，具备更高的服务吞吐性能；而CCE也原生集成了Istio项目，并提供高性能ServiceMesh数据面，面向非侵入式场景提供Kubernetes-Native的微服务治理能力。

  

随着Kubernetes的全面成熟与大规模应用，如何落地Kubernetes是企业实施云战略需要考虑的迫切问题。

落地Kubernetes除了对Kubernetes平台自身的熟悉与掌握之外，如何对现有业务及基础设施进行容器化改造、如何应对Kubernetes对业务现有交付与运维流程的冲击、如何深入思考容器与Kubernetes给企业所带来的转型化思考都是需要一并考虑的问题。

引入围绕着Kubernetes的各类工具化服务能够让企业快速获取业界最佳实践，平滑迁移现有软硬件资产，减小对现有业务交付与运维流程的冲击，使得企业平稳落地Kubernetes并合理优化现有业务，最终达成提升业务交付效率、简化基础设施管理的目的。
 

推荐阅读

[深度长文！！！华为云专家为你解读云原生领域的热点话题和发展趋势](http://mp.weixin.qq.com/s?__biz=MzIzNzU5NTYzMA==&mid=2247483708&idx=1&sn=69941c6bd282f311edf192c59d0bd764&chksm=e8c77fbddfb0f6ab485684cc3f861f30495405de57bc0295b2bc89a60af876fb6f5ceab589f7&scene=21#wechat_redirect)

[在 K8S 大规模场景下， Service 性能如何优化？](https://mp.weixin.qq.com/s?__biz=MzIzNzU5NTYzMA==&mid=2247483742&idx=1&sn=3540b2b3073f4d3831186dbde35d6e9b&chksm=e8c77fdfdfb0f6c9c1ff3acf0553d046143b599760050f64ec52fbfb12b11af33eb15297c6bf&scene=21#wechat_redirect)

[大海航行靠舵手 华为云靠什么征服K8S？](https://mp.weixin.qq.com/s?__biz=MzIzNzU5NTYzMA==&mid=2247483729&idx=1&sn=ab00daadfa8d2f502d481b038e85cdb4&chksm=e8c77fd0dfb0f6c66e3bd1dfaadfaacd4fb12201bf18d886b5fc2207c6fdf92a7bafccb55dfc&scene=21#wechat_redirect)

  

  

点击阅读原文，了解云容器引擎CCE

本文转载自：[企业落地Kubernetes的问题与对策](https://mp.weixin.qq.com/s?__biz=MzIzNzU5NTYzMA==&mid=2247483766&idx=1&sn=74ca05a880948c193387a4c572e0838f&chksm=e8c77ff7dfb0f6e134ba5217c0da29f62eab38c8c64e604919b1340fda9fda5f9e3914b49573&mpshare=1&scene=1&srcid=1009g9l7CyD3WgIIL9eVsBvM#rd)