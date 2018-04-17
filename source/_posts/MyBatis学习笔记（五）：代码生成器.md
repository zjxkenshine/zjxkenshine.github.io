---
title: MyBatis学习笔记（五）：代码生成器
date: 2018-03-29 20:15:28
tags: MyBatis
categories: J2EE框架

---
## 0.学习准备
1. 学习资料：
书本：《MyBatis从入门到精通》
视频：某培训班视频
资料：[MyBatis中文文档](http://www.mybatis.org/mybatis-3/zh/index.html)
[MBG官方文档](http://www.mybatis.org/generator/)

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
				userId="root" password=""
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

### 方式一：java编写代码运行(推荐)
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

### 方式二：命令提示符cmd运行
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

### 方式四：用Ecilpse插件运行（推荐）
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
共6个属性，三个与分隔符有关，三个与分隔符无关。
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

**2)plugin标签：**
1. 用来定义一个插件，用于扩展或者修改MBG生成的代码，不常用，可以配置0个或者多个，个数不受限制。
2. 只有一个Type标签，其中填插件的全限定名。
3. 常用的有缓存插件，序列化插件，RowBounds插件，ToString插件等。
4. 详情可以查看官方文档：
<http://www.mybatis.org/generator/plugins.html>

**3)commentGenenrator标签：**
1. 用来配置如何生成注释信息，最多可以配置一个，也可以不配置。
2. 该标签有一个可选属性：type,可以指定用户的实现类，该类需要实现org.mybatis.generator.api.CommentGenerator接口，且必须有一个默认的构造方法。
type属性，默认值为DEFAULT,默认实现org.mybatis.generator.internal.DefaultCommentGenerator。
3. 默认的实现类有三个属性，需要通过property属性进行配置：
	- suppressAllComments：阻止生成注释，默认值为false。
	- suppressData：阻止生成的注释包含时间戳，默认false。
	- addRemarComment：注释是否添加数据库表的备注信息，默认false。
4. 一般情况下MBG生成的时间注释信息没有任何价值，所以一般情况下配置如下：
		<commentGenerator>
			<property name="suppressDate" value="true"/>
			<property name="addRemarkComments" value="true"/>
		</commentGenerator>
5. 如果不满意默认实现，可以自定义实现默认的注释，需要实现CommentGenerator接口，如下。

**4)自定义commentGenenrator标签：**
1. 在Simple的mybatis.generator包下创建MyCommentGenerator.java。
2. MyCommentGenerator.java代码如下：
		package Mybatis.generator;
		
		import java.util.Properties;
		import org.mybatis.generator.api.IntrospectedColumn;
		import org.mybatis.generator.api.IntrospectedTable;
		import org.mybatis.generator.api.dom.java.Field;
		import org.mybatis.generator.config.PropertyRegistry;
		import org.mybatis.generator.internal.DefaultCommentGenerator;
		import org.mybatis.generator.internal.util.StringUtility;
		
		public class MyCommentGenerator extends DefaultCommentGenerator{
			//默认实现类的可配置参数没有提供给子类可以访问的方法,需要自己定义
			private boolean suppressAllComments;  //阻止生成注释
			private boolean addRemarkComments;		//是否生成数据库表的注释
			//设置用户配置的参数
			public void addConfigurationProperties(Properties properties){
				//调用父类方法保证父类方法可以正常使用
				super.addConfigurationProperties(properties);
				//获取supressAllComment参数值
				suppressAllComments = Boolean.parseBoolean(properties.getProperty(PropertyRegistry.COMMENT_GENERATOR_SUPPRESS_ALL_COMMENTS));
				//获取addRemarkComments参数值
				addRemarkComments = Boolean.parseBoolean(properties.getProperty(PropertyRegistry.COMMENT_GENERATOR_ADD_REMARK_COMMENTS));
			}
			
			//给字段添加注释信息
			public void addFieldComment(Field field,IntrospectedTable introspectedTable,IntrospectedColumn introspectedColumn){
				//如果阻止生成注释则直接返回
				if(suppressAllComments){
					return;
				}
				//文档注释开始
				field.addJavaDocLine("/**");  //开始
				//获取数据库字段的备注信息
				String remarks =introspectedColumn.getRemarks();
				//根据参数和备注信息判断是否添加备注信息
				if(addRemarkComments&&StringUtility.stringHasValue(remarks)){
					String[] remarkLinesStrings=remarks.split(System.getProperty("line.separator"));
					for(String remarkLine:remarkLinesStrings){
						field.addJavaDocLine(" * "+remarkLine);
					}
				}
				//注释中保留数据库字段名
				field.addJavaDocLine(" * "+introspectedColumn.getActualColumnName());
				field.addJavaDocLine(" */");
			}
		}
3. 在generatorConfig.xml中如下配置：
		<commentGenerator type="Mybatis.generator.MyCommentGenerator">
			<property name="suppressDate" value="true"/>
			<property name="addRemarkComments" value="true"/>
		</commentGenerator>
4. 运行MBG，并查看生成的model类：
![](http://p5ki4lhmo.bkt.clouddn.com/00025MyBatis%E5%AD%A6%E4%B9%A05-18.jpg)
5. 注意：
MBG是通过JDBC的DatabaseMetaData方式来回去数据库表和字段的备注信息的。常用数据库只有MySqlZ完全支持，Oracal需要配置后才完全支持(配置方式在下一个标签)。

### jdbcConnection,javaTypeResolver和javaModelGenerator标签
**1)jdbcConnection标签**
1. 用于指定要连接的数据库信息，必选，并且只能一个。
2. 如果JDBC驱动不在classpath下，就要通过classPathEntry标签引入jar包。
所以最好将JDBC驱动放到classpath路径下。
3. 属性：
必选属性：
--`driverClass`：访问数据库的jdbc驱动程序的全限定名
--`connectionURL`：访问数据库的JDBC连接URL
可选属性：
--`userId`：访问数据库的用户ID。
--`password`：访问数据库密码。
4. 基本的配置：
		<jdbcConnection driverClass="com.mysql.jdbc.Driver" connectionURL="jdbc:mysql://localhost:3306/mybatis" userId="root" password="">
		</jdbcConnection>
5. 配置Oracal以获取注释信息(上一个标签)：
		<jdbcConnection driverClass="" connectionURL="" userId="" password="">
			<property name="remarksReporting" value="true">
		</jdbcConnection>

**2)javaTypeResolver标签**
1. 该标签用来指定JDBC类型和Java类型时如何转换的，最多可以配置一个。
2. 和commentGenerator一样有一个type属性且默认实现为DEFAULT。可以修改，但是不建议修改。
3. 还有一个可配置的property标签，可以配置一个名为`forceBigDecimals`的属性，可以控制是否强制将DECIMAL和NUMERIC类型的JDBC字段转换为java类型的java.math.BigDecimal，默认为false,一般不需要配置。
4. 默认转换规则：
精度>0且长度>18，使用java.math.BigDecimal。
精度=0且长度在10~18,使用java.lang.Long。
精度=0且长度范围在5~9,使用java.lang.Integer。
精度=0且长度<5,使用java.lang.Short。
如果配置了`forceBigDecimals`则在任何情况下都是由BigDecimal。
5. 常用配置如下(一般情况下就不用使用这标签配置)：
		<javaTypeResolver>
			<property name="forceBigDecimals" values="false" />
		</javaTypeResolver>

**3)javaModelGenerator标签**
1. 用来控制生成的属性值。最好是context标签中配置defaultModelType属性配置为flat,使一个表只生成一个实体类。这样比较好控制。该标签必须有且只能有一个。
2. 有两个必选属性：
	- `targetPackage`：生成实体类存放的包名，可能会受到其他配置的影响。
	- `targetProject`：指定目标的项目路径，可以用相对路径或者绝对路径。
3. property子标签属性：
	- `constructorBased`：该属性只对MyBatis3有效，设为true则会使用构造方法入参，false则是使用setter方法入参。
	- `enableSubPackages`：如果为true，MBG会根据`catalog`和`schema`来生成子包，默认为false，使用targetPackage属性值。
	- `imutable`：配置实体类属性是否为可变，默认为false，可以改变。改为true，那么constructorBased不管设置成什么，都会使用构造方法入参。
	- `rootClass`：设置所有实体类的基类。设置的话要使用类的全限定名称。(可以通过classPathEntry引入jar包，或者classpath方式)，引入jar包能加载rootClass则MBGb不会覆盖和父类中相同的属性。
	- `trimString`：去掉空格。默认为false，不去空格。
4. 配置练习：
		<javaModelGenerator targetPackage="mbg.model" targetProject="src\main\java">
			<property name="enableSubPackages" value="false"/>
			<property name="trimStrings" value="false"/>
		</javaModelGenerator>


### sqlMapGenerator,javaClientGenerator和table标签
**1)sqlMapGenerator标签**
1. 用于配置SQL映射生成器（Mapper.xml文件）的属性，可选且最多配置1个。
2. targetRuntime设置为Mybatis3且javaClientGenerator配置需要XML时才必须配置一个。
如果没有配置javaClientGenerator时有以下规则：
	- 指定了一个sqlMapGenerator，那么MBG只生成XML的sql映射文件和实体类。
	- 如果没有指定sqlMapGenerator，则只生成实体类。
3. 只有两个必选属性(和实体类的差不多)：
	- `targetPackage`：生成映射文件存放的包名，可能会受到其他配置的影响。
	- `targetProject`：指定目标`targetPackage`的项目路径，可以用相对路径或者绝对路径。
4. 还有一个可选的prpoerty子标签属性`enableSubPackages`:
如果为true，MBG会根据`catalog`和`schema`来生成子包，默认为false，使用targetPackage属性值。
5. 基本配置示例：
		<!-- 配置映射位置 -->
		<sqlMapGenerator targetPackage="mbg.mapper" targetProject="src\main\resources">
			<property name="enableSubPackages" value="false"/>
		</sqlMapGenerator>

**2)javaClientGenerator标签**
1. 该标签用于配置java客户端生成器（Mapper接口）的属性，该标签可选且最多配置一项。不配置该标签则不会生成Mapper接口。
2. 必选属性(三个)：
	- `type`：用于选择客户端的代码(接口)生成器，可以自己实现，需要继承org.mybatis.generator.codegen.AbstractJavaClientGenerator类，且必须有一个默认为空的构造方法。
	根据targetRuntime的设置分为两类：
	Mybatis3：
		- `ANNOTATRFMAPPER`：基于注解的Mapper，不会有对应的xml文件生成
		- `MIXEDMAPPER`：xml和注解混合模式，SQL provider会被XML代替。
		- `XMLMAPPER`：所有方法都在XML中，接口用依赖Xml文件。
		
	Mybatis3Simple：
		- `ANNOTATRFMAPPER`：基于注解的Mapper，不会有对应的xml文件生成
		- `XMLMAPPER`：所有方法都在XML中，接口用依赖Xml文件。
	- `targetPackage`：生成Mapper文件存放的包名，可能会受到其他配置的影响。
	- `targetProject`：指定目标`targetPackage`的项目路径，可以用相对路径
3. 还支持几个peoperty子标签，但是不常用书上为介绍。
4. 在一般配置中的Type推荐使用`XMLMAPPER`,将接口和XML完全分离。
常用配置如下：
		<javaClientGenerator targetPackage="mbg.dao" type="XMLMAPPER" targetProject="src\main\java">
		</javaClientGenerator>

**3)table标签及属性介绍：**
1. table标签非常重要，用于配置需要通过内省的数据库的表。
只有通过该标签配置过的表才能通过前面的配置生成相应的代码。至少要配置一个，也可以配置多个。
2. 必须配置的属性：tableName
填写需要生成的表名，可以使用通配符匹配多个表，也可以使用下列代码匹配所有表：
		<table tableName="%"></table>
3. 可选属性:
		1.schema： 数据库的schema,指定该标签表名会变为schema.tableName的形式,可使用通配符。
		2.catalog： 数据库的catalog，指定该标签表名会变为catalog.tableName的形式。
		3.alias： 为数据表设置的别名，如果设置了alias，那么生成的所有的SELECT SQL语句中，列名会变成：alias_actualColumnName(别名_实际列名)
		4.domainObjectName： 生成的对象实体类的名字，如果不设置，直接使用表名作为对象类的名字；可以设置为somepck.domainName，那么会自动把domainName类再放到somepck包里面；
        5.enableInsert（默认true）： 指定是否生成insert语句；
        6.enableSelectByPrimaryKey（默认true）： 指定是否生成按照主键查询对象的语句（就是getById或get）；
        7.enableSelectByExample（默认true）：MyBatis3Simple为false，指定是否生成动态查询语句；
        8.enableUpdateByPrimaryKey（默认true）：指定是否生成按照主键修改对象的语句（即update)；
        9.enableDeleteByPrimaryKey（默认true）：指定是否生成按照主键删除对象的语句（即delete）；
        10.enableDeleteByExample（默认true）：MyBatis3Simple为false，指定是否生成动态删除语句；
        11.enableCountByExample（默认true）：MyBatis3Simple为false，指定是否生成动态查询总条数语句（用于分页的总条数查询）；
        12.enableUpdateByExample（默认true）：MyBatis3Simple为false，指定是否生成动态修改语句（只修改对象中不为空的属性）；
        13.modelType：参考context元素的defaultModelType，相当于覆盖；
        14.delimitIdentifiers：参考tableName的解释，注意，默认的delimitIdentifiers是双引号，如果类似MYSQL这样的数据库，使用的是`（反引号，那么还需要设置context的beginningDelimiter和endingDelimiter属性）
        15.delimitAllColumns：设置是否所有生成的SQL中的列名都使用标识符引起来。默认为false，delimitIdentifiers参考context的属性
4. table标签可用的property子标签属性：
		1.constructorBased： 该属性只对MyBatis3有效，设为true则会使用构造方法入参，false则是使用setter方法入参(和javaModelCenerator的该属性一样)
		2.ignoreQualifiersAtRuntime： 默认为false，如果设置为true，在生成的SQL中，table名字不会加上catalog或schema前缀
		3.immutable： 参考javaModelCenerator的该属性，一样
		4.modelOnly： 是否只生成实体类，会覆盖enablexxx方法，不会生成CRUD方法，如果设置为true，只生成实体类，如果还配置了sqlMapGenerator，那么在mapper XML文件中，只生成resultMap元素
		5.rootClass： 参考javaModelGenerator 的 rootClass 属性
		6.rootInterface： 参考javaClientGenerator 的  rootInterface 属性
		7.runtimeCatalog： 如果设置了运行时的Catalog，那么在生成的SQL中，使用该指定的catalog，而不是table元素上的catalog
		8.runtimeSchema： 如果设置了运行时的Schema，那么在生成的SQL中，使用该指定的schema，而不是table元素上的schema 
		9.runtimeTableName： 如果设置了运行时的TableName，那么在生成的SQL中，使用该指定的tablename，而不是table元素上的tablename 
		10.selectAllOrderByClause： 只针对MyBatis3Simple有用，如果选择的runtime是MyBatis3Simple，那么会生成一个SelectAll方法，如果指定了selectAllOrderByClause，那么会在该SQL中添加指定的这个order条件；
		11.useActualColumnNames： 如果设置为true，生成的model类会直接使用column本身的名字，而不会再使用驼峰命名方法，比如BORN_DATE，生成的属性名字就是BORN_DATE,而不会是bornDate
		12.useColumnIndexes： 设为true,MBG生成resultMap时会使用列的索引而不是结果中列的顺序。
		13.useCompounfPropertyNames： 如果为true，MBG生成属性名的时候将列名和备注名连接起来。
5. 除了property子标签外还包括以下子标签：
	- generatedKey（0个或者1个）
	- columnRnamingRule(0个或者1个)
	- columnoverride(0个或者多个)
	- ignoreColumn（0个或者多个）

**4)table的generatedKey子标签详解**
1. generateKey标签功能：
用来指定自动生成的主键的属性。（identity字段或者sequences序列），如果指定了该标签MGB将在生成的Insert的SQL映射文件中插入一个`<selectKey>`标签，这个标签非常重要且只能配置一个。
2. 包含两个必选属性：
`column`：生成列的列名
`sqlStatement`：返回值的SQL语句，如果这是一个identity列，则可以使用其中一个预定义的特殊值，预定义值如下：
	- `Cloudscape`:相当于selectKey的SQL为： VALUES IDENTITY_VAL_LOCAL()
	- `DB2`:相当于selectKey的SQL为： VALUES IDENTITY_VAL_LOCAL()
	- `DB2_MF`:相当于selectKey的SQL为：SELECT IDENTITY_VAL_LOCAL() FROM SYSIBM.SYSDUMMY1
	- `Derby`:相当于selectKey的SQL为：VALUES IDENTITY_VAL_LOCAL()
	- `HSQLDB`:相当于selectKey的SQL为：CALL IDENTITY()
	- `Informix`:相当于selectKey的SQL为：select dbinfo('sqlca.sqlerrd1') from systables where tabid=1
	- `MySql`:相当于selectKey的SQL为：SELECT LAST_INSERT_ID()
	- `SqlServer`:相当于selectKey的SQL为：SELECT SCOPE_IDENTITY()
	- `SYBASE`:相当于selectKey的SQL为：SELECT @@IDENTITY
	- `JDBC`:相当于在生成的insert元素上添加useGeneratedKeys="true"和keyProperty属性
3. 包含两个可选属性：
	- `indetity`：设置为true时会被标记为indentity列，并且selectKey标签会被插入在Insert标签（order=AFTER）。设置为false时selectKey会插入到Insert之前（oracal序列），默认为false.
	即使Type属性指定为post,仍需要将该属性列设为true,这会作为MBG插入列表中删除该列的标识
	- `type`：type=post且indetity=true时生成的selectKey的order=AFTER;type=pre,indetity只能为false,生成的selectKey中order=BEFORE.
4. 配置示例1（针对MySql,SQL Server等自增类型主键）：
		<table tableName="user login info" domainObjectName="UserLoginInfo">
			<generatedKey column="id" sqlStatement="Mysql"/>
		</table>
5. 配置示例2（针对Oracal序列）:
		<table tableName="user login info" domainObjectName="UserLoginInfo">
			<generatedKey column="id" sqlStatement="SELECT SEQ_ID.nextval from xxx"/>
		</table>

**5)table的其他三个子标签详解**
1. **columnRenamingRole标签：**
可以在生成列之前对列进行重命名。对于那些由于存在同一个前缀字段而想在生成属性名时去除前缀的表非常有用。
有一个必选属性：
`searchString`：用于定义将要被替换的字符串的正则表达式。
有一个可选属性：、
`replaceString`：替换搜索到的每一个匹配项。没有指定则使用空字符串，代表删除。
简单示例：去掉每个字段开头的"stu_"
		<columnRenamingRole searchString="^stu_" replaceString="">
2. 其它标签会对columnRenamingRole标签产生影响：
columnOverride(下个标签)匹配一列时columnRenamingRole标签的配置会被忽略。
table的Property属性useActualColumnNames也会对此标签产生影响。（详情查看[官方文档](http://www.mybatis.org/generator/plugins.html)）
3. **columnOverride标签：**
将某些默认计算的属性值更改为指定的值。可选且可以配置多个。
有一个必选属性：
`column`：表示要重写的列名。
有多个可选属性（可通过property子标签配置）：
	- `property`：要使用的java属性的名称。如果没有指定，MBG会根据列名生成。
	- `javaType`：列的属性值为完全限定的Java类型，可以覆盖javaTypeResolver设置的基本类型
	- `jdbcType`：列的JDBC类型，可以覆盖javaTypeResolver设置的类型。
	- `typerHandler`：类型处理器（不会判断是否可用，所以要正确指定）
	- `delimitrdColimnName`：是否给列名增加分隔符（是关键字时选true，其他情况如空格会自动处理）
4. columnOverride配置示例：
		<columnOverride column="LONG_VARCHAR_FILED" javaType="java.lang.String" jdbcType="VARCHAR">
		</columnOverride>
5. **ignoreColumn标签：**
设置一个MGB忽略的列，如果设置了改列，那么在生成的domain中，生成的SQL中，都不会有该列出现。
有一个必选标签：
`column`:指定要忽略的列的名字；
有一个可选标签：
`delimitedColumnName`：参考table元素的delimitAllColumns配置,是否区分大小写，默认为false，不区分大小写。

---
## 4.Example的使用
**1)配置并生成含有Example的代码**
1. 在MBG的context中将targetRuntime配置为MyBatis3时MBG会生成和Example相关的对象方法。
下面是书上的使用Country表进行测试的代码。
2. 配置(Eclipse插件的配置)：
		<?xml version="1.0" encoding="UTF-8"?>
		<!DOCTYPE generatorConfiguration
		  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
		"http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
		
		<!-- 配置生成器 -->
		<generatorConfiguration>
			<classPathEntry location="D:/Test/mybatistest/mysql-connector-java-5.1.35.jar" />
			<!-- 配置对象环境,注意这里是 MyBatis3才能生成Example代码 -->
			<context id="MySqlContext" targetRuntime="MyBatis3" defaultModelType="flat">
				<property name="javaFileEncoding" value="UTF-8"/>
				<!-- 配置注释生成器 -->
				<commentGenerator  type="Mybatis.generator.MyCommentGenerator">
					<property name="suppressDate" value="true"/>
					<property name="addRemarkComments" value="true"/>
				</commentGenerator>
				<!-- 必须配置的项，连接数据库 -->
				<jdbcConnection driverClass="com.mysql.jdbc.Driver" connectionURL="jdbc:mysql://localhost:3306/mybatis"
				userId="root" password="zjx1754294529"
				>
				</jdbcConnection>
				<!-- 配置生成的实体类位置 -->
				<javaModelGenerator targetPackage="mbg.model" targetProject="simple2\src\main\java">
					<property name="trimStrings" value="true"/>
				</javaModelGenerator>
				<!-- 配置映射位置 -->
				<sqlMapGenerator targetPackage="mbg.mapper" targetProject="simple2\src\main\resources">
				</sqlMapGenerator>
				<!-- 配置接口位置 -->
				<javaClientGenerator targetPackage="mbg.dao" type="XMLMAPPER" targetProject="simple2\src\main\java">
				</javaClientGenerator>
				<!-- 配置数据库表 -->
				<table tableName="country">
					<generatedKey column="id" sqlStatement="Mysql"/>
				</table>
			</context>
		</generatorConfiguration>
3. 使用插件生成代码,查看dao层的接口方法，发现多了很多方法(和Example相关的)：
![](http://p5ki4lhmo.bkt.clouddn.com/00025MyBatis%E5%AD%A6%E4%B9%A05-19.jpg)
且Model中多了个CountyExample对象，里面有非常多方法：
![](http://p5ki4lhmo.bkt.clouddn.com/00025MyBatis%E5%AD%A6%E4%B9%A05-20-1.jpg)
![](http://p5ki4lhmo.bkt.clouddn.com/00025MyBatis%E5%AD%A6%E4%B9%A05-20-2.jpg)
除此之外还有超级多和字段相关的方法，用于代替SQL语句。

**2)测试selectByExample方法（了解Example的基本用法）：**
1. 创建测试包mbg.mappr,并将Simple的BaseMapperTest.java及核心配置文件，log4j配置文件等复制到Simple2，并做相应的修改。
2. 在mbg.mappr包下创建CountryMapperTest.java并添加testExample方法，代码如下：
		package mbg.mapper;
		
		import java.util.List;
		import mbg.dao.CountryMapper;
		import mbg.model.Country;
		import mbg.model.CountryExample;
		import org.apache.ibatis.session.SqlSession;
		import org.junit.Test;
		import Mybatis.simple.base.BaseMapperTest;
		
		public class CountryMapperTest extends BaseMapperTest{
			@Test
			public void testExample(){
				//获取sqlSession
				SqlSession sqlSession=getSqlSession();
				try{
					//获取sqlSession
					CountryMapper countryMapper =sqlSession.getMapper(CountryMapper.class);
					//创建Example对象
					CountryExample example=new CountryExample();
					//设置排序规则
					example.setOrderByClause("id desc,countryname asc");
					//设置是否distinct去重
					example.setDistinct(true);
					//创建条件
					CountryExample.Criteria criteria=example.createCriteria();
					//id>=1
					criteria.andIdGreaterThanOrEqualTo((byte) 1);
					//id<4
					criteria.andIdLessThan((byte) 4);
					//countrycode like '%u%'
					//注意此处必须自己写通配符
					criteria.andCountrycodeLike("%U%");
					//OR的情况
					CountryExample.Criteria or=example.or();
					//countryname=中国
					or.andCountrynameEqualTo("中国");
					//执行查询
					List<Country> countList=countryMapper.selectByExample(example);
					for(Country coun:countList){
						System.out.println(coun);
					}
				}finally{
					//关闭sqlSession
					sqlSession.close();
				}
			}
		}
3. 测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00025MyBatis%E5%AD%A6%E4%B9%A05-21.jpg)
最后的那仨是因为未在model类添加toString方法。
4. 使用了Example就可以不用自己写简单的Java语句了。
要注意的是使用like和or时的情况。

**3)测试更新的两个方法：**
1. 和UPDATE有关的有两个方法：
updateByExample和updateByExampleSelective,两个方法的区别是对象属性为空时，第一个方法会将值更新为null,而第二个方法不会。
2. 测试updateByExampleSelective：
添加测试方法：
		@Test
		public void testExampleUpdateByExampleSelective(){
			//获取sqlSession
			SqlSession sqlSession=getSqlSession();
			try{
				//获取sqlSession
				CountryMapper countryMapper =sqlSession.getMapper(CountryMapper.class);
				//创建Example对象
				CountryExample example=new CountryExample();
				//创建条件（只能有一个createCriteria）
				CountryExample.Criteria criteria=example.createCriteria();
				//条件为id>2，更新id>2的
				criteria.andIdGreaterThan((byte) 2);
				//创建一个要设置的对象
				Country country=new Country();
				//设置国家名字为China
				country.setCountryname("China");
				//执行更新
			//	countryMapper.updateByExample(country, example);  //报错说id不能为空
				countryMapper.updateByExampleSelective(country, example);
				//执行查询
				List<Country> countList=countryMapper.selectByExample(example);
				//将符合条件的结果输出
				for(Country coun:countList){
					System.out.println(coun);
				}
			}finally{
				sqlSession.rollback();
				sqlSession.close();
			}		
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00025MyBatis%E5%AD%A6%E4%B9%A05-22.jpg)

**4)测试删除和统计的方法：**
1. 删除方法：deleteByExample
统计方法：countyByExample
2. 测试代码：
		@Test
		public void testCountDeleteByExample(){
			//获取sqlSession
			SqlSession sqlSession=getSqlSession();
			try{
				//获取sqlSession
				CountryMapper countryMapper =sqlSession.getMapper(CountryMapper.class);
				//创建Example对象
				CountryExample example=new CountryExample();
				//创建条件（只能有一个createCriteria）
				CountryExample.Criteria criteria=example.createCriteria();
				//删除所有id>2的国家
				criteria.andIdGreaterThan((byte) 2);
				//执行删除
				countryMapper.deleteByExample(example);
				//使用CountByExample方法查看删除后是否为0条
				Assert.assertEquals(0, countryMapper.countByExample(example));
			}finally{
				sqlSession.rollback();
				sqlSession.close();
			}
		}
3. 测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00025MyBatis%E5%AD%A6%E4%B9%A05-23.jpg)

---
## 5.MBG最全配置介绍

	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE generatorConfiguration
	  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
	"http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
	<!-- 配置生成器 -->
	<generatorConfiguration>
	<!-- 可以用于加载配置项或者配置文件，在整个配置文件中就可以使用${propertyKey}的方式来引用配置项
	    resource：配置资源加载地址，使用resource，MBG从classpath开始找，比如com/myproject/generatorConfig.properties        
	    url：配置资源加载地质，使用URL的方式，比如file:///C:/myfolder/generatorConfig.properties.
	    注意，两个属性只能选址一个;
	    
	    另外，如果使用了mybatis-generator-maven-plugin，那么在pom.xml中定义的properties都可以直接在generatorConfig.xml中使用
	<properties resource="" url="" />
	 -->
	 
	 <!-- 在MBG工作的时候，需要额外加载的依赖包
	    location属性指明加载jar/zip包的全路径
	<classPathEntry location="/Program Files/IBM/SQLLIB/java/db2java.zip" />
	  -->
	  
	<!-- 
	    context:生成一组对象的环境 
	    id:必选，上下文id，用于在生成错误时提示
	    defaultModelType:指定生成对象的样式
	        1，conditional：类似hierarchical；
	        2，flat：所有内容（主键，blob）等全部生成在一个对象中；
	        3，hierarchical：主键生成一个XXKey对象(key class)，Blob等单独生成一个对象，其他简单属性在一个对象中(record class)
	    targetRuntime:
	        1，MyBatis3：默认的值，生成基于MyBatis3.x以上版本的内容，包括XXXBySample；
	        2，MyBatis3Simple：类似MyBatis3，只是不生成XXXBySample；
	    introspectedColumnImpl：类全限定名，用于扩展MBG
	-->
	<context id="mysql" defaultModelType="hierarchical" targetRuntime="MyBatis3Simple" >
	
	    <!-- 自动识别数据库关键字，默认false，如果设置为true，根据SqlReservedWords中定义的关键字列表；
	        一般保留默认值，遇到数据库关键字（Java关键字），使用columnOverride覆盖
	     -->
	    <property name="autoDelimitKeywords" value="false"/>
	    <!-- 生成的Java文件的编码 -->
	    <property name="javaFileEncoding" value="UTF-8"/>
	    <!-- 格式化java代码 -->
	    <property name="javaFormatter" value="org.mybatis.generator.api.dom.DefaultJavaFormatter"/>
	    <!-- 格式化XML代码 -->
	    <property name="xmlFormatter" value="org.mybatis.generator.api.dom.DefaultXmlFormatter"/>
	    
	    <!-- beginningDelimiter和endingDelimiter：指明数据库的用于标记数据库对象名的符号，比如ORACLE就是双引号，MYSQL默认是`反引号； -->
	    <property name="beginningDelimiter" value="`"/>
	    <property name="endingDelimiter" value="`"/>
	    
	    <!-- 必须要有的，使用这个配置链接数据库
	        @TODO:是否可以扩展
	     -->
	    <jdbcConnection driverClass="com.mysql.jdbc.Driver" connectionURL="jdbc:mysql:///pss" userId="root" password="admin">
	        <!-- 这里面可以设置property属性，每一个property属性都设置到配置的Driver上 -->
	    </jdbcConnection>
	    
	    <!-- java类型处理器 
	        用于处理DB中的类型到Java中的类型，默认使用JavaTypeResolverDefaultImpl；
	        注意一点，默认会先尝试使用Integer，Long，Short等来对应DECIMAL和 NUMERIC数据类型； 
	    -->
	    <javaTypeResolver type="org.mybatis.generator.internal.types.JavaTypeResolverDefaultImpl">
	        <!-- 
	            true：使用BigDecimal对应DECIMAL和 NUMERIC数据类型
	            false：默认,
	                scale>0;length>18：使用BigDecimal;
	                scale=0;length[10,18]：使用Long；
	                scale=0;length[5,9]：使用Integer；
	                scale=0;length<5：使用Short；
	         -->
	        <property name="forceBigDecimals" value="false"/>
	    </javaTypeResolver>
	    
	    
	    <!-- java模型创建器，是必须要的元素
	        负责：1，key类（见context的defaultModelType）；2，java类；3，查询类
	        targetPackage：生成的类要放的包，真实的包受enableSubPackages属性控制；
	        targetProject：目标项目，指定一个存在的目录下，生成的内容会放到指定目录中，如果目录不存在，MBG不会自动建目录
	     -->
	    <javaModelGenerator targetPackage="com._520it.mybatis.domain" targetProject="src/main/java">
	        <!--  for MyBatis3/MyBatis3Simple
	            自动为每一个生成的类创建一个构造方法，构造方法包含了所有的field；而不是使用setter；
	         -->
	        <property name="constructorBased" value="false"/>
	        
	        <!-- 在targetPackage的基础上，根据数据库的schema再生成一层package，最终生成的类放在这个package下，默认为false -->
	        <property name="enableSubPackages" value="true"/>
			
	        <!-- for MyBatis3 / MyBatis3Simple
	            是否创建一个不可变的类，如果为true，
	            那么MBG会创建一个没有setter方法的类，取而代之的是类似constructorBased的类
	         -->
	        <property name="immutable" value="false"/> 
			
	        <!-- 设置一个根对象，
	            如果设置了这个根对象，那么生成的keyClass或者recordClass会继承这个类；在Table的rootClass属性中可以覆盖该选项
	            注意：如果在key class或者record class中有root class相同的属性，MBG就不会重新生成这些属性了，包括：
	                1，属性名相同，类型相同，有相同的getter/setter方法；
	         -->
	        <property name="rootClass" value="com._520it.mybatis.domain.BaseDomain"/>
	        
	        <!-- 设置是否在getter方法中，对String类型字段调用trim()方法 -->
	        <property name="trimStrings" value="true"/>
	    </javaModelGenerator>
	    
	    <!-- 生成SQL map的XML文件生成器，
	        注意，在Mybatis3之后，我们可以使用mapper.xml文件+Mapper接口（或者不用mapper接口），
	            或者只使用Mapper接口+Annotation，所以，如果 javaClientGenerator配置中配置了需要生成XML的话，这个元素就必须配置
	        targetPackage/targetProject:同javaModelGenerator
	     -->
	    <sqlMapGenerator targetPackage="com._520it.mybatis.mapper" targetProject="src/main/resources">
	        <!-- 在targetPackage的基础上，根据数据库的schema再生成一层package，最终生成的类放在这个package下，默认为false -->
	        <property name="enableSubPackages" value="true"/>
	    </sqlMapGenerator>
	    
	    <!-- 对于mybatis来说，即生成Mapper接口，注意，如果没有配置该元素，那么默认不会生成Mapper接口 
	        targetPackage/targetProject:同javaModelGenerator
	        type：选择怎么生成mapper接口（在MyBatis3/MyBatis3Simple下）：
	            1，ANNOTATEDMAPPER：会生成使用Mapper接口+Annotation的方式创建（SQL生成在annotation中），不会生成对应的XML；
	            2，MIXEDMAPPER：使用混合配置，会生成Mapper接口，并适当添加合适的Annotation，但是XML会生成在XML中；
	            3，XMLMAPPER：会生成Mapper接口，接口完全依赖XML；
	        注意，如果context是MyBatis3Simple：只支持ANNOTATEDMAPPER和XMLMAPPER
	    -->
	    <javaClientGenerator targetPackage="com._520it.mybatis.mapper" type="ANNOTATEDMAPPER" targetProject="src/main/java">
	        <!-- 在targetPackage的基础上，根据数据库的schema再生成一层package，最终生成的类放在这个package下，默认为false -->
	        <property name="enableSubPackages" value="true"/>
	        
	        <!-- 可以为所有生成的接口添加一个父接口，但是MBG只负责生成，不负责检查
	        <property name="rootInterface" value=""/>
	         -->
	    </javaClientGenerator>
	    
	    <!-- 选择一个table来生成相关文件，可以有一个或多个table，必须要有table元素
	        选择的table会生成一下文件：
	        1，SQL map文件
	        2，生成一个主键类；
	        3，除了BLOB和主键的其他字段的类；
	        4，包含BLOB的类；
	        5，一个用户生成动态查询的条件类（selectByExample, deleteByExample），可选；
	        6，Mapper接口（可选）
	    
	        tableName（必要）：要生成对象的表名；
	        注意：大小写敏感问题。正常情况下，MBG会自动的去识别数据库标识符的大小写敏感度，在一般情况下，MBG会
	            根据设置的schema，catalog或tablename去查询数据表，按照下面的流程：
	            1，如果schema，catalog或tablename中有空格，那么设置的是什么格式，就精确的使用指定的大小写格式去查询；
	            2，否则，如果数据库的标识符使用大写的，那么MBG自动把表名变成大写再查找；
	            3，否则，如果数据库的标识符使用小写的，那么MBG自动把表名变成小写再查找；
	            4，否则，使用指定的大小写格式查询；
	        另外的，如果在创建表的时候，使用的""把数据库对象规定大小写，就算数据库标识符是使用的大写，在这种情况下也会使用给定的大小写来创建表名；
	        这个时候，请设置delimitIdentifiers="true"即可保留大小写格式；
	        
	        可选：
	        1，schema：数据库的schema；
	        2，catalog：数据库的catalog；
	        3，alias：为数据表设置的别名，如果设置了alias，那么生成的所有的SELECT SQL语句中，列名会变成：alias_actualColumnName
	        4，domainObjectName：生成的domain类的名字，如果不设置，直接使用表名作为domain类的名字；可以设置为somepck.domainName，那么会自动把domainName类再放到somepck包里面；
	        5，enableInsert（默认true）：指定是否生成insert语句；
	        6，enableSelectByPrimaryKey（默认true）：指定是否生成按照主键查询对象的语句（就是getById或get）；
	        7，enableSelectByExample（默认true）：MyBatis3Simple为false，指定是否生成动态查询语句；
	        8，enableUpdateByPrimaryKey（默认true）：指定是否生成按照主键修改对象的语句（即update)；
	        9，enableDeleteByPrimaryKey（默认true）：指定是否生成按照主键删除对象的语句（即delete）；
	        10，enableDeleteByExample（默认true）：MyBatis3Simple为false，指定是否生成动态删除语句；
	        11，enableCountByExample（默认true）：MyBatis3Simple为false，指定是否生成动态查询总条数语句（用于分页的总条数查询）；
	        12，enableUpdateByExample（默认true）：MyBatis3Simple为false，指定是否生成动态修改语句（只修改对象中不为空的属性）；
	        13，modelType：参考context元素的defaultModelType，相当于覆盖；
	        14，delimitIdentifiers：参考tableName的解释，注意，默认的delimitIdentifiers是双引号，如果类似MYSQL这样的数据库，使用的是`（反引号，那么还需要设置context的beginningDelimiter和endingDelimiter属性）
	        15，delimitAllColumns：设置是否所有生成的SQL中的列名都使用标识符引起来。默认为false，delimitIdentifiers参考context的属性
	        
	        注意，table里面很多参数都是对javaModelGenerator，context等元素的默认属性的一个复写；
	     -->
	    <table tableName="userinfo" >
	        
	        <!-- 参考 javaModelGenerator 的 constructorBased属性-->
	        <property name="constructorBased" value="false"/>
	        
	        <!-- 默认为false，如果设置为true，在生成的SQL中，table名字不会加上catalog或schema； -->
	        <property name="ignoreQualifiersAtRuntime" value="false"/>
	        
	        <!-- 参考 javaModelGenerator 的 immutable 属性 -->
	        <property name="immutable" value="false"/>
	        
	        <!-- 指定是否只生成domain类，如果设置为true，只生成domain类，如果还配置了sqlMapGenerator，那么在mapper XML文件中，只生成resultMap元素 -->
	        <property name="modelOnly" value="false"/>
	        
	        <!-- 参考 javaModelGenerator 的 rootClass 属性 
	        <property name="rootClass" value=""/>
	         -->
	         
	        <!-- 参考javaClientGenerator 的  rootInterface 属性
	        <property name="rootInterface" value=""/>
	        -->
	        
	        <!-- 如果设置了runtimeCatalog，那么在生成的SQL中，使用该指定的catalog，而不是table元素上的catalog 
	        <property name="runtimeCatalog" value=""/>
	        -->
	        
	        <!-- 如果设置了runtimeSchema，那么在生成的SQL中，使用该指定的schema，而不是table元素上的schema 
	        <property name="runtimeSchema" value=""/>
	        -->
	        
	        <!-- 如果设置了runtimeTableName，那么在生成的SQL中，使用该指定的tablename，而不是table元素上的tablename 
	        <property name="runtimeTableName" value=""/>
	        -->
	        
	        <!-- 注意，该属性只针对MyBatis3Simple有用；
	            如果选择的runtime是MyBatis3Simple，那么会生成一个SelectAll方法，如果指定了selectAllOrderByClause，那么会在该SQL中添加指定的这个order条件；
	         -->
	        <property name="selectAllOrderByClause" value="age desc,username asc"/>
	        
	        <!-- 如果设置为true，生成的model类会直接使用column本身的名字，而不会再使用驼峰命名方法，比如BORN_DATE，生成的属性名字就是BORN_DATE,而不会是bornDate -->
	        <property name="useActualColumnNames" value="false"/>
	        
	        
	        <!-- generatedKey用于生成生成主键的方法，
	            如果设置了该元素，MBG会在生成的<insert>元素中生成一条正确的<selectKey>元素，该元素可选
	            column:主键的列名；
	            sqlStatement：要生成的selectKey语句，有以下可选项：
	                Cloudscape:相当于selectKey的SQL为： VALUES IDENTITY_VAL_LOCAL()
	                DB2       :相当于selectKey的SQL为： VALUES IDENTITY_VAL_LOCAL()
	                DB2_MF    :相当于selectKey的SQL为：SELECT IDENTITY_VAL_LOCAL() FROM SYSIBM.SYSDUMMY1
	                Derby     :相当于selectKey的SQL为：VALUES IDENTITY_VAL_LOCAL()
	                HSQLDB    :相当于selectKey的SQL为：CALL IDENTITY()
	                Informix  :相当于selectKey的SQL为：select dbinfo('sqlca.sqlerrd1') from systables where tabid=1
	                MySql     :相当于selectKey的SQL为：SELECT LAST_INSERT_ID()
	                SqlServer :相当于selectKey的SQL为：SELECT SCOPE_IDENTITY()
	                SYBASE    :相当于selectKey的SQL为：SELECT @@IDENTITY
	                JDBC      :相当于在生成的insert元素上添加useGeneratedKeys="true"和keyProperty属性
	        <generatedKey column="" sqlStatement=""/>
	         -->
	        
	        <!-- 
	            该元素会在根据表中列名计算对象属性名之前先重命名列名，非常适合用于表中的列都有公用的前缀字符串的时候，
	            比如列名为：CUST_ID,CUST_NAME,CUST_EMAIL,CUST_ADDRESS等；
	            那么就可以设置searchString为"^CUST_"，并使用空白替换，那么生成的Customer对象中的属性名称就不是
	            custId,custName等，而是先被替换为ID,NAME,EMAIL,然后变成属性：id，name，email；
	            
	            注意，MBG是使用java.util.regex.Matcher.replaceAll来替换searchString和replaceString的，
	            如果使用了columnOverride元素，该属性无效；
	            
	        <columnRenamingRule searchString="" replaceString=""/>
	         -->
			
	         <!-- 用来修改表中某个列的属性，MBG会使用修改后的列来生成domain的属性；
	            column:要重新设置的列名；
	            注意，一个table元素中可以有多个columnOverride元素哈~
	          -->
	         <columnOverride column="username">
	            <!-- 使用property属性来指定列要生成的属性名称 -->
	            <property name="property" value="userName"/>
	            
	            <!-- javaType用于指定生成的domain的属性类型，使用类型的全限定名
	            <property name="javaType" value=""/>
	             -->
	             
	            <!-- jdbcType用于指定该列的JDBC类型 
	            <property name="jdbcType" value=""/>
	             -->
	             
	            <!-- typeHandler 用于指定该列使用到的TypeHandler，如果要指定，配置类型处理器的全限定名
	                注意，mybatis中，不会生成到mybatis-config.xml中的typeHandler
	                只会生成类似：where id = #{id,jdbcType=BIGINT,typeHandler=com._520it.mybatis.MyTypeHandler}的参数描述
	            <property name="jdbcType" value=""/>
	            -->
	            
	            <!-- 参考table元素的delimitAllColumns配置，默认为false
	            <property name="delimitedColumnName" value=""/>
	             -->
	         </columnOverride>
	         
	         <!-- ignoreColumn设置一个MGB忽略的列，如果设置了改列，那么在生成的domain中，生成的SQL中，都不会有该列出现 
	            column:指定要忽略的列的名字；
	            delimitedColumnName：参考table元素的delimitAllColumns配置，默认为false
	            
	            注意，一个table元素中可以有多个ignoreColumn元素
	         <ignoreColumn column="deptId" delimitedColumnName=""/>
	         -->
	    </table>
	</context>
	</generatorConfiguration>

---