title: scals学习笔记-类和对象1
date: 2017-03-07 20:11:13
tags:
- 乡村销客
- 大数据
categories: scals
---
 
# 1.单例对象

<!-- more -->
在某些场景下,我们可能不需要创建对象,而是想直接调用方法,但是scala语言并不支持静态成员变量,scals通过单例对象解决该问题,单例对象的创建方式如下:

```bash
object Student{
	private var studengNo:Int=0;
	def uniqueStudentNo()={
		studentNo+=1
		studentNo
	}
	def main(args:Arrary[String]:Unit)={
		println(Student.uniqueStudentNo())
	}
}
```
object Student 编码后将生成两个字节码文件

Student$.class
Student.class

利用javap命令查看字节码文件内容有:

```bash
javap -private Student$
警告: 二进制文件Student$包含cn.scala.xtwy.Student$
Compiled from "Student.scala"
public final class cn.scala.xtwy.Student$ {
  public static final cn.scala.xtwy.Student$ MODULE$;
  private int studentNo;
  public static {};
  private int studentNo();
  private void studentNo_$eq(int);
  public int uniqueStudentNo();
  private cn.scala.xtwy.Student$();
}

javap -private Student
警告: 二进制文件Student包含cn.scala.xtwy.Student
Compiled from "Student.scala"
public final class cn.scala.xtwy.Student {
  public static void main(java.lang.String[]);
  public static int uniqueStudentNo();
}

```

他们都是final类型的,而且student不难看出,object Student最终生成了两个类,分别是Student和Student$ 的构造方法是私有的,通过静态成员域 public static final cn.scala.xtwy.Student$ MODULE$;对student$进行引用,这其实是java语言中单例实现方式

单例对象的使用方式同java语言类引用静态成员是一样的.

# 半生对象于伴生类

在前面的单例对象的基础之上,我们在objec Studeng 所在的文件内定义了一个class Student,此时object Student被称为class Student的伴生对象,而 class student 被称作object Student的伴生类 :

```bash
class Student(var name:String,age:Int)

object Student {
  private var studentNo:Int=0;
  def uniqueStudentNo()={
    studentNo+=1
    studentNo
  }
  def main(args: Array[String]): Unit = {
    println(Student.uniqueStudentNo())
  }
}

//生成的字节码文件如下：
D:\ScalaWorkspace\ScalaChapter06_2\bin\cn\scala\xtwy>javap -private Student
警告: 二进制文件Student包含cn.scala.xtwy.Student
Compiled from "Student.scala"
public class cn.scala.xtwy.Student {
  private java.lang.String name;
  private int age;
  public static void main(java.lang.String[]);
  public static int uniqueStudentNo();
  public java.lang.String name();
  public void name_$eq(java.lang.String);
  public int age();
  public void age_$eq(int);
  public cn.scala.xtwy.Student(java.lang.String, int);
}

D:\ScalaWorkspace\ScalaChapter06_2\bin\cn\scala\xtwy>javap -private Student$
警告: 二进制文件Student$包含cn.scala.xtwy.Student$
Compiled from "Student.scala"
public final class cn.scala.xtwy.Student$ {
  public static final cn.scala.xtwy.Student$ MODULE$;
  private int studentNo;
  public static {};
  private int studentNo();
  private void studentNo_$eq(int);
  public int uniqueStudentNo();
  public void main(java.lang.String[]);
  private cn.scala.xtwy.Student$();
}

```
从上面的代码中不难看出, 其实伴生对象与伴生类本质上是不同的两个类,只不过伴生类与伴生对象之间可以相互访问到对方的成员包括私有的成员变量或者方法,例如:

``` bash
class Student(var name:String,var age:Int){
	private var sex:Int=0
	//直接访问伴生对象的私有成员
	def printCompanionObject()=println(Student.studentNo)
}

object Student{
  private var studentNo:Int=0;
  def uniqueStudentNo()={
    studentNo+=1
    studentNo
  }
  def main(args: Array[String]): Unit = {
    println(Student.uniqueStudentNo())
    val s=new Student("john",29)
    //直接访问伴生类Student中的私有成员
    println(s.sex)
  }
}
```
# apply方法

在前几节中我们提到，通过利用apply方法可以直接利用类名创建对象，例如前面在讲集合的时候，可以通过val intList=List(1,2,3)这种方式创建初始化一个列表对象，其实它相当于调用val intList=List.apply(1,2,3)，只不过val intList=List(1,2,3)这种创建方式更简洁一点，但我们必须明确的是这种创建方式仍然避免不了new，它后面的实现机制仍然是new的方式，只不过我们自己在使用的时候可以省去new的操作。下面就让我们来自己实现apply方法，代码如下：

```bash
//定义Student类，该类称为伴生类，因为在同一个源文件里面，我们还定义了object Student
class Student(var name:String,var age:Int){
  private var sex:Int=0
  //直接访问伴生对象的私有成员
  def printCompanionObject()=println(Student.studentNo)

}

//伴生对象
object Student {
  private var studentNo:Int=0;
  def uniqueStudentNo()={
    studentNo+=1
    studentNo
  }
  //定义自己的apply方法
  def apply(name:String,age:Int)=new Student(name,age)
  def main(args: Array[String]): Unit = {
    println(Student.uniqueStudentNo())
    val s=new Student("john",29)
    //直接访问伴生类Student中的私有成员
    println(s.sex)
    //直接利用类名进行对象的创建，这种方式实际上是调用前面的apply方法进行实现，这种方式的好处是避免了自己手动new去创建对象
    val s1=Student("john",29)
    println(s1.name)
    println(s1.age)
  }
}
```

# 应用程序对象

利用IDE开发scala应用程序时,在运行程序时必须制定main方法作为程序的入口,例如:

```bash
object Student {
  //必须定义mian方法作为程序的入口才能执行
  def main(args: Array[String]): Unit = {
    val s1=Student("john",29)
    println(s1.name)
    println(s1.age)
  }
}

```

除了这种方式之外,scala还提供了一种机制,通过扩展App,在scala IDE for Eclipse 里通过new-> scala app方式创建的

也可以在代码直接指定:
```bash
//扩展App后,程序可以直接运行,而不需要自己定义main方法,代码更简洁
object AppDemo extends App{	
	println("App Demo")
}

```

# 抽象类

抽象类是一种不能被实例化的类,抽象类中包括了若干不能完整定义的方法,这些方法由子类扩展定义自己的实现.

```bash
//scala中的抽象类定义
abstract class Animal {
  def eat:Unit
}

//对应字节码文件
javap -private Animal.class

Compiled from "human.scala"
public abstract class cn.scala.xtwy.Animal {
  public abstract void eat();
  public cn.scala.xtwy.Animal();
}

```

除抽象方法外,抽象类中还有抽象字段:
```bash
abstract class Animal{
	//抽象字段
	//前面我们提到,一般类中定义字段的话必须初始化,而抽象类中则没有这样的要求
	var height:Int
	def eat:Unit
}

//Person继承Animal，对eat方法进行了实现
//通过主构造器对height参数进行了初始化
class Person(var height:Int) extends Animal{
  //对父类中的方法进行实现，注意这里面可以不加override关键字
  def eat()={
    println("eat by mouth")
  }

}

//通过扩展App创建程序的入口
object Person extends App{
  new Person(10).eat()
}
```

```bash
//上面这几个类会产生以下四个字节码文件

2015/07/22  23:28    <DIR>          .
2015/07/22  23:28    <DIR>          ..
2015/07/22  23:28               675 Animal.class
2015/07/22  23:28             2,143 Person$.class
2015/07/22  23:28               699 Person$delayedInit$body.class
2015/07/22  23:28             1,741 Person.class


//字节码内容如下：
//Animal类对应的字节码，可以看到，字节码中包括了抽象字段height的getter和setter方法，只不过它们都是抽象的
javap -private Animal.class

Compiled from "Person.scala"
public abstract class cn.scala.xtwy.Animal {
  public abstract int height();
  public abstract void height_$eq(int);
  public abstract void eat();
  public cn.scala.xtwy.Animal();
}

//Person类对应的字节码文件
javap -private Person.class

Compiled from "Person.scala"
public class cn.scala.xtwy.Person extends cn.scala.xtwy.Animal {
  private int height;
  public static void main(java.lang.String[]);
  public static void delayedInit(scala.Function0<scala.runtime.BoxedUnit>);
  public static java.lang.String[] args();
  public static void scala$App$_setter_$executionStart_$eq(long);
  public static long executionStart();
  public int height();
  public void height_$eq(int);
  public void eat();
  public cn.scala.xtwy.Person(int);
}

//伴生对象Person对应的字节码文件内容
javap -private Person$.clas
s
Compiled from "Person.scala"
public final class cn.scala.xtwy.Person$ implements scala.App {
  public static final cn.scala.xtwy.Person$ MODULE$;
  private final long executionStart;
  private java.lang.String[] scala$App$$_args;
  private final scala.collection.mutable.ListBuffer<scala.Function0<scala.runtim
e.BoxedUnit>> scala$App$$initCode;
  public static {};
  public long executionStart();
  public java.lang.String[] scala$App$$_args();
  public void scala$App$$_args_$eq(java.lang.String[]);
  public scala.collection.mutable.ListBuffer<scala.Function0<scala.runtime.Boxed
Unit>> scala$App$$initCode();
  public void scala$App$_setter_$executionStart_$eq(long);
  public void scala$App$_setter_$scala$App$$initCode_$eq(scala.collection.mutabl
e.ListBuffer);
  public java.lang.String[] args();
  public void delayedInit(scala.Function0<scala.runtime.BoxedUnit>);
  public void main(java.lang.String[]);
  private cn.scala.xtwy.Person$();
}

```


