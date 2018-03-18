---
title: Maven学习笔记（一）：环境搭建与常用命令
date: 2018-03-17 22:21:44
tags: Maven
categories: 开发工具

---
## 0.学习要点
1. 学习视频
慕课网《项目管理利器——maven》

2. 学习重点
	- 什么是Maven
	- 安装配置Maven
	- 使用命令行构建HelloWorld项目
	- 常用

---
## 1.Maven简介及名词介绍：
1. 什么是Maven?
Maven是一个项目管理工具（构建工具），它包含了一个**项目对象模型**，一组**标准集合**，一个**项目生命周期**（敲代码，编译，测试，打包，部署等），一个**依赖管理系统**和用来运行定义在生命周期阶段中插件目标的逻辑。
简单理解：使用Maven可以创建项目，管理项目。
其他定义：
Maven是基于项目对象模型（POM），可以通过一小段描述信息来管理项目的构建、报告和文档的软件项目管理工具。方便管理第三方jar包。

2. 项目生命周期：
![](http://p5ki4lhmo.bkt.clouddn.com/00013Maven%E5%AD%A6%E4%B9%A01-01.jpg)
传统的项目构建：java命令，编译，手动测试，手动清理，手动打包，手动部署，每一步都是手动。
Maven是自动化构建工具，能自动完成整个项目的构建。

3. 有哪些项目构建工具
Maven,Ant,gradle等。

---
## 2.Maven的安装配置
**1)Maven下载：**
<https://maven.apache.org/download.cgi>
下载及解压到某一目录，会看到内部如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00013Maven%E5%AD%A6%E4%B9%A01-02.jpg)
其中：

bin目录: 包含了mvn运行的脚本，命令行会调用这些脚本
内部有一个m2.config配置文件

boot: 包含了一个类加载器的框架，Maven使用它加载自己的类库

conf: 一些配置文件，如settings.xml。

lib: Maven运行时所需要的类库，自身需要的+第三方类库。

**2)配置环境变量：**
计算机-->右键，属性(或者点进文件管理-->计算机，系统属性)-->高级系统设置-->环境变量-->(用户)系统变量-->新建，新建如下变量：
![](http://p5ki4lhmo.bkt.clouddn.com/00013Maven%E5%AD%A6%E4%B9%A01-03.jpg)
变量值指向Maven的安装路径，我的是`D:\apache-maven-3.5.0\apache-maven-3.5.0`。
然后修改Path变量，在Path变量中添加`%M2_OME%\bin`,不同变量之间记得加分号。这是为了能让系统更方便地使用bin目录下的脚本。和Linux中的Path一样。
- 如何查看是否配置成功:
打开命令提示符输入`mvn -v`,能看到maven的版本就说明配置成功了：
![](http://p5ki4lhmo.bkt.clouddn.com/00013Maven%E5%AD%A6%E4%B9%A01-04.jpg)

---
## 2.简单构建一个HelloWorld
**1)Maven的目录结构：**
![](http://p5ki4lhmo.bkt.clouddn.com/00013Maven%E5%AD%A6%E4%B9%A01-05.jpg)
**2）创建一个HelloWorld**
在某个地方新建一个文件，文件内部有一个src目录存放java代码，src目录如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00013Maven%E5%AD%A6%E4%B9%A01-06.jpg)
其中main与test中都有一个java目录用于存放代码。
在main.java中创建一个包hello,并创建一个java文件，代码如下：

		package hello;
		
		public class HelloWorld{
			public String sayHello(){
				return "HelloWorld";
			}
		}
在test.java中创建相应的包并创建测试类(`TestHello.java`)：

		package HelloTest;
		
		import org.junit.*;
		import org.junit.Assert.*;
		import hello.*;
		
		public class TestHello{
			@Test
			public void testhello(){
				Assert.assertEquals("HelloWorld",new HelloWorld().sayHello());
			}
		}
切记一定要导hello包，否则后面测试会出问题。
然后需要一个`pom.xml`文件来管理(以下是pom的基本框架):

		<?xml version="1.0" encoding="UTF-8"?>
		
		<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
		    <modelVersion>4.0.0</modelVersion>
			<!--modelVersion，maven版本，固定格式,不用修改-->
			
		</project>
然后需要添加项目：

		<groupId>hello</groupId>
	    <artifactId>maven01-model</artifactId>
	    <version>0.0.1-SNAPSHOT</version>
		<!--groupId项目包名，不是测试包名-->
		<!--artifactId是模块名(标识)-->
		<!--version是版本号,SNAPSHOT是快照版本-->
由于用到了junit，需要添加junit依赖：

		<!--添加依赖-->
		<dependencies>
	        <dependency>
	            <groupId>junit</groupId>
	            <artifactId>junit</artifactId>
				<version>4.10</version>
	        </dependency>
			<!--这后面可以添加多个依赖-->
		</dependencies>
完整的`pom.xml`如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00013Maven%E5%AD%A6%E4%B9%A01-07-1.jpg)
然后将pom.xml文件放在maven01根目录（src在的那一层）。
这样一个简单的maven项目就构建完成了。

**3)测试maven项目：**
1. 编译：
使用cd切换到maven项目根目录（我的是`D:\apache-maven-3.5.0\Code\Maven01`）,或者直接在根目录[Shift]+[Ctrl]+右键选择在此处打开命令行，
输入命令：`mvn compile`对项目进行编译。如下图：
![](http://p5ki4lhmo.bkt.clouddn.com/00013Maven%E5%AD%A6%E4%B9%A01-08-1.jpg)
如果是第一次运行`mnv compile`会下载一些依赖的jar包(记得联网)，请耐心等待。若失败，看看有没有网，看看代码有没有写错。
2. 测试：
输入`mvn test`对项目进行测试，但是我在测试时出现了错误：
![](http://p5ki4lhmo.bkt.clouddn.com/00013Maven%E5%AD%A6%E4%B9%A01-09-01.jpg)
原来是我以前用过的仓库中已经有了这个插件了，可能解析不了。把这个插件删了，再次运行`mvn test`，会发现maven已经开始自动下载插件了：
![](http://p5ki4lhmo.bkt.clouddn.com/00013Maven%E5%AD%A6%E4%B9%A01-10.jpg)
终于测试成功(因为忘记导hello包而改了些地方):
![](http://p5ki4lhmo.bkt.clouddn.com/00013Maven%E5%AD%A6%E4%B9%A01-11.jpg)

3. 查看测试后的项目：
根目录默认生成了一个Target文件夹。里面有.class字节码文件与测试报告等。

4. 关于maven的插件（更多详情以后看了书《Maven实战》再写）
maven本身并没有什么功能，它的主要功能都是依靠插件来完成的,对于junit而言在我们在使用mvn test 命令对项目进行测试时会去中央仓库中下载一个名为maven-surefire-plugin的插件，然后在src/test/java/目录下去查找，是否有Test开头或结束的类，如果找到则默认执行所有测试用例并生成测试报告，如果想指定对单独方法进行测试可以使用类似mvn test -Dtest=HelloTest ,该命令支持通配符，mvn tes -Dtest=Hello*Test,这样就会运行测试以Hello开头和Test结尾的所有测试用例， 也可以用逗号分隔一次执行多个测试用例.

**3)项目打包：**
输入`mvn package`可以将项目打包成jar包。
若出错则将仓库maven插件中的`maven-jar-plugin`删除再运行。
打完包后的package文件如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00013Maven%E5%AD%A6%E4%B9%A01-12.jpg)

---
## 3.常用的构建命令
**1)前面出现过的命令：**
`mvn -v`:查看maven版本
`mvn compile`:在根目录编译项目，编译的内容在target中
`mvn test`:运行test下的测试用例，测试报告在target中
`mvn package`:打包项目，打的jar在target目录下

**2)其他常用构建的命令：**
1. `mvn clean`：删除项目目录下的target目录
若出错则将仓库maven插件中的`maven-clean-plugin`删除再运行。
执行后根目录的`target`文件被删除。

2. **mvn install命令导入到本地仓库**
新建一个与Maven01差不多的Maven02项目，内部构造如下
		Maven02
			-src
				-main
					-java
						-Hello
							-Speek.java
				-test
					-java
						-TestSpeek
							-TestHello.java
			-pom.xml
`Speek.java`代码如下：
		package Hello;
		import hello.*;
		public class Speek{
			public String say(){
				return new HelloWorld().sayHello();
			}
		}
`TestHello.java`代码如下：
		package TestSpeek;
		import org.junit.*;
		import org.junit.Assert.*;
		import Hello.Speek;
		public class TestHello{
			@Test
			public void test01(){
				Assert.assertEquals("HelloWorld",new Speek().say());
			}
		}
pom.xml中参数修改：
		<groupId>Hello</groupId>
	    <artifactId>maven02</artifactId>
此时使用`mvn compile`会报错说找不到hello这个类，因为没有导Maven01包。
这时候就可以使用`mvn install`了：
在Maven01（注意是01）下打开命令行，输入`mvn install`:
![](http://p5ki4lhmo.bkt.clouddn.com/00013Maven%E5%AD%A6%E4%B9%A01-13.jpg)
若出错则将仓库maven插件中的`maven-install-plugin`删除再运行。
`mvn install`**命令可以将包含引用类的项目打成jar包放到本地仓库。**
可以在本地仓库中看见引入的包，说明install执行成功：
![](http://p5ki4lhmo.bkt.clouddn.com/00013Maven%E5%AD%A6%E4%B9%A01-14.jpg)
然后在Maven02中修改pom.xml,将Maven01作为依赖引入(在dependencies中添加):
		<dependency>
            <groupId>hello</groupId>
			<artifactId>maven01-model</artifactId>
			<version>0.0.1-SNAPSHOT</version>
        </dependency>
然后再使用`mvn compile`进行编译，编译成功：
![](http://p5ki4lhmo.bkt.clouddn.com/00013Maven%E5%AD%A6%E4%B9%A01-15.jpg)
再使用`mvn test`测试,测试成功：
![](http://p5ki4lhmo.bkt.clouddn.com/00013Maven%E5%AD%A6%E4%B9%A01-16.jpg)
完整的`pom.xml`：
![](http://p5ki4lhmo.bkt.clouddn.com/00013Maven%E5%AD%A6%E4%B9%A01-17.jpg)

**3)maven引入jar包的流程：**
使用`mvn compile`编译源代码时会查看是否有第三方jar包，有的话会查看pom.xml是否引入该jar包的坐标，若本地仓库有则将它加入到项目的ClassPath，若本地仓库没有则去maven中央仓库下载。

---
## 4.Maven自动构建目录骨架
**1)规定：**
main.java文件夹中存放项目代码
test.java文件夹中存放测试代码
**2)使用archetype插件创建根目录：**
在Code文件夹下创建Maven03空目录，并使用命令行cmd进入，输入`mvn archetype :generate`，会让你按照提示进行创建：
![](http://p5ki4lhmo.bkt.clouddn.com/00013Maven%E5%AD%A6%E4%B9%A01-18.jpg)
会让你设定：组名，模块号，版本，包名，和pom.xml中设置的差不多。最后输入yes表示确认。
创建成功:
![](http://p5ki4lhmo.bkt.clouddn.com/00013Maven%E5%AD%A6%E4%B9%A01-19.jpg)
会自动创建和Maven01,Maven02相同的目录并创建HelloWorld用例。
**3）使用另一种方式创建：**

		mvn archetype： -DgroupId=组织名，公司网址的反写+项目名 -DartifactId=项目名-模块名 -Dversion=版本号 -Dpackage=代码所在的包名
一次性指定所有的参数，效果和第一种方式相同，不再测试。

---
## 5.Maven中坐标和仓库的概念：
**1)坐标与构件：**
1. Maven中的构件都是由坐标来唯一标识的，坐标的基本组成：
		  <dependency>
		     <groupId></groupId>
		     <artifactId></artifactId>
		     <version></version>     
		  </dependency>
2. 约定：
项目包名，组名（groupId），模块名最好是相关联的。

**2)仓库：**
- Maven中的构件都存储在仓库中
仓库一般分为两种：本地仓库，远程仓库。
找到maven下的lib文件夹，我的是:`D:\apache-maven-3.5.0\apache-maven-3.5.0\lib`，找到`maven-model-builder-3.5.0.jar`这个jar包，打开`org\apache\maven\model`目录下的`pom-4.0.0.xml`(一个超级pom)文件，可以看到远程仓库(全球中央仓库)的信息及地址:
<https://repo.maven.apache.org/maven2>
如下图:
![](http://p5ki4lhmo.bkt.clouddn.com/00013Maven%E5%AD%A6%E4%B9%A01-20.jpg)
基本上一般开发所用到的开源jar包都可以在这里找到。找不到则会报错。

**3)配置镜像仓库：**
- 国外的仓库有时太慢，用国内相同的镜像仓库代替（和Gitee跟Github的关系差不多）。
打开maven底下的conf文件夹，找到settings.xml并打开，我的位置是：
`D:\apache-maven-3.5.0\apache-maven-3.5.0\conf\settings.xml`
在140-150左右的地方可以找到<mirrors></mirrors>标签，使用阿里云的镜像则可以这样修改：
		<mirror>
	        <id>nexus-aliyun</id>
	        <mirrorOf>central</mirrorOf>
	        <name>Nexus aliyun</name>
	        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
	     </mirror>
也可以使用maven镜像（视频中的）:
![](http://p5ki4lhmo.bkt.clouddn.com/00013Maven%E5%AD%A6%E4%B9%A01-21.jpg)
`<mirrorOf>central</mirrorOf>`也可以使用*来指定所有的镜像仓库，一旦配置了镜像，所有针对原仓库的访问都会失效（国外中央仓库失效)。

**4)更改仓库位置(repo)：**
Maven的仓库是默认放在你用户文件夹的.m2隐藏文件下的，我的原仓库的位置是：
`C:\Users\poppy\.m2\repository`。
但是一般不会帮仓库放到C盘中，所以要修改仓库地址:

1. 新建一个文件夹repo用作新仓库：
我的目录是:`D:\apache-maven-3.5.0\repo`;

2. 找到conf的settin.xml文件（改镜像仓库的那个），在第50-60行作用可以看见这个标签:
		<localRepository></localRepository>
在这里面写上上面新建的目录：
		<localRepository>D:/apache-maven-3.5.0/repo</localRepository>
保存。
3. 将settings.xml复制一份放入到repo目录下：
这样以后更新maven版本就不用更新settings了。

4. 计算机-->右键，属性(或者点进文件管理-->计算机，系统属性)-->高级系统设置-->环境变量-->(用户)系统变量:
查看一下是否有个用户变量`M2_REPO`指向原来的`C:\Users\****\.m2\repository`仓库呢，可以编辑就改掉把，不能编辑就删掉重建，或者直接把新仓库路径写入path。（详情可以参考Maven的配置）

---
## 6.学习总结:
`mvn -v`:查看maven版本号
`mvn compile`:在根目录编译项目，编译的内容在target中
`mvn test`:运行test下的测试用例，测试报告在target中
`mvn package`:打包项目，打的jar在target目录下
`mvn clean`:删除项目目录下的target目录   
`mvn install`将项目打jar包并添加到本地仓库中

在CMD中自动创建maven目录的两种方式：
`mvn archetype:generate`按照提示进行选择
```
mvn archetype:gennerate -DgroupId=组织名，一般公司网址的反写+项目名
	                           -DartifactId=项目名-模块名
	                           -Dversion=版本号
	                           -Dpackage=代码所存在的包名
```
一次性指定所有的属性

---