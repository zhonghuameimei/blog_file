---
title: Effective Java
date: 2021-05-014 16:31:24
tags: 代码优化
categories: Java编码书籍
---

### Effective Java

本书主要解决的三种需求：习惯和高效的用法。

#### 一、创建和销毁对象
##### 1、用静态工厂方法代替构造器
**优势：**
1. 静态工厂方法拥有名称

2. 不必在每次调用它们的时候都创建一个新对象

3. 它们可以返回原返回类型的任何子类型的对象

4. 返回的对象的类可以随着每次调用而发生变化，这取决于静态工厂方法的参数值

5. 方法返回的对象所属的类，在编写包含该静态工厂方法的类时可以不存在

<!-- more -->

**缺点：**

1. 类如果不含公有的或者受保护的构造器，就不能被子类化

2. 程序员很难发现它们
##### 2、遇到多个构造器参数时要考虑使用构建器

~~~java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        //Required param
        private final int servingSize;
        private final int servings;
        //Optional param
        private int calories;
        private int fat;
        private int sodium;
        private int carbohydrate;

        public Builder(int servingSize, int servings){
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val){
            calories = val;
            return this;
        }
        public Builder fat(int val){
            fat = val;
            return this;
        }
        public Builder sodium(int val){
            sodium = val;
            return this;
        }
        public Builder carbohydrate(int val){
            carbohydrate = val;
            return this;
        }

        public NutritionFacts build(){
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder){
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }

    @Override
    public String toString() {
        return "NutritionFacts{" +
                "servingSize=" + servingSize +
                ", servings=" + servings +
                ", calories=" + calories +
                ", fat=" + fat +
                ", sodium=" + sodium +
                ", carbohydrate=" + carbohydrate +
                '}';
    }

    public static void main(String[] args){
        NutritionFacts nutritionFacts = new NutritionFacts.Builder(240,12).calories(3).fat(2).sodium(6).carbohydrate(3).build();
        System.out.println(nutritionFacts);
    }
}
~~~

##### 3、用私有构造器或者枚举类型强化_Singleton_属性



##### 4、通过私有构造器强化不可实例化的能力
##### 5、优先考虑依赖注入来引用资源
##### 6、避免创建不必要的对象
1. 优先使用基本类型而不是装箱基本类型
~~~java
private static long sum(){
	Long sum = 0L;
	for(int i = 0; i <= Integer.MAX_VALUE; i++){
		sum += i;
	}
	return sum;
}
~~~
##### 7、消除过期的对象引用
##### 8、避免使用终结方法和清除方法
**缺点：**
1. 不能保证会被及时执行
2. 会造成严重性能损失
3. 严重的安全问题
##### 9、try-with-resources 优先 try-finally

在处理必须关闭的资源时，始终要优先考虑用 try-with-resources ，而不是 try-finally 这样得到的代码将更加简洁、清晰，产生的异常也更有价值 有了 try-with-resources 语句，在使用必须关闭的资源时，就能更轻松地正确编写代码了 实践证明， 这个用 try-finally 是不可能做到的

~~~java
//try-with-resources
try(BufferedReader reader = new BufferedReader(new FileReader("/test"))){
    reader.readLine();
}catch (IOException e) {
    e.printStackTrace();
}
//try-finally
BufferedReader reader = null;
try{
    reader = new BufferedReader(new FileReader("/test"));
    reader.readLine();
}catch (IOException e) {
    e.printStackTrace();
}finally {
    try {
        reader.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
~~~

#### 二、对于所有对象都通用的方法

##### 1、覆盖 equals 时请遵守通用约定

##### 2、覆盖 equals 时总要覆盖 hash Code

**Object规范：** 

1. 在应用程序的执行期间，只要对象的 equals 方法的比较操作所用到的信息没有被 修改，那么对同一个对象的多次调用， hashCode 方法都必须始终返回同一个值 在一个应用程序与另一个程序的执行过程中，执行 hashCode 方法所返回的值可以 不一致
2. 如果两个对象根据 equals(Object)方法比较是相等的，那么调用这两个对象中的 hashCode 方法都必须产生同样的整数结果
3. 如果两个对象根据 equals(Object)方法比较是不相等的，那么调用这两个对象 中的 hashCode 方法，则不一定要求 hashCode 方法必须产生不同的结果 但是程 序员应该知道，给不相等的对象产生截然不同的整数结果，有可能提高散列表(hash table) 的性能

##### 3、始终要覆盖 toString

虽然 Object 提供了 toString 方法的 个实现，但它返回的字符串通常并不是类 的用户所期望看到的 它包含类的名称，以及 个“＠”符号，接着是散列码的无符号 十六进制表示法，例如 PhoneNumber@163b91 toString 的通用约定指出，被返回 的字符串应该是一个“简洁的但信息丰富，并且易于阅读的表达形式” 尽管有人认为 PhoneNumber@163b91 算得上是简洁和易于阅读了，但是与 707-867-5309 比较起来，它还算不上是信息丰富的 toString 约定进一步指出，“建议所有的子类都覆盖这个方法 这是一个很好的建议

##### 4、谨慎的覆盖 clone

##### 5、考虑实现 Comparable 接口

#### 三、类和接口

##### 1、使类和成员的可访问性最小化

##### 2、要在公有类而非公有域中使用访问方法

##### 3、使可变性最小化

##### 4、复合优先于继承

##### 5、要么设计继承并提供文档说明， 要么禁止继承

##### 6、接口优于抽象类

##### 7、为后代设计接口

##### 8、接口只用于定义类型

##### 9、类层次优于标签类

##### 10、静态成员类优于非静态成员类

##### 11、限制源文件为单个顶级类