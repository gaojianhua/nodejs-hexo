title: scala学习笔记-继承和组合
date: 2017-04-05 20:11:13
tags:
- 乡村销客
- 大数据
categories: scala
---

# 1.类的继承

<!-- more -->
下面的代码演示了scala类的继承

```bash
//Person类
class Person(name:String,age:Int){

}

//Student继承Person类
class Student(name:String,age:Int,var studentNo:String) extends Person(name,age){

}

object demo{
  def main(args: Array[String]): Unit = {
     val student=new Student("john",18,"1024")
  }
}

```

等同于下面的java代码

```bash
//Person类
class Person{
    private String name;
    private int age;
    public Person(String name,int age){
       this.name=name;
       this.age=age;
    }
}

//Student继承Person类
class Student extends Person{
    private String studentNo;
    public Student(string name,int age,String studentNo){
        super(name,age);
        this.sutdentNo=studentNo;
    }
}
```

# 2. 构造函数执行顺序

下面的代码演示scala在继承的时候，构造函数的执行顺序

```bash
class Person(name:String,age:Int){
	println("Constructing Persion")
}
class Student(name:String ,age:Int,var studentNo:String) extends Persion(name,age){
	println("Contstructing Student")
}

object demo{
	def main(args:Arrary[String]):Unit = {
	//下面的语句执行时会打印下列内容
     //Constructing Person
     //Constructing Student
     //也就是说，构造Student这前，首先会调用Person的主构造方法
     val student=new Student("john",18,"1024")
}
}


```

等同于代码java

```bash
//Person类
class Person{
    private String name;
    private int age;
    public Person(String name,int age){

       this.name=name;
       this.age=age;
       System.out.println("Constructing Person");
    }
}

//Student继承Person类
class Student extends Person{
    private String studentNo;
    public Student(string name,int age,String studentNo){

        super(name,age);
        this.sutdentNo=studentNo;
        System.out.println("Constructing Student");
    }
}
```

# 3.方法重写

方法重写指的是当子类继承父类的时候,从父类继承过来的方法不能满足子类的需求,子类希望有自己的实现,这时需要对父类的方法进行重写,
方法重写是实现多态和动态绑定的关键
scala中的方法重写同java一样,也是利用override关键字标识重写父类的算法,

```bash
class Person(name:String,age:Int){
	def walk():Unit=println("walk like a normal persion")
}
class Student(name:String,age:Int,var)extends Persion(name,age){
	override def walk():Unit={
		super.walk()
		println("walk like a elegant swan")
	}
}

object demo{
	def main(args:Array[String]):Unit={
		val s = new Student("john",18,"1024")
		s.walk()
	}
}


//代码运行输出内容
walk like a normal person
walk like a elegant swan
```

不得不提的是,如果父类是抽象类,则override关键字可以不加,
假设抽象类为AbstractClass，子类为SubClass），在SubClass类中，AbstractClass对应的抽象方法如果没有实现的话，那SubClass也必须定义为抽象类，否则的话必须要有方法的实现，这样的话，加不加override关键字都是可以的。下面是一个实例代码：
```bash
//抽象的Person类
abstract class Person(name:String,age:Int){
  def walk():Unit
}

//Student继承抽象Person类
class Student(name:String,age:Int,var studentNo:String) extends Person(name,age){
  //重写抽象类中的walk方法，可以不加override关键字
  def walk():Unit={
    println("walk like a elegant swan")
  }
}

object demo{
  def main(args: Array[String]): Unit = {
     val s=new Student("john",18,"1024")
     s.walk()
  }
}

```

# 4.匿名类

当某个类在程序中只使用一次时,可以定义为匿名类,匿名类
的定义如下:
```bash
//抽象的Person类
abstract class Person(name:String,age:Int){ 
  def walk():Unit
}


object demo{
  def main(args: Array[String]): Unit = {
     //下面的代码定义了一个匿名类，并且进行了实例化
     //直接new Person("john",18)，后面跟的是类的内容
     //我们知道，Person是一个抽象类，它是不能被实例化的
     //这里能够直接new操作是因为我们扩展了Person类，只不
     //过这个类是匿名的，只能使用一次而已
     val s=new Person("john",18){
       override def walk()={
         println("Walk like a normal Person")
       }
     }
     s.walk()   
  }
}
```
# 5.多态与动态绑定

"多态"(polymorphic) 也叫"动态绑定"(dynamic Bingding),"迟绑定"(Late Bingding) 指在执行期间(非编译期间)判断引用对象的实际类型,根据其实际类型调用其相应的方法.即指子类的引用可以赋给父类,程序在运行时根据实际类型调用对应的方法
下面的代码演示了scala中的多态和动态绑定:

```bash
//抽象Person类
abstract class Persion (var name:String,var age:Int){
	def walk():Unit
	//talkTo方法,参数为Persion类型
	def talkTo(p:Person):Unit
}
class Student(name:String,age:Int)extends Person(name,age){
	private var studentNo:Int=0
	def walk()=println("walk like a elegant swan")
	//重写父类的talkTo方法
	def talkTo(p:Person)={
		println("talkTo() method in Student")
		println(this.name+"is talkTo "+p.name)
	}

}

class Teacher(name:String,age:Int) extends Person(name,age){
  private var teacherNo:Int=0

  def walk()=println("walk like a elegant swan")

   //重写父类的talkTo方法
  def talkTo(p:Person)={
    println("talkTo() method in Teacher")
    println(this.name+" is talking to "+p.name)
  }
}


object demo{
  def main(args: Array[String]): Unit = {

     //下面的两行代码演示了多态的使用
     //Person类的引用可以指向Person类的任何子类
     val p1:Person=new Teacher("albert",38)
     val p2:Person=new Student("john",38)

     //下面的两行代码演示了动态绑定
     //talkTo方法参数类型为Person类型
     //p1.talkTo(p2)传入的实际类型是Student
     //p2.talkTo(p1)传入的实际类型是Teacher
     //程序会根据实际类型调用对应的不同子类中的talkTo()方法
     p1.talkTo(p2)
     p2.talkTo(p1)
  }
}

```

# 6.继承与组合的使用

继承可以重用父类的代码,从而简化程序设计,继承是is-a的关系,apple is a kind of fruit (苹果是一种水果).
还有一种代码重用的方式是组合,组合是has-a的关系(one person has a head ) . 继承前面已经讲过了,这边只给出组合的使用代码:

```bash
class Head
class Body
class Hand

//persion类
abstract class　Person (var name:String,var age:Int){
	// 各类的实例作为该类对象的一部分,通过各类的实例方法实现代码重用
	val head:Head=null
	val body:Body=null
	val hand:Hand=null
}
```
继承与组合使用总结：

一 继承

　　继承是Is a 的关系，比如说Student继承Person,则说明Student is a Person。继承的优点是子类可以重写父类的方法来方便地实现对父类的扩展。 
　　继承的缺点有以下几点： 
　　1 父类的内部细节对子类是可见的。 
　　2 子类从父类继承的方法在编译时就确定下来了，所以无法在运行期间改变从父类继承的方法的行为。 
3 如果对父类的方法做了修改的话（比如增加了一个参数），则子类的方法必须做出相应的修改。所以说子类与父类是一种高耦合，违背了面向对象思想。
二 组合
	组合也就是设计类的时候把要组合的类的对象假如到该类中作为自己的成员变量. 组合的优点:
1.当前的对象只能通过所包含的那个对象去调用其方法,所以所包含的对象的内部细节对当前对象时不可见的.
2.当前对象与包含对象是低耦合关系,如果修改包含对象的类中的代码不需要修改当前对象类的代码.
3.当前对象可以在运行时动态绑定所包含的对象.可以通过set方法给包含对象赋值.
组合缺点:
1容易产生过多的对象.
2.为了能组合多个对象.必须对接口进行定义.
由此可见,组合比继承更具灵活性和稳定性,所以再设计的时候优先使用组合. 只有当下列条件满足时才考虑使用继承
1.子类是一种特殊的类型,而不是父类的一个角色
2.子类的实例不需要变成另一个对象
3.子类扩展,而不是覆盖或者使用父类的功能失效.








