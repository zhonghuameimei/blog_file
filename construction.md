#### 一、外观模式
**外观模式（Facade Pattern）**隐藏系统的复杂性，并向客户端提供了一个客户端可以访问系统的接口。这种类型的设计模式属于结构型模式，它向现有的系统添加一个接口，来隐藏系统的复杂性。
这种模式涉及到一个单一的类，该类提供了客户端请求的简化方法和对现有系统类方法的委托调用

<!--more-->

##### 意图
为子系统中的一组接口提供一个一致的界面，外观模式定义了一个高层接口，这个接口使得这一子系统更加容易使用
##### 使用场景
1. 为复杂的模块或子系统提供外界访问的模块
2. 子系统相对独立
3. 预防低水平人员带来的风险
##### 实现
我们将创建一个  _Shape_  接口和实现了  _Shape_  接口的实体类。下一步是定义一个外观类  _ShapeMaker_。
_ShapeMaker_  类使用实体类来代表用户对这些类的调用。_FacadePatternDemo_  类使用  _ShapeMaker_  类来显示结果。
![外观模式的 UML 图](https://www.runoob.com/wp-content/uploads/2014/08/20201015-facade.svg)
**创建接口**
```java
package test06;

public interface Shape {

	public void draw();
}
```
**创建实现接口的实体类**
```java
package test06;

public class Square implements Shape{

	@Override
	public void draw() {
		System.out.println("Draw Square");
	}
}
```
```java
package test06;

public class Rectangle implements Shape{

	@Override
	public void draw() {
		System.out.println("Draw Rectangle");
	}
}
```
```java
package test06;

public class Circle implements Shape{

	@Override
	public void draw() {
		System.out.println("Draw Circle");
	}
}
```
**创建一个外观类**
```java
package test06;

public class ShapeMaker {
	
	private Circle circle;
	
	private Square square;
	
	private Rectangle rectangle;

	public ShapeMaker(){
		this.circle = new Circle();
		this.square = new Square();
		this.rectangle = new Rectangle();
	}
	
	public void drawCircle(){
		circle.draw();
	}
	
	public void drawSquare(){
		square.draw();	
	}

	public void drawRectangle(){
		rectangle.draw();
	}
}
```
**使用该外观类画出各种类型的形状（测试类）**
```java 
package test06;

public class Demo {

	public static void main(String[] args) {
		ShapeMaker shapeMaker = new ShapeMaker();
		shapeMaker.drawCircle();
		shapeMaker.drawRectangle();
		shapeMaker.drawSquare();
	}
}
```
#### 二、桥接模式
**桥接模式**是用于把抽象化与实现化解耦，使得二者可以独立变化。这种类型的设计模式属于结构型模式，它通过提供抽象化和实现化之间的桥接结构，来实现二者的解耦。
这种模式涉及到一个作为桥接的接口，使得实体类的功能独立于接口实现类。这两种类型的类可被结构化改变而互不影响。
##### 意图
将抽象部分与实现部分分离，使它们都可以独立的变化
##### 使用场景
1. 如果一个系统需要在构件的抽象化角色和具体化角色之间增加更多的灵活性，避免在两个层次之间建立静态的继承联系，通过桥接模式可以使它们在抽象层建立一个关联关系
2. 对于那些不希望使用继承或因为多层次继承导致系统类的个数急剧增加的系统，桥接模式尤为适用
3. 一个类存在两个独立变化的维度，且这两个维度都需要进行扩展
##### 实现
我们有一个作为桥接实现的 _DrawAPI_ 接口和实现了 _DrawAPI_ 接口的实体类 _RedCircle_、_GreenCircle_。_Shape_ 是一个抽象类，将使用 _DrawAPI_ 的对象。_BridgePatternDemo_ 类使用 _Shape_ 类来画出不同颜色的圆。
![桥接模式的 UML 图](https://www.runoob.com/wp-content/uploads/2014/08/20201015-bridge.svg)
**创建桥接实现接口**
```java
package test04;

public interface DrawAPI {

	public void drawCircle(int radius, int x, int y);
}
```
**创建实现桥接接口具体实现类**
```java
package test04;

public class RedCircle implements DrawAPI{

	@Override
	public void drawCircle(int radius, int x, int y) {
		System.out.println("Drawing Circle[ color: red, radius: "
		         + radius +", x: " +x+", "+ y +"]");
	}
}
```
```java
package test04;

public class GreenCircle implements DrawAPI{

	@Override
	public void drawCircle(int radius, int x, int y) {
		System.out.println("Drawing Circle[ color: red, radius: "
		         + radius +", x: " +x+", "+ y +"]");
	}
}
```
**使用 DrawAPI 创建抽象类**
```java
package test04;

public abstract class Shape {

	DrawAPI drawAPI;
	
	protected Shape(DrawAPI drawAPI){
		this.drawAPI = drawAPI;
	}
	
	public abstract void draw();
}
```
**创建实现了 Shape 抽象类的实体类**
```java
package test04;

public class Circle extends Shape{
	
	private int x, y, radius;

	protected Circle(int x, int y, int radius, DrawAPI drawAPI) {
		super(drawAPI);
		this.x = x;  
	    this.y = y;  
	    this.radius = radius;
	}

	@Override
	public void draw() {
		drawAPI.drawCircle(radius, x, y);
	}
}
```
**测试类**
```java
package test04;

public class Demo {

	public static void main(String[] args) {
		Shape redCircle = new Circle(100,100, 10, new RedCircle());
		Shape greenCircle = new Circle(100,100, 10, new GreenCircle());
		 
		redCircle.draw();
		greenCircle.draw();
	}
}
```
#### 三、适配器模式
**适配器模式（Adapter Pattern）**是作为两个不兼容的接口之间的桥梁。这种类型的设计模式属于结构型模式，它结合了两个独立接口的功能。
这种模式涉及到一个单一的类，该类负责加入独立的或不兼容的接口功能。举个真实的例子，读卡器是作为内存卡和笔记本之间的适配器。您将内存卡插入读卡器，再将读卡器插入笔记本，这样就可以通过笔记本来读取内存卡。
我们通过下面的实例来演示适配器模式的使用。其中，音频播放器设备只能播放 mp3 文件，通过使用一个更高级的音频播放器来播放 vlc 和 mp4 文件。
##### 意图
将一个类的接口转换成客户希望的另外一个接口。适配器模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。
##### 何时使用

1. 系统需要使用现有的类，而此类的接口不符合系统的需要
2. 想要建立一个可以重复使用的类，用于与一些彼此之间没有太大关联的一些类，包括一些可能在将来引进的类一起工作，这些源类不一定有一致的接口
3. 通过接口转换，将一个类插入另一个类系中（比如老虎和飞禽，现在多了一个飞虎，在不增加实体的需求下，增加一个适配器，在里面包容一个虎对象，实现飞的接口。）
##### 关键步骤
适配器继承或依赖已有的对象，实现想要的目标接口。
##### 实现
我们有一个  _MediaPlayer_  接口和一个实现了  _MediaPlayer_  接口的实体类  _AudioPlayer_。默认情况下，_AudioPlayer_  可以播放 mp3 格式的音频文件。
我们还有另一个接口  _AdvancedMediaPlayer_  和实现了  _AdvancedMediaPlayer_  接口的实体类。该类可以播放 vlc 和 mp4 格式的文件。
我们想要让  _AudioPlayer_  播放其他格式的音频文件。为了实现这个功能，我们需要创建一个实现了  _MediaPlayer_  接口的适配器类  _MediaAdapter_，并使用  _AdvancedMediaPlayer_  对象来播放所需的格式。
_AudioPlayer_  使用适配器类  _MediaAdapter_  传递所需的音频类型，不需要知道能播放所需格式音频的实际类。_AdapterPatternDemo_  类使用  _AudioPlayer_  类来播放各种格式。
![适配器模式的 UML 图](https://www.runoob.com/wp-content/uploads/2014/08/20201204-adapter.png)
**创建媒体播放器接口**
```java
package test03;

public interface MediaPlayer {
	
	public void play(String audioType, String fileName);
	
}
```
**创建实现播放功能的播放器类**
```java
package test03;

public class AudioPlayer implements MediaPlayer{
	
	MediaAdapter mediaAdapter;

	public void play(String audioType, String fileName) {
		if (audioType.equalsIgnoreCase("mp3")) {
			System.out.println("player " + fileName);
		}else if (audioType.equalsIgnoreCase("vlc") || audioType.equalsIgnoreCase("mp4")) {
			mediaAdapter = new MediaAdapter(audioType);
			mediaAdapter.play(audioType, fileName);
		}else {
			System.out.println("Invalid media. "+ audioType + " format not supported");
		}
	}
}
```
**创建其他媒体播放器接口**
```java
package test03;

public interface OtherPlayer {

	public void Vlcplayer(String fileName);
	
	public void Mp4player(String fileName);
}
```
**创建实现其他媒体播放器接口的实体类**
```java
package test03;

public class VlcPlayer implements OtherPlayer{

	@Override
	public void Vlcplayer(String fileName) {
		System.out.println("Vlc "+fileName);
	}

	@Override
	public void Mp4player(String fileName) {
		// TODO Auto-generated method stub
	}
}
```
```java
package test03;

public class Mp4Player implements OtherPlayer{

	@Override
	public void Vlcplayer(String fileName) {
		// TODO Auto-generated method stub
	}

	@Override
	public void Mp4player(String fileName) {
		System.out.println("Mp4 "+fileName);
	}
}
```
**创建其他播放器与媒体播放器之间的适配器**
```java
package test03;

public class MediaAdapter implements MediaPlayer{
	
	OtherPlayer otherPlayer = null;
	
	public MediaAdapter(String audioType){
		if (audioType.equalsIgnoreCase("vlc")) {
			otherPlayer = new VlcPlayer();
		}else if (audioType.equalsIgnoreCase("mp4")) {
			otherPlayer = new Mp4Player();
		}
	}

	@Override
	public void play(String audioType, String fileName) {
		if(audioType.equalsIgnoreCase("vlc")){
			otherPlayer.Vlcplayer(fileName);
		}else if(audioType.equalsIgnoreCase("mp4")){
			otherPlayer.Mp4player(fileName);
        }		
	}
}
```
**测试类**
```java
package test03;

public class Demo {

	public static void main(String[] args) {
		AudioPlayer audioPlayer = new AudioPlayer();
		audioPlayer.play("mp3", "mp3");
		audioPlayer.play("vlc", "vlc");
		audioPlayer.play("mp4", "mp4");
		audioPlayer.play("kugou", "kugou");
	}
}
```
#### 四、装饰者模式
**装饰器模式（_Decorator Pattern_）**允许向一个现有的对象添加新的功能，同时又不改变其结构。这种类型的设计模式属于结构型模式，它是作为现有的类的一个包装。
这种模式创建了一个装饰类，用来包装原有的类，并在保持类方法签名完整性的前提下，提供了额外的功能。
##### 意图
动态地给一个对象添加一些额外的职责。就增加功能来说，装饰器模式相比生成子类更为灵活。
##### 关键代码
1. _Component_ 类充当抽象角色，不应该具体实现
2. 修饰类引用和继承 _Component_ 类，具体扩展类重写父类方法
##### 使用场景
1. 扩展一个类的功能
2. 动态增加功能，动态撤销
##### 实现
我们将创建一个  _Shape_  接口和实现了  _Shape_  接口的实体类。然后我们创建一个实现了  _Shape_  接口的抽象装饰类  _ShapeDecorator_，并把  _Shape_  对象作为它的实例变量。
_RedShapeDecorator_  是实现了  _ShapeDecorator_  的实体类。
_DecoratorPatternDemo_  类使用  _RedShapeDecorator_  来装饰  _Shape_  对象。
![装饰器模式的 UML 图](https://www.runoob.com/wp-content/uploads/2014/08/20210204-decorator-1-decorator.svg)
**创建一个接口**
```java
package test05;

public interface Shape {

	public void draw();
}
```
**创建实现接口的实体类**
```java 
package test05;

public class Rrctangle implements Shape{

	@Override
	public void draw() {
		System.out.println("Draw Rrctangle");
	}
}
```
```java 
package test05;

public class Circle implements Shape{

	@Override
	public void draw() {
		System.out.println("Draw Circle");
	}
}
```
**创建实现了  _Shape_  接口的抽象装饰类**
```java
package test05;

public abstract class ShapeDecorator implements Shape{

	protected Shape decoratorShape;
	
	public ShapeDecorator(Shape decoratorShape){
		this.decoratorShape = decoratorShape;
	}

	@Override
	public void draw() {
		decoratorShape.draw();
	}
}
```
**创建扩展了  _ShapeDecorator_  类的实体装饰类**
```java
package test05;

public class RedShapeDecorator extends ShapeDecorator{

	public RedShapeDecorator(Shape decoratorShape) {
		super(decoratorShape);
	}
	
	@Override
	public void draw() {
		decoratorShape.draw();
		setRedBorder(decoratorShape);
	}
	
	private void setRedBorder(Shape decoratedShape){
	    System.out.println("Border Color: Red");
	}
}
```
**测试类**
```java 
package test05;

public class Demo {

	public static void main(String[] args) {
		Shape shape = new Circle();
		shape.draw();
		ShapeDecorator shapeDecorator = new RedShapeDecorator(shape);
		shapeDecorator.draw();
	}
}
```



