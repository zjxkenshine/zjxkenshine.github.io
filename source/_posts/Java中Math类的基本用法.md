---
title: Java中Math类的基本用法
date: 2018-03-02 11:06:55
tags:
- Java
- 类
categories: Java基础

---
## 0.重新学习
昨天刷Java题的时候有一道题考:
>Math.floor(-8.9)返回的什么值?

然后义无反顾地了选了-9，取整嘛，然后就懵逼了，取整函数返回值竟然是double类型，所以决定把Java的Math类(java.lang.Math)再学习测试一遍。(jdk1.8)

---
## 1.取整函数
1. 介绍:
(1).floor:地板，向下取整，**返回值为double**。
		 public static double floor(double a){...}
测试代码:
		public static void main(String[] args) {
			//floor
			System.out.println("Math.floor()");
			System.out.println(Math.floor(-8.9));
			System.out.println(Math.floor(8.9));
			System.out.println(Math.floor(-0.5));
			System.out.println(Math.floor(-0.0));
			System.out.println(Math.floor(0));
		}
测试结果:
		Math.floor()
		-9.0
		8.0
		-1.0
		-0.0
0.0
(2).ceil:天花板，向上取整，**返回值为double**。
		public static double ceil(double a){...}
传入的值都是**double**。double进double出。
测试代码:
		public static void main(String[] args) {
			//ceil
			System.out.println("Math.ceil()");
			System.out.println(Math.ceil(-8.9));
			System.out.println(Math.ceil(8.9));
			System.out.println(Math.ceil(-0.5));
			System.out.println(Math.ceil(-0.0));
			System.out.println(Math.ceil(0));
		}
测试结果:
		Math.ceil()
		-8.0
		9.0
		-0.0
		-0.0
		0.0
(3).rint:四舍，但五不一定入，碰到5向取较近偶数，返回值为double。
		public static double rint(double a) {...}
测试代码:
		public static void main(String[] args){
			//rint
			System.out.println("Math.rint()");
			System.out.println(Math.rint(10.5));
			System.out.println(Math.rint(10.51));
			System.out.println(Math.rint(11.5));
			System.out.println(Math.rint(-0.5));
			System.out.println(Math.rint(-1.5));
		}
测试结果:
		Math.rint()
		10.0
		11.0
		12.0
		-0.0
		-2.0
(4).round:四舍五入，参数为float时返回int值，double时返回long值。
		public static long round(double a) {...}
		public static int round(float a) {...}
测试代码:
	public static void main(String[] args){
		//round
		System.out.println("Math.round()");
		System.out.println(Math.round(10.5));
		System.out.println(Math.round(10.4));
		System.out.println(Math.round(11.5));
		System.out.println(Math.round(-0.5));
		System.out.println(Math.round(-0.6));
	}
测试结果:
		Math.round()
		11
		10
		12
		0
		-1
注意-0.5四舍五入为0，-0.6四舍五入为-1。

---
## 2.常用方法
### 根与绝对值
- 平方根
Math.sqrt():传入的参数是**double类型**，返回值为double。
		public static double sqrt(double a) {}
测试代码:
		public static void main(String[] args) {
			//sqrt平方根
			System.out.println("Math.sqrt()");
			System.out.println(Math.sqrt(2.0));
			System.out.println(Math.sqrt(-2.0));
			System.out.println(Math.sqrt(9));
			System.out.println(Math.sqrt(0.0));
		}
测试结果:
		Math.sqrt()
		1.4142135623730951
		NaN
		3.0
		0.0
传入一个负值sqrt并不报错。
- 立方根
Math.cbrt():传入的参数是**double类型**，返回值为double。
		public static double cbrt(double a) {
测试代码:
		public static void main(String[] args)
			System.out.println("Math.cbrt()");
			System.out.println(Math.cbrt(1.0));
			System.out.println(Math.cbrt(8.0));
			System.out.println(Math.cbrt(-8.0));
			System.out.println(Math.cbrt(-0.0));
			System.out.println(Math.cbrt(0.0));
			System.out.println(Math.cbrt(-0.0)==Math.cbrt(0.0));
		}
测试结果:
		Math.cbrt()
		1.0
		2.0
		-2.0
		-0.0
		0.0
		true
- 绝对值
Math.abs():可以传入int,long,float,fouble类型，返回值与传入的值类型相同。
测试代码：
	public static void main(String [] args){
		//abs绝对值
		System.out.println("Math.abs");
		System.out.println(Math.abs(1.01));
		System.out.println(Math.abs(-0.0));
		System.out.println(Math.abs(-10.01));
	}
测试结果:
		Math.abs
		1.01
		0.0
		10.01

### 最大最小值
- 最大值:
Math.max():可以传入int,long,float,fouble类型，返回值与传入的值类型相同。
		System.out.println(Math.max(1,0.0));  //1.0
		System.out.println(Math.max(3.5,-4.0));  //3.5
		System.out.println(Math.max(3.5,4.0));  //4.0
		System.out.println(Math.max(3,4));  //4
注释后为运行结果。
- 最小值
Math.min():可以传入int,long,float,fouble类型，返回值与传入的值类型相同。
		System.out.println(Math.min(1.0,0));  //0.0
		System.out.println(Math.min(3.5,-4.0));  //-4.0
		System.out.println(Math.min(3.5,4.0));  //3.5
		System.out.println(Math.min(3,4));  //3

---
## 3.随机数
- Math.random():可以生成大于等于0.0、小于1.0的double型随机数。
		int num=(int)(Math.random()*n);         //返回大于等于0小于n之间的随机整数
		int num0=m+(int)(Math.random()*n);      //返回大于等于m小于m+n（不包括m+n）之间的随机整数  
		int num1=(int)(random*100)；    //生成0-100的随机整数
- Math.random()还可以随机生成字符:
1.随机生成a-z之间的字符:
		(char)('a'+Math.random()*('z'-'a'+1));
2.随机生成char1-char2之间的字符:
		(char)(cha1+Math.random()*(cha2-cha1+1));
- 也可以使用Random类来生成随机数:
关于Randon类的使用可以参考博客:[JAVA的Random类的用法详解](http://www.jb51.net/article/83028.htm)。

---
## 4.幂与对数
1. a的b次幂
Math.pow(a，b):传入两个double类型的参数，返回值为double。
		public static double pow(double a, double b) {...}
测试代码:
	public static void main(String [] args){
		System.out.println("Math.pow");
		System.out.println(Math.pow(2, 3));
		System.out.println(Math.pow(3, 2));
		System.out.println(Math.pow(0.5, 2.3));
	}
测试结果:
		Math.pow
		8.0
		9.0
		0.2030630990890589
2. e为底数的幂
exp(x)： 返回e^x的值,double
expm1(x)： 返回e^x - 1的值,double
		System.out.println(Math.exp(0));  //1.0
3. 对数：
Math.log(x):返回log以e为底x的值，double
Math.log10(x):返回log以10为底x的值，double
Math.logp(x):返回log以e-1为底x的值，double
		System.out.println(Math.log(Math.E));  //1.0
		System.out.println(Math.log10(100));  //2.0
		System.out.println(Math.log1p(Math.E-1)); //1.0

---
## 5.常量
Math.PI与Math.E,可以直接使用。
Math.PI:比任何其他值都更接近 pi（即圆的周长与直径之比）的 double 值。 
Math.E:比任何其他值都更接近 pi（即圆的周长与直径之比）的 double 值。

---
## 6.三角函数与反三角函数
1. 三角函数:
Math.sin():求余弦，返回值double；
Math.cos():求正弦，返回值double；
Math.tan():求正切，返回值double；
		System.out.println(Math.sin(Math.PI/2));  //返回值1.0
		System.out.println(Math.cos(0));    //1.0
		System.out.println(Math.cos(Math.PI));    //-1.0
		System.out.println(Math.tan(0));    //0.0
但是注意有特殊情况:
		System.out.println(Math.sin(Math.PI)); 
		System.out.println(Math.cos(Math.PI/2));
		System.out.println(Math.tan(Math.PI/2));  //tan90度不报错
返回值为:
		1.2246467991473532E-16
		6.123233995736766E-17
		1.633123935319537E16
2. 反三角函数:
Math.asin():求反余弦，返回值double；
Math.acos():求反正弦，返回值double；
Math.atan():求反正切，返回值double；
		System.out.println(Math.asin(0));  //0.0
		System.out.println(Math.asin(-1));  //-PI/2小数值
		System.out.println(Math.acos(1));  //0.0
		System.out.println(Math.atan(1));  //PI/4小数值
3. atan2(y,x)：求向量(x,y)与x轴夹角，返回值为double
		System.out.println(Math.atan2(1.0, 1.0));//输出 π/4 的小数值

---
## 7.转换（弧度和角度）
Math.toDegrees(a): 弧度换角度,返回值为double

	public static double toDegrees(double angrad) {
		return angrad * 180.0 / PI;
	}
Math.toRadians(a): 角度换弧度,返回值为double

	public static double toRadians(double angdeg) {
		return angdeg / 180.0 * PI;
	}
测试代码:
```
System.out.println(Math.toDegrees(Math.PI));//输出180.0  
System.out.println(Math.toRadians(180));//输出 π 的值```

---
## 8.其他基本方法
1. Math.copySign(x,y):
*返回用y的符号取代x的符号后新的x值* 
		System.out.println(Math.copySign(-1.0, 2.0));//输出1.0  
		System.out.println(Math.copySign(2.0, -1.0));//输出-2.0  
2. Math.nextAfter(a,b):
*返回(a,b)或(b,a)间与a相邻很近的浮点数 b可以比a小*
        System.out.println(Math.nextAfter(1.2, 2.7));//输出1.2000000000000002  
        System.out.println(Math.nextAfter(1.2, -1));//输出1.1999999999999997 
b比a大，输出比a大一点点的浮点数；
b比a小，输出比a小一点点的浮点数。
3. Math.nextUp(a)
*返回比a大一点点的浮点数*
        System.out.println(Math.nextUp(1.2));//输出1.2000000000000002  
4. nextDown(a)
*返回比a小一点点的浮点数*
        System.out.println(Math.nextDown(1.2));//输出1.1999999999999997  

---
## 9.测试代码（总）
```
public class Main {  
  
    public static void main(String[] args) {  
        // TODO Auto-generated method stub  
        System.out.println(Math.E);//比任何其他值都更接近 e（即自然对数的底数）的 double 值。  
        System.out.println(Math.PI);//比任何其他值都更接近 pi（即圆的周长与直径之比）的 double 值。  
        /* 
         * 1.abs绝对值函数 
         * 对各种数据类型求绝对值 
         */  
        System.out.println(Math.abs(-10));//输出10  
          
        /* 
         * 2.三角函数与反三角函数 
         * cos求余弦 
         * sin求正弦 
         * tan求正切 
         * acos求反余弦 
         * asin求反正弦 
         * atan求反正切 
         * atan2(y,x)求向量(x,y)与x轴夹角 
         */  
        System.out.println(Math.acos(-1.0));//输出圆周率3.14...  
        System.out.println(Math.atan2(1.0, 1.0));//输出 π/4 的小数值  
          
        /* 
         * 3.开根号 
         * cbrt(x)开立方 
         * sqrt(x)开平方 
         * hypot(x,y)求sqrt(x*x+y*y)在求两点间距离时有用sqrt((x1-x2)^2+(y1-y2)^2) 
         */  
        System.out.println(Math.sqrt(4.0));//输出2.0  
        System.out.println(Math.cbrt(8.0));//输出2.0  
        System.out.println(Math.hypot(3.0, 4.0));//输出5.0  
          
        /* 
         * 4.最值 
         * max(a,b)求最大值 
         * min(a,b)求最小值 
         */  
        System.out.println(Math.max(1, 2));//输出2  
        System.out.println(Math.min(1.9, -0.2));//输出-0.2  
        /* 
         * 5.对数 
         * log(a) a的自然对数(底数是e) 
         * log10(a) a 的底数为10的对数 
         * log1p(a) a+1的自然对数 
         * 值得注意的是，前面其他函数都有重载，对数运算的函数只能传double型数据并返回double型数据 
         */  
        System.out.println(Math.log(Math.E));//输出1.0  
        System.out.println(Math.log10(10));//输出1.0  
        System.out.println(Math.log1p(Math.E-1.0));//输出1.0  
        /* 
         * 6.幂 
         * exp(x) 返回e^x的值 
         * expm1(x) 返回e^x - 1的值 
         * pow(x,y) 返回x^y的值 
         * 这里可用的数据类型也只有double型 
         */  
        System.out.println(Math.exp(2));//输出E^2的值  
        System.out.println(Math.pow(2.0, 3.0));//输出8.0  
          
        /* 
         * 7.随机数 
         * random()返回[0.0,1.0)之间的double值 
         * 这个产生的随机数其实可以通过*x控制 
         * 比如(int)(random*100)后可以得到[0,100)之间的整数 
         */  
        System.out.println((int)(Math.random()*100));//输出[0,100)间的随机数  
          
        /* 
         * 8.转换 
         * toDegrees(a) 弧度换角度 
         * toRadians(a) 角度换弧度 
         */  
        System.out.println(Math.toDegrees(Math.PI));//输出180.0  
        System.out.println(Math.toRadians(180));//输出 π 的值  
        /* 
         * 9.其他 
         */  
          
        //copySign(x,y) 返回 用y的符号取代x的符号后新的x值  
        System.out.println(Math.copySign(-1.0, 2.0));//输出1.0  
        System.out.println(Math.copySign(2.0, -1.0));//输出-2.0  
          
        //ceil(a) 返回大于a的第一个整数所对应的浮点数(值是整的，类型是浮点型)  
        //可以通过强制转换将类型换成整型  
        System.out.println(Math.ceil(1.3443));//输出2.0  
        System.out.println((int)Math.ceil(1.3443));//输出2  
          
        //floor(a) 返回小于a的第一个整数所对应的浮点数(值是整的，类型是浮点型)  
        System.out.println(Math.floor(1.3443));//输出1.0  
          
        //rint(a) 返回最接近a的整数的double值  
        System.out.println(Math.rint(1.2));//输出1.0  
        System.out.println(Math.rint(1.8));//输出2.0  
          
          
        //nextAfter(a,b) 返回(a,b)或(b,a)间与a相邻的浮点数 b可以比a小  
        System.out.println(Math.nextAfter(1.2, 2.7));//输出1.2000000000000002  
        System.out.println(Math.nextAfter(1.2, -1));//输出1.1999999999999997    
        //所以这里的b是控制条件  
          
        //nextUp(a) 返回比a大一点点的浮点数  
        System.out.println(Math.nextUp(1.2));//输出1.2000000000000002  
          
        //nextDown(a) 返回比a小一点点的浮点数  
        System.out.println(Math.nextDown(1.2));//输出1.1999999999999997     
    }  
}  ```
---
## 10. 使用Math类
Math类不能实例化对象：
`Math m = new Math();m.sqrt(4.0); ` 
这样会出错:

	Exception in thread "main" java.lang.Error: Unresolved compilation problem: 
		The constructor Math() is not visible
	
		at MathTest.test08.main(test08.java:5)
因为Math类的构造方法时这样的:
`private Math() {}`
根本不能创建对象。

---
## 11.参考博客
(1)[Java之Math类使用小结](http://blog.csdn.net/tomorrowtodie/article/details/52590688)
(2)[java中Math常用方法](https://www.cnblogs.com/whiteme/p/7234243.html)

---