---
title: 行为型模式
date: 2021-05-07 16:58:24
tags: 行为型模式
categories: 设计模式
---
#### 一、观察者模式
当对象间存在一对多关系时，则使用观察者模式（Observer Pattern）。比如，当一个对象被修改时，则会自动通知依赖它的对象。观察者模式属于行为型模式。
<!-- more -->
##### 1、意图
定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新
##### 2、应用场景
1. 一个抽象模型有两个方面，其中一个方面依赖于另一个方面。将这些方面封装在独立的对象中使它们可以各自独立地改变和复用
2. 一个对象的改变将导致其他一个或多个对象也发生改变，而不知道具体有多少对象将发生改变，可以降低对象之间的耦合度
3. 一个对象必须通知其他对象，而并不知道这些对象是谁
4. 需要在系统中创建一个触发链，A对象的行为将影响B对象，B对象的行为将影响C对象……，可以使用观察者模式创建一种链式触发机制
##### 3、实现
观察者模式使用三个类 Subject、Observer 和 Client。Subject 对象带有绑定观察者到 Client 对象和从 Client 对象解绑观察者的方法。我们创建  _Subject_  类、_Observer_  抽象类和扩展了抽象类  _Observer_  的实体类。
_ObserverPatternDemo_，我们的演示类使用  _Subject_  和实体类对象来演示观察者模式。
![观察者模式的 UML 图](./picture/observer_pattern_uml_diagram.jpg)

**创建主题类**
```java
package test09;

import java.util.ArrayList;
import java.util.List;

public class Subject {

	private int state;
	
	private List<Observer> observers = new ArrayList<>();

	public int getState() {
		return state;
	}
	
	public void attach(Observer observer){
		observers.add(observer);
	}

	public void setState(int state) {
		this.state = state;
		notifyObserver();
	}

	public void notifyObserver(){
		for (Observer observer : observers) {
			observer.update(); 
		}
	}
}
```
**创建观察类**
```java 
package test09;

public abstract class Observer {

	protected Subject subject;
	
	abstract void update();
}
```
**创建观察实现类**
```java
package test09;

public class BinaryObserver extends Observer{
	
	public BinaryObserver(Subject subject) {
		this.subject = subject;
		this.subject.attach(this);
	}

	@Override
	void update() {
		System.out.println("Binary String: "+subject.getState());
	}

}
```
**测试类**
```java
package test09;

public class Demo {

	public static void main(String[] args) {
		Subject subject = new Subject();
		new BinaryObserver(subject);
		
		subject.setState(1);
		
	}
}
```
**java自带观察者模式工具类（_Observer_ _Observable_）**
```java
package test0901;

import java.util.Observable;

public class Subject extends Observable{

	private int state;

	public int getState() {
		return state;
	}

	public void setState(int state) {
		this.state = state;
		setChanged();
		notifyObservers();
	}
}
```
```java
package test0901;

import java.util.Observable;
import java.util.Observer;

public class BinaryObserver implements Observer{

	public BinaryObserver(Subject subject) {
		subject.addObserver(this);
	}
	
	@Override
	public void update(Observable observable, Object arg) {
		System.out.println("状态发生改变"+ ((Subject)observable).getState());
	}
	
}
```
```java
package test0901;

public class Demo {

	public static void main(String[] args) {
		Subject subject = new Subject();
		new BinaryObserver(subject);
	}
}
```




