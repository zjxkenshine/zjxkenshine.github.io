---
title: 《Java多线程编程核心技术》笔记（六）：单例模式与多线程
date: 2018-03-05 23:23:35
tags:
- Java
- 多线程
categories: 学习笔记

---
## 0.学习要点
- 简单了解单例模式
- 如何使单例模式遇到多线程是安全的、正确的。

---
## 1.立即加载/"饿汉模式"
1. [单例模式](https://baike.baidu.com/item/%E5%8D%95%E4%BE%8B%E6%A8%A1%E5%BC%8F/5946627?fr=aladdin):
**一个类只有一个对象实例。**
2. 关于立即加载/饿汉模式：
*立即加载就是使用类的时候已经将对象创建完毕了。在调用方法前，实例已经被创建了。*
3. 测试代码:
单例类:
		public class Object6_02 {
			//立即加载/饿汉模式
			private static Object6_02 obj=new Object6_02();     //提前加载
			private Object6_02() {
			}
			public static Object6_02 getInstance(){
				//缺点是不能有其他实例变量，因为getInstance方法没有同步，可能会出现非线程安全
				return obj;
			}
		}
在线程类中使用`Object6_02.getInstance().hashCode()`输出对象的hashCode。
在测试类中实例化多个线程对象，并start()，得到的结果是同一个hashCode,说明是只有一个对象。

---
## 2.延迟加载/"懒汉模式"简介及其缺点
1. 延时加载/懒汉模式:
*在调用get方法时实例才被创建，常见的实现方法是在get()中new实例化。*
2. 延迟加载（会出错）:
		public class Object6_03 {
			//延迟加载/懒汉模式解析
			public static Object6_03 obj;
			public Object6_03() {
			}
			
			public static Object6_03 getInstance(){
				//延迟加载
				if(obj!=null){
				}else{
					obj=new Object6_03();
				}
				return obj;
			}
		}
在多线程环境下使用`Object6_02.getInstance().hashCode()`创建对象并打印hashCode会出现非线程安全，会取出多个实例对象。
实测在`public static Object6_03 obj；`前加volatile关键字无法解决问题。

---
## 3.延迟加载/懒汉模式缺点的三种解决方法
1. 声明synchronize关键字：
		public class Object6_05 {
			//延迟加载/懒汉模式的缺点的解决方法1：声明synchronize关键字
			private static Object6_05 obj;
			public Object6_05() {
			}
			//声明synchronize关键字
			synchronized public static Object6_05 getInstance(){
				try{
					if(obj!=null){
					}else{
						//模拟创建对象之前做一些准备工作
						Thread.sleep(3000);
						obj=new Object6_05();
					}
				}catch (InterruptedException e) {
					e.printStackTrace();
				}
				return obj;
			}
		}
不足：效率非常低下，同步运行，下一个线程想要取得对象，必须等上一个线程释放锁之后才可以执行
2. synchronized同步代码块:
将getInstance方法更改如下:
		public static Object6_05 getInstance(){
			try{
				synchronized (Object6_05.class) {
					if(obj!=null){				
					}else{
						Thread.sleep(3000);  //创建实例化准备工作
						obj=new Object6_05();
					}
				}
			}catch (InterruptedException e) {	
				e.printStackTrace();
			}
			return obj;
		}
但是仍然会有三秒的开支。
3. 针对某些重要代码进行同步:
		if(obj!=null){				
		}else{
			Thread.sleep(3000);
			synchronized (Object6_05.class) {
				obj=new Object6_05();
			}
		}
但是遇到多线程还是会出现多个对象。没有解决根本问题。
4. 最终方案:**DCL双检查锁机制**
*双检查锁就是在同步代码块调用之前检查一遍，再在同步代码块内部再检查一遍。*
代码如下:
		public class Object6_08 {
			//延迟加载/懒汉模式的缺点的解决方法4：DCL双检查锁机制(Double-Check Locking)
			private volatile static Object6_08 obj;
			private Object6_08() {
			}
			public static Object6_08 getInstance() {
				try{
					if(obj!=null){
						
					}else{
						Thread.sleep(3000);
						synchronized (Object6_08.class) {  //一重锁
							if(obj==null){  //二重检查
								obj=new Object6_08();
							}
						}
					}
				}catch (InterruptedException e) {
					e.printStackTrace();
				}
				return obj;
			}
		}

---
## 4.使用其他方法实现安全的单例模式
### 4.1 使用静态内置类实现单例模式

		public class Object{
			//使用静态内置类实现单例模式
			//静态内部类ObjectHandler
			private static class ObjectHandler{
				private static Object obj=new Object();
			}
			
			public Object() {
			}
			public static Object getInstance(){
				return ObjectHandler.obj;
			}
		}
### 4.2 序列化与反序列化时的单例模式实现
[序列化](https://baike.baidu.com/item/%E5%BA%8F%E5%88%97%E5%8C%96/2890184):*将对象的状态信息转换为可以存储或传输的形式的过程。*
4.1的代码在遇到序列化的对象时仍会出现不同对象。输出与读取的对象不同。
解决方法:
在反序列化中加上readResolve()方法。
单例类代码:

	public class Object6_10 implements Serializable{
		//序列化与反序列化的单例模式实现
		private static final long serialVersionUID=4448L;
		//内部类方法
		private static class Object6_10Handler{
			private static final Object6_10 obj=new Object6_10();
		}
		
		public Object6_10() {
		}
		//第一次实例化时要调用该方法
		public static Object6_10 getInstance(){
			System.out.println("getInstance"+Object6_10Handler.obj.hashCode());
			return Object6_10Handler.obj;
		}
		//readResolve反序列化时自动调用
		protected Object readResolve() throws ObjectStreamException{
			System.out.println("调用了readResolve()方法");
			System.out.println("readResolve"+Object6_10Handler.obj.hashCode());
			return Object6_10Handler.obj;
		}
	}
测试类：

		public class Test6_10 {
			//序列化与反序列化的单例模式实现
			public static void main(String[] args) {
				try{
					Object6_10 obj=new Object6_10().getInstance();  //实例化时显式调用getInstance
					FileOutputStream fosRef=new FileOutputStream(new File("666.txt"));  //输出一个文件
					ObjectOutputStream oosRef=new ObjectOutputStream(fosRef);
					oosRef.writeObject(obj);  //向文件输出一个Object6_10类对象
					oosRef.close();
					fosRef.close();
		
					System.out.println(obj.hashCode());
					
					FileInputStream fisRef=new FileInputStream(new File("666.txt"));  //读取该文件
					ObjectInputStream oisRef=new ObjectInputStream(fisRef);
					Object6_10 obj1=(Object6_10) oisRef.readObject();  //从文件中读取Object6_10类对象
					oisRef.close();
					fisRef.close();
		
					System.out.println(obj1.hashCode());
				}catch (Exception e) {
					e.printStackTrace();
				}
			}
		}
运行结果:

		getInstance366712642
		366712642
		调用了readResolve()方法
		readResolve366712642
		366712642
更多关于序列化，反射等和单例模式的知识可以访问博客:
<https://www.cnblogs.com/ttylinux/p/6498822.html?utm_source=itdadao&utm_medium=referral>
### 4.3 使用static代码块实现

	public class Object6_11 {
		//使用static代码实现单例模式
		private static Object6_11 ins=null;
		public Object6_11() {
		}
		
		//static代码块
		static{
			ins=new Object6_11();
		}
		public static Object6_11 get(){
			return ins;
		}
	}
在线程中使用`Object6_11.get().hashCode()`输出，在多线程环境下可以实现单例模式。但在序列化与反序列化时仍要加readResolve()方法。

---
## 5.使用枚举类实现单例模式
1. 介绍:
**最被推荐使用的一种方式，在使用枚举类时构造方法会自动调用，自带了readResolve的解决方式**，但注意使用时不能违反“[职责单一原则](https://baike.baidu.com/item/%E5%8D%95%E4%B8%80%E8%81%8C%E8%B4%A3%E5%8E%9F%E5%88%99/9456515?fr=aladdin)”,不能将enum类暴露在外。
2. 一个例子，使用枚举实现单例模式--JDBC连接
		public class Object6_13 {
			//完善使用enum枚举数据类型实现单例模式
			//内部枚举类
			public enum mySingleton{
				connectionFactory;
				private Connection conn;
				private mySingleton(){
					try{
						System.out.println("调用了Object6_12的构造");
						String url="jdbc:mysql://localhost:3306/employee?characterEncoding=utf8&useSSL=true";
						String username="root";
						String PASSWORD = "******";
						String diver="com.mysql.jdbc.Driver";
						Class.forName(diver);
						conn=DriverManager.getConnection(url,username,PASSWORD);
					}catch (Exception e){
						e.printStackTrace();
					}
				}
				public Connection get(){
					return conn;
				}
			}
			public static Connection getcon(){
				return mySingleton.connectionFactory.get();
			}
		}
测试类:
		public class Test6_13 {
			//完善使用enum枚举数据类型实现单例模式
			public static void main(String[] args) {
				Thread6_13 t1=new Thread6_13();
				Thread6_13 t2=new Thread6_13();
				Thread6_13 t3=new Thread6_13();
				t1.start();
				t2.start();
				t3.start();
			}
		}

---