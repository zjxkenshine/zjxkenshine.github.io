---
title: Maven学习笔记（二）：Maven核心知识及在eclipese中使用
date: 2018-03-18 20:07:50 
tags: Maven
categories: 开发工具

---
## 0.学习要点
1. 学习视频
[慕课网《项目管理利器——maven》](https://www.imooc.com/learn/443)

2. 学习重点
	- 如何在eclipse中安装插件
	- eclipse中使用maven创建项目
	- Maven的生命周期
	- pom.xml
	- 关于依赖

---
## 1.eclipse中使用maven创建项目
**1)在eclipse中安装M2E插件：**
1. 首先最重要的一点，是看看你的eclipse中是否已经安装了maven插件了:
打开你的Eclipse-->Windows-->Preferences查看是否有maven插件。
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-01.jpg)
一般来说Eclipse4.0以上的版本一般都自带了Maven插件，我的是Eclipse 4.4.3，本来是有Maven插件的，为了学习，卸载了←_←。
2. 安装M2E（Maven to Eclipese)插件：
首先给你个下载M2E的网站：
<http://www.eclipse.org/m2e/m2e-downloads.html>
进去之后，是这个样子的：
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-04.jpg)
点击第一个或第二个，它会教你怎么用的(就是下面的)。
复制地址栏`http://download.eclipse.org/technology/m2e/milestones/1.8/`
接着切回Eclipese,打开Help-->Install New Software打开界面并输入该地址，选择Maven并下载：
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-02.jpg)
注意：首先你得连网，一开始显示Pending等一会儿就好了。
这里下载完，会有选择项，一路next下去，finish后还会下载很多东西，等下载完成重启，再点开Preferences:
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-03.jpg)
插件安装完毕了。

 eclipse是默认运行在jre之上的而maven需要jdk的支持，需要tools.jar，tools.jar默认放在jdk/lib目录中，所以修改eclipse的JRE，Windows-->Prefrence-->java-->Installed JREs-->ADD-->Next-->把本地JDK的目录放到JRE home中-->最后勾选JDK
详情参考慕课网的课程。

3. 配置本地Maven：
点击Maven-->Installations-->Add添加本地项目：
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-05.jpg)
然后点击User Settings修改配置文件与仓库地址：
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-06.jpg)

**2)Eclipse创建Maven项目：**
File-->New-->Project找到Maven Project:
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-07.jpg)
一路next到这个：
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-08-1.jpg)
会显示很多种模板，选择quickstart就可以了。
接下来配置坐标(和pom.xml差不多)：
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-09.jpg)
点击finish就完成啦！
点开项目，发现目录结构和手动创建的是一样的，而且pom.xml也已经为我们自动创建好了：
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-10.jpg)

**3)运行Maven项目：**
maven项目上右键-->Run AS-->Maven Build..进入到如下界面：
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-11.jpg)
输入compile进行编译，输入test进行测试：
Console窗口下的显示和命令行的差不多。点击Run。
若失败提醒你maven-xxxx-plugins出错，去仓库删掉它再来运行。
打包则输入package命令，打包后刷新项目则会看到这样：
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-12.jpg)
打包成功。

**4)视频课程中运行错误的解决方法：**
1. 如果版本不匹配 则mvn -v查看maven的jdk版本，然后在eclipse中配置当前使用的jdk
2. run as-->Maven build...-->在goals中compile（可以在此处使用其他的命令，如：package -->run
 若报-Dmaven.multiModuleProjectDirectory错误，则在选项-->java-->installed JRES 中设置jdk的参数，
 添加上“-Dmaven.multiModuleProjectDirectory=$M2_HOME”
3. 具体我也没遇到，就不多写了。

**5)如何卸载M2E：**
打开Eclipse-->Help-->Installation Details
在这里可以更新可以卸载。

---
## 2.Maven的生命周期与插件：
**1）Maven的生命周期：**
完整的项目构建过程包括：
清理 编译 测试 打包 集成测试 验证 部署
2. Maven抽象出了一套项目构建的生命周期，而插件是对Maven抽象的具体实现。
3. Maven中定义了三套**相互独立**的生命周期:
&nbsp;&nbsp;&nbsp;&nbsp;clean  清理项目
&nbsp;&nbsp;&nbsp;&nbsp;cleandefault  构建项目
&nbsp;&nbsp;&nbsp;&nbsp;cleansite  生成项目站点
每一套生命周期中又包含一些阶段，每个生命周期中的阶段都是有顺序的，每个阶段都依赖于前一个阶段，阶段顺序执行但不会触发另外两套生命周期中的任何阶段。
如执行package会先执行编译与测试。

**2)各个生命周期的阶段：**
1. clean 清理项目：
	- pre-clean 执行清理前的项目
	- clean 清理上一个构建的项目
	- post-clean 执行清理后的项目
2. default 构建项目（核心生命周期）（部分常用阶段）：
	- compile 编译
	- test 测试
	- package 打包
	- install 打包到本地仓库
3. site 根据pom生成站点：
	- pre-site 生成项目站点前需要完成的工作
	- site 生成站点文档
	- post-site 生成项目站点后要玩成的工作
	- site-deploy 部署生成的站点到服务器上

**3)Maven的插件的使用：**
1. 单单对于Maven而言它并没有什么执行任务的功能，maven中的所有命令也是调用插件实现的，可以在maven的官网中查看插件的描述：
<http://maven.apache.org/plugins/index.html>

2. 使用插件的基本步骤：
pom.xml中添加插件坐标-->绑定某一个生命周期中的某个阶段-->运行

3. pom.xml中的配置：
		<build>
			<plugins>
				<plugin>
					<!--定义插件坐标-->
					<groupId></groupId><!--项目组 id-->
					<artifactId></artifactId><!--项目 id-->
					<version></version><!--版本号-->
					<executions>
						<execution>
							<!--绑定生命周期的阶段-->
							<phase></phase>
							<goals>
								<!--目标-->
								<goal></goal>
							</goals>
						</execution>
					</executions>
				</plugin>
			</plugins>
		</build>

4. 如让项目在打包（package）时把源码也打包（source插件），pom.xml中这么写:
		<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
		  <modelVersion>4.0.0</modelVersion>
		
		  <groupId>MVNTest01</groupId>
		  <artifactId>mvn-test</artifactId>
		  <version>0.0.1-SNAPSHOT</version>
		  <packaging>jar</packaging>
		
		  <name>mvn-test</name>
		  <url>http://maven.apache.org</url>
		
		  <properties>
		    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		  </properties>
		
		  <dependencies>
		    <dependency>
		      <groupId>junit</groupId>
		      <artifactId>junit</artifactId>
		      <version>3.8.1</version>
		      <scope>test</scope>
		    </dependency>
		  </dependencies>
		  
		  <build>
			<plugins>
				<plugin>
					<!-- 定义插件坐标-->
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-source-plugin</artifactId>
					<version>3.0.1</version>
					<executions>
						<execution>
							<!--绑定生命周期的阶段-->
							<phase>package</phase>
							<goals>
								<!--目标-->
								<goal>jar-no-fork</goal>
							</goals>
						</execution>
					</executions>
				</plugin>
			</plugins>
		  </build>
		</project>
其中gaols中的值可以通过点击官网插件看到，是source的不同命令：
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-13.jpg)
然后使用package命令，发现先运行了编译和测试，然后生成了两个jar包。


---
## 3.pom.xml解析
1. pom文件时Maven的核心管理文件，用于依赖管理，组织管理构建信息的管理。
pom中有非常多的标签，这里只讲常用的。
2. 常见标签
		<!--pom根元素，包含了pom的一些约束信息-->
		<project>
			<!--固定的版本，必须元素-->
			<modelVersion>4.0.0</modelVersion>
			
			<!--坐标信息-->
			<!--主项目id,项目的实际地址-->
			<groupId>反写的公司网址+项目名</groupId>  
			<!--模块id,项目中的功能模块-->
			<artifactId>项目名+模块名</artifactId>
			<!--项目版本号，一般由三个数字组成,第一个0表示大版本号，第二个0表示分支版本号，第三个1表示小版本号-->
			<!--SNAPSHOT 快照版本
				alpha 内部版本，内测版本
				beta 公测版本
				Release 稳定版
				GA 正式发布版-->
			<version>0.0.1-SNAPSHOT</version> 
			<!--打包后的扩展名，默认是jar，还可以是war,zip,pom-->
			<packaging>jar</packaging>
			
			<!--项目描述名-->
			<name></name>
			<!--项目地址-->
			<url></url>
			<!--项目描述，介绍-->
			<discription></discription>
			<!--开发人员列表-->
			<developers></developers>
			<!--许可证信息-->
			<licenses></licenses>
			<!--组织信息-->
			<organization></organization>
			
			<!--设置编码为utf-8-->
			<!--定义变量，了直接在后面使用${标签名}调用-->
			<properties>
				<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
			</properties>
			
			<!--依赖开始-->
			<!--依赖列表-->
			<dependencies>
				<!--依赖项（多个）-->
				<dependency>
					<groupId></groupId>
					<artifactId></artifactId>
					<version></version>
					<!--类型-->
					<type></type>
					<!--依赖范围,如写test则只在测试内有效-->
					<scope></scope>
					<!--设置依赖是否可选，默认false；false:子项目默认继承；true:子项目需要显式引用该依赖-->
					<optional>true/false</optional>
					<!--排除依赖传递的列表-->
					<exclusions>
						<!--排除依赖项(多个)，如Ajar包依赖B,B依赖C，A就是传递依赖C-->
						<exclusion></exlusion>
					</exclusions>
				</dependency>
			</dependencies>
			
			<!--依赖管理，内部的依赖不会运行
				定义在父模块中，子模块继承用的-->
			<dependencyManagement>
				<dependencies>
					<dependency></dependency>
				</dependencies>
			</dependencyManagement>
			
			<!--build为构建的的行为提供相应的支持-->
			<build>
				<!--build中常用的是插件，插件列表-->
				<plugins>
					<!--插件，不止一个-->
					<plugin>
						<!--插件的坐标-->
						<groupId></groupId>
						<artifactId></artifactId>
						<version></version>
					</plugin>
				</plugins>
			</bulid>
			
			<!--子模块中对父模块的pom的继承-->
			<parent></parent>
			<!--模块聚合运行，可以同时指定多个模块进行编译测试-->
			<modules>
				<modules></modules>
			</modules>
			
		</project>

---
## 4.依赖范围：

	<dependency>
		<groupId></groupId>
		<artifactId></artifactId>
		<version></version>
		<type></type>
		<!--依赖范围,如写test则只在测试内有效-->
		<scope></scope>
		...
	</dependency>
1）`<scope>classpath</scope>`
开发时需要使用某一个框架（jar包），就要将该框架的jar包引入到项目的classpath中，Maven中有三种classpath：**编译，测试，运行**。
**2）scope的属性：**
打开官网对于scope的描述：
<https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html>
可以看到有6种值：
	- compile：默认级别，编译测试运行都有效
	- provided：在编译和测试时有效
	- runtime：在测试和运行时有效（如jdbc的驱动）
	- test：只在测试时有效（如junit）
	- system：编译与测试有效，但是要与本地的系统相关联，可移植性差
	- import：只使用在<dependencyManagement>标签中，表示从其他项目中继承过来的依赖。
	
还有一些例子,自己去看吧。

---
## 5.依赖传递
**1)传递依赖：**
A依赖于B，B依赖于C，则A与C是传递依赖
**2)测试：**
1. 创建三个Maven项目
mvna,mvnb,mvnc:
2. 配置mvnb依赖mvnc:
		  <dependencies>
		    <dependency>
		      <groupId>junit</groupId>
		      <artifactId>junit</artifactId>
		      <version>3.8.1</version>
		      <scope>test</scope>
		    </dependency>
		    <dependency>
		     	<groupId>MVNC</groupId>
		  	 	<artifactId>mvnc</artifactId>
		  		<version>0.0.1-SNAPSHOT</version>
		    </dependency>
		  </dependencies>
使用mvnc右键运行中使用install将mnvc打包到本地仓库。
（运行install会把前面的编译测试打包都执行一遍）
3. 配置mvna依赖mnvb:
		  <dependencies>
		    <dependency>
		      <groupId>junit</groupId>
		      <artifactId>junit</artifactId>
		      <version>3.8.1</version>
		      <scope>test</scope>
		    </dependency>
		     <dependency>
				<groupId>MVNB</groupId>
				<artifactId>mvnb</artifactId>
				<version>0.0.1-SNAPSHOT</version>
		    </dependency>
		  </dependencies>
使用mvnb右键运行中使用install将mnvb打包到本地仓库。
最后测试mvna,测试成功:
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-14.jpg)

**3)排除传递依赖：**
先查看一下MVNA的依赖：
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-15.jpg)
使用`<exclusions><exclusion></exclusion></exclusions>`排除传递依赖，MVNA排除MVNC依赖可以这么写：

		  <dependencies>
		    <dependency>
		      <groupId>junit</groupId>
		      <artifactId>junit</artifactId>
		      <version>3.8.1</version>
		      <scope>test</scope>
		    </dependency>
		     <dependency>
		     	<groupId>MVNB</groupId>
		  	 	<artifactId>mvnb</artifactId>
		  		<version>0.0.1-SNAPSHOT</version>
		  		<exclusions>
		  			<exclusion>
		  				<groupId>MVNC</groupId>
						  <artifactId>mvnc</artifactId>
		  			</exclusion>
		  		</exclusions>
		    </dependency>
		  </dependencies>
再次编译，查看结果,传递依赖已经消除了：
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-16.jpg)

**4)修改默认jdk：**
1. 先修改jdk：
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-17.jpg)
Maven默认的JDK是1.5，在这一行上右键-->Properites进行修改
虽然不修改也没什么问题，但是不好看,而且实际使用的是1.7
2. 修改Maven默认配置：
打开settings.xml,找到最底下的`<profiles></profiles>`在里面添加如下代码即可：
		<profile>
			<id>jdk-1.7</id>
			<activation>
				<activeByDefault>true</activeByDefault>
				<jdk>1.7</jdk>
		    </activation>	
			<properties>
				<maven.compiler.source>1.7</maven.compiler.source>
				<maven.compiler.target>1.7</maven.compiler.target>
				<maven.compiler.compilerVersion>1.7</maven.compiler.compilerVersion>
			</properties>
		</profile>

---
## 6.依赖冲突：
1)依赖冲突是指A与B依赖了一个不同版本的相同构件AC,BC，若有模块D同时依赖A与B，那么使用时是哪个版本呢？
**2)两条原则：**
1. **短路优先原则**：
A<--B<--C<--X(1jar)
A<--D<--X(2jar)
则会使用下一个版本X(2jar)。
2. **路径相同  则依赖配置近的优先**：
如1的情景，若A先被添加依赖，则使用AC,若B先被添加依赖，则BC先

---
## 7.聚合与继承
**1)聚合**:
1. 之前在Maven中若想将多个模块都install入仓库中，必须一个一个install。Maven中有一个方式可以将多个项目放到一起运行==>>**聚合**。
2. 新建一个mvnd项目：
如何一起聚合mvnabc一起编译测试打包入库呢？
修改mvnd的pom.xml如下：
		<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
		  <modelVersion>4.0.0</modelVersion>
		
		  <groupId>MVND</groupId>
		  <artifactId>mvnd</artifactId>
		  <version>0.0.1-SNAPSHOT</version>
		  <packaging>pom</packaging>  <!--注意这里改为pom-->
		
		  <name>mvnd</name>
		  <url>http://maven.apache.org</url>
		
		  <properties>
		    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		  </properties>
		
		  <dependencies>
		    <dependency>
		      <groupId>junit</groupId>
		      <artifactId>junit</artifactId>
		      <version>3.8.1</version>
		      <scope>test</scope>
		    </dependency>
		  </dependencies>
		  
		  <!--使用module标签聚合-->
		  <modules>
		  	<module>../mvna</module>
		  	<module>../mvnb</module>
		  	<module>../mvnc</module>
		  </modules>
		  
		</project>
3. 在mvnd上运行install,会发现mvnabc的jar包都添加到了仓库中。
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-18.jpg)

**2)继承：**
查看每个项目，发现都有Junit,那么就可以用继承来实现：
三个项目的pom都写了junit的依赖,可以将junit依赖写在父maven项目pom中；packaging为pom；使用dependencyManagement
新建一个MVNE项目，修改pom.xml如下：

		<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
		  <modelVersion>4.0.0</modelVersion>
		
		  <groupId>MVNE</groupId>
		  <artifactId>mvne</artifactId>
		  <version>0.0.1-SNAPSHOT</version>
		  <packaging>pom</packaging>  <!--注意这里改为pom-->
		
		  <name>mvnd</name>
		  <url>http://maven.apache.org</url>
		
		  <properties>
		    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
			<junit version>3.8.1<junit version>
		  </properties>
		
		  <dependencyManagement>
		  <dependencies>
		    <dependency>
		      <groupId>junit</groupId>
		      <artifactId>junit</artifactId>
		      <version>${junit version}</version>
		      <scope>test</scope>
		    </dependency>
		  </dependencies>
		  </dependencyManagement>
		  
		</project>
且可以将main和test文件夹删除。
然后mvna继承mvne可以这样写(部分)：

		<parent>
		  <groupId>MVNE</groupId>
		  <artifactId>mvne</artifactId>
		  <version>0.0.1-SNAPSHOT</version>
		</parent>
		<dependencies>
		    <dependency>
		      <groupId>junit</groupId>
		      <artifactId>junit</artifactId>
		    </dependency>
		  </dependencies>
就不用在子类里写版本号及依赖范围了。

---
## 8.maven新建一个web项目并发布到Netty
**1)创建web项目：**
1. File-->New -->Project-->Maven Project然后一路next到达这个界面：
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-19.jpg)
选择webapp模板，然后设置组名为MVNWEB,项目名为mvnweb,finish。记得将eclipse模式切换为javaEE哦。
2. 打开之后是这样的:
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-20.jpg)
若index.jsp出现错误，直接跳到配置部分先。
3. 发现系统没有帮我们创建main与test目录，需要我们自己添加。
添加**resource folder**，使目录变为Maven标准结构：
从上图可知，需要创建src/main/java/;src/test/java;src/test/resourse的source folder,注意不是package，是source folder。
如果无法创建source folder:
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-21.jpg)
可以先创建个src/test/jav再重命名为java。
或者可以Windows-->Show View-->Navigator,在这个视图内就可以创建了,创建完Folder后返回工程视图Project Explore，右键-->Maven-->Update Project即可。
4. 检验ClassPath路径：
项目右键-->Build Path-->configure bulid path,看到如下界面的resoures,检查所有的outputFile是不是target目录:
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-22.jpg)
5. 选择动态web模块：
右键-->属性-->Project Facets-->勾上Dinamic web与JS
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-23.jpg)
这时我发现我原来没报错的index.jsp页面报错了。

**2)更多配置：**
1. jsp页面报错：
原因是没有servlet插件，没报错可以跳过。
可以去到Maven的中央仓库h或阿里云镜像去查找Java Servlet API。
<http://www.mvnrepository.com/artifact/javax.servlet/javax.servlet-api>或阿里的<http://maven.aliyun.com/nexus/>
打开pom.xml，添加如下依赖：
		<dependency>
		    <groupId>javax.servlet</groupId>
		    <artifactId>javax.servlet-api</artifactId>
		    <version>4.0.0</version>
			<!-- 只在编译和测试时运行 -->
		    <scope>provided</scope>
		</dependency>
然后再右键Maven-->Update Project后发现项目变成了熟悉的样子：
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-24.jpg)
有个叉叉，但是找不出哪里错了，不管了。
2. 修改部署时的默认配置:
项目右键-->属性，找到Development Assembly，把测试用的代码全部删除:
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-25.jpg)
这样一个web项目就创建成功了。

**3)打包(war)并拷贝到jetty容器：**
1. 使用jetty作为web容器:
<http://www.mvnrepository.com/artifact/org.eclipse.jetty/jetty-maven-plugin>
或<http://maven.aliyun.com/nexus/>
在pom.xml的builder标签中添加插件：
	    <plugins>
	    	<plugin>
	   	 		<groupId>org.eclipse.jetty</groupId>
	    		<artifactId>jetty-maven-plugin</artifactId>
	    		<version>9.4.8.v20171121</version>
			</plugin>
	    </plugins>
一定要添加插件，不要添加依赖。
2. 运行jetty
项目右键-->RUN AS-->Maven Builder,输入`jetty:run`:
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-26.jpg)
启动成功:
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-27.jpg)
我配置的是阿里云镜像，所以一开始用官网的失败了，后来去阿里云里找就可以了。
3. 再次配置pom.xml使其在打包后运行jetty:run:
		<plugins>
	    	<plugin>
	   	 		<groupId>org.mortbay.jetty</groupId>
	  			<artifactId>jetty-maven-plugin</artifactId>
	  			<version>8.1.16.v20140903</version>
	  			<executions>
	  				<execution>
	  					<!-- 在打包成功后使用jetty:run来运行jetty服务 -->
	  					<phase>package</phase>
	  					<goals>
	  						<goal>run</goal>
	  					</goals>
	  				</execution>
	  			</executions>
			</plugin>
	    </plugins>
然后停止这次Jetty服务，再次clean package(Maven Builder):
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-28.jpg)
4. 成功后就可以使用localhost:8080访问了
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-29.jpg)
5. 完整的pom.xml代码如下:
		<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
		  <modelVersion>4.0.0</modelVersion>
		  <groupId>MVNWEB</groupId>
		  <artifactId>mvnweb</artifactId>
		  <packaging>war</packaging>
		  <version>0.0.1-SNAPSHOT</version>
		  <name>mvnweb Maven Webapp</name>
		  <url>http://maven.apache.org</url>
		  <dependencies>
		    <dependency>
		      <groupId>junit</groupId>
		      <artifactId>junit</artifactId>
		      <version>3.8.1</version>
		      <scope>test</scope>
		    </dependency>
		    <dependency>
		    <groupId>javax.servlet</groupId>
		    <artifactId>javax.servlet-api</artifactId>
		    <version>4.0.0</version>
		    <!-- 只在编译和测试时运行 -->
		    <scope>provided</scope>
			</dependency>
		  </dependencies>
		  <build>
		    <finalName>mvnweb</finalName>
		    <plugins>
		    	<plugin>
		   	 		<groupId>org.mortbay.jetty</groupId>
		  			<artifactId>jetty-maven-plugin</artifactId>
		  			<version>8.1.16.v20140903</version>
		  			<executions>
		  				<execution>
		  					<!-- 在打包成功后使用jetty:run来运行jetty服务 -->
		  					<phase>package</phase>
		  					<goals>
		  						<goal>run</goal>
		  					</goals>
		  				</execution>
		  			</executions>
				</plugin>
		    </plugins>
		  </build>
		</project>

---
## 9.将项目打包发布到Tomcat
1. 进入tomcat官网然后点击Maven Plugin
<http://tomcat.apache.org/maven-plugin.html>
2. 修改pom.xml如下：
		<plugins>
    		<plugin>
  		  	<groupId>org.apache.tomcat.maven</groupId>
          	<artifactId>tomcat7-maven-plugin</artifactId>
          	<version>2.2</version>
  			<executions>
  				<execution>
  					<!-- 在打包成功后使用jetty:run来运行jetty服务 -->
  					<phase>package</phase>
  					<goals>
  						<goal>run</goal>
  					</goals>
  				</execution>
  			</executions>
		</plugin>
3. clean package项目:
启动成功:
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-30.jpg)
在浏览器中查看（对比jetty,要加项目名）
![](http://p5ki4lhmo.bkt.clouddn.com/00015Maven%E5%AD%A6%E4%B9%A02-31.jpg)
4. 完整的pom.xml配置（附带jetty的）：
		<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
		  <modelVersion>4.0.0</modelVersion>
		  <groupId>MVNWEB</groupId>
		  <artifactId>mvnweb</artifactId>
		  <packaging>war</packaging>
		  <version>0.0.1-SNAPSHOT</version>
		  <name>mvnweb Maven Webapp</name>
		  <url>http://maven.apache.org</url>
		  <dependencies>
		    <dependency>
		      <groupId>junit</groupId>
		      <artifactId>junit</artifactId>
		      <version>3.8.1</version>
		      <scope>test</scope>
		    </dependency>
		    <dependency>
		    <groupId>javax.servlet</groupId>
		    <artifactId>javax.servlet-api</artifactId>
		    <version>4.0.0</version>
		    <!-- 只在编译和测试时运行 -->
		    <scope>provided</scope>
			</dependency>
		  </dependencies>
		  <build>
		    <finalName>mvnweb</finalName>
		    <plugins>
		    	<plugin>
		    	<!--
		   	 		<groupId>org.mortbay.jetty</groupId>
		  			<artifactId>jetty-maven-plugin</artifactId>
		  			<version>8.1.16.v20140903</version>  -->
		  			
		  		  <groupId>org.apache.tomcat.maven</groupId>
		          <artifactId>tomcat7-maven-plugin</artifactId>
		          <version>2.2</version>
		  			<executions>
		  				<execution>
		  					<!-- 在打包成功后使用jetty:run来运行jetty服务 -->
		  					<phase>package</phase>
		  					<goals>
		  						<goal>run</goal>
		  					</goals>
		  				</execution>
		  			</executions>
				</plugin>
				
		    </plugins>
		  </build>
		</project>

5. 现在就可以开始玩Maven啦！！！

---