title: scala学习笔记-包和引入
date: 2017-03-07 20:11:13
tags:
- 乡村销客
- 大数据
categories: scala
---

# 包的作用与定义

<!-- more -->
同java中的包.c++中的命名空间一样,scala中的包主要用于大型工程代码的组织同时也解决命名冲突的问题,scala中的包与java有着诸多的相似之处,但是scala语言中的包更加灵活.

```bash

//将代码组织到cn.scala.xtwy包中
package cn.scala.xtwy

abstract class Animal {
  //抽象字段(域）
  var height:Int
  //抽象方法
  def eat:Unit
}

class Person(var height:Int) extends Animal{
  override def eat()={
    println("eat by mouth")
  }

}

object Person extends App{
  new Person(10).eat()
}

```
# 包的作用域与引入（import）的使用方法
```bash
package cn{
  package scala{
    //在包cn.scala下创建了一个Utils单例
    object Utils{
      def toString(x:String){
        println(x)
      }
      //外层包无法直接访问内层包，下面这一行代码编译通不过
     //def getTeacher():Teacher=new Teacher("john")
     //如果一定要使用的话，可以引入包
     import cn.scala.xtwy._
     def getTeacher():Teacher=new Teacher("john")
    }
    //定义了cn.scala.xtwy
    package xtwy{
      class Teacher(var name:String) {
           //演示包的访问规则
           //内层包可以访问外层包中定义的类或对象，无需引入
           def printName()={Utils.toString(name)}
      }

    }
  }
}
object appDemo{
        //scala允许在任何地方进行包的引入，_的意思是引入该包下的所有类和对象
        import cn.scala._
        import cn.scala.xtwy._
        def main(args: Array[String]): Unit = {
            Utils.toString(new Teacher("john").name)
            new Teacher("john").printName() 
        }

}
```


# 访问控制

在java语言中,主要通过public  private protected及默认控制来实现包中类成员的访问控制,当定义一个类时,如果类成员不加任何访问控制符时,表示该成员在定义该类的包中可见. 
在scala中没有public关键字,仅有private和protected访问控制符,当一个类成员不加private和protected时,他的访问权限就是public. 下面逐个解释:

1. provate成员

private成员同java是一样的，所有带该关键字修饰的成员仅能在定义它的类或对象中使用，在外部是不可见的

```bash
class Student(var name:String,var age:Int){
	private var sex:Int=0
	class Course(var cNamen:String,val gpa:Float){
		//可以直接访问其外部类的私有成员
    	def getStudentSex(student:Student)= student.sex
	}
}

// 班级类
class Class{
	//下面这条语句统计通不过，因为sex是私有的
  // def getStudentSex(student:Student)=student.sex
}


object Student {
  private var studentNo:Int=0;
  def uniqueStudentNo()={
    studentNo+=1
    studentNo
  }
  def apply(name:String,age:Int)=new Student(name,age)

  def main(args: Array[String]): Unit = {
    println(Student.uniqueStudentNo())
    val s=new Student("john",29)
    //直接访问伴生类Student中的私有成员
    println(s.sex)

    val s1=Student("john",29)
    println(s1.name)
    println(s1.age)

    //使用内部类
    val c1=new s1.Course("Scala",3.0f)

  }
}

```

2.protected 成员

在java语言中，protected成员不但可以被该类及其子类访问，也可以被同一个包中的其它类使用，但在scala中，protected成员只能被该类及其子类访问

```bash
class SuperClass {
  protected def f()=println(".....")
}

class SubClass extends SuperClass{
  f()
}

class OtherClass{
  //下面这个语句会报错
  //f()
}


```

3. 无修饰符成员

无修饰符的成员同java的public , 可以在任何位置进行访问
4.范围保护

在scala中提供了更为灵活的访问控制方法，private、protected除了可以直接修饰成员外，还可以以private[X]、protected[X]的方式进行更为灵活的访问控制，这种访问控制的意思是可以将private、protected限定到X，X可以是包、类，还可以是单例对象
```bash

package cn{
  class UtilsTest{
     //编译通不过，因为Utils利用private[scala]修饰，只能在scala及其子包中使用
    //Utils.toString()
  }
  package scala{
    //private[scala]限定Utils只能在scala及子包中使用
    private[scala] object Utils{
      def toString(x:String){
        println(x)
      }
      import cn.scala.xtwy._
      def getTeacher():Teacher=new Teacher("john")

    }
    package xtwy{
      class Teacher(var name:String) {
           def printName()={Utils.toString(name)}
      }

    }
  }
}
object appDemo{
        import cn.scala._
        import cn.scala.xtwy._
        def main(args: Array[String]): Unit = {
            //编译通不过，同UtilsTest
            //Utils.toString(new Teacher("john").name)
            new Teacher("john").printName() 
        }

}
```

private[this]，限定只有该类的对象才能访问，称这种成员为对象私有成员

```bash
package cn.scala.xtwy;

class Teacher(var name: String) {
  private[this] def printName(tName:String="") :Unit= { println(tName) }
  //调用private[this] printName方法
  def print(n:String)=this.printName(n)
}

object Teacher{
   //private[this]限定的成员，即使伴生对象Teacher也不能使用
  //def printName=new Teacher("john").printName()
}

object appDemo {
  def main(args: Array[String]): Unit = {
          //编译不能通过
         //new Teacher("john").printName()
  }

}
```

private，定义的类及伴生对象可以访问

```bash
package cn.scala.xtwy;

class Teacher(var name: String) {
  private def printName(tName:String="") :Unit= { println(tName) }
  //可以访问
  def print(n:String)=this.printName(n)
}

object Teacher{
  //伴生对象可以访问
  def printName=new Teacher("john").printName()
}

object appDemo {
  def main(args: Array[String]): Unit = {
          //不能访问
         //new Teacher("john").printName()
  }

}

```

下面给出的是访问规则表

修饰符	访问范围
无任何修饰符	任何地方都可以使用
private[scala]	在定义的类中可以访问，在scala包及子包中可以访问
private[this]	只能在定义的类中访问，即使伴生对象也不能访问团
private	在定义的的类及伴生对象中可以访问，其它地方不能访问
protected[scala]	在定义的类及子类中可以访问，在scala包及子包中可以访问，
protected[this]	只能在定义的类及子类中访问，即使伴生对象也不能访问
protected	在定义的类及子类中访问，伴生对象可以访问，其它地方不能访问

# 包对象

包对象主要用于常量,工具函数,使用时直接通过包名引用,
