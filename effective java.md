---
title: Effective Java
date: 2021-05-14 16:31:24
tags: 代码优化
categories: Java编码书籍
---

### Effective Java

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

##### 2、用实例域代替序数

永远不要根据枚举的序数导出与他关联的值，而是将他保存在实例域中

##### 3、 EnumSet 位域

##### 4、用接口拟可扩展的枚举

##### 5、注解优先于命名模式

所有的程序员都应该使用 Java 平台所提供的预定义的注解类型还要考虑使用 IDE 或者静态分析工具所提供的任何注解 这种注解可以提升由这些工具所提供的诊断信息的质量 但是要注意这些注解还没有标准化，因此 如果变换工具或者形成标准，就有很多工作要做了

##### 6、坚持使用 Override 注解

在你想要的每个方法声明中使用 Override 注解来覆盖超类声明，编译器就可以替你防止大量的错误，但有一个例外 在具体的类中，不必标注你确信覆盖了抽象方法声明的方法（虽然这么做也没有什么坏处）

#### 六、Lambda 和 Stream

##### 1、Lambda 优先于匿名类

~~~java
Collections.sort(new ArrayList<String>(), new Comparator<String>() {
    @Override
    public int compare(String o1, String o2) {
        return Integer.compare(o1.length(),o2.length());
    }
});

Collections.sort(new ArrayList<String>(),(o1, o2) -> Integer.compare(o1.length(),o2.length()));

Collections.sort(new ArrayList<String>(),Comparator.comparingInt(String::length));
~~~

##### 2、 方法引用优先于 Lambda

 只要方法引用更加简洁、清晰，就用方法引用；如果方法引用并不简洁，就坚持使用 Lambda

~~~java
List<String> list = new ArrayList<>();
list.add("1");
list.add("2");
list.forEach(System.out::println);
    
list.forEach(str -> System.out.println(str));
~~~

##### 3、坚持使用标准的函数接口

1. 不要用带包装类型的基础函数接 口来代替基本函数接口
2. 必须始终用 ＠Functionallnterface 注解对自己编写的函数接口进行标注

##### 4、谨慎使用 Stream

1. 避免利用 Stream 来处理 char 值

2. 滥用 Stream 会使程序代码难以读懂和维护
3. 重构现有代码来使用 Stream ，并且只在必要的时候才在新代码中使用

有些任务最好用 Stream 完成，有些则要用迭代 而有许多任务则最好是结合使 用这两种方法来一起完成 具体选择用哪一种方法，并没有硬性 速成的规则，但是可以参 考一些有意义的启发 在很多时候，会很清楚应该使用哪一种方法；有些时候，则不太明显 如果实在不确定用 Stream 用迭代比较好，那么就两种都试试，看看哪一种更好用吧

##### 5、Stream 要优先用 Collection 作为返回类型

有些用户可能想要当作 Stream 处理，而其他用户可能想要使用迭代，要尽量两边兼顾，如果可以返回集合，就返回集合，如果集合中已经有元素，或者序列中的元素数量很少，足以创建一个新的集合，那么就返回一 个标准的集合，如 ArrayList 否则，就要考虑实现一个定制的集合，如幂集（ power set)  范例中所示，如果无法返回集合，就返回 Stream 或者 lterable，感觉哪一种更自然即可，如果在未来的 Java 发行版本中， Stream 接口声明被修改成扩展了 Iterable 接口，就可以放心地返回 Stream 了，因为它们允许进行 Stream 处理和迭代

##### 6、谨慎使用 Stream 并行

尽量不要并行 Stream pipeline ，除非有足够的理由相信它能保证计算的正确性，并且能加快程序的运行速度，如果对 Stream 进行不恰当的并行操作，可能导致程序运行失败，或者造成性能灾难 如果确信并行是可行的，并发运行时一定要确保代码正确，并 在真实环境下认真地进行性能测量如果代码正确，这些实验也证明它有助于提升性能，只有这时候，才可以在编写代码时并行 Stream

#### 七、方法

##### 1、检查参数的有效性

每当编写方法或者构造器的时候，应该考虑它的参数有哪些限制，应该把这些限制写到文档中，并且在这个方法体的开头处，通过显式的检查来实施这些限制。养成这样的习惯是非常重要的，只要有效性检查有一次失败，你为必要的有效性检查所付出的努力便都可以连本带利地得到偿还了

##### 2、必要时进行保护性拷贝

1. 对于参数类型可以被不可信任方子类化的参数，请不要使用 clone 方法进行保护性拷贝
2. 保护性拷贝是在检查参数的有效性之前进行的，并且有效性检查是针对拷贝之后的对象， 而不是针对原始的对象
3. 对于构造器的每个可变参数进行保护性拷贝是必要的

~~~java
private Date start;

private Date end;

public method(Date start, Date end){
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
}
~~~

如果一个类包含有从客户端得到或者返回到客户端的可变组件，这个类就必须保护性地拷贝这些组件，如果烤贝的成本受到限制，并且类信任它的客户端不会不恰当地修改组件，就可以在文档中指明客户端的职责是不得修改受到影响的组件，以此来代替保护性拷贝

##### 3、谨慎设计方法签名

##### 4、慎用重载

能够重载方法并不意味着就应该重载方法，一般情况下，对于多个具有相同参数数目的方法来说，应该尽量避免重载方法。在某些情况下，特别是涉及构造器的时候，要遵循这条建议也许是不可能的 在这种情况下，至少应该避免这样的情形，同一组参数只需经过类型转换就可以被传递给不同的重载方法 如果不能避免这种情形，例如，因为正在改造一个现有的类以实现新的接口，就应该保证：当传递同样的参数时·，所有重载 方法的行为必须 一致，如果不能做到这一点，程序员就很难有效地使用被重载的方法或者构造器，同时也不能理解它为什么不能正常地工作

##### 5、慎用可变参数

每次调用可变参数方法都会导致一次数组分配和初始化

在定义参数数目不定的方法时，可变参数方法是一种很方便的方式，在使用可变参数之前， 要先包含所有必要的参数，并且要关注使用可变参数所带来的性能影响

##### 6、返回长度的数组或者集合，而不是null

永远不要返回 null ，而不返回一个零长度的数组和集合。如何返回 null ，那样会使API更难用，容易出错

##### 7、为所有导出的 API 元素编写文档注释

#### 八、通用编程

##### 1、将局部变的作用域最小化

1. 要使局部变量的作用域最小化 ，最有力的方法就是在第一次要使用它的地方进行声明
2. 几乎每一个局部变量的声明都应该包含一个初始化表达式
3. for 循环就优先于 while 循环

##### 2、for-each 循环优先于传统的 for 循环

与传统的 for 循环相比， for-each 循环在简洁性、灵活性以及出错预防性方面都占有绝对优势，并且没有性能惩罚的问题，因此，当可以选择的时候， for-each 循环应该优先于 for 循环

##### 3、了解和使用类库

##### 4、如果需要精确的答案，请避免使用 float 和 double

##### 5、基本类型优先于装箱基本类型

~~~java
Long sum = 0L;
for (int i = 0; i < 10; i++) {
    sum += i;
}
~~~

基本类型要优先于装箱基本类型。基本类型更加简单， 也更加快速。如果必须使用装箱基本类型，要特别小心！ 自动装箱减少了使用装箱基本类型类性，但是并没有减少它的风险，当程序用==操作符比较两个装箱基本类型时，它做了个同一性比较，这几乎肯定不是你所希望的，当程序进行涉及装箱和拆箱基本类型的混合类型计算时，它会进行拆箱， 当程序进行拆箱时，会抛出 NullPointerException异常，最后，当程序装箱了基本类型值时，会导致较高的资源消耗和不必要的对象创建。

##### 6、如果其他类型更适合，则尽量避免使用字符串

1. 字符串不适合代替其他的值类型

2. 字符串不适合代替枚举类型

3. 字符串不适合代替聚合类型

##### 7、了解字符串连接的性能

不要使用字符串连接操作符来合并多个字符串，除非性能无关紧要，则应该使用 StringBuilder 的 append 方法 另一种做法是使用字符数组，或者每次只处理一个字符串，而不是将它们组合起来

~~~java
String s = "one";
for (int i = 0; i < 10; i++) {
    s = s + i;
}

StringBuilder stringBuilder = new StringBuilder("one");
for (int i = 0; i < 10; i++) {
    stringBuilder.append(i);
}
~~~

##### 8、通过接口引用对象

~~~java
Set<Son> sonSet = new LinkedHashSet<>(); 
~~~

给定的对象是否具有适当的接口应该是很显然的，如果是，用接口引用对象就会使程序更加灵活，如果没有适合的接口，就用类层次结构中提供了必要功能的最小的具体类来引用对象吧

##### 9、接口优先于反射机制

##### 10、谨慎地使用本地方法

使用本地方法有一些严重的缺陷，因为本地语言不是安全的（详见第 50 条），所以使用本地方法的应用程序也不再能免受内存毁坏错误的影响，因为本地语言是与平台相关的，使用本地方法的应用程序也不再是可自由移植的，使用本地方法的应用程序更难调试，如果不小心，本地方法还可能降低性能，因为回收垃圾器不是自动的，甚至无法追踪本机内存使用情况（详见第 条），而且在进入和退出本地代码时，还需要相关的开 最后一点，需要“胶合代码”的本地方法编写起来单调乏味，并且难以阅读

##### 11、谨慎地进行优化

1. 不要为了性能而牺牲合理的结构，要努力编写好的程序而不是快的程序
2. 要努力避免那些限制性能的设计决策
3. 要考虑 API 设计决策的性能后果
4. 在每次试图做优化之前和之后，要对性能进行测量

#### 九、异常

##### 1、只针对异常的情况才使用异常

1. 异常应该只用于异常的情况下；它们永远不应该用于正常的控制流

2. 设计良好的 API 不应该强迫它的客户端为了正常的控制流而使用异常

异常是为了在异常情况下使用而设计的 不要将它们用于普通的控制流，也不要编写迫使它们这么做的 API

##### 2、对可恢复的情况使用受检异常，又指启程错误使用运行时异常

在谨慎使用的前提之下，受检异常可以提升程序的可读性；如果过度使用， 将会使 API 使用起来非常痛苦，如果调用者无法恢复失败，就应该抛出未受检异常，如果可以恢复，并且想要迫使调用者处理异常的条件，首选应该返回一个 optional 当且仅当 万一失败时，这些无法提供足够的信息，才应该抛出受检异常

##### 3、优先使用标准的异常

精确使用异常

##### 4、抛出与抽象对应的异常

如果不能阻止或者处理来自更低层的异常，一般的做法是使用异常转译， 只有在低层方法的规范碰巧可以保证“它所抛出的所有异常对于更高层也是合适的”情况下，才可以将异常从低层传播到高层 异常链对高层和低层异常都提供了最佳的功能：它允 许抛出适当的高层异常，同时又能捕获低层的原因进行失败分析

##### 5、不要忽略异常

~~~java
try {
    ...
}catch (SomeException e){
    //如果选择忽略异常，catch 块中应该包含一条注释，说明为什么可以这么做
}
~~~

不管异常代表了可预见的异常条件，还是编程错误，用空的 catch 块忽略它，都将导致程序在遇到错误的情况下悄然地执行下去然后，有可能在将来的某个点上，当程序不能再容忍与错误源明显相关的问题时，它就会失败正确地处理异常能够彻底避免失败 只要将异常传播给外界，至少会导致程序 迅速失败，从而保留了有助于调试该失败条件的信息

##### 6、在细节消息中包含失败-捕获信息

1.  异常的细节信息应该包含“对该异常有贡献”的所有参数和域的值
2.  不要在细节消息中包含密码、密钥以及类似的信息！

#### 十、并发

##### 1、同步访问共享的可变数据

**手动停止线程，不要使用线程 Thread.stop() 方法**

~~~java
public class StopThread {

    private static boolean stopRequested;

    private static synchronized void setStopRequested(){
        stopRequested = true;
    }

    private static synchronized boolean stopRequested(){
        return stopRequested;
    }

    public static void main(String[] args) throws Exception{
        Thread thread = new Thread(() -> {
            int i = 0;
            while (!stopRequested()){
                System.out.println(i++);
            }
            System.out.println("stop i = "+i);
        });
        thread.start();

        TimeUnit.SECONDS.sleep(1);
        setStopRequested();
    }
}

~~~

volatile 保证线程可见性

~~~java
public class StopThread {

    private static volatile boolean stopRequested;

    public static void main(String[] args) throws Exception{
        Thread thread = new Thread(() -> {
            int i = 0;
            while (!stopRequested){
                System.out.println(i++);
            }
            System.out.println("stop i = "+i);
        });
        thread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}

~~~

当多个线程共享可变数据的时候，每个读或者写数据的线程都必须执行同步，如果没有同步，就无法保证一个线程所做的修改可以被另一个线程获知。未能同步共享可变数据会造成程序的活性失败（ liveness failure ）和安全性失败（ safety failure ）。这样的失败是最难调试的，它们可能是间歇性的，且与时间相关，程序的行为在不同的虚拟机上可能根本不同，如果只需要线程之间的交互通信，而不需要互斥， volatile 修饰符就是一种可以接受的同步形式，但要正确地使用它可能需要一些技巧

##### 2、避免过度同步

为了避免死锁和数据破坏，千万不要从同步区域内部调用外来方法，更通俗地讲，要尽量将同步区域内部的工作量限制到最少，当你在设计一个可变类的时候，要考虑一下它们是否应该自己完成同步操作，在如今这个多核的时代，这比永远不要过度同步来得更重要，只有当你有足够的理由一定要在内部同步类的时候，才应该这么做，同时还应该将这个决定清楚地写到文档中

##### 3、executor、task 和 stream 优先于线程

##### 4、并发工具优先于 wait 和 notify

并发集合（ ConcurrentHashMap ）、同步器（ CountDownLatch、semaphore、CyclicBarrier、Phaser ）

1. 对于间歇式的定时，始终应该优先使用 System. nanoTime ，而不是使用 System.currentTimeMillis
2. 应该优先使用 ConcurrentHashMap ，而不是使用 Collections. synchronizedMap
3. 始终应该使用 wait 循环模式来调用 wait 方法；永远不要在循环之外调用 wait 方法
4. 没有理由在新代码中使用 wait 方法和 notify 方法，即使有，也是极少的

##### 5、线程安全性的文档化

1. lock 域应该始终声明为 final

##### 6、慎用延迟初始化

大多数的域应该正常地进行初始化，而不是延迟初始化，如果为了达到性能目标，或者为了破坏有害的初始化循环，而必须延迟初始化一个域，就可以使用相应的延迟初始化方法，对于实例域，就使用双重检查模式（ double-check idiom ）；对于静态域，则使用 lazy initialization holder class idiom，对于可以接受重复初始化的实例域，也可以考虑使用单重检查模式（ single-check idiom ）。

##### 7、不要依赖于线程调度器

要编写出健壮、响应良好、可移植的多线程应用程序，最好的办法是确保可运行线程的平均数量不明显多于处理器的数量。 这使得线程调度器没有更多的选择，它只需要运行这 些可运行的线程，直到它们不再可运行为止，即使在根本不同的线程调度算法下，这些程序的行为也不会有很大的变化，注意可运行线程的数量并不等于线程的总数量，前者可能更多，在等待的线程并不是可运行的

不要让应用程序的正确性依赖于线程调度器，否则，得到的应用程序将既不健壮，也不具有可移植性，同样，不要依赖 Thread.yield 或者线程优先级，这些机制都只是影响到调度器，线程优先级可以用来提高一个已经能够正常工作的程序的服务质量， 但永远不应该用来“修正” 个原本并不能工作的程序

#### 十一、序列化

##### 1、其他方法优先于 Java 序列化

序列化是很危险的，应该予以避免，如果是重新设计一个系统，一定要用跨平台的结构化数据表示法代替，如 JSON 或者 protobuf 不要反序列化不被信任的数据，如果必须这么做，就要使用对象的反序列化过滤，但要注意的是，它并不能确保阻止所有的攻击，不要编写可序列化的类，如果必须这么做 一定要倍加小心地进行试验

##### 2、谨慎地实现 Serializable 接口

1. 一旦一个类被发布，就大大降低了 “改变这个类的实现”的灵活性
2. 它增加了出现 Bug 和安全漏洞的可能性
3. 随着类发行新的版本，相关的测试负担也会增加

##### 3、考虑使用自定义的序列化形式

