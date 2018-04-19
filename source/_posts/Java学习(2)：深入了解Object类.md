---
title: Java学习(2)：深入了解Object类
date: 2018-03-05 11:04:27
tags:
- Java
- 类
categories: Java基础

---
## 1.关于Object类
Object类是所有类的超类，是java中唯一没有父类的类，如果没有明确地指出超类，Object就被认为是这个类的超类

---
## 2.Object类的常用方法
1. equals()方法:
判断是否相等。
		public boolean equals(Object obj) {
	        return (this == obj);
	    }
Object的equals实现其实就是 ==, 判断的其实是地址,而String的equals比较的是值，说明它对equals进行了重写。
这是String的equals()方法源码：
		public boolean equals(Object anObject) {  
	       if (this == anObject) {  
	           return true;  
	       }  
	       if (anObject instanceof String) {  
	           String anotherString = (String)anObject;  
	           int n = value.length;  
	           if (n == anotherString.value.length) {  
	               char v1[] = value;  
	               char v2[] = anotherString.value;  
	               int i = 0;  
	               while (n-- != 0) {  
	                   if (v1[i] != v2[i])  
	                       return false;  
	                   i++;  
	               }  
	               return true;  
	           }  
	       }  
	       return false;  
	   }
需要使用equals()方法判断值时，要对它进行重写。
2. hashCode()方法:
**返回该对象的哈希码。**
		public native int hashCode();
[hashCode](http://baike.sogou.com/v72625696.htm?fromTitle=hashcode)是jdk根据对象的地址 或者字符串或者数字算 出来的int类型哈希码的数值 值。支持此方法是为了提高哈希表（例如java.util.Hashtable提供的哈希表）的性能。
**协定**:
a.在 Java 应用程序执行期间，在对同一对象多次调用 hashCode 方法时，必须一致地返回相同的整数，前提是将对象进行 equals 比较时所用的信息没有被修改。
b.如果根据 equals(Object) 方法，两个对象是相等的，那么对这两个对象中的每个对象调用 hashCode 方法都必须生成相同的整数结果，注：这里说的equals(Object) 方法是指Object类中未被子类重写过的equals方法。
c.如果根据equals(java.lang.Object)方法，两个对象不相等，那么对这两个对象中的任一对象上调用 hashCode 方法**不一定生成不同的整数结果**。但是，程序员应该意识到，为不相等的对象生成不同整数结果可以提高哈希表的性能。
3. toString()方法:
**返回该对象的字符串表示，很重要。**
		public String toString() {  
	        return getClass().getName() + "@" + Integer.toHexString(hashCode());  
	    }
getClass().getName():获取字节码文件的对应全路径名例如java.lang.Object
Integer.toHexString(hashCode()):将哈希值转成16进制数格式的字符串。
如代码:
		public static void main(String[] args) {  
		    System.out.println(new Object().toString());  
		}  
返回值为:
		java.lang.Object@15db9742
一般的JavaBean为了方便都会重写toString方法。因为只要对象与一个字符串通过操作符“+”连接起来，Java编译器就会自动地调用toString方法，以便获得这个对象的字符串描述

---
## 2.线程相关的方法
1. wait()方法
当前线程必须拥有此对象监视器。**该线程发布对此监视器的所有权并等待**，直到其他线程通过调用 notify 方法，或 notifyAll 方法通知在此对象的监视器上等待的线程醒来。
		public final native void wait(long timeout) throws InterruptedException; 
		public final void wait() throws InterruptedException {     
			wait(0);
		}
		//更加精确的等待
		public final void wait(long timeout, int nanos) throws InterruptedException { 
			//微秒数大于0
	        if (timeout < 0) {
	            throw new IllegalArgumentException("timeout value is negative");     
	        }
			//大于1纳秒少于一微秒
	        if (nanos < 0 || nanos > 999999) {     
	            throw new IllegalArgumentException(     
	                "nanosecond timeout value out of range");     
	        }     
		    //大于500000纳秒进1毫秒
		    if (nanos >= 500000 || (nanos != 0 && timeout == 0)) {
		        timeout++;
		    }
			wait(timeout);
		}
2. notify：
**唤醒正在等待的线程**:
	/*唤醒在此对象监视器上等待的单个线程。*/    
    public final native void notify();     
	/*唤醒在此对象监视器上等待的所有线程。*/    
    public final native void notifyAll(); 

---
## 3.clone()方法
1. clone方法是**用来复制一个对象**。不同于“=”。
对于值类型的数据是可以通过“=”来实现复制的。但是对于引用类型的对象，“=”只能复制其内存地址，使对象的引用指向同一个对象，而不会创建新的对象。clone则可以创建与原来对象相同的对象。非常有用。
		protected native Object clone() throws CloneNotSupportedException; 
2. 简单使用：
		Object a;
		b=a.clone();
function(a)a作为参数进行操作，会改变a对象的内容。
function(b)对b进行操作a对象不变。
相当于在内存中又复制出一个 a对象。不会改变原先a 对象。
扩展：[自定义类的克隆](https://www.cnblogs.com/acode/p/6306887.html)

---
## 4.getClass()方法
** 返回此 Object 的运行时类。** 
`public final native Class<?> getClass();`
如代码：
```
	public static void main(String[] args) {
		System.out.println(new Object().getClass());
	}
```
返回值为：
`class java.lang.Object`

---
## 5.finalize()方法与registerNatives()方法
1. finalize()方法:
**子类覆盖 finalize() 方法以整理系统资源或者执行其他清理工作。finalize() 方法是在垃圾收集器删除对象之前对这个对象调用的。**
		 void finalize() throws Throwable { }
扩展:[Java finalize方法使用](http://blog.csdn.net/carolzhang8406/article/details/6705831)
2. registerNatives()方法:
一个本地方法,具体是用C(C++)在DLL中实现的,然后通过[JNI](https://baike.baidu.com/item/JNI/9412164?fr=aladdin)调用:
源代码:
	    private static native void registerNatives();
	    /*对象初始化时自动调用此方法*/
	    static {
	        registerNatives();
	    }

---
## 5.native本地方法
一个Native Method是这样一个java的方法：该方法的实现由非java语言实现，比如C。这个特征并非java所特有，很多其它的编程语言都有这一机制。

---
## 6.Object类源代码

	package java.lang;
	public class Object {
	    /*一个本地方法,具体是用C(C++)在DLL中实现的,然后通过JNI调用*/
	    private static native void registerNatives();
	    /*对象初始化时自动调用此方法*/
	    static {
	        registerNatives();
	    }
	    /*返回此Object的运行时类*/
	    public final native Class<?> getClass();
	    /*
	    hashCode的常规协定是：
	    1.在java应用程序执行期间,在对同一对象多次调用hashCode()方法时,必须一致地返回相同的整数,前提是将对象进行equals比较时所用的信息没有被修改。从某一应用程序的一次执行到同一应用程序的另一次执行,该整数无需保持一致。
	    2.如果根据equals(object)方法,两个对象是相等的,那么对这两个对象中的每个对象调用hashCode方法都必须生成相同的整数结果。
	    3.如果根据equals(java.lang.Object)方法,两个对象不相等,那么对这两个对象中的任一对象上调用hashCode()方法不要求一定生成不同的整数结果。但是,程序员应该意识到,为不相等的对象生成不同整数结果可以提高哈希表的性能。
	    */
	    public native int hashCode();
	
	    /*这里比较的是对象的内存地址,跟String.equals方法不同,它比较的只是对象的值*/
	    public boolean equals(Object obj) {
	        return (this == obj);
	    }
	    /*本地clone方法,用于对象的复制*/
	    protected native Object clone() throws CloneNotSupportedException;
	    /*
	    返回该对象的字符串表示,非常重要的方法
	    getClass().getName();获取字节码文件的对应全路径名例如java.lang.Object
	    Integer.toHexString(hashCode());将哈希值转成16进制数格式的字符串。
	    */
	    public String toString(){
	        return getClass().getName() + "@" + Integer.toHexString(hashCode());
	    }
	    /*唤醒在此对象监视器上等待的单个线程*/
	    public final native void notity();
	    /*唤醒在此对象监视器上等待的所有线程*/
	    public final native void notifyAll();
	    /*
	    在其他线程调用此对象的notify()方法或notifyAll()方法前,导致当前线程等待。换句话说,此方法的行为就好像它仅执行wait(0)调用一样。
	    当前线程必须拥有此对象监视器。该线程发布对此监视器的所有权并等待,直到其他线程通过调用notify方法或notifyAll方法通知在此对象的监视器上等待的线程醒来,然后该线程将等到重新获得对监视器的所有权后才能继续执行。
	    */
	    public final void wait() throws InterruptedException(){
	        wait(0);
	    }
	    /*在其他线程调用此对象的notify()方法或notifyAll()方法,或者超过指定的时间量前,导致当前线程等待*/
	    public final native void wait(long timeout) throws InterruptedException;
	    public final void wait(long timeout, int nanos) throws InterruptedException {
	        if(timeout < 0){
	            throw new IllegalArgumentException("timeout value is negative");
	        }
	        //nanos不能大于等于1000000ms也就是1000s
	        if(nanos < 0 || nanos > 999999){
	            throw new IllegalArgumentException("nanosecond timeout value out of range");
	        }
	        //
	        if(nanos >= 500000 || (nanos != 0 && timeout == 0)){
	            timeout++;
	        }
	        wait(timeout);
	    }
	
	    /*当垃圾回收期确定不存在对该对象的更多引用时,由对象的垃圾回收器调用此方法。*/
	    protected void finalize() throws Throwable{}
	}

---
## 7.参考博客
[1].[java中Object类 源代码详解](http://blog.csdn.net/u014344668/article/details/78894298)
[2].[java Object类常用方法浅解](http://blog.csdn.net/xu511739113/article/details/52328727)

---