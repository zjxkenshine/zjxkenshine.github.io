---
title: MyBatis学习笔记（五）：代码生成器
date: 2018-03-29 20:15:28
tags: MyBatis
categories: Java框架

---
## 0.学习准备
1. 学习资料：
书本：《MyBatis从入门到精通》
视频：某培训班视频
资料：[MyBatis中文文档](http://www.mybatis.org/mybatis-3/zh/index.html)

---
## 1.代码生成器MBG简介
1. MyBatis Generator,简称MBG，是MyBatis开发团队提供的一个强大的代码生成器。
2. MBG可以通过不同的配置生成不同类型的代码，包含了数据库表对应的实体类，Mapper接口类，Mapper xml文件和Example对象等。
3. MBG的版本和Mybatis的版本没有必然的关系。

---
## 2.简单的配置示例：
**1)配置代码**
1. 在src/main/resources中创建一个generator目录，创建一个generatorConfig.xml文件。
2. generatorConfig.xml中配置如下内容：
		<?xml version="1.0" encoding="UTF-8"?>
		<!DOCTYPE generatorConfiguration
		  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
		"http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
		<!-- 配置生成器 -->
		<generatorConfiguration>
			<!-- 配置对象环境 -->
			<context id="MySqlContext" targetRuntime="MyBatis3Simple" defaultModelType="flat">
			
				<!-- 配置起始与结束标识符 -->
				<property name="beginningDemiliter" value="`"/>
				<property name="endingDemiliter" value="`"/>
				
				<!-- 配置注释生成器 -->
				<commentGenerator>
					<property name="suppressDate" value="true"/>
					<property name="addRemarkComments" value="true"/>
				</commentGenerator>
				
				<!-- 必须配置的项，连接数据库 -->
				<jdbcConnection driverClass="com.mysql.jdbc.Driver" connectionURL="jdbc:mysql://localhost:3306/mybatis"
				userId="root" password="zjx1754294529"
				>
				</jdbcConnection>
				
				<!-- 配置生成的实体类位置 -->
				<javaModelGenerator targetPackage="mbg.model" targetProject="src\main\java">
					<property name="trimStrings" value="true"/>
				</javaModelGenerator>
				
				<!-- 配置映射位置 -->
				<sqlMapGenerator targetPackage="mbg.mapper" targetProject="src\main\resources">
				</sqlMapGenerator>
			
				<!-- 配置接口位置 -->
				<javaClientGenerator targetPackage="mbg.dao" type="XMLMAPPER" targetProject="src\main\java">
				</javaClientGenerator>
				
				<!-- 配置数据库表 -->
				<table tableName="%">
					<generatedKey column="id" sqlStatement="Mysql"/>
				</table>
			</context>
		</generatorConfiguration>

**2)上述配置的简单介绍：**
1. `<context id="MySqlContext" targetRuntime="MyBatis3Simple" defaultModelType="flat">`
	- `context`的`targetRuntime`属性设置为MyBatis3Simple是为了避免生成Example相关的代码和方法。如果需要则改为Mybatis3.
	- `defaultModelType="flat"`目的是使每个表只生成一个实体类
2. 数据库使用mysql,所以前后的分隔符都设为"`"
3. 注释生成器的配置：
```
	<commentGenerator>
	<property name="suppressDate" value="true"/>
		<property name="addRemarkComments" value="true"/>
	</commentGenerator>```
配置了生成数据库的注释信息并且禁止在注释中生成日期。
4. `jdbcConnection`简单配置了要连接数据源的信息
5. `javaModelGenerator，sqlMapGenerator，javaClientGenerator`分别配置了实体类，XML映射文件与接口的位置。
`javaClientGenerator`的type使用XMLMAPPER，会使接口和XML完全分离。
6. `table`标签使用`%`来匹配数据库中所有的表。所有表都有主键字段id。
sqlStatement则是配置当前的数据库类型（mysql）。

---
## 2.运行MyBatis Generator
1. 运行MyBatis有很多种方式，比较常用的有4种：
	- 使用java编写代码运行
	- 从命令提示符运行
	- 使用Maven插件运行
	- 使用Eclipse插件运行
2. 每种方式都有各自的优点和缺点

### 方式一：使用java编写代码运行(推荐)
**1)首先需要添加jar包：**
1. 可以手动下载添加：
<https://github.com/mybatis/generator/releases>
2. 也可以使用Maven引入依赖，在pom.xml中添加如下依赖：
		<!-- MyBatis代码生成器 -->
		<dependency>
			<groupId>org.mybatis.generator</groupId>
			<artifactId>mybatis-generator-core</artifactId>
			<version>1.3.3</version>
		</dependency>

**2)编写Generator类代码：**
1. 在项目中添加Mybatis.generator包，并创建Generator类，如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00025MyBatis%E5%AD%A6%E4%B9%A05-01.jpg)
2. 在Generator中添加如下代码：
		package Mybatis.generator;
		
		import java.io.InputStream;
		import java.util.ArrayList;
		import java.util.List;
		
		import org.mybatis.generator.api.MyBatisGenerator;
		import org.mybatis.generator.config.Configuration;
		import org.mybatis.generator.config.xml.ConfigurationParser;
		import org.mybatis.generator.internal.DefaultShellCallback;
		
		//读取MBG配置并生成代码
		public class Generator {
			public static void main(String[] args) throws Exception {
				//MBG执行过程中的警告信息
				List<String> warnings=new ArrayList<String>();
				//生成的代码重复时覆盖原代码
				boolean overwrite =true;
				//读取MBG配置文件
				InputStream is=Generator.class.getResourceAsStream("/generator/generatorConfig.xml");
				ConfigurationParser cp=new ConfigurationParser(warnings);
				Configuration config =cp.parseConfiguration(is);
				is.close();
				
				DefaultShellCallback callback=new DefaultShellCallback(overwrite);
				
				//创建MBG
				MyBatisGenerator myBatisGenerator =new MyBatisGenerator(config, callback, warnings);
				//执行代码生成器
				myBatisGenerator.generate(null);
				//输出警告信息
				for(String warning : warnings){
					System.out.println(warning);
				}
			}
		}
3. 运行测试：
没有什么过多的消息：
![](http://p5ki4lhmo.bkt.clouddn.com/00025MyBatis%E5%AD%A6%E4%B9%A05-02.jpg)
再次运行会提示代码覆盖：
![](http://p5ki4lhmo.bkt.clouddn.com/00025MyBatis%E5%AD%A6%E4%B9%A05-03.jpg)
4. 查看生成效果：
![](http://p5ki4lhmo.bkt.clouddn.com/00025MyBatis%E5%AD%A6%E4%B9%A05-04.jpg)

**3)java编码方式的优缺点：**
1. 优点：
使用方便，generatorConfig.xml配置的一些特殊类，只要再当前项目或者当前项目的classPath中就可以直接使用。
其他方式则需要额外的配置才能使用。
2. 缺点：
和当前项目是绑定在一起的，如果有多个子模块则需要配置多个generatorConfig.xml和Generator.java,不太方便。
3. 综合评价：
虽然有缺陷，但是这种方式出现的问题最少，推荐使用。

### 方式二：使用命令提示符cmd运行
**1)运行前准备：**
1. 只能使用手动导包的方式：
下载地址：
<https://github.com/mybatis/generator/releases>
2. 需要将jar包和generatorConfig.xml文件放到一起
并且将jdbc连接驱动的jar包也放到一起（mysql-connector-java-5.1.35.jar）
3. 修改配置:在配置文件开头添加classPathEntry，代码如下：
		<generatorConfiguration>
		    <!-- 数据库驱动包位置 -->
		    <classPathEntry location="D:/Test/mybatistest/mysql-connector-java-5.1.35.jar" />
			<!-- 配置对象环境 -->
			<context id="MySqlContext" targetRuntime="MyBatis3Simple" defaultModelType="flat">
			<!--后面相同-->
		</generatorConfiguration>
4. 并且在generatorConfig.xml同级目录添加src文件夹，并在src目录下添加java与resources文件夹，添加完后的大致结果是这样的：
![](http://p5ki4lhmo.bkt.clouddn.com/00025MyBatis%E5%AD%A6%E4%B9%A05-05.jpg)

**2)使用cmd自动生成代码：**
在当前目录下【shift】+右键，在此处打开命令窗口。
1. 输入以下命令就可以生成代码：
		java -jar mybatis-generator-core-1.3.3.jar -configfile generatorConfig.xml -overwrite
运行测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00025MyBatis%E5%AD%A6%E4%B9%A05-06.jpg)
查看运行后的目录结构：
![](http://p5ki4lhmo.bkt.clouddn.com/00025MyBatis%E5%AD%A6%E4%B9%A05-07.jpg)
但是这样生成的代码是GBK编码的，一般需要修改为UTF-8编码。
2. 自动生成UTF-8编码代码的命令：
		java -Dfile.encoding=UTF-8 -jar mybatis-generator-core-1.3.3.jar -configfile generatorConfig.xml -overwrite
只需要执行这一条就可以了，上一条不用执行。

**3)参数：**
1. MBG的参数：
	- `-configfile filename`：指定配置文件的名称
	- `-overwriote（可选）`：指定了该参数后，如果生成的Java文件存在同名文件则覆盖。未指定是不会覆盖，会为生成的文件指定一个唯一的名字。
	- `-verbose（可选）`：指定该参数，MBG会使用java日志而不会使用Log4J。
	- `-contextids context1,context2,...(可选)`：指定该参数，后面的context会被执行，context1,context2需要与配置文件context标签的id一致。只有这样指定的context才会被激活。没有参数，所有的context都激活。
	- `-tables table1,table2,...（可选）`：指定该参数，逗号隔开的表会被运行，需与数据库的表名一致，不指定该参数则所有的表都被执行。
2. 使用到的java参数：
	- `-jar xxx.jar`：执行jar包
	- `--Dfile.encoding=xxx`:修改编码
	- `-cp`：需要使用其他jar包时可以将其他jar包添加到classpath下，使用该参数时不能使用`-jar`

**4)命令行方式的优缺点：**
1. 优点(特点)：
完全独立与项目，针对不同的项目配置不同的generatorConfig.xml文件。
命令虽然写起来麻烦，但是可以创建批处理文件，.bat/.sh，只要点击就可以运行了。
2. 缺点：
如果想要和自己实现的gererator类合起来使用的话会比较麻烦。

### 方式三：使用maven插件运行
1. 先在项目(空项目)上【右键】-->run as-->maven build，在goals一栏输入install，或者直接在命令行输入`mvn install`将项目打包到本地仓库。
2. 在pom.xml中添加如下配置：
		<plugin>
		<groupId>org.mybatis.generator</groupId>
		<artifactId>mybatis-generator-maven-plugin</artifactId>
		<version>1.3.3</version>
		<configuration>
			<configurationFile>
				${basedir}/src/main/resources/generator/generatorConfig.xml
			</configurationFile>
			<overwrite>true</overwrite>
			<verbose>true</verbose>
		</configuration>
		<dependencies>
			<dependency>
				<groupId>mysql</groupId>
				<artifactId>mysql-connector-java</artifactId>
				<version>5.1.38</version>
			</dependency>
			<dependency>
				<groupId>Mybatis</groupId>
				<artifactId>simple</artifactId>
				<version>0.0.1-SNAPSHOT</version>
			</dependency>
		</dependencies>
		</plugin>
3. 标签说明：
插件中配置`</dependencies>`比较特殊。将JDBC驱动和当前项目的依赖添加到该插件下，就不用在配置文件中使用classPathEntry来配置了。
添加当前项目的依赖后当前项目中定制了MBG某个实现后就可以直接找到了，不过需要将该项目使用install打包到本地仓库。

**2)运行测试：**
1. 新建一个项目MyBatis/simple2，添加核心配置文件，generatorConfig.xml等。目录如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00025MyBatis%E5%AD%A6%E4%B9%A05-08-0.jpg)
2. 执行install打包：
![](http://p5ki4lhmo.bkt.clouddn.com/00025MyBatis%E5%AD%A6%E4%B9%A05-08.jpg)
3. 在pom.xml中添加上述的插件依赖，注意`<configurationFile>`需要指定配置文件所在的路径。项目【右键】-->run as-->maven build，出现如下框框，输入如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00025MyBatis%E5%AD%A6%E4%B9%A05-09.jpg)
注意那个name是本次操作的名字，执行过一次可以在run的下拉工具栏选择直接运行。
点击运行:
![](http://p5ki4lhmo.bkt.clouddn.com/00025MyBatis%E5%AD%A6%E4%B9%A05-10.jpg)
会有很长的一段。
4. 执行完毕后再查看目录：
![](http://p5ki4lhmo.bkt.clouddn.com/00025MyBatis%E5%AD%A6%E4%B9%A05-11.jpg)
已经自动生成了代码，但是要记得在mybatis核心配置文件中配置对应的映射。
5. 自动生成，不会生成测试类。

**4)Maven插件方式的优缺点：**
1. 优点：
比较简单，使用方便
2. 缺点：
与第一种方式一样，和项目绑定在一起。
不支持代码合并（前两种也不支持）

### 方式四：使用Ecilpse插件自动生成
**1)使用Eclipse插件运行的优点**
1. 上述三种方式的共同缺点：不支持代码合并
2. 自动生成代码注释中会有一个@mbggerated的标志：
![](http://p5ki4lhmo.bkt.clouddn.com/00025MyBatis%E5%AD%A6%E4%B9%A05-12.jpg)
这个标志的意思是这个字段或方法是自动生成的，重新生成时会被覆盖。若没有这个标签，则不会被覆盖。但是前三种方法不支持该标签。
3. 四种运行方式只有Eclipse插件的方式支持@mbggenerator注解。

**2)安装Eclipse插件：**
1. 仍然是使用这个网址下载：
<https://github.com/mybatis/generator/releases>
选择eclipse并下载：
![](http://p5ki4lhmo.bkt.clouddn.com/00025MyBatis%E5%AD%A6%E4%B9%A05-13.jpg)
2. 打开【help】-->【Install New Software】-->【add】,添加刚刚下载的文件，并指定名字，然后选中所有的MyBatis Generator（不要选带尾巴那个，否则会出错）,然后一路next，finish,会下载一些东西，然后让你重启。
![](http://p5ki4lhmo.bkt.clouddn.com/00025MyBatis%E5%AD%A6%E4%B9%A05-14-1.jpg)
3. 成功安装并重启后在xml文件点击右键会有这个功能：
![](http://p5ki4lhmo.bkt.clouddn.com/00025MyBatis%E5%AD%A6%E4%B9%A05-15.jpg)

**3)使用Eclipese插件自动生成代码：**
1. 先将上面自动生成的代码全都删除：
![](http://p5ki4lhmo.bkt.clouddn.com/00025MyBatis%E5%AD%A6%E4%B9%A05-16.jpg)
2. 修改generatorConfig.xml：
JDBC驱动需要classPathEntry进行配置，还需要在context下添加一个javaFileEncoding的property标签配置为UTF-8：
		....
		<!-- 数据库驱动包位置 -->
		<classPathEntry location="D:/Test/mybatistest/mysql-connector-java-5.1.35.jar" />
	
		<!-- 配置对象环境 -->
		<context id="MySqlContext" targetRuntime="MyBatis3Simple" defaultModelType="flat">
		
			<!-- 配置起始与结束标识符 -->
			<property name="beginningDemiliter" value="`"/>
			<property name="endingDemiliter" value="`"/>
			<property name="javaFileEncoding" value="UTF-8"/>
		....
然后修改路径添加项目名：
将`src\main\java`,改为`simple2\src\main\java`,以此类推。
3. generatorConfig.xml，右键，选择Generator MyBatis/Ibatis Artifacts,运行，结果如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00025MyBatis%E5%AD%A6%E4%B9%A05-17.jpg)

---
## 3.xml配置详解
**1)大体配置内容介绍**
1. MBG的XML文件头：
		<?xml version="1.0" encoding="UTF-8"?>
		<!DOCTYPE generatorConfiguration
		  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
		"http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
`mybatis-generator-config_1_0.dtd`定义了该配置文件中所有标签和属性的用法及限制。
2. xml文件头之后需要加上XML文件的根节点：
generatorConfiguration:
		<generatorConfiguration>
		<！--所有的配置内容-->
		</generatorConfiguration>
1,2的两部分是MBG配置文件的必须的部分。
后面是自定义配置，但是要按照严格的顺序进行配置。
3. 后面是generatorConfiguration的三个子标签，依次为：properties，classPathEntry和context。其中context最为重要，剩下的那些标签都是context的子标签。
4. **properties标签：**
用来指定外部的属性元素，最多可以配置1个，也可以不配置。
可以指定一个需要在配置中解析使用的外部属性文件,可以在配置中使用${property}来引用属性文件中的属性值。在后面配置JDBC时很有用。
property标签包含了两个属性：url和resource,**但是只能使用其中一个属性，同时使用会报错**
	- resource：指定classpath下的属性文件。
	- url：指定文件系统上的特点位置。
5. **classPathEntry标签：**
可以配置多个也可以不配置。
最常用的用法是通过属性location指定JDBC驱动路径：
`<classPathEntry location="D:/Test/mybatistest/mysql-connector-java-5.1.35.jar" />`
还可以用来指定rootClass属性配置类所在的jar包。
6. **context标签**
用于指定生成一组对象环境，如指定连接的数据库，要生成对象的类型和要生成的表。运行MBG时还可以指定要运行的context。

**2)context属性和子标签：**
1. 属性：
	- id：必填属性，唯一确定context,可以在运行MBG时使用。
	- defaultModelType:定义了MBG如何生成实体类，有以下可选值：
		- conditional：默认值，如果一个表主键只有一个字段，那么不会为该字段生成单独的实体类，会将该字段合并到基本实体类中。(先看后面的hierarchical属性再理解)
		- flat：为每张表生成一个实体类，包含了表中的所有字段，容易使用。
		- hierarchical：如果表有主键，该模型会产生一个单独的主键实体类，如果表还有BLOB字段，则会生成一个包含所有BLOB字段的单独实体类。然后其他剩下的字段生成一个单独的实体类。MBG会在所有生成的实体类之间维护一个继承关系。
	- targetRuntime：用于指定代码运行时的环境
		- MyBatis3：默认值，会生成与Example相关的方法
		- MyBatis3Simle：不会生成与Example相关的方法
	- introspectedColumnImpl：该参数可以指定扩展org.mybatis.generator.api.Introspected Column类的实现。
2. 一般情况下的context配置：
`<context id="Mysql" defaultModelType="flat">`
如果不希望生成和Example查询有关的内容这样配置：
`<context id="Mysql" targetRuntime="MyBatis3Simle" defaultModelType="flat">`
3. context的子标签：其它标签基本都是context的子标签。
**子标签的使用有严格的顺序**：
	- property（0个或多个）
	- plugin(0个或多个)
	- commentGenerator（0个或1个）
	- jdbcConnection（1个）
	- javaTypeResolver(0或1个)
	- javaModelGenerator(1个)
	- sqlMapGenerator(0或1个)
	- javaClientGenerator(0个或1个)
	- table(1个或多个)

### property,plugin,commentGenerator标签
**1)property标签：**
共6个name属性值，三个与分隔符有关，三个与分隔符无关。
1. 关于数据库分隔符：
如有一个表名为user info(中间有一个空格)，查询时直接使用这个表名会出错，这时就需要分隔符 \` 将表名括起来，Mysql中使用 \`user info\`，而在SQL Sever中则使用 [user info]。将内容作为一个整体进行处理。
2. property标签与分隔符相关的属性值(name属性的值)：
	- autoDelimitKeyword:字动给关键字添加分隔符属性。字段名或表名和MBG中的关键字列表的某一关键字一样时，会自动加上 \` 做分隔符。
	- beginningDelimiter：开始的分隔符
	- endingDelimiter：结束分隔符
3. Mysql的配置：
		<context...>
		<property name="autoDelimitKeyword" value="true"/>
		<property name="beginningDemiliter" value="`"/>
		<property name="endingDemiliter" value="`"/>
		...
		</context>
4. property的另外三个与分隔符无关的属性：
	- javaFileEncoding：java文件的编码，默认当前运行环境的编码。
	- javaFormatter：不常用，格式化java代码
	- xmlFormatter：不常用，格式化xml代码

 

---


---