---
title: 创建型模式
date: 2021-04-27 14:25:27
tags: 创建型模式
categories: 设计模式
---
**创建型模式**：这些设计模式提供了一种在创建对象的同时隐藏创建逻辑的方式，而不是使用 new 运算符直接实例化对象。这使得程序在判断针对某个给定实例需要创建哪些对象时更加灵活。包括（[单例模式](#singleton)、[原型模式](#propertype)、[工厂模式](#factory)、[抽象工厂模式](#abstract)、[建造者模式](#build)）<!--more-->

#### <span id = "singleton">一、单例模式</span>
**单例模式**（Singleton Pattern）是 Java 中最简单的设计模式之一。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。
这种模式涉及到一个单一的类，该类负责创建自己的对象，同时确保只有单个对象被创建。这个类提供了一种访问其唯一的对象的方式，可以直接访问，不需要实例化该类的对象。

##### 注意：
 1. 单例类只能有一个实例。
 2. 单例类必须自己创建自己的唯一实例。
 3. 单例类必须给所有其他对象提供这一实例。
##### 1、懒汉式：线程不安全
```java
public  class  Singleton  {  
	private  static  Singleton  instance;
	//构造函数私有化，不允许其他类创建
	private  Singleton  (){}  
	public  static  Singleton  getInstance()  {
	    if  (instance == null)  {  
		   instance = new  Singleton(); 
		}  
		return  instance; 
	} 
}
```
##### 2、懒汉式：线程安全
```java
public class Singleton  {  
	private static Singleton instance;
	//构造函数私有化，不允许其他类创建
	private Singleton(){}  
	public static synchronized Singleton getInstance() {
	    if  (instance == null)  {  
		   instance = new  Singleton(); 
		}  
		return instance; 
	} 
}
```
##### 3、饿汉式：线程安全（推荐）
```java 
public class Singleton{
	private static Singleton instance = new Singleton();
	public static Singleton getInstance(){
		return instance;
	}
}
```
##### 4、双重校验锁：线程安全
```java
public class Singleton  {  
	private volatile static Singleton instance;
	//构造函数私有化，不允许其他类创建
	private Singleton (){}  
	public static Singleton getInstance() {
	    if  (instance == null)  {  
		   synchronized(Singleton.class){
			   if(instance == null){
				   instance = new  Singleton(); 
			   }
		   }
		}  
		return  instance; 
	} 
}
```
##### 5、枚举：多线程安全
```java
public enum Singleton { 
	INSTANCE; 
	public void whateverMethod() {} 
}
```

#### <span id = "propertype">二、原型模式</span>
用于创建重复的对象，同时又保证性能，属于创建型模式，提供了一种创建对象的最佳方式。这种模式实现了原型接口，该接口用于创建该对象的克隆，当创建对象的代价较大时，则会使用这种模式
##### 何时使用
 1. 当一个系统应该独立于它的产品创建，构成和表示时
 2. 当要实例化的类是在运行时刻指定时，例如，通过动态装载
 3. 为了避免创建一个与产品类层次平行的工厂类层次时
 4. 当一个类的实例只能有几个不同状态组合中的一种时。建立相应数目的原型并克隆它们可能比每次用合适的状态手工实例化该类更方便一些
##### 关键步骤
1. 实现克隆操作，在 JAVA 继承 Cloneable，重写 clone()
2. 原型模式同样用于隔离类对象的使用者和具体类型（易变类）之间的耦合关系，它同样要求这些"易变类"拥有稳定的接口
##### 深克隆/浅克隆
**深克隆：**深拷贝会拷贝所有的属性,并拷贝属性指向的动态分配的内存。当对象和它所引用的对象一起拷贝时即发生深拷贝。深拷贝相比于浅拷贝速度较慢并且花销较大。
**浅克隆：**浅拷贝是按位拷贝对象，它会创建一个新对象，这个对象有着原始对象属性值的一份精确拷贝。如果属性是基本类型，拷贝的就是基本类型的值；如果属性是内存地址（引用类型），拷贝的就是内存地址 ，因此如果其中一个对象改变了这个地址，就会影响到另一个对象

##### 代码实现

**1、创建实现Cloneable接口的抽象类**
```java
package test02;

public abstract class Shape implements Cloneable{

	private String id;
	
	protected String type;
	
	abstract void draw();

	public String getId() {
		return id;
	}

	public void setId(String id) {
		this.id = id;
	}

	public String getType() {
		return type;
	}

	public void setType(String type) {
		this.type = type;
	}
	
	public Object clone(){
		Object clone = null;
		try {
			clone = super.clone();
		} catch (Exception e) {
			e.printStackTrace();
		}
		return clone;
	}
}
```
**2、创建继承Shape类的实体类Rectangle**
```java
package test02;

public class Rectangle extends Shape{

	public Rectangle(){
	    type = "Rectangle";
	}
	 
	@Override
	public void draw() {
		System.out.println("Inside Rectangle::draw() method.");
	}
}
```
**3、创建继承Shape类的实体类Circle**
```java
package test02;

public class Circle extends Shape{

	public Circle(){
	    type = "Circle";
	}
	
	@Override
	void draw() {
		System.out.println("Inside Circle::draw() method.");
	}
}
```
**4、创建继承Shape类的实体类Square**
```java
package test02;

public class Square extends Shape{

	public Square(){
	    type = "Square";
	}
	
	@Override
	void draw() {
		System.out.println("Inside Square::draw() method.");
	}
}
```
**5、创建虚拟数据存储以及深/浅克隆的类**
```java
package test02;

import java.util.Hashtable;

public class ShapeCache {
	
	private static Hashtable<String, Shape> shapeMap = new Hashtable<String, Shape>();

	//浅拷贝
	public static Shape getShape(String shapeId) {
		Shape cachedShape = shapeMap.get(shapeId);
		return (Shape) cachedShape.clone();
	}
	
	//深拷贝
	public static Shape getDeepShape(String shapeId) {
		Shape cachedShape = shapeMap.get(shapeId);
		Shape cachedShape1 = new Shape() {
			@Override
			void draw() {
				System.out.println("new Class");
				
			}
		};
		cachedShape1.setId(cachedShape.getId());
		cachedShape1.setType(cachedShape.getType());
		return cachedShape1;
	}
	
	public static void loadCache() {
        Circle circle = new Circle();
        circle.setId("1");
        shapeMap.put(circle.getId(),circle);
 
        Square square = new Square();
        square.setId("2");
        shapeMap.put(square.getId(),square);
 
        Rectangle rectangle = new Rectangle();
        rectangle.setId("3");
        shapeMap.put(rectangle.getId(),rectangle);
    }
}
```
**6、测试类**

```java
package test02;

public class Demo {

	public static void main(String[] args) {
		ShapeCache.loadCache();
		
		Shape deepCloneShape = ShapeCache.getDeepShape("1");
		deepCloneShape.draw();
		Shape cloneShape1 = ShapeCache.getShape("1");
		Shape cloneShape4 = ShapeCache.getShape("1");
		System.out.println(cloneShape1 == cloneShape4);
		System.out.println(cloneShape1 == deepCloneShape);
		Shape cloneShape2 = ShapeCache.getShape("2");
		cloneShape2.draw();
		Shape cloneShape3 = ShapeCache.getShape("3");
		cloneShape3.draw();
	}
}
```

