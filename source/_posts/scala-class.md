title: scals学习笔记-类和对象1
date: 2017-03-07 20:11:13
tags:
- 乡村销客
- 大数据
categories: scals
---
 
# 1.类定义.创建对象

<!-- more -->

```bash
//采用关键字class定义
class Persion{
	//类成员必须初始化,否则报错
	//这里定义一个公有成员
	var name:String=null
}
```

Person 类在编译后会生成Person.class 文件

利用java -private Person 命令查看字节码文件内容

```bash
javap -private Person

警告: 二进制文件Person包含cn.scala.xtwy.Person
Compiled from "Person.scala"
public class cn.scala.xtwy.Person {
  private java.lang.String name;
  public java.lang.String name();
  public void name_$eq(java.lang.String);
  public cn.scala.xtwy.Person();
}

```

从字节码文件内容可以看到：虽然我们只在Person类中定义了一个类成员（域）name，类型为String，
但Scala会默认帮我们生成name()与name_=（）及构造函数Person()。其中name()对应java中的getter方法，name_=()对应java中的setter方法（由于JVM中不允许出现=，所以用$eq代替。
值得注意的是定义的是公有成员，但生成的字节码中却是以私有的方式实现的，生成的getter、setter方法是公有的 
因此，可以直接new操作创建Person对象

```bash
//默认已经有构造函数,所以可以直接new

val p = new Persion()
//直接调用getter和setter方法
//setter方法
p.name_="nike"
//getter方法
p.name

//直接修改,但是调用的是 p.name_ = ("nike")
p.name="nike1"

```

你也可以自定义setter 和getter方法

```bash
class Persion{
	//定义私有变量
	private var privateName:String=null
	//getter方法
	def name=privateName
	//setter方法
	def name_=(name:String){
	  this.privateName=name
	}

}

```

```bash
javap -private Person
警告: 二进制文件Person包含cn.scala.xtwy.Person
Compiled from "Person.scala"
public class cn.scala.xtwy.Person {
  private java.lang.String privateName;
  private java.lang.String privateName();
  private void privateName_$eq(java.lang.String);
  public java.lang.String name();
  public void name_$eq(java.lang.String);
  public cn.scala.xtwy.Person();
}

```

从字节码中可以看出: (1) 定义成私有成员, getter和setter也是私有的 . (2)直接能访问的是我们自己定义的getter和setter方法,下面是调用方式

```bash
scala> val p=new Person()
p: Person = Person@12d0b54

scala> p.name
res29: String = null

//直接赋值法
scala> p.name="john"
p.name: String = john

scala> p.name
res30: String = john

```

从代码执行的结果看, 我们知道: 通过p.name="nike" 这种方式进行赋值,调用者
 并需要知道是其通过方法调用还是通过字段访问来进行操作的,这便是著名的__ 统一访问原则__

 如果类的成员域是val类型的变量,则只会生成getter方法

 ```bash
class Person {
  //类成员必须初始化，否则会报错
  //这里定义的是一个val公有成员
  val name:String="john"
}

javap -private Person
警告: 二进制文件Person包含cn.scala.xtwy.Person
Compiled from "Person.scala"
public class cn.scala.xtwy.Person {
  private final java.lang.String name;
  public java.lang.String name();
  public cn.scala.xtwy.Person();
}

 ```

 从字节码文件中可以看出：val变量对应的是java中的final类型变量，只生成了getter方法

 如果将成员域定义为private[this] ,则不会生成getter和setter方法

 ```bash
calss Persion{
	//private[this] 修饰
	private[this] var name:String='nike'

}


javap -private Person
警告: 二进制文件Person包含cn.scala.xtwy.Person
Compiled from "Person.scala"
public class cn.scala.xtwy.Person {
  private java.lang.String name;
  public cn.scala.xtwy.Person();
}

 ```

在java语言中,在定义javaBean的时候生成的都是setXXX(),getXxx()方法,但是scala语言生成的getter和setter方法并不是这样的,如果也需要程序自动生成getter和setter方法,则需要引入 scala.reflect.BeanProperty然后采用注解的方式修改变量

```bash
class Persion{
	//@BeanProperty 用于生成get 和set方法
	@BeanProperty var name:String="nike"
}


javap -private Person
警告: 二进制文件Person包含cn.scala.xtwy.Person
Compiled from "Person.scala"
public class cn.scala.xtwy.Person {
  private java.lang.String name;
  public java.lang.String name();
  public void name_$eq(java.lang.String);
  public void setName(java.lang.String);
  public java.lang.String getName();
  public cn.scala.xtwy.Person();
}

```

下面是getter.setter方法产生的规则

__ val/var  name __ 

generated Methods
public name name_=(var only)

when to use
to implement a property that is publicly
accessible and backed by a field

__ @BeanProperty val/var name __

generated Methods
public name 
getName() 
name_=(var only)
setName(...)(var only)

when to use
To interoperate with JavaBeans

private val/var name 

generated Methods
provate name 
name_=(var only)

To confine the field to the methods of this class, just like in java . Use private unless you really want a public property

private[this] val/var name 

none 

to confine the field to methods invoked on the same object . not commonly used



类主构造器

主构造器的定义与类的定义交织在一起, 将构造器参数直接放在类名称之后,如下代码:

```bash
//以下代码不但定义了一个Persion类,还定义了主构造器,参数为 String , Int
class Person (val name:String,val age:int)


javap -private Person
警告: 二进制文件Person包含cn.scala.xtwy.Person
Compiled from "Person.scala"
public class cn.scala.xtwy.Person {
  private final java.lang.String name;
  private final int age;
  public java.lang.String name();
  public int age();
  public cn.scala.xtwy.Person(java.lang.String, int);
}

//不难看出：上面的代码与下列java语言编写的代码等同
public class Person{
  private final String name;
  private final int age;
  public Person(String name,int age){
       this.name=name;
       this.age=age;
  }
  public String getName(){ return name}
  public int getAge() {return age}
}

//具体使用操作如下：
scala> val p=new Person("john",29)
p: Person = Person@abdc0f

scala> p.name
res31: String = john

scala> p.age
res32: Int = 29

```


前面我们定义的Persion类是一种无参主构造器
```bash
//Person类具有无参主构建器
class Person {
  println("constructing Person....")
  val name:String="john"
}

scala> val p=new Person()
constructing Person....
p: Person = Person@79895f

```

主构造器还可以使用默认参数

```bash
//默认参数的主构建器
class Person(val name:String="",val age:Int=18){
  println("constructing Person ........")
  override def toString()= name + ":"+ age
}

scala> val p=new Person
constructing Person ........
p: Person = :18

scala> val p=new Person("john")
constructing Person ........
p: Person = john:18

```

主构造器中的参数还可以加访问控制符

```bash

//默认参数的主构建器，参数带访问控制符号
//age变成私有成员，其getter方法是私有的，外部不能访问
class Person(val name:String="",private val age:Int=18){
  println("constructing Person ........")
  override def toString()= name + ":"+ age
}

```

当主构造器的参数不用var或val修饰的时候，参数会生成类的私有val成员，并且不会产生getter和setter方法

```bash

//不加变量修饰符
class Person(name:String,age:Int){
  println("constructing Person ........")
  override def toString()= name + ":"+ age
}

javap -private Person
警告: 二进制文件Person包含cn.scala.xtwy.Person
Compiled from "Person.scala"
public class cn.scala.xtwy.Person {
  private final java.lang.String name;
  private final int age;
  public java.lang.String toString();
  public cn.scala.xtwy.Person(java.lang.String, int);
}

//与下面类定义等同
class Person(private[this] val name:String,private[this] val age:Int){
  println("constructing Person ........")
  override def toString()= name + ":"+ age
}

javap -private Person
警告: 二进制文件Person包含cn.scala.xtwy.Person
Compiled from "Person.scala"
public class cn.scala.xtwy.Person {
  private final java.lang.String name;
  private final int age;
  public java.lang.String toString();
  public cn.scala.xtwy.Person(java.lang.String, int);
}

```

值得注意的是,将上述Persion类中的toString()方法去掉,则类中无任何地方使用了主构造器的参数,此时主构造器参数不会生成类成员.

```bash

即将
//不加变量修饰符
class Person(name:String,age:Int){
  println("constructing Person ........")
  override def toString()= name + ":"+ age
}
改成：
class Person( val name:String,age:Int){
  println("constructing Person ........")
}

其字节码文件如下：
D:\ScalaWorkspace\ScalaChapter06\bin\cn\scala\xtwy>javap -private Person
警告: 二进制文件Person包含cn.scala.xtwy.Person
Compiled from "Person.scala"
public class cn.scala.xtwy.Person {
  public cn.scala.xtwy.Person(java.lang.String, int);
}
//可以看出，主构造器参数不会生成类成员


```

下面图给出了scala中的主构造器参数生成类成员和方法的规则


name: String 

object-private field , or no field no method uses name

private val/var name:String

private field ,private getter/setter

val/var name:String 

private field,public getter/setter

@BeanProperty val/var name : String 

private field , public scala and javabeans
getters and setters


在某些情况下, 可能需要禁用主构建器,代码如下
```bash
//类名后面紧跟private关键字可以将主构建器设为私有，不允许外部使用
class Person private(var name:String,var age:Int){
	println("constructing Persion....")
}
//生成的字节码文件如下，可以看到其构建函数已经为private了
javap -private Person
警告: 二进制文件Person包含cn.scala.xtwy.Person
Compiled from "Person.scala"
public class cn.scala.xtwy.Person {
  private java.lang.String name;
  private int age;
  public java.lang.String name();
  public void name_$eq(java.lang.String);
  public int age();
  public void age_$eq(int);
  private cn.scala.xtwy.Person(java.lang.String, int);
}

//此时不能直接这么用
scala> val p=new Person("john",19)
<console>:9: error: constructor Person in class Person cannot be accessed in obj
ect $iw
       val p=new Person("john",19)

```

辅助构造函数
前面讲了，如果禁用掉了主构建器，则必须使用辅助构造函数来创建对象。辅助构造函数具有两个特点：（1）辅助构建器的名称为this，java中的辅助构造函数与类名相同，这常常会导致修改类名时出现不少问题，scala语言避免了这样的问题；（2）调用辅助构造函数时，必须先调用主构造函数或其它已经定义好的构造函数。

3.1 我们首先看一下只有辅助构造函数的Person类


```bash
//只有辅助构造函数的类
class Person{
  //类成员
  private var name:String=null
  private var age:Int=18
  private var sex:Int=0

  //辅助构造器
  def this(name:String){
    this()
    this.name=name
  }
  def this(name:String,age:Int){
    this(name)
    this.age=age
  }
   def this(name:String,age:Int,sex:Int){
    this(name,age)
    this.sex=sex
  }
}

//字节码文件
D:\ScalaWorkspace\ScalaChapter06\bin\cn\scala\xtwy>javap -private Person
警告: 二进制文件Person包含cn.scala.xtwy.Person
Compiled from "Person.scala"
public class cn.scala.xtwy.Person {
  private java.lang.String name;
  private int age;
  private int sex;
  private java.lang.String name();
  private void name_$eq(java.lang.String);
  private int age();
  private void age_$eq(int);
  private int sex();
  private void sex_$eq(int);
  public cn.scala.xtwy.Person();
  public cn.scala.xtwy.Person(java.lang.String);
  public cn.scala.xtwy.Person(java.lang.String, int);
  public cn.scala.xtwy.Person(java.lang.String, int, int);
}

//在定义辅助构造函数时，需要注意构造函数的顺序
class Person{
  //类成员
  private var name:String=null
  private var age:Int=18
  private var sex:Int=0

 //辅助构造器
 def this(name:String,age:Int,sex:Int){
    this(name,age)//此处会发生编译错误，这是因为def this(name:String,age:Int)没有被定义
    this.sex=sex
  }

  def this(name:String){
    this()
    this.name=name
  }
  def this(name:String,age:Int){
    this(name)
    this.age=age
  }

}
```

3.2 带主构造函数、辅助构造函数的Person类
```bash
//具有主构建函数和辅助构建函数的Person类
class Person(var name:String,var age:Int){
  //类成员
  private var sex:Int=0

  //辅助构造器
   def this(name:String,age:Int,sex:Int){
    this(name,age)
    this.sex=sex
  }
}

生成的字节码文件如下：
D:\ScalaWorkspace\ScalaChapter06\bin\cn\scala\xtwy>javap -private Person
警告: 二进制文件Person包含cn.scala.xtwy.Person
Compiled from "Person.scala"
public class cn.scala.xtwy.Person {
  private java.lang.String name;
  private int age;
  private int sex;
  public java.lang.String name();
  public void name_$eq(java.lang.String);
  public int age();
  public void age_$eq(int);
  private int sex();
  private void sex_$eq(int);
  public cn.scala.xtwy.Person(java.lang.String, int);
  public cn.scala.xtwy.Person(java.lang.String, int, int);
}
``` 

在主构造函数小节当中我们提到，有时候可能会禁用掉主构造函数，此时只能通过辅助构造函数来创建对象

```bash
//禁用主构造函数
class Person private(var name:String,var age:Int){
  //类成员
  private var sex:Int=0

  //辅助构造器
   def this(name:String,age:Int,sex:Int){
    this(name,age)
    this.sex=sex
   }

}

//其字节码文件内容如下
D:\ScalaWorkspace\ScalaChapter06\bin\cn\scala\xtwy>javap -private Person
警告: 二进制文件Person包含cn.scala.xtwy.Person
Compiled from "Person.scala"
public class cn.scala.xtwy.Person {
  private java.lang.String name;
  private int age;
  private int sex;
  public java.lang.String name();
  public void name_$eq(java.lang.String);
  public int age();
  public void age_$eq(int);
  private int sex();
  private void sex_$eq(int);
  private cn.scala.xtwy.Person(java.lang.String, int);
  public cn.scala.xtwy.Person(java.lang.String, int, int);
}
```


