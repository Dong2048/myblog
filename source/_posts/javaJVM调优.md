---
title: java虚拟机介绍
date: 2021-11-15 09:18:46
tags: JDK
category: 笔记
top_img: https://s3.bmp.ovh/imgs/2021/12/816a5a30f6cee438.jpg
cover: https://s3.bmp.ovh/imgs/2021/12/816a5a30f6cee438.jpg
---
### jdk基本说明
#### 1、jdk由如下内容组成：
1、java命令：javac、javap、java、java db等命令。
2、JRE支撑java程序运行的环境，包括类库、javaJVM虚拟机。
### java虚拟机基本说明
#### 2、JVM(虚拟机)介绍
JVM是指用软件的方式模拟具有完整硬件系统功能、运行在一个完全隔离环境中的完整计算机系统，是物理机的软件实现。
#### 3、Helloworld执行流程
> Helloworld.java
> ⭣
> javac
> ⭣
> Helloworld.calss
> ⭣
> java
> ⭣
> JVN 「从软件层面屏蔽不同操作系统在底层硬件与指令上的区别」
> ⭣
> 「Windows」windows机器码 or 「Linux」linux机器码
### 4、Math类
```
public class Math{
	public static final int initdata = 666;
	public static User user =new User();
	
	public int compute{ //一个方法对应一块栈帧内存区域
		int a =1;
		int b=2;
		int c=(a - b) * 10;
		return c;
	}
	
	public static void main(String[] args){
		Math math=new Math();
		math.compute();
	}
}
```
![](https://preview.cloud.189.cn/image/imageAction?param=654E72CBF5F80CFCCCEB6C3CF76A753D6617BE65C27CDF426F8A10EFDD629F857C93BF8AD4828B529ED65339A02C7ADEFB99A88A961A89593B372AB03BB63CED4DFAE6A89154C9DA43E826745286C62EBE4215F3469A117A2DA32260C296156B09DD08473ABFE722206FA73F5470897C731BB29E)
### 4、JVM虚拟机组成
JVM虚拟机分为：类装载子系统、运行时数据区（内存区域）、字节码执行引擎，三部分组成。
当执行math类方法时「类装载子系统」将字节码文件加载到「内存区域」里,「字节码执行引擎」执行「内存区域」中**方法区**代码。
#### 运行时数据区（内存区域）
1、堆：存放对象  
分为年轻代（Edan区、s区0、s区1）老年代。
2、方法区（元空间）存放常量、静态变量、类信息。
3、栈（线程）
栈主要存储**局部变量**
只要一个线程开始运行，JVM虚拟机就会给此线程分配一个线程栈。
运行几个线程就在栈中分配几个内存给线程栈使用。
当线程运行方法时，在线程栈中分配一个内存空间存储此方法的局部变量，此内容空间称为栈帧内存空间。
栈主要组成部分：局部变量、操作数栈、动态链接、方法出口。
	（1）局部变量
	（2）操作数栈  
		 赋值局部变量、对局部变量进行计算时临时分配的空间。
	（3）动态链接   
		 存储指向运行时常量池中该栈帧所属方法的引用。
	（4）方法出口
		 方法返回的方式：正常返回、异常返回。

栈数据结构“先进后出”与程序嵌套调用的逻辑顺序相匹配。
4、本地方法栈
	Navtive 方法是 Java 通过 JNI 直接调用本地 C/C++ 库。  
    当线程调用 Java 方法时，虚拟机会创建一个栈帧并压入 Java 虚拟机栈。然而当它调用的是 native 方法时，虚拟机会保持 Java 虚拟机栈不变，也不会向 Java 虚拟机栈中压入新的栈帧，虚拟机只是简单地动态连接并直接调用指定的 native 方法。
5、程序计数器
程序计数器是每一个线程都有，存储要执行代码在方法区的（行号）内存地址。
为什么使用程序计数器，java是多线程运行，当权限级别更高的线程占用了CPU，当前线程被挂起，权限级别告的线程运行结束后，继续执行当前线程，通过程序计数器定位从哪里继续执行，而不是从头开始执行。
### java虚拟机调优
#### 1、Arthas    
>（1）dashboard 查看系统运行状态。
>（2）thread [进程号]查看问题进程代码。
>（3）jad [方法类url，如：com.xxx.xxxService] 查看线上运行代码




