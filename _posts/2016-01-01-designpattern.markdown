---
layout:     post
title:      "Design Pattern之Java设计模式"
subtitle:   " \"可复用面向对象的基础.\""
date:       2016-01-01 16:00:00
author:     "WangChongjie"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 设计模式
---
Christopher Alexander说过："每一个模式描述了一个在我们周围不断重复发生的问题，以及该问题的解决方案的核心。这样，就能一次又一次地使用该方案而不必做重复的劳动"。
本文列举一些常用设计模式的代码示例，即Principle，Pattern、Practice之模式部分。

## 常用的设计模式归类

创建型模式：

```xml
ABSTRACT FACTORY 抽象工厂模式
BUILDER 生成器模式
FACTORY METHOD 工厂方法模式
PROTOTYPR 原型模式
SINGLETON 单例模式
```

结构型模式：

```xml
ADAPTER 适配器模式
BRIDGE 桥接模式
COMPOSITE 组装模式
DECORATOR 装饰器模式
FACADE 外观模式
FLYWEIGHT 享元模式
PROXY 代理模式
```

行为模式：

```xml
CHAIN OF RESPONSIBILITY 职责链模式
COMMAND 命令模式
INTERPRETER 解释器模式
ITERATOR 迭代器模式
FACADE 外观模式
MEDIATOR 中介者模式
MEMENTO 备忘录模式
OBSERVER 观察者模式
STATE 状态模式
TEMPLATE METHOD 模板模式
VISITOR 访问者模式
MEMENTO 备忘录模式
STRATEGY 策略模式
```

## ADAPTER模式

Shape.java

```java
package adapter;

public interface Shape {
	public void Draw();
	public void Border();
}
```

Text.java

```java
package adapter;

public class Text {
	private String content;
	public Text(){
		
	}
	public void setContent(String str){
		content = str;
	}
	public String getContent(){
		return content;
	}
}
```

TextShapeObject.java

```java
package adapter;

public class TextShapeObject implements Shape{
	private Text txt;
	public TextShapeObject(Text t){
		txt = t;
	}
	public void Draw(){
		System.out.println("Draw a Shap!implement Shap interface!");
	}
	public void Border(){
		System.out.println("Set the Border of Shap!implement Shap interface!");
	}
	public void setContent(String str){
		txt.setContent(str);
	}
	public String getContent(){
		return txt.getContent();
	}
	
	public static void main(String[] args){
		Text myText = new Text();
		TextShapeObject myTextShapeObject = new TextShapeObject(myText);
		myTextShapeObject.Draw();
		myTextShapeObject.Border();
		myTextShapeObject.setContent("A Text Shape");
		System.out.println(myTextShapeObject.getContent());
	}
}
```

TextShapeClass.java

```java
package adapter;

public class TextShapeClass  extends Text implements Shape {
    public TextShapeClass() {
    }
    public void Draw() {
        System.out.println("Draw a shap ! Impelement Shape interface !");
    }
    public void Border() {
        System.out.println("Set the border of the shap ! Impelement Shape interface !");
    }
    public static void main(String[] args) {
        TextShapeClass myTextShapeClass = new TextShapeClass();
        myTextShapeClass.Draw();
        myTextShapeClass.Border();
        myTextShapeClass.setContent("A test text !");
        System.out.println("The content in Text Shape is :" + myTextShapeClass.getContent());
    }
}
```

## Observer模式

ConcreteSubject.java

```java
package observer;

import java.util.*;
import java.io.*;

public class ConcreteSubject implements Subject {
    private LinkedList observerList;
    private Vector strVector;
    
    public ConcreteSubject() {
        observerList =  new LinkedList();
        strVector = new Vector();
    }
    public void attach(Observer o) {
        observerList.add(o);
    }
    public void detach(Observer o) {
        observerList.remove(o);
    }
    public void sendNotify() {
        for(int i = 0; i < observerList.size(); i++) {
            ((Observer)observerList.get(i)).update(this);   
        }
    }
    public void setState(String act, String str) {
        if(act.equals("ADD")) {
            strVector.add(str);
        } else if(act.equals("DEL")) {
            strVector.remove(str);
        }
    }
    public Vector getState() {
        return strVector;
    }
}
```

observer.java

```java
package observer;

public interface Observer {
	public void update(Subject s);
}

```

ObserverA.java

```java
package observer;

import java.util.Vector;

public class ObserverA implements Observer{
	private Vector strVector;
	private Subject sub;
	public ObserverA(Subject s){
		sub = s;
	}
	public void update(Subject subject){
		strVector = subject.getState();
        System.out.println("----- ObserverA will be updated -----");
        for(int i = 0; i < strVector.size(); i++) {
            System.out.println("Num " + i + " is :" + (String)strVector.get(i));
        }
	}
	public void change(String action,String str){
		sub.setState(action, str);
	}
	public void notifySub(){
		sub.sendNotify();
	}
}
```

ObserverB.java

```java
package observer;

import java.io.*;
import java.util.*;

public class ObserverB implements Observer {
	private Vector strVector;

	public ObserverB() {
		strVector = new Vector();
	}

	public void update(Subject subject) {
		strVector = (Vector) (subject.getState()).clone();
		// ----- Sorted vector ---------------------------
		for (int i = strVector.size(); --i >= 0;) {
			for (int j = 0; j < i; j++) {
				String str1 = (String) strVector.get(j);
				String str2 = (String) strVector.get(j + 1);
				if ((str1.compareTo(str2)) > 0) {
					strVector.setElementAt(str2, j);
					strVector.setElementAt(str1, j + 1);
				}
			}
		}
		System.out.println("----- ObserverB will be updated -----");
		for (int i = 0; i < strVector.size(); i++) {
			System.out.println("Num " + i + " is :" + (String) strVector.get(i));
		}
	}

}
```

Subject.java

```java
package observer;

import java.util.Vector;

public interface Subject {
	public abstract void attach(Observer o);
	public abstract void detach(Observer o);
	public abstract void sendNotify();
	
	public abstract Vector getState();
	public abstract void setState(String act,String str);
}
```

Test.java

```java
package observer;

public class Test  {
    public static void main(String[] args) {
        Subject mySub = new ConcreteSubject();
        ObserverA myObserverA = new ObserverA(mySub);
        ObserverB myObserverB = new ObserverB();
        mySub.attach(myObserverA);
        mySub.attach(myObserverB);

        mySub.setState("ADD", "One --- 1");
        mySub.setState("ADD", "Tow --- 2");
        mySub.sendNotify();

        myObserverA.change("DEL", "Tow --- 2");
        myObserverA.change("ADD", "Three --- 3");
        myObserverA.change("ADD", "Four --- 4");
        myObserverA.notifySub();  
    }
}
```