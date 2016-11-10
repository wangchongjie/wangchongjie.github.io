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

## 适配器模式模式

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