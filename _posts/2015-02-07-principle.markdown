---
layout:     post
title:      "软件系统设计原则综述"
subtitle:   " \"最佳实践淬炼.\""
date:       2015-02-07 19:00:00
author:     "WangChongjie"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 技术沉淀
---
Facebook的格言是"Move fast and break things",快速行动打破常规，后来扎克伯格将这句话改为"Move fast，with stable infra"，快速行动，稳定架构。不管是打破常规，还是稳定架构，都有些经验原则可循。本文将综述IT领域常用的设计原则，这些原则对编程、架构、设计都会有一定的指导意义。

## DRY原则

DRY是"Don't Repeat Yourself"。其反义词是WET "Write Everything Twice"。其含义是小到一行代码，一个方法，一个类，大至一个模块，一个服务，一个系统，当有类似的需求时，均不要copy拷贝，而需考虑如何进行抽象复用。不要重复自己写过的代码，珍爱时间，拒绝重复性工作。

## KISS原则

KISS是"Keep it simple，stupid"。软件设计中应注意简约的原则。大部分系统设计地越简约，系统复杂度越低，反而越健壮。系统变复杂时，引入的问题也会更多。当一个系统变得复杂时，应考虑将其拆解为多个简单的组件，做好足够的分解和抽象。较简单的解决方法总是更有弹性、柔性，更容易分阶段进行。

## LKP最少知道原则

LKP最少知道原则，是Least Knowledge Principle，又称为迪米特法则（Law of Demeter）。常常用在面向对象软件设计中，它是"松耦合原则"的一个具体实例，是指在面向对象编程中，每一个软件单元应该尽可能少地与其它单元发生作用。应用该原则之后，软件系统能够得到更高的可维护性。因为对象只暴露少数的接口，而隐藏自己的内部结构和实现原理。当内部发生变化时，不会影响整个系统的运作。

## CAP原则

CAP原则又称CAP定理，指的是在一个分布式系统中， Consistency（一致性）、 Availability（可用性）、Partition tolerance（分区容错性），三者不可兼得。系统设计之初就要考虑折衷和取舍。

```xml
  一致性（C）：在分布式系统中的所有数据备份，在同一时刻是否同样的值。
  可用性（A）：在集群中一部分节点故障后，集群整体是否还能响应客户端的读写请求。
  分区容错性（P）：分布式系统在遇到任何网络分区故障的时候，是否仍能够保证对外提供满足一致性和可用性的服务
```

## BASE原则

BASE是为了解决关系数据库，因强一致性导致的可用性降低而提出的解决方案。

```xml
  基本可用（Basically Available）
  软状态（Soft state）
  最终一致（Eventually consistent）
```

## SRP单一职责原则 

Simple responsibility pinciple，对于雨一个类，应该仅有一个引起它变化的原因。如果你能想到多于一个的动机去改变一个类,那么这个类就具有多于一个的职责。应该把多于的指责分离出去，分别再创建一些类来完成每一个职责。

## OCP开-闭原则

Open-Closed Principle，一个软件实体应当对扩展开发，对修改关闭。设计一个模块时,
应使这个模块可以在不被修改的前提下被扩展。可保持系统一定稳定性的基础上，对系统进行扩展。这是面向对象设计（OOD）的基石，也是最重要的原则。
 
## LSP里氏代换原则

Liskov Substitution Principle，该原则由Barbar Liskov提出，是继承复用的基石。一个软件实体如果使用的是一个基类的话,那么一定适用于其子类,而且它根本不能察觉出基类对象和子类对象的区别。这样，基类才能真正被复用，而衍生类也能够在基类的基础上增加新功能。

## DIP依赖倒置原则

Dependence Inversion Principle，抽象不应当依赖于细节，细节应当依赖于抽象。这样构建的代码或模块才是可复用的。

## ISP接口隔离原则

Interface Segregation Principle，一个类对另外一个类的依赖是建立在最小的接口上，通过接口来对实现进行隔离。降低程序的耦合性。

## CARP合成/聚合复用原则

Composite/Aggregate Reuse Principle，要尽量使用合成/聚合，尽量不要使用继承。如此构建的代码，在结构上更为灵活可控。

## To Be Continue

未完待续...
