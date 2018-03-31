---
title: Maven常见问题与解决方案
date: 2018-03-20 18:04:59
tags: Maven
categories: 开发工具

---
## 0.问题列表
1. 编译时报错怎么解决：
		Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.3:compile (default-compile)
2. jetty运行时出异常：
		Failed to execute goal org.eclipse.jetty:jetty-maven-plugin:9.4.8.v20171121:run (default-cli) on project mvnweb: Execution default-cli of goal org.eclipse.jetty:jetty-maven-plugin:9.4.8.v20171121:run failed: Unable to load the mojo 'run' in the plugin 'org.eclipse.jetty:jetty-maven-plugin:9.4.8.v20171121' due to an API incompatibility: org.codehaus.plexus.component.repository.exception.ComponentLookupException: org/eclipse/jetty/maven/plugin/JettyRunMojo : Unsupported major.minor version 52.0
3. 配置MyBatis代码生成器插件时出错：
		Failed to execute goal org.mybatis.generator:mybatis-generator-maven-plugin:1.3.3:generate (default-cli) on project simple2: configfile D:\四海兴唐\workspace\simple2\src\main\resource\generator\generatorConfig.xml does not exist -> [Help 1]
4. 编译时报编译插件版本警告：
		[WARNING] Some problems were encountered while building the effective model for Mybatis:simple2:jar:0.0.1-SNAPSHOT
		[WARNING] 'build.plugins.plugin.version' for org.apache.maven.plugins:maven-compiler-plugin is missing. @ line 57, column 13

---
## 1.问题1~5解决方案
**1)编译时报错怎么解决**：
完整的错误信息：
Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.3:compile (default-compile)
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.3:compile (default-compile) on project xinghe-interaction: Compilation failure
[ERROR] No compiler is provided in this environment. Perhaps you are running on a JRE rather than a JDK?
[ERROR] -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoFailureException
[ERROR] 
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR] mvn <goals> -rf :xinghe-interaction
解决方案：


**2）jetty运行时出异常：**
Failed to execute goal org.eclipse.jetty:jetty-maven-plugin:9.4.8.v20171121:run (default-cli) on project mvnweb: Execution default-cli of goal org.eclipse.jetty:jetty-maven-plugin:9.4.8.v20171121:run failed: Unable to load the mojo 'run' in the plugin 'org.eclipse.jetty:jetty-maven-plugin:9.4.8.v20171121' due to an API incompatibility: org.codehaus.plexus.component.repository.exception.ComponentLookupException: org/eclipse/jetty/maven/plugin/JettyRunMojo : Unsupported major.minor version 52.0
出错原因：
jetty-maven-plugin的版本与开发环境的jdk不符。
解决办法(添加其他版本的jetty-maven-plugin):

		<groupId>org.mortbay.jetty</groupId>
		<artifactId>jetty-maven-plugin</artifactId>
		<version>8.1.16.v20140903</version> 


**3)编译时报错怎么解决**：
错误：
Failed to execute goal org.mybatis.generator:mybatis-generator-maven-plugin:1.3.3:generate (default-cli) on project simple2: configfile D:\四海兴唐\workspace\simple2\src\main\resource\generator\generatorConfig.xml does not exist -> [Help 1]
解决方案：
找不到`generatorConfig.xml`,查看指定该文件的路径是否正确

**4)编译时报编译插件版本警告：**
[WARNING] Some problems were encountered while building the effective model for Mybatis:simple2:jar:0.0.1-SNAPSHOT
[WARNING] 'build.plugins.plugin.version' for org.apache.maven.plugins:maven-compiler-plugin is missing. @ line 57, column 13
原因：未加版本号

		<plugin>
			<artifactId>maven-compiler-plugin</artifactId>
			<configuration>
				<source>1.7</source>
				<target>1.7</target>
			</configuration>
		</plugin>
加上一行：
		<version>2.3.2</version> 
等指定版本就行了，不过不加也没什么问题。


---