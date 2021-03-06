# 微服务

让我们讨论一下软件开发的未来。

改变正在进行之中。我们正越来越多地走向一个以技术为核心的世界。在这个时代保持竞争优势变得越来越困难。当组织试图利用低效的平台、流程和结构进行扩展时，其执行能力可能会逐渐停止。已有10年历史的科技公司已经经历了这些规模扩张的痛苦，大多数公司都使用了相同的方法来克服这些挑战。

是时候把世界上最成功公司的竞争优势带给其他人了。因此，让我们谈谈微服务，一种创造竞争优势的方式。

## 什么是微服务？

微服务是一种软件体系架构模式，用于将大型单体应用程序分解为更小的可管理的独立服务，这些服务通过与语言无关的协议进行通信，每个服务都专注于做好一件事。
行业专家对微服务的定义。

> 具有有限上下文的松散耦合的面向服务的体系结构。
</br><font size=1>Adrian Cockcroft</font>

> 一种将单个应用程序开发为一组小型服务的方法，每个服务运行在自己的流程中，并以轻量级机制进行通信
</br><font size=1>Martin Fowler</font>

微服务的概念并不新鲜，这是对面向服务的体系结构的重新设想，但是采用了一种与unix进程和管道更全面地一致的方法。

微服务架构的设计哲学:

- 服务是小粒度的，作为一种单一的业务用途，类似于unix的“只做一件事，做好它”的哲学

- 组织文化应该包含部署和测试的自动化。这减轻了管理和运维的负担

- 文化和设计原则应该包含失败和错误，类似于抗脆弱(anti-fragile )系统。

## 为什么是microservices？

随着组织同时扩展技术和人员数量，管理单一代码库变得更加困难。在一段时间内，我们都习惯了Twitter的fail whale，因为他们试图通过一个单一的系统来扩展他们的用户群和产品特性集。微服务使Twitter能够将其应用程序分解为更小的服务，这些服务可以由许多不同的团队单独管理。每个团队负责由许多微服务组成的业务功能，这些微服务可以独立于其他团队进行部署。

<img src="https://micro.mu/docs/images/micro-service-architecture.png"/>

通过第一手经验，我们已经了解到微服务系统能够加快开发周期、提高生产力和更好的可伸缩系统。

让我们来谈谈它的一些好处:

1. **更容易扩展开发**——团队围绕不同的业务需求进行组织，并管理自己的服务。
2. **更容易理解**——微服务要小得多，通常是1000 LOC或更少。
3. **更容易频繁地部署新版本的服务**——可以独立地部署、扩展和管理服务。
4. **改进的容错性和隔离** ——关注点的分离将一个服务中的问题与另一个服务中的问题的影响降到最低。
5. **提高执行速度** ——团队通过独立开发、部署和管理微服务，更快地交付业务需求。
6 **可重用服务和快速原型设计**——unix在微服务中根深蒂固的哲学允许您重用现有的服务，并更快地在其上构建全新的功能。

## 什么是Micro

Micro是一个微服务生态系统，专注于提供产品、服务和解决方案，以支持现代软件驱动企业的创新。我们计划成为任何与微服务相关的实际资源，并将使公司能够为自己的业务利用这一技术。从早期的原型设计一直到大规模的生产部署。

我们看到这个行业正在发生根本性的转变。摩尔定律是有效的，我们每天都能得到越来越多的计算能力。然而，我们无法完全实现这种新能力。现有的工具和开发实践在这个新时代无法扩展。没有为开发人员提供从单体代码基转移到更有效的设计模式的工具。大多数公司不可避免地会因为单一的设计而达到收益递减的地步，并不得不进行大规模的研发再造工作。Netfix、Twitter、Gilt和Hailo都是最好的例子。他们最终都建立了自己的微服务平台。

我们的愿景是提供基本的构建模块，使任何人更容易采用微服务。