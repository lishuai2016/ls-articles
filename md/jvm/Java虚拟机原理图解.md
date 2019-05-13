---
title: Java虚拟机原理图解
categories: 
- jvm
tags:
---

#Java虚拟机原理图解

[参考](https://blog.csdn.net/u010349169/column/info/jvm-principle)


目录：
1、[class文件基本组织结构]
NO1. 魔数(magic)
NO2.版本号(minor_version,major_version)
NO3.常量池计数器(constant_pool_count)
NO4.常量池数据区(constant_pool[contstant_pool_count-1])
NO6.访问标志(access_flags)[类或者接口的]
NO7.类索引(this_class)
NO8.父类索引(super_class)
NO9.接口计数器(interfaces_count)
NO10.接口信息数据区(interfaces[interfaces_count])
NO11.字段计数器(fields_count)
NO12.字段信息数据区(fields[fields_count])
NO13.方法计数器(methods_count)
NO14.方法信息数据区(methods[methods_count])
NO15.属性计数器(attributes_count)
NO16.属性信息数据区(attributes[attributes_count])

2、[Class文件中的常量池详解]
NO1.常量池在class文件的什么位置？
NO2.常量池的里面是怎么组织的？
NO3.常量池项 (cp_info) 的结构是怎样的？
NO4.常量池 能够表示那些信息？
NO5.int和float数据类型的常量在常量池中是怎样表示和存储的？( ----介绍 常量池项  CONSTANT_Integer_info, CONSTANT_Float_info)
NO6.long和 double数据类型的常量在常量池中是怎样表示和存储的？（----介绍 常量池项 CONSTANT_Long_info, CONSTANT_Double_info）
NO7.String类型的字符串常量在常量池中是怎样表示和存储的？( ----介绍 常量池项 CONSTANT_String_info,CONSTANT_Utf8_info)
NO8.类文件中定义的类名和类中使用到的类在常量池中是怎样被组织和存储的？（----介绍 常量池项 CONSTANT_Class_info）
NO9.类中引用到的field字段在常量池中是怎样描述的？( ----介绍 常量池项   CONSTANT_Fieldref_info, CONSTANT_Name_Type_info)
NO10.类中引用到的method方法在常量池中是怎样被描述的？(----介绍 常量池项  CONSTANT_Methodref_info)
NO11.类中引用到某个接口中定义的method方法在常量池中是怎样描述的？(----介绍 常量池项  CONSTANT_InterfaceMethodref_info)
NO12.CONSTANT_MethodType_info
NO13.CONSTANT_MethodHandle_info
NO14.CONSTANT_InvokeDynamic_info

3、[class文件中的访问标志、类索引、父类索引、接口索引集合]
1). 访问标志、类索引、父类索引、接口索引集合 在class文件中的位置
2). 访问标志(access_flags)能够表示什么？
3). 类索引(this_class)是什么？
4). 父类索引(super_class)是什么？
5). 接口索引集合(interfaces)是什么？


4 、[class文件中的字段表集合--field字段在class文件中是怎样组织的]

 
5、 [class文件中的方法表集合--method方法在class文件中是怎样组织的]


备注：[方法区类的存储方式]
1、引导类加载器将类信息加载到方法区中，以特定方式组织，对于某一个特定的类而言，
在方法区中它应该有 [运行时常量池]、[类型信息]、[字段信息]、[方法信息]、[类加载器的引用]，对应class实例的引用等信息;
2、类加载器的引用,由于这些类是由引导类加载器(Bootstrap Classloader)进行加载的，而 引导类加载器是有C++语言实现的，所以是无法访问的，
故而该引用为NULL;
3、对应class实例的引用， 类加载器在加载类信息放到方法区中后，会创建一个对应的Class 类型的实例放到堆(Heap)中, 
作为开发人员访问方法区中类定义的入口和切入点;




# 1、class文件基本组织结构
我们知道，我们写好的.java 源代码，最后会被Java编译器编译成后缀为.class的文件，该类型的文件是由字节组成的文件，
又叫字节码文件。那么，class字节码文件里面到底是有什么呢？它又是怎样组织的呢？让我们先来大概了解一下他的组成结构吧。
<div align="center"> <img src="/images/Java虚拟机原理图解1.png"/> </div><br>

## 1.1、魔数(magic)
所有的由Java编译器编译而成的class文件的前[4个字节]都是“0xCAFEBABE”  
它的作用在于：当JVM在尝试加载某个文件到内存中来的时候，会首先判断此class文件有没有JVM认为可以接受的“签名”，
即JVM会首先读取文件的前4个字节，判断该4个字节是否是“0xCAFEBABE”，如果是，则JVM会认为可以将此文件当作class文件来加载并使用。

## 1.2、版本号(minor_version,major_version)
随着Java本身的发展，Java语言特性和JVM虚拟机也会有相应的更新和增强。目前我们能够用到的JDK版本如：1.5，1.6，1.7，
还有现如今最新的1.8。发布新版本的目的在于：在原有的版本上增加新特性和相应的JVM虚拟机的优化。而随着主版本发布的次版本，
则是修改相应主版本上出现的bug。我们平时只需要关注主版本就可以了。
[主版本号和次版本号在class文件中各占两个字节]，副版本号占用第5、6两个字节，而主版本号则占用第7，8两个字节。
JDK1.0的主版本号为45，以后的每个新主版本都会在原先版本的基础上加1。若现在使用的是JDK1.7编译出来的class文件，则相应的主版本号应该是51,
对应的7，8个字节的十六进制的值应该是 0x33。

一个 JVM实例只能支持特定范围内的主版本号 （Mi 至Mj） 和 0 至特定范围内 （0 至 m） 的副版本号。
假设一个 Class 文件的格式版本号为 V， 仅当Mi.0 ≤ v ≤ Mj.m成立时，这个 Class 文件才可以被此 Java 虚拟机支持。
不同版本的 Java 虚拟机实现支持的版本号也不同，高版本号的 Java 虚拟机实现可以支持低版本号的 Class 文件，反之则不成立。
JVM在加载class文件的时候，会读取出主版本号，然后比较这个class文件的主版本号和JVM本身的版本号，
如果JVM本身的版本号 < class文件的版本号，JVM会认为加载不了这个class文件，会抛出我们经常见到的"java.lang.UnsupportedClassVersionError:
 Bad version number in .class file " Error 错误；反之，JVM会认为可以加载此class文件，继续加载此class文件。

小贴士：
1. 有时候我们在运行程序时会抛出这个Error 错误："java.lang.UnsupportedClassVersionError: Bad version number in .class file"。
上面已经揭示了出现这个问题的原因，就是在于当前尝试加载class文件的JVM虚拟机的版本 低于class文件的版本。
解决方法：1.重新使用当前jvm编译源代码，然后再运行代码；2.将当前JVM虚拟机更新到class文件的版本。

2. 怎样查看class文件的版本号？
可以借助于文本编辑工具，直接查看该文件的7，8个字节的值，确定class文件是什么版本的。
当然快捷的方式使用JDK自带的javap工具，如当前有Programmer.class 文件，进入此文件所在的目录，
然后执行 ”javap -v Programmer“,结果会类似如下所示：
<div align="center"> <img src="/images/Java虚拟机原理图解2.png"/> </div><br>

## 1.3、常量池计数器(constant_pool_count)
常量池是class文件中非常重要的结构，它描述着整个class文件的字面量信息。 
常量池是由一组constant_pool结构体数组组成的，而数组的大小则由常量池计数器指定。
常量池计数器constant_pool_count 的值 =constant_pool表中的成员数+ 1。
constant_pool表的索引值只有在大于 0 且小于constant_pool_count时才会被认为是有效的。

## 1.4、常量池数据区(constant_pool[contstant_pool_count-1])
常量池，constant_pool是一种表结构,它包含 Class 文件结构及其子结构中引用的所有字符串常量、 类或接口名、字段名和其它常量。 
常量池中的每一项都具备相同的格式特征——第一个字节作为类型标记用于识别该项是哪种类型的常量，称为 “tag byte” 。
常量池的索引范围是 1 至constant_pool_count−1。常量池的具体细节我们会稍后讨论。

## 1.6、访问标志(access_flags)
访问标志，access_flags 是一种掩码标志，用于表示某个类或者接口的访问权限及基础属性。
<div align="center"> <img src="/images/Java虚拟机原理图解3.png"/> </div><br>


## 1.7、类索引(this_class)
类索引，this_class的值必须是对constant_pool表中项目的一个有效索引值。
constant_pool表在这个索引处的项必须为CONSTANT_Class_info 类型常量，表示这个 Class 文件所定义的类或接口。

## 1.8、父类索引(super_class)
父类索引，对于类来说，super_class 的值必须为 0 或者是对constant_pool 表中项目的一个有效索引值。
如果它的值不为 0，那 constant_pool 表在这个索引处的项必须为CONSTANT_Class_info 类型常量，表示这个 Class 文件所定义的类的直接父类。
当前类的直接父类，以及它所有间接父类的access_flag 中都不能带有ACC_FINAL 标记。对于接口来说，
它的Class文件的super_class项的值必须是对constant_pool表中项目的一个有效索引值。
constant_pool表在这个索引处的项必须为代表 java.lang.Object 的 CONSTANT_Class_info 类型常量 。
如果 Class 文件的 super_class的值为 0，那这个Class文件只可能是定义的是java.lang.Object类，只有它是唯一没有父类的类。

## 1.9、接口计数器(interfaces_count)
接口计数器，interfaces_count的值表示当前类或接口的直接父接口数量。

## 1.10、接口信息数据区(interfaces[interfaces_count])
接口表，interfaces[]数组中的每个成员的值必须是一个对constant_pool表中项目的一个有效索引值， 它的长度为 interfaces_count。
每个成员 interfaces[i]  必须为 CONSTANT_Class_info类型常量，其中 0 ≤ i <interfaces_count。
在interfaces[]数组中，成员所表示的接口顺序和对应的源代码中给定的接口顺序（从左至右）一样，即interfaces[0]对应的是源代码中最左边的接口。

## 1.11、字段计数器(fields_count)
字段计数器，fields_count的值表示当前 Class 文件 fields[]数组的成员个数。 fields[]数组中每一项都是一个field_info结构的数据项，
它用于表示该类或接口声明的类字段或者实例字段。

## 1.12、.字段信息数据区(fields[fields_count])
字段表，fields[]数组中的每个成员都必须是一个fields_info结构的数据项，用于表示当前类或接口中某个字段的完整描述。
fields[]数组描述当前类或接口声明的所有字段，但不包括从父类或父接口继承的部分。

## 1.13、方法计数器(methods_count)
方法计数器， methods_count的值表示当前Class 文件 methods[]数组的成员个数。Methods[]数组中每一项都是一个 method_info 结构的数据项。

## 1.14、方法信息数据区(methods[methods_count])
方法表，methods[] 数组中的每个成员都必须是一个 method_info 结构的数据项，用于表示当前类或接口中某个方法的完整描述。
如果某个method_info 结构的access_flags 项既没有设置 ACC_NATIVE 标志也没有设置ACC_ABSTRACT 标志
，那么它所对应的方法体就应当可以被 Java 虚拟机直接从当前类加载，而不需要引用其它类。
 method_info结构可以表示类和接口中定义的所有方法，包括实例方法、类方法、实例初始化方法方法和类或接口初始化方法方法 。
 methods[]数组只描述当前类或接口中声明的方法，不包括从父类或父接口继承的方法。

## 1.15、属性计数器(attributes_count)
属性计数器，attributes_count的值表示当前 Class 文件attributes表的成员个数。attributes表中每一项都是一个attribute_info 结构的数据项。

## 1.16、属性信息数据区(attributes[attributes_count])
属性表，attributes 表的每个项的值必须是attribute_info结构。
在Java 7 规范里，Class文件结构中的attributes表的项包括下列定义的属性： 
InnerClasses  、 EnclosingMethod 、 Synthetic  、Signature、SourceFile，SourceDebugExtension 、Deprecated、RuntimeVisibleAnnotations 、
RuntimeInvisibleAnnotations以及BootstrapMethods属性。

对于支持 Class 文件格式版本号为 49.0 或更高的 Java 虚拟机实现，必须正确识别并读取attributes表中的Signature、
RuntimeVisibleAnnotations和RuntimeInvisibleAnnotations属性。对于支持Class文件格式版本号为 51.0 或更高的 Java 虚拟机实现，
必须正确识别并读取 attributes表中的BootstrapMethods属性。
Java 7 规范 要求任一 Java 虚拟机实现可以自动忽略 Class 文件的 attributes表中的若干 （甚至全部） 它不可识别的属性项。
任何本规范未定义的属性不能影响Class文件的语义，只能提供附加的描述信息 。
根据上述的叙述，我们可以将class的文件组织结构概括成以下面这个结构体：
<div align="center"> <img src="/images/Java虚拟机原理图解4.png"/> </div><br>

# 2、Class文件中的常量池详解

## 2.1、常量池在class文件的什么位置？
class文件基本组织结构中已经提到了class的文件结构，在class文件中的魔数、副版本号、主版本之后，紧接着就是常量池的数据区域了。

## 2.2、常量池的里面是怎么组织的？
常量池的组织很简单，前端的两个字节占有的位置叫做常量池计数器(constant_pool_count)，它记录着常量池的组成元素常量池项(cp_info) 的个数。
紧接着会排列着constant_pool_count-1个常量池项(cp_info)。如下图所示：
<div align="center"> <img src="/images/Java虚拟机原理图解5.png"/> </div><br>

## 2.3、常量池项 (cp_info) 的结构是什么？
每个常量池项(cp_info) 都会对应记录着class文件中的某中类型的字面量。让我们先来了解一下常量池项(cp_info)的结构吧：
<div align="center"> <img src="/images/Java虚拟机原理图解6.png"/> </div><br>
    
JVM虚拟机规定了不同的tag值和不同类型的字面量对应关系如下：
<div align="center"> <img src="/images/Java虚拟机原理图解7.png"/> </div><br>

所以根据cp_info中的tag 不同的值，可以将cp_info 更细化为以下结构体：
CONSTANT_Utf8_info,CONSTANT_Integer_info,CONSTANT_Float_info,CONSTANT_Long_info,
CONSTANT_Double_info,CONSTANT_Class_info,CONSTANT_String_info,CONSTANT_Fieldref_info,
CONSTANT_Methodref_info,CONSTANT_InterfaceMethodref_info,CONSTANT_NameAndType_info,CONSTANT_MethodHandle_info,
CONSTANT_MethodType_info,CONSTANT_InvokeDynamic_info。
<div align="center"> <img src="/images/Java虚拟机原理图解8.png"/> </div><br>

现在让我们看一下细化了的常量池的结构会是类似下图所示的样子：
<div align="center"> <img src="/images/Java虚拟机原理图解9.png"/> </div><br>

          
## 2.4、常量池能够表示那些信息？
<div align="center"> <img src="/images/Java虚拟机原理图解10.png"/> </div><br>

 

## 2.5、int和float数据类型的常量在常量池中是怎样表示和存储的？(CONSTANT_Integer_info, CONSTANT_Float_info)
Java语言规范规定了 int类型和Float 类型的数据类型占用 4 个字节的空间。那么存在于class字节码文件中的该类型的常量是如何存储的呢？
相应地，在常量池中，将 int和Float类型的常量分别使用CONSTANT_Integer_info和 Constant_float_info表示，他们的结构如下所示：
<div align="center"> <img src="/images/Java虚拟机原理图解11.png"/> </div><br>

举例：建下面的类 IntAndFloatTest.java，在这个类中，我们声明了五个变量，但是取值就两种int类型的10 和Float类型的11f。

```java
package com.louis.jvm;
 
public class IntAndFloatTest {
	
	private final int a = 10;
	private final int b = 10;
	private float c = 11f;
	private float d = 11f;
	private float e = 11f;
	
}
```

然后用编译器编译成IntAndFloatTest.class字节码文件，我们通过javap -v IntAndFloatTest 指令来看一下其常量池中的信息，
可以看到虽然我们在代码中写了两次10 和三次11f，但是常量池中，就只有一个常量10 和一个常量11f,如下图所示:
<div align="center"> <img src="/images/Java虚拟机原理图解12.png"/> </div><br>

从结果上可以看到常量池第#8 个常量池项(cp_info) 就是CONSTANT_Integer_info,值为10；
第#23个常量池项(cp_info) 就是CONSTANT_Float_info,值为11f。
代码中所有用到 int 类型 10 的地方，会使用指向常量池的指针值#8 定位到第#8 个常量池项(cp_info)，即值为 10的结构体 CONSTANT_Integer_info，
而用到float类型的11f时，也会指向常量池的指针值#23来定位到第#23个常量池项(cp_info) 即值为11f的结构体CONSTANT_Float_info。如下图所示：
<div align="center"> <img src="/images/Java虚拟机原理图解13.png"/> </div><br>


## 2.6、long和 double数据类型的常量在常量池中是怎样表示和存储的？(CONSTANT_Long_info、CONSTANT_Double_info )
Java语言规范规定了 long 类型和 double类型的数据类型占用8 个字节的空间。那么存在于class 字节码文件中的该类型的常量是如何存储的呢？
相应地，在常量池中，将long和double类型的常量分别使用CONSTANT_Long_info和Constant_Double_info表示，他们的结构如下所示：
<div align="center"> <img src="/images/Java虚拟机原理图解14.png"/> </div><br>

举例：建下面的类 LongAndDoubleTest.java，在这个类中，我们声明了六个变量，但是取值就两种Long 类型的-6076574518398440533L 和Double 
类型的10.1234567890D。

```java
package com.louis.jvm;
 
public class LongAndDoubleTest {
	
	private long a = -6076574518398440533L;
	private long b = -6076574518398440533L;
	private long c = -6076574518398440533L;
	private double d = 10.1234567890D;
	private double e = 10.1234567890D;
	private double f = 10.1234567890D;
}
```
然后用编译器编译成 LongAndDoubleTest.class 字节码文件，我们通过javap -v LongAndDoubleTest指令来看一下其常量池中的信息，
可以看到虽然我们在代码中写了三次-6076574518398440533L 和三次10.1234567890D，但是常量池中，就只有一个常量-6076574518398440533L 
和一个常量10.1234567890D,如下图所示:
<div align="center"> <img src="/images/Java虚拟机原理图解15.png"/> </div><br>

 从结果上可以看到常量池第 #18 个常量池项(cp_info) 就是CONSTANT_Long_info,值为-6076574518398440533L ；
 第 #26个常量池项(cp_info) 就是CONSTANT_Double_info,值为10.1234567890D。
代码中所有用到 long 类型-6076574518398440533L 的地方，会使用指向常量池的指针值#18 定位到第 #18 个常量池项(cp_info)，
即值为-6076574518398440533L 的结构体CONSTANT_Long_info，而用到double类型的10.1234567890D时，
也会指向常量池的指针值#26 来定位到第 #26 个常量池项(cp_info) 即值为10.1234567890D的结构体CONSTANT_Double_info。如下图所示：
<div align="center"> <img src="/images/Java虚拟机原理图解16.png"/> </div><br>


## 2.7、String类型的字符串常量在常量池中是怎样表示和存储的？（CONSTANT_String_info、CONSTANT_Utf8_info）
对于字符串而言，JVM会将字符串类型的字面量以UTF-8 编码格式存储到在class字节码文件中。这么说可能有点摸不着北，
我们先从直观的Java源码中中出现的用双引号"" 括起来的字符串来看，在编译器编译的时候，都会将这些字符串转换成CONSTANT_String_info结构体，
然后放置于常量池中。其结构如下所示：
<div align="center"> <img src="/images/Java虚拟机原理图解17.png"/> </div><br>

如上图所示的结构体，CONSTANT_String_info结构体中的string_index的值指向了CONSTANT_Utf8_info结构体，
而字符串的utf-8编码数据就在这个结构体之中。如下图所示：
<div align="center"> <img src="/images/Java虚拟机原理图解18.png"/> </div><br>

请看一例，定义一个简单的StringTest.java类，然后在这个类里加一个"JVM原理" 字符串，然后，我们来看看它在class文件中是怎样组织的。

```java
package com.louis.jvm;
 
public class StringTest {
	private String s1 = "JVM原理";
	private String s2 = "JVM原理";
	private String s3 = "JVM原理";
	private String s4 = "JVM原理";
}
```
将Java源码编译成StringTest.class文件后，在此文件的目录下执行 javap -v StringTest 命令，会看到如下的常量池信息的轮廓：
<div align="center"> <img src="/images/Java虚拟机原理图解19.png"/> </div><br>
 
(PS :使用javap -v 指令能看到易于我们阅读的信息，查看真正的字节码文件可以使用HEXWin、NOTEPAD++、UtraEdit 等工具。)
在面的图中，我们可以看到CONSTANT_String_info结构体位于常量池的第#15个索引位置。
而存放"Java虚拟机原理" 字符串的 UTF-8编码格式的字节数组被放到CONSTANT_Utf8_info结构体中，该结构体位于常量池的第#16个索引位置。
上面的图只是看了个轮廓，让我们再深入地看一下它们的组织吧。请看下图：
<div align="center"> <img src="/images/Java虚拟机原理图解20.png"/> </div><br>

由上图可见：“JVM原理”的UTF-8编码的数组是：4A564D E5 8E 9FE7 90 86，并且存入了CONSTANT_Utf8_info结构体中。

  

## 2.8、类文件中定义的类名和类中使用到的类在常量池中是怎样被组织和存储的？(CONSTANT_Class_info)
JVM会将某个Java 类中所有使用到了的类的完全限定名 以二进制形式的完全限定名 封装成CONSTANT_Class_info结构体中，
然后将其放置到常量池里。CONSTANT_Class_info 的tag值为 7 。其结构如下：
<div align="center"> <img src="/images/Java虚拟机原理图解21.png"/> </div><br>
        

Tips：类的完全限定名和二进制形式的完全限定名
在某个Java源码中，我们会使用很多个类，比如我们定义了一个 ClassTest的类，并把它放到com.louis.jvm 包下，
则 ClassTest类的完全限定名为com.louis.jvm.ClassTest，将JVM编译器将类编译成class文件后，此完全限定名在class文件中，
是以二进制形式的完全限定名存储的，即它会把完全限定符的"."换成"/" ，
即在class文件中存储的 ClassTest类的完全限定名称是"com/louis/jvm/ClassTest"。
因为这种形式的完全限定名是放在了class二进制形式的字节码文件中，所以就称之为 二进制形式的完全限定名。

举例，我们定义一个很简单的ClassTest类，来看一下常量池是怎么对类的完全限定名进行存储的。

```java
package com.jvm;
import  java.util.Date;
public class ClassTest {
	private Date date =new Date();
}
```
将Java源码编译成ClassTest.class文件后，在此文件的目录下执行 javap -v ClassTest 命令，会看到如下的常量池信息的轮廓：
<div align="center"> <img src="/images/Java虚拟机原理图解22.png"/> </div><br>

如上图所示，在ClassTest.class文件的常量池中，共有 3 个CONSTANT_Class_info结构体，分别表示ClassTest 中用到的Class信息。
 我们就看其中一个表示com/jvm.ClassTest的CONSTANT_Class_info 结构体。它在常量池中的位置是#1，它的name_index值为#2，
 它指向了常量池的第2 个常量池项，如下所示:
<div align="center"> <img src="/images/Java虚拟机原理图解23.png"/> </div><br>
 
注意：
对于某个类而言，其class文件中至少要有两个CONSTANT_Class_info常量池项，用来表示自己的类信息和其父类信息。
(除了java.lang.Object类除外，其他的任何类都会默认继承自java.lang.Object）如果类声明实现了某些接口，
那么接口的信息也会生成对应的CONSTANT_Class_info常量池项。

除此之外，如果在类中使用到了其他的类，只有真正使用到了相应的类，JDK编译器才会将类的信息组成CONSTANT_Class_info常量池项放置到常量池中。
如下图：

```java
package com.louis.jvm;
 
import java.util.Date;
 
public  class Other{
	private Date date;
	
	public Other()
	{
		Date da;
	}
}
```
上述的Other的类，在JDK将其编译成class文件时，常量池中并没有java.util.Date对应的CONSTANT_Class_info常量池项，为什么呢?
在Other类中虽然定义了Date类型的两个变量date、da，但是JDK编译的时候，认为你只是声明了“Ljava/util/Date”类型的变量，
并没有实际使用到Ljava/util/Date类。将类信息放置到常量池中的目的，是为了在后续的代码中有可能会反复用到它。
很显然，JDK在编译Other类的时候，会解析到Date类有没有用到，发现该类在代码中就没有用到过，所以就认为没有必要将它的信息放置到常量池中了。

将上述的Other类改写一下，仅使用new Date()，如下图所示：
<div align="center"> <img src="/images/Java虚拟机原理图解24.png"/> </div><br>

```java
package com.louis.jvm;
 
import java.util.Date;
 
public  class Other{
	public Other()
	{
		new Date();
	}
}
```
这时候使用javap -v Other ，可以查看到常量池中有表示java/util/Date的常量池项：

总结：
1.对于某个类或接口而言，其自身、父类和继承或实现的接口的信息会被直接组装成CONSTANT_Class_info常量池项放置到常量池中；  
2. 类中或接口中使用到了其他的类，只有在类中实际使用到了该类时，该类的信息才会在常量池中有对应的CONSTANT_Class_info常量池项；
3. 类中或接口中仅仅定义某种类型的变量，JDK只会将变量的类型描述信息以UTF-8字符串组成CONSTANT_Utf8_info常量池项放置到常量池中，上面在类中的private Date date;JDK编译器只会将表示date的数据类型的“Ljava/util/Date”字符串放置到常量池中。

## 2.9、类中引用到的field字段在常量池中是怎样描述的？(CONSTANT_Fieldref_info, CONSTANT_Name_Type_info)
一般而言，我们在定义类的过程中会定义一些 field 字段，然后会在这个类的其他地方（如方法中）使用到它。
有可能我们在类的方法中只使用field字段一次，也有可能我们会在类定义的方法中使用它很多很多次。
举一个简单的例子，我们定一个叫Person的简单java bean，它有name和age两个field字段，如下所示：

```java
package com.louis.jvm;
 
public class Person {
 
	private String name;
	private int age;
	
	public String getName() {
		return name;
	}
	
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	
	public void setAge(int age) {
		this.age = age;
	}
}
```
 
在上面定义的类中，我们在Person类中的一系列方法里，多次引用到namefield字段 和agefield字段，
对于JVM编译器而言，name和age只是一个符号而已，并且它在由于它可能会在此类中重复出现多次，所以JVM把它当作常量来看待，
将name和age以field字段常量的形式保存到常量池中。
将它name和age封装成 [CONSTANT_Fieldref_info] 常量池项，放到常量池中，在类中引用到它的地方，直接放置一个指向field字段所在常量池的索引。
上面的Person类，使用javap -v Person指令，查看class文件的信息，你会看到，在Person类中引用到age和namefield字段的地方，
都是指向了常量池中age和namefield字段对应的常量池项中。表示field字段的常量池项叫做CONSTANT_Fieldref_info。
<div align="center"> <img src="/images/Java虚拟机原理图解25.png"/> </div><br>

怎样描述某一个field字段的引用？
<div align="center"> <img src="/images/Java虚拟机原理图解26.png"/> </div><br>
<div align="center"> <img src="/images/Java虚拟机原理图解27.png"/> </div><br>
<div align="center"> <img src="/images/Java虚拟机原理图解28.png"/> </div><br>
<div align="center"> <img src="/images/Java虚拟机原理图解29.png"/> </div><br>


实例解析： 现在，让我们来看一下Person类中定义的namefield字段在常量池中的表示。通过使用javap -v Person会查看到如下的常量池信息：
<div align="center"> <img src="/images/Java虚拟机原理图解30.png"/> </div><br>
<div align="center"> <img src="/images/Java虚拟机原理图解31.png"/> </div><br>

请读者看上图中namefield字段的数据类型，它在#6个常量池项，以UTF-8编码格式的字符串“Ljava/lang/String;” 
表示，这表示着这个field 字段是java.lang.String 类型的。关于field字段的数据类型，class文件中存储的方式和我们在源码中声明的有些不一样。
请看下图的对应关系：
<div align="center"> <img src="/images/Java虚拟机原理图解32.png"/> </div><br>

[注意]
如果我们在类中定义了field 字段，但是没有在类中的其他地方用到这些字段，它是不会被编译器放到常量池中的。
读者可以自己试一下。（当然了，定义了但是没有在类中的其它地方引用到这种情况很少。）
[只有在类中的其他地方引用到了，才会将他放到常量池中。]




## 2.10、类中引用到的method方法在常量池中是怎样描述的？(CONSTANT_Methodref_info, CONSTANT_Name_Type_info)
1.举例：
还是以Person类为例。在Person类中，我们定义了setName(String name)、getName()、setAge(int age)、getAge()这些方法：   

```java
package com.louis.jvm;
 
public class Person {
 
	private String name;
	private int age;
	
	public String getName() {
		return name;
	}
	
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	
	public void setAge(int age) {
		this.age = age;
	}
	
}
```
虽然我们定义了方法，但是这些方法没有在类总的其他地方被用到（即没有在类中其他的方法中引用到），所以它们的方法引用信息并不会放到常量中。
现在我们在类中加一个方法 getInfo()，调用了getName()和getAge() 方法：

	public String getInfo()
	{
		
		return getName()+"\t"+getAge();
	}

这时候JVM编译器会将getName()和getAge()方法的引用信息包装成CONSTANT_Methodref_info结构体放入到常量池之中。
<div align="center"> <img src="/images/Java虚拟机原理图解33.png"/> </div><br>

这里的方法调用的方式牵涉到Java非常重要的一个术语和机制，叫动态绑定。这个动态绑定问题以后在单独谈谈。

2.  怎样表示一个方法引用？
请看下图：
<div align="center"> <img src="/images/Java虚拟机原理图解34.png"/> </div><br>
<div align="center"> <img src="/images/Java虚拟机原理图解35.png"/> </div><br>
<div align="center"> <img src="/images/Java虚拟机原理图解36.png"/> </div><br>

3.  方法描述符的组成
<div align="center"> <img src="/images/Java虚拟机原理图解37.png"/> </div><br>
<div align="center"> <img src="/images/Java虚拟机原理图解38.png"/> </div><br>
 

4.  getName() 方法引用在常量池中的表示
<div align="center"> <img src="/images/Java虚拟机原理图解39.png"/> </div><br>




## 2.11、类中引用到某个接口中定义的method方法在常量池中是怎样描述的？(CONSTANT_InterfaceMethodref_info, CONSTANT_Name_Type_info)
当我们在某个类中使用到了某个接口中的方法，JVM会将用到的接口中的方法信息方知道这个类的常量池中。
比如我们定义了一个Worker接口，和一个Boss类，在Boss类中调用了Worker接口中的方法，这时候在Boss类的常量池中会有Worker接口的方法的引用表示。

```java
package com.louis.jvm;
 
/**
 * Worker 接口类
 * @author luan louis
 */
public interface Worker{
	
	public void work();
 
}
package com.louis.jvm;
 
/**
 * Boss 类，makeMoney()方法 调用Worker 接口的work
 * @author louluan
 */
public class Boss {
	
	public void makeMoney(Worker worker)
	{
		worker.work();
	}
 
}
```
我们对Boss.class执行javap -v  Boss,然后会看到如下信息：
<div align="center"> <img src="/images/Java虚拟机原理图解40.png"/> </div><br>

如上图所示，在Boss类的makeMoney()方法中调用了Worker接口的work()方法，机器指令是通过invokeinterface指令完成的，
invokeinterface指令后面的操作数，是指向了Boss常量池中Worker接口的work()方法描述，表示的意思就是：“我要调用Worker接口的work()方法”。

Worker接口的work()方法引用信息，JVM会使用CONSTANT_InterfaceMethodref_info结构体来描述，CONSTANT_InterfaceMethodref_info定义如下：
<div align="center"> <img src="/images/Java虚拟机原理图解41.png"/> </div><br>

CONSTANT_InterfaceMethodref_info结构体和上面介绍的CONSTANT_Methodref_info 结构体很基本上相同，它们的不同点只有：
1.CONSTANT_InterfaceMethodref_info 的tag 值为11，而CONSTANT_Methodref_info的tag值为10；
2. CONSTANT_InterfaceMethodref_info 描述的是接口中定义的方法，而CONSTANT_Methodref_info描述的是实例类中的方法；


小试牛刀
关于方法的描述,完全相同CONSTANT_InterfaceMethodref_info和上述的CONSTANT_Methodref_info 结构体完全一致，
这里就不单独为CONSTANT_InterfaceMethodref_info绘制结构图了，请读者依照CONSTANT_Methodref_info的描述，
结合本例子关于Worker接口和Boss类的关系，使用javap -v Boss,查看常量池信息，然后根据常量池信息，自己动手绘制work() 方法在常量池中的结构。


 

## 2.12、CONSTANT_MethodType_info，CONSTANT_MethodHandle_info，CONSTANT_InvokeDynamic_info
如果你从我的《常量池详解》NO1节看到了NO11节，那么恭喜你，你已经学会了几乎所有的常量池项！只要你掌握了上述的常量池项，
你就可以读懂你平常所见到的任何一个class文件的常量池了。
至于NO12所列出来的三项：CONSTANT_MethodType_info,CONSTANT_MethodHandle_info,CONSTANT_InvokeDynamic_info，我想对你说，暂时先不管它吧。
这三项主要是为了让Java语言支持动态语言特性而在Java 7 版本中新增的三个常量池项，只会在极其特别的情况能用到它，
在class文件中几乎不会生成这三个常量池项。   其实我花了一些时间来研究这三项，并且想通过各种方式生成这三项，不过没有成功，
最后搞的还是迷迷糊糊的。从我了解到的信息来看，Java 7对动态语言的支持很笨拙，并且当前没有什么应用价值，然后就对着三项的研究先放一放了。

 
# 3、class文件中的访问标志、类索引、父类索引、接口索引集合
讲完了class文件中的常量池，我们就相当于克服了class文件中最麻烦的模块了。现在，我们来看一下class文件中紧接着常量池后面的几个东西：
访问标志、类索引、父类索引、接口索引集合。

## 3.1. 访问标志、类索引、父类索引、接口索引集合 在class文件中的位置
<div align="center"> <img src="/images/Java虚拟机原理图解42.png"/> </div><br>


## 3.2. 访问标志(access_flags)能够表示什么？
访问标志（access_flags）紧接着常量池后，占有两个字节，总共16位，如下图所示：
<div align="center"> <img src="/images/Java虚拟机原理图解43.png"/> </div><br>

当JVM在编译某个类或者接口的源代码时，JVM会解析出这个类或者接口的访问标志信息，然后，
将这些标志设置到访问标志（access_flags）这16个位上。JVM会考虑如下设置如下访问表示信息：

a. 我们知道，每个定义的类或者接口都会生成class文件（这里也包括内部类，在某个类中定义的静态内部类也会单独生成一个class文件）。
对于定义的类，JVM在将其编译成class文件时，会将class文件的访问标志的第11位设置为1 。第11位叫做ACC_SUPER标志位；
对于定义的接口，JVM在将其编译成class文件时，会将class文件的访问标志的第8位 设置为 1 。第8位叫做ACC_INTERFACE标志位；

b. class文件表示的类或者接口的访问权限有public类型的和包package类型的。
如果类或者接口被声明为public类型的，那么，JVM将其编译成class文件时，会将class文件的访问标志的第16位设置为1 。第16位叫做ACC_PUBLIC标志符；

c. 类是否为抽象类型的，即我们定义的类有没有被abstract关键字修饰，即我们定义的类是否为抽象类。
如果我们形如：
public  abstract  class MyClass{......} 
定义某个类时，JVM将它编译成class文件的时候，会将class文件的访问标志的第7位设置为1 。
第7位叫做ACC_ABSTRACT标志位。 另外值得注意的是，对于定义的接口，JVM在编译接口的时候也会对class文件的访问标志上的ACC_ABSTRACT标志位设置为 1；

d. 该类是否被声明了final类型,即表示该类不能被继承。
此时JVM会在编译class文件的过程中，会将class文件的访问标志的第12位设置为 1 。第12位叫做ACC_FINAL标志位；

e.如果我们这个class文件不是JVM通过java源代码文件编译而成的，而是用户自己通过class文件的组织规则生成的，
那么，一般会对class文件的访问标志第4位设置为 1 。通过JVM编译源代码产生的class文件此标志位为 0，第4位叫做ACC_SYNTHETIC标志位；

f. 枚举类，对于定义的枚举类如：public enum EnumTest{....}，JVM也会对此枚举类编译成class文件，这时，对于这样的class文件，
JVM会对访问标志第2位设置为 1 ，以表示它是枚举类。第2位叫做ACC_ENUM标志位；

g. 注解类，对于定义的注解类如：public @interface{.....},JVM会对此注解类编译成class文件，对于这样的class文件，
JVM会将访问标志第3位设置为1，以表示这是个注解类，第3位叫做ACC_ANNOTATION标志位。

当JVM确定了上述标志位的值后，就可以确定访问标志（[access_flags]）的值了。实际上JVM上述标志会根据上述确定的标志位的值，
对这些标志位的值取或，便得到了访问标志（access_flags）。如下图所示:
<div align="center"> <img src="/images/Java虚拟机原理图解44.png"/> </div><br>


举例：定义一个最简单的类Simple.java，使用编译器编译成class文件，然后观察class文件中的访问标志的值，以及使用javap -v Simple 查看访问标志。

 package com.louis.jvm;
 
public class Simple {
 
}
使用UltraEdit查看编译成的class文件，如下图所示：
<div align="center"> <img src="/images/Java虚拟机原理图解45.png"/> </div><br>

上述的图中黄色部分表示的是常量池部分,常量池后面紧跟着就是访问标志，它的十六进制值为0x0021,二进制的值为：00000000 00100001，
由二进制的1的位数可以得出第11、16位为1，分别对应ACC_SUPER标志位和ACC_PUBLIC标志位。
也可以通过一下运算：
 0x0021 = 0x0001 | 0x0020,  即：   访问标志表示的标志是ACC_PUBLIC + ACC_SUPER

为了验证我们的运算，使用javap -v Simple查看反编译信息如下：
（小技巧：使用javap -v Simple指令的结果展示在命令提示符下显示不友好，一般我是使用javap -v Simple > temp.txt，将结果重定向到文件中，然后查看文件）
<div align="center"> <img src="/images/Java虚拟机原理图解46.png"/> </div><br>


 

## 3.3. 类索引(this_class)是什么？
我们知道一般情况下一个Java类源文件经过JVM编译会生成一个class文件，也有可能一个Java类源文件中定义了其他类或者内部类，
这样编译出来的class文件就不止一个，但每一个class文件表示某一个类，至于这个class表示哪一个类，便可以通过 [类索引] 这个数据项来确定。

JVM通过类的完全限定名确定是某一个类。
类索引的作用，就是为了指出class文件所描述的这个类叫什么名字。
类索引紧接着访问标志的后面，占有两个字节，在这两个字节中存储的值是一个指向常量池的一个索引，该索引指向的是CONSTANT_Class_info常量池项，
<div align="center"> <img src="/images/Java虚拟机原理图解47.png"/> </div><br>
          
以上面定义的Simple.class 为例，如下图所示，查看他的类索引在什么位置和取什么值。
<div align="center"> <img src="/images/Java虚拟机原理图解48.png"/> </div><br>
          
由上可知，它的类索引值为0x0001,那么，它指向了常量池中的第一个常量池项，那我们再看一下常量池中的信息。使用javap -v Simple,常量池中有以下信息：
<div align="center"> <img src="/images/Java虚拟机原理图解49.png"/> </div><br>
         
可以看到常量池中的第一项是CONSTANT_Class_info项，它表示一个"com/louis/jvm/Simple"的类名。即类索引是告诉我们这个class文件所表示的是哪一个类。
<div align="center"> <img src="/images/Java虚拟机原理图解50.png"/> </div><br>


## 3.4、父类索引(super_class)是什么？
Java支持单继承模式，除了java.lang.Object 类除外，每一个类都会有且只有一个父类。class文件中紧接着类索引(this_class)之后的两个字节区域表示父类索引，
跟类索引一样，父类索引这两个字节中的值指向了常量池中的某个常量池项CONSTANT_Class_info，表示该class表示的类是继承自哪一个类。
<div align="center"> <img src="/images/Java虚拟机原理图解51.png"/> </div><br>



## 3.5、接口索引集合(interfaces)是什么？
一个类可以不实现任何接口，也可以实现很多个接口，为了表示当前类实现的接口信息，class文件使用了如下结构体描述某个类的接口实现信息:
<div align="center"> <img src="/images/Java虚拟机原理图解52.png"/> </div><br>

由于类实现的接口数目不确定，所以接口索引集合的描述的前部分叫做接口计数器（interfaces_count），
接口计数器占用两个字节，其中的值表示着这个类实现了多少个接口，紧跟着接口计数器的部分就是接口索引部分了，
每一个接口索引占有两个字节，接口计数器的值代表着后面跟着的接口索引的个数。接口索引和类索引和父类索引一样，
其内的值存储的是指向了常量池中的常量池项的索引，表示着这个接口的完全限定名。

举例：定义一个Worker接口，然后类Programmer实现这个Worker接口，然后我们观察Programmer的接口索引集合是怎样表示的。

/**
 * Worker 接口类
 * @author luan louis
 */
public interface Worker{
	
	public void work();
 
}
package com.louis.jvm;
 
public class Programmer implements Worker {
 
	@Override
	public void work() {
		System.out.println("I'm Programmer,Just coding....");
	}
}

<div align="center"> <img src="/images/Java虚拟机原理图解53.png"/> </div><br>


# 4 、class文件中的字段表集合--field字段在class文件中是怎样组织的



 
# 5、 class文件中的方法表集合--method方法在class文件中是怎样组织的





# 6、JVM机器指令集








 










