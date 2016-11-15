---
layout:     post
title:      "软件系统设计原则综述"
subtitle:   " \"最佳实践淬炼.\""
date:       2014-12-08 19:00:00
author:     "WangChongjie"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 技术沉淀
---
Facebook的格言是"Move fast and break things",快速行动打破常规，后来扎克伯格将这句话改为"Move fast，with stable infra"，快速行动，稳定架构。不管是打破常规，还是稳定架构，有有些经验原则可循。本文将综述IT领域常用的设计原则，这些原则对编程、架构、设计都会有一定的指导意义。

## DRY原则

DRY是"Don't Repeat Yourself"。其反义词是WET "Write Everything Twice"。其含义是小到一行代码，一个方法，一个类，大至一个模块，一个服务，一个系统，当有类似的需求时，均不要copy拷贝，而需考虑如何进行抽象复用。不要重复自己写过的代码，珍爱时间，拒绝重复性工作。

## KISS原则

KISS是"Keep it simple，stupid"。软件设计中应注意简约的原则。大部分系统设计地越简约，系统复杂度越低，反而越健壮。系统变复杂时，引入的问题也会更多。当一个系统变得复杂时，应考虑将其拆解为多个简单的组件，做好足够的分解和抽象。较简单的解决方法总是更有弹性、柔性，更容易分阶段进行。

## LKP最少知道原则

LKP最少知道原则，是Least Knowledge Principle，又称为迪米特法则（Law of Demeter）。常常用在面向对象软件设计中，它是"松耦合原则"的一个具体实例，是指在面向对象编程中，每一个软件单元应该尽可能少地与其它单元发生作用。应用该原则之后，软件系统能够得到更高的可维护性。因为对象只暴露少数的接口，而隐藏自己的内部结构和实现原理。当内部发生变化时，不会影响整个系统的运作。

## To Be Continue

