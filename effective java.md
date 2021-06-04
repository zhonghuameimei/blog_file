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
	//构建器
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

**注意：**享有特权的客户端可以借助 AccessibleObject.setAccessible 方法， 通过反射机制调用私有构造器 如果需要抵御这种攻击，可以修改构造器，让它在被要求创建第 个实例的时候抛出异常

**私有构造器**

~~~java
public class Elvis {
    private static final elvis = new Elvis();
    private Elvis(){}
    public static Elvis getInstance(return elvis){}
}
~~~

**枚举**

~~~java
public enum Elvis {
    INSTANCE;
}
~~~

##### 4、通过私有构造器强化不可实例化的能力

~~~java
public class UtilityClass {
    private UtilityClass(){
        throw  new AssertionError();//防止内部调用，同时无法被子类化
    }
}
~~~

##### 5、优先考虑依赖注入来引用资源

~~~java
//简称依赖注入
public class SpellChecker {
    private final Lexicon dictionary;
    public SpellChecker(Lexicon dictionary){
        this.dictionary = Objects.requireNonNull(dictionary);
    }
}

public class Lexicon {
}
~~~

##### 6、避免创建不必要的对象
~~~java
public class NullClass {
    public static void main(String[] args) {
        //每次执行都会创建一个String对象
        String s = new String("s");
    }

    //每次调用在matches内部都会创建Pattern对象
    public static boolean isRomanNumeral1(String s){
        return s.matches("[A-Z]");
    }

    //将pattern初始化并缓存，作为不可变的实列，仅此一份
    private static final Pattern pattern = Pattern.compile("[A-Z]");
    public static boolean isRomanNumeral2(String s){
        return pattern.matcher(s).matches();
    }

    //自动装箱会创建对象实列，优先使用基本类型而不是装箱基本类型
    private static long sum(){
        Long sum = 0L;
        for(int i = 0; i <= Integer.MAX_VALUE; i++){
            sum += i;
        }
        return sum;
    }
}
~~~
##### 7、消除过期的对象引用

清空对象引用应该是一种例外 而不是一种规范行为

##### 8、避免使用终结方法（finalizer）和清除方法（cleaner）
**缺点：**

1. 不能保证会被及时执行
2. 会造成严重性能损失
3. 严重的安全问题
##### 9、try-with-resources 优先 try-finally

在处理必须关闭的资源时，始终要优先考虑用 try-with-resources ，而不是 try-finally 这样得到的代码将更加简洁、清晰，产生的异常也更有价值 有了 try-with-resources 语句，在使用必须关闭的资源时，就能更轻松地正确编写代码了 实践证明， 这个用 try-finally 是不可能做到的

~~~java
public class tryClass {

    public void tryWithResource(String filePath){
        //try-with-resources
        try(BufferedReader reader = new BufferedReader(new FileReader(filePath))){
            reader.readLine();
        }catch (IOException e) {
            e.printStackTrace();
        }
    }

    public void tryFinally(String filePath){
        //try-finally
        BufferedReader reader = null;
        try{
            reader = new BufferedReader(new FileReader(filePath));
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
    }
}
~~~

#### 二、对于所有对象都通用的方法

##### 1、覆盖 equals 时请遵守通用约定

约定内容：

**自反性（reflexive）：**对于任何非 null 的引用值 x，x.equals(x) 必须返回 true

 （对象必须等于其自身）

**对称性（symmetric）：**对于任何非null的引用值 x 和 y，当且仅当 y.equals(x) 返回 true 时，x.equals(y) 必须返回 true

（任何两个对象对于“他们是否相等”的问题都必须保持一致）

**传递性（transitive）：**对于任何非 null 的引用值 x，y 和 z，如果 x.equals(y) 返回 true ，并且 y.equals(z) 也返回 true ，那么 x.equals(z) 也必须返回 true

（如果一个对象等于第二个对象，而第二个对象又等于第三个对象，则第一个对象等于第三个对象）

**一致性（consistent）：**对于任何非 null 引用值 x 和 y，只要 equals 的比较操作在对象中所用的信息没有被修改，多次调用 x.equals(y) 就会一致地返回 true, 或者一致地返回 false

（如果两个对象相等，它们就 必须始终保持相等，除非它们中有一个对象（或者两个都）被修改了）

##### 2、覆盖 equals 时总要覆盖 hashCode

**Object规范：** 

1. 在应用程序的执行期间，只要对象的 equals 方法的比较操作所用到的信息没有被 修改，那么对同一个对象的多次调用， hashCode 方法都必须始终返回同一个值 在一个应用程序与另一个程序的执行过程中，执行 hashCode 方法所返回的值可以 不一致
2. 如果两个对象根据 equals(Object)方法比较是相等的，那么调用这两个对象中的 hashCode 方法都必须产生同样的整数结果
3. 如果两个对象根据 equals(Object)方法比较是不相等的，那么调用这两个对象 中的 hashCode 方法，则不一定要求 hashCode 方法必须产生不同的结果 但是程 序员应该知道，给不相等的对象产生截然不同的整数结果，有可能提高散列表(hash table) 的性能

##### 3、始终要覆盖 toString

虽然 Object 提供了 toString 方法的 个实现，但它返回的字符串通常并不是类 的用户所期望看到的 它包含类的名称，以及 个“＠”符号，接着是散列码的无符号 十六进制表示法，例如 PhoneNumber@163b91 toString 的通用约定指出，被返回 的字符串应该是一个“简洁的但信息丰富，并且易于阅读的表达形式” 尽管有人认为 PhoneNumber@163b91 算得上是简洁和易于阅读了，但是与 707-867-5309 比较起来，它还算不上是信息丰富的 toString 约定进一步指出，“建议所有的子类都覆盖这个方法 这是一个很好的建议

##### 4、谨慎的覆盖 clone

##### 5、考虑实现 Comparable 接口

~~~java
 public static void main(String[] args) {
     Student student = new Student();
     Student student1 = new Student();
     comparable.compare(student,student1);
     comparator1.compare(student,student1);
 }

static Comparator comparable = new Comparator() {
    @Override
    public int compare(Object o1, Object o2) {
        return Integer.compare(o1.hashCode(),o2.hashCode());
    }
};

static Comparator comparator1 = Comparator.comparing(o -> o.hashCode());
~~~



#### 三、类和接口

##### 1、使类和成员的可访问性最小化

##### 2、要在公有类而非公有域中使用访问方法

公有类永远都不应该暴露可变的域 虽然还是有问题，但是让公有类暴露 不可变的域，其危害相对来说比较小但有时候会需要用包级私有的或者私有的嵌套类来暴露域，无论这个类是可变的还是不可变的

##### 3、使可变性最小化

**类不可变遵循的规则：**

1. 不要提供任何会修改对象状态的方法（也称为设值方法）
2. 保证类不会被扩展
3. 声明所有的域都是final
4. 声明所有的域都为私有
5. 确保对于任何可变组件的互斥访问

##### 4、复合优先于继承

##### 5、要么设计继承并提供文档说明， 要么禁止继承

##### 6、接口优于抽象类

##### 7、为后代设计接口

##### 8、接口只用于定义类型

常量接口模式是对接口的不良使用

~~~java
public interface ObjectStreamConstants {

    final static short STREAM_MAGIC = (short)0xaced;

    final static short STREAM_VERSION = 5;

    final static byte TC_BASE = 0x70;
}
~~~

##### 9、类层次优于标签类

**标签类：**标签类过于冗长、容易出错，并且效率低下

~~~java
class Figure{
    enum Shape {RECTANGLE, CIRCLE};
    final Shape shape;
    double length;
    double width;
    double radius;
    Figure(double radius){
        this.shape = Shape.CIRCLE;
        this.radius = radius;
    }

    Figure(double length, double width){
        this.shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area(){
        switch(shape){
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default: throw new AssertionError(shape);
        }
    }
}
~~~

**类层次**

~~~java
public interface Shape {
    double area();
}

class Reclangle implements Shape{

    double length;
    double width;

    public Reclangle(double length, double width){
        this.length = length;
        this.width = width;
    }

    @Override
    public double area() {
        return length * width;
    }
}

class Circle implements Shape{

    double radius;

    Circle(double radius){
        this.radius = radius;
    }

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}
~~~

##### 10、静态成员类优于非静态成员类

##### 11、限制源文件为单个顶级类

#### 四、泛型

##### 1、请不要使用原生态类型

~~~java
Set set1 = new HashSet();; //原生态类型
Set<String> set2 = new HashSet<>(); //参数化类型，泛型
Set<?> set3 = new HashSet<>(); //无限制通配符类型
Set<? extends Number> set4 = new HashSet<>(); //有限制通配符类型
~~~

##### 2、消除非受检的警告

```java
@SuppressWarnings("unchecked")
Set<String> set2 = new HashSet<>();
```

##### 3、列表优于数组

**数组：**运行时才会发现错误

**列表：**编译时就能发现错误

数组和泛型有着截然不同的类型规则。数组是协变且可以具体化的；泛型是不可变的且可以被擦除的。因此，数组提供了运行时的类型安全，但是没有编译时的类型安全，反之，对于泛型也一样。一般来说，数组和泛型不能很好地混合使用。如果你发现自己将它们混合起来使用，并且得到了编译时错误或者警告，你的第一反应就应该是用列表代替数组

##### 4、优先考虑泛型

使用泛型比使用需要在客户端代码中进行转换的类型来得更加安全，也更 加容易 在设计新类型的时候，要确保它们不需要这种转换就可以使用 这通常意味着要把 类做成是泛型的 只要时间允许，就把现有的类型都泛型化 这对于这些类型的新用户来说 会变得更加轻松，又不会破坏现有的客户端

##### 5、优先考虑泛型方法

泛型方法就像泛型一样，使用起来比要求客户端转换输入参数并返回值的 方法来得更加安全，也更加容易，就像类型一样 ，你应该确保方法不用转换就能使用，这通 常意味着要将它们泛型化，并且就像类型一样，还应该将现有的方法泛型化，使新用户使用起来更加轻松 ，且不会破坏现有的客户端

~~~java
public class CollectionLearn extends Stack{

    public static void main(String[] args) {
        List<Vehicle> cars = new ArrayList<>();
        swap(cars);
    }

    public static void swap(List<? extends Vehicle> vehicles){

    }
}
~~~



##### 6、利用有限制通配符来提示 API 的灵活性

##### 7、谨慎并用泛型和可变参数

##### 8、优先考虑类型安全的异构容器

~~~java
public class Favorites {

    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorites(Class<T> type, T instance){
        favorites.put(Objects.requireNonNull(type),instance);
    }

    public <T> T getFavorites(Class<T> type){
        return type.cast(favorites.get(type));
    }
}
~~~



#### 五、枚举和注解

##### 1、用 enum 代替 int 变量

~~~java
public enum PayrollDay {

    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY(PayType.WEEKEND), SUNDAY(PayType.WEEKEND);

    private final PayType payType;

    PayrollDay(){ this.payType = PayType.WEEKDAY; }

    PayrollDay(PayType payType){ this.payType = payType; }

    int pay(int minsWorked, int payRate){
        return payType.pay(minsWorked,payRate);
    }

    private enum PayType{
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate){
                return minsWorked <= MINX_PER_SHIFT ? 0 : (minsWorked - MINX_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate){
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int mins, int payRate);

        private static final int MINX_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate){
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked,payRate);
        }
    }
}
~~~

```java
public enum Calculate {

    PLUS{
        int doCalculate(int num, int num2){
            return num + num2;
        }
    },
    MINUS{
        int doCalculate(int num, int num2){
            return num - num2;
        }
    },
    TIMES{
        int doCalculate(int num, int num2){
            return num * num2;
        }
    },
    DIVIDE{
        int doCalculate(int num, int num2){
            return num / num2;
        }
    };

    abstract int doCalculate(int num, int num2);
}
```





