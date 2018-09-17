---
title: Java 中int和Integer的区别
date: 2016-08-04 11:41:57
tags: [java, int, Integer, j2se]
categories: java
---

# Java中int和Integer的区别

## Java各种数据类型详细介绍及其区别

- 基本类型，或者叫做内置类型，是JAVA中不同于类的特殊类型。

Java中的简单类型从概念上分为四种：实数、整数、字符、布尔值。但是有一点需要说明的是，Java里面只有八种原始类型，其列表如下：

实数：double、float

整数：byte、short、int、long

字符：char

布尔值：boolean

复杂类型和基本类型的内存模型本质上是不一样的，简单数据类型的存储原理是这样的：所有的简单数据类型不存在“引用”的概念，简单数据类型都是直接存储在内存中的内存栈上的，数据本身的值就是存储在栈空间里面，而Java语言里面只有这八种数据类型是这种存储模型；而其他的只要是继承于Object类的复杂数据类型都是按照Java里面存储对象的内存模型来进行数据存储的，使用Java内存堆和内存栈来进行这种类型的数据存储，简单地讲，“引用”是存储在有序的内存栈上的，而对象本身的值存储在内存堆上的。

- Java的简单数据讲解列表如下：

int：int为整数类型，在存储的时候，用4个字节存储，范围为-2,147,483,648到2,147,483,647，在变量初始化的时候，int类型的默认值为0。

short：short也属于整数类型，在存储的时候，用2个字节存储，范围为-32,768到32,767，在变量初始化的时候，short类型的默认值为0，一般情况下，因为Java本身转型的原因，可以直接写为0。

long：long也属于整数类型，在存储的时候，用8个字节存储，范围为-9,223,372,036,854,775,808到9,223,372,036, 854,775,807，在变量初始化的时候，long类型的默认值为0L或0l，也可直接写为0。

byte：byte同样属于整数类型，在存储的时候，用1个字节来存储，范围为-128到127，在变量初始化的时候，byte类型的默认值也为0。

float：float属于实数类型，在存储的时候，用4个字节来存储，范围为32位IEEEE 754单精度范围，在变量初始化的时候，float的默认值为0.0f或0.0F，在初始化的时候可以写0.0。

double：double同样属于实数类型，在存储的时候，用8个字节来存储，范围为64位IEEE 754双精度范围，在变量初始化的时候，double的默认值为0.0。

char：char属于字符类型，在存储的时候用2个字节来存储，因为Java本身的字符集不是用ASCII码来进行存储，是使用的16位Unicode字符集，它的字符范围即是Unicode的字符范围，在变量初始化的时候，char类型的默认值为'u0000'。

boolean：boolean属于布尔类型，在存储的时候不使用字节，仅仅使用1位来存储，范围仅仅为0和1，其字面量为true和false，而boolean变量在初始化的时候变量的默认值为false。

Integer是int的封装类，里面有很多进行处理的静态方法

Integer是对象而int不是，内存的分配位置也不一样

Integer的属性和其他类一样的！在方法里都是引用传递，而原始类型是值传递！

jdk1.5以后可以从int自动装箱Integer类。

int是为了兼容以前的编程语言使用的基本类型，目的是让程序效率更高，以为它是直接分配到栈上的。所以它不是对象，不能有类似 int.operation()的操作。

Integer是java中一切都是对象这个大前提下的int的包装类型，可以使用方法，是个对象，是用new分配到堆上的。

jdk1.5后，引入了类似c#中的自动装、拆箱，使得Integer i = 1；这样的表达直接可行。

int是一种基本数据类型，而Integer是相应于int的类类型，称为对象包装。

实现这种对象包装的目的主要是因为类能够提供必要的方法，用于实现基本数据类型的数值与可打印字符串之间的转换，以及一些其他的实用程序方法；

另外，有些数据结构库类只能操作对象，而不支持基本数据类型的变量，包装类提供一种便利的方式，能够把基本数据类型转换成等价的对象，从而可以利用数据结构库类进行处理。

int 是基本类型，(int)(Math.Random()*100)就是一个数，可以进行加见乘除。 Integer是class ,那么 new Integer(temp)就是一个对象了，可以用到Integer这个class的方法，例如用intvalue()可以返回这个int的值。int a=1;这是基本数据类型是能参与运算的.而Integer b= new Integer(1);c=b.floatvalue;Float a= new Float(null);是可以的用Float初始化一个对象这个a有很多方法而float a;就没有因为原始没有引用类，java 提供两种不同的类型：引用类型（或者封装类型，Warpper）和原始类型（或内置类型，Primitive）。Int是java的原始数据类型，Integer是java为int提供的封装类。Java为每个原始类型提供了封装类。

- 原始类型 封装类

boolean Boolean

char Character

byte Byte

short Short

int Integer

long Long

float Float

double Double

引用类型和原始类型的行为完全不同，并且它们具有不同的语义。引用类型和原始类型具有不同的特征和用法，它们包括：大小和速度问题，这种类型以哪种类型的数据结构存储，当引用类型和原始类型用作某个类的实例数据时所指定的缺省值。对象引用实例变量的缺省值为 null，而原始类型实例变量的缺省值与它们的类型有关。

## 1 、Boolean VS boolean

public final class Boolean extends Object  implementsSerializable,Comparable

Boolean 类将基本类型为boolean的值包装在一个对象中。一个Boolean类型的对象只包含一个类型为boolean的字段。此外，此类还为boolean和String的相互转换提供了许多方法，并提供了处理 boolean时非常有用的其他一些常量和方法。

## 2、 Byte VS byte

public final class Byte extends Number implements Comparable Byte类将基本类型 byte的值包装在一个对象中。一个Byte类型的对象只包含一个类型为 byte的字段。此外，该类还为 byte和 String的相互转换提供了几种方法，并提供了处理 byte时非常有用的其他一些常量和方法。

## 3、 Character VS char

public final class Character extends Object  implements Serializable, Comparable

Character类在对象中包装一个基本类型char的值。

Character类型的对象包含类型为char的单个字段。此外，该类提供了几种方法，以确定字符的类别（小写字母，数字，等等），并将字符从大写转换成小写，反之亦然。

## 4 、Double VS double

public final class Double extends Number implements Comparable Double类在对象中包装了一个基本类型double的值。每个Double类型的对象都包含一个double类型的字段。此外，该类还提供了多个方法，可以将double转换为String，将String转换为double，还提供了其他一些处理double时有用的常量和方法。

## 5、 Float VS float

public final class Float extends Number implements Comparable

Float类在对象中包装了一个float基本类型的值。Float类型的对象包含一个float类型的字段。此外，此类提供了几种方法，可在float类型和String类型之间互相转换，并且还提供了处理float类型时非常有用的其他一些常量和方法。

## 6、 Integer VS int

public final class Integer extends Number implements Comparable

Integer类在对象中包装了一个基本类型int的值。Integer类型的对象包含一个int类型的字段。

此外，该类提供了多个方法，能在int类型和String类型之间互相转换，还提供了处理int类型时非常有用的其他一些常量和方法。

## 7 Long VS long

public final class Long extends Number implements Comparable

Long类在对象中封装了基本类型long的值。每个Long类型的对象都包含一个long类型的字段。

此外，该类提供了多个方法，可以将long转换为String，将String转换为long，除此之外，还提供了其他一些处理long时有用的常量和方法。

## 8、 Short VS short

public final class Short extends Number implements Comparable

Short类在对象中包装基本类型short的值。一个Short类型的对象只包含一个short类型的字段。另外，该类提供了多个方法，可以将short转换为String，将String转换为short，同时还提供了其他一些处理short时有用的常量和方法。

## 9、public final class Voidextends Object

Void 类是一个不可实例化的占位符类，它保持一个对代表 Java 关键字 void 的 Class 对象的引用。

类的对象才能为null，不能把null赋值给一个变量不能，如int m=null;但可以String s=null;因为String是个类。

