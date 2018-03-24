---
title: MyBatis学习笔记（二）：xml的基本用法--简单的RBAC实现
date: 2018-03-23 15:59:35
tags: MyBatis3
categories: Java框架

---
## 0.学习准备：
1. 学习资料：
书本：《MyBatis从入门到精通》
视频：某培训班视频
资料：[MyBatis官方文档](http://www.mybatis.org/mybatis-3/zh/index.html)

2. 本章重点
	- rbac的原理及数据库表设计
	- xml的基本用法
	- 使用xml进行增删改查
	- Mapper接口动态代理

---
## 1.创建RBAC数据库及实体类：
**1)RBAC介绍及需求**
1. RBAC:
Role-Based Access Control,基于角色的访问控制。
简单的权限管理。
2. 权限管理的需求：
一个用户拥有若干角色，一个角色拥有若干权限。
权限就是对某个资源的某种操作：增删改查。
用户与角色之间，角色与权限之间一般都是多对多的关系。
rbac的简单数据库需求：
![](http://p5ki4lhmo.bkt.clouddn.com/00019MyBatis%E5%AD%A6%E4%B9%A02-01.jpg)
3. 分别要创建5个表：
用户表(user),角色表（role）,权限表(privilege),用户角色关联表（user_role）,角色权限关联表(role_privilege)

**2)创建数据库：**
1. 创建数据表：
创建user用户表：
		CREATE TABLE user(
		id BIGINT NOT NULL auto_increment COMMENT '用户ID',
		user_name VARCHAR(50) COMMENT '用户名',
		user_password VARCHAR(50) COMMENT '密码',
		user_email VARCHAR(50) COMMENT '邮箱',
		user_info text COMMENT '简介',
		head_img BLOB COMMENT '头像',
		create_time datetime COMMENT '创建时间',
		PRIMARY KEY(id)
		)CHARSET utf8;
创建role角色表：
		CREATE TABLE role(
		id BIGINT NOT NULL auto_increment COMMENT '用户ID',
		role_name VARCHAR(50) COMMENT '角色名',
		enabled INT COMMENT '有效标志',
		create_by BIGINT COMMENT '创建人',
		create_time datetime COMMENT '创建时间',
		PRIMARY KEY (id)
		)CHARSET utf8;
创建user_role联系表：
		CREATE TABLE user_role(
		user_id BIGINT COMMENT '用户ID',
		role_id BIGINT COMMENT '角色ID'
		)CHARSET utf8;
权限表与role_privilege关联表与创建方式与上述相同。
创建完的数据库如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00019MyBatis%E5%AD%A6%E4%B9%A02-02.jpg)
2. 添加数据：
为了方便测试学习，给各个表添加数据如下:
![](http://p5ki4lhmo.bkt.clouddn.com/00019MyBatis%E5%AD%A6%E4%B9%A02-03.jpg)
3. 创建实体类：
MyBatis默认的是**下划线转驼峰**的命名方式，所以实体类一般按照这种方式进行创建(对应字段)。
Java工程仍用前面的，在src/main/java/Mybatis/simple/model下：
创建User类：
		package Mybatis.simple.model;
		import java.util.Arrays;
		import java.util.Date;
		//用户表
		public class User {
			//用户id
			private Long id;
			//用户名
			private String userName;
			//密码
			private String userPassword;
			//邮箱
			private String userEmail;
			//简介
			private String userInfo;
			//头像
			private byte[] headImg;
			//创建时间
			private Date createTime;
			
			//set,get方法，toString方法
		
		}
创建Role类：
		package Mybatis.simple.model;
		import java.util.Date;
		//角色表
		public class Role {
			//角色id
			private Long id;
			//用户名
			private String roleName;
			//有效标志
			private int enabled;
			//创建人ID
			private long createBy;
			//创建时间
			private Date creatTime;

			//set,get方法，toString方法
		}
创建UserRole类：
		package Mybatis.simple.model;
		//用户角色关系
		public class UserRole {
			//用户ID
			private Long userId;
			//角色ID
			private Long roleId;
			public Long getUserId() {
				return userId;
			}
			public void setUserId(Long userId) {
				this.userId = userId;
			}
			public Long getRoleId() {
				return roleId;
			}
			public void setRoleId(Long roleId) {
				this.roleId = roleId;
			}
			@Override
			public String toString() {
				return "UserRole [userId=" + userId + ", roleId=" + roleId + "]";
			}
		}
Privilege类和RolePrivilege类的创建和上面的差不多。

---
## 3.最常用的XML使用方式--接口代理
**1)接口代理调用：**
1. MyBatis最大的强大之处在于映射语句，笔记1使用sqlSession的方法是属于映射器方法。
映射器传多个参数时，需要使用Map等才能实现传值，比较麻烦。
2. MyBatis3.0相比MyBatis2.0最大的区别就是支持**接口调用方法**。
MyBatis使用Java的动态代理可以直接通过接口来调用方法，而不需要提供接口的实现类，更不需要在实现类中使用SqlSession通过命名空间.id来调用Sql方法。
有多个参数时，通过参数注解@Param设置参数的名字，省去了手动构造Mapper的过程。
3. 接口可以配合XML使用，也可以配合注解使用。XML可以单独使用，但是注解必须要在接口中使用。
4. 接口使用更加方便且已经被广泛使用。

**2)接口+XML的使用方式：**
1. 在src/main/sources/Mybatis/simple/mapper目录下创建与表对应的映射XML文件：
UserMapper.xml,RoleMapper.xml,UserRoleMapper.xml,PrivilegeMaper.xml,RolePrivilegeMapper.xml
2. 在src/main/java下创建Mybatis/simple/mapper目录，在此目录下创建5个对应的接口：
UserMapper.java,RoleMapper.java,UserRoleMapper.java,PrivilegeMaper.java,RolePrivilegeMapper.java(为上次的CountryMapper也创建一个)
新建的接口的内容如下，如UserMapper:
		package Mybatis.simple.mapper;
		
		public interface UserMapper {
		
		}
此时的所有XML都是空的状态。接下来逐步添加。
创建好的目录是这个样子的：
![](http://p5ki4lhmo.bkt.clouddn.com/00019MyBatis%E5%AD%A6%E4%B9%A02-04.jpg)
3. 配置Mapper.xml使其与接口相关联。(namespace=接口全名）
如配置UserMapper.xml：
		<?xml version="1.0" encoding="UTF-8"?>
		
		<!DOCTYPE mapper
		  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
		  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
		  
		  <mapper namespace="Mybatis.simple.mapper.UserMapper">
		  </mapper>
配置RoleMapper.xml：
		<?xml version="1.0" encoding="UTF-8"?>
		
		<!DOCTYPE mapper
		  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
		  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
		  
		  <mapper namespace="Mybatis.simple.mapper.RoleMapper">
		  </mapper>
配置UserRoleMapper.xml：
		<?xml version="1.0" encoding="UTF-8"?>
		
		<!DOCTYPE mapper
		  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
		  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
		  
		  <mapper namespace="Mybatis.simple.mapper.UserRoleMapper">
		  </mapper>
其余两个XML的配置和上面的相同。
Mapper接口与XML文件相关联时，需要配置XML文件的namespace为对应接口的全限定名称。
4. 配置MyBatis核心配置文件：
一般方法：
		<mappers>
			<mapper resource="Mybatis/simple/mapper/CountryMapper.xml"/>
			<mapper resource="Mybatis/simple/mapper/UserMapper.xml"/>
			<mapper resource="Mybatis/simple/mapper/RoleMapper.xml"/>
			<mapper resource="Mybatis/simple/mapper/UserRoleMapper.xml"/>
			<mapper resource="Mybatis/simple/mapper/PrivilegeMapper.xml"/>
			<mapper resource="Mybatis/simple/mapper/RolePrivilegeMapper.xml"/>
		</mappers>
特殊方法：
因为`Mybatis/simple/mapper`下的所有映射文件都有相应的Mapper接口，所以可以一起配置：
		<mappers>
			<package name="Mybatis.simple.mapper"/>
		</mappers>
使用`<package>`标签的name属性。

**3)特殊配置方法的工作过程：**
使用`<package name="Mybatis.simple.mapper"/>`时的工作流程：
- 1.MyBatis会先查找Mybatis/simple/mapper包下所有接口，循环对每个接口进行2-4步的操作
- 2.判断接口的命名空间是否已经存在（已经加载），若存在则报错，不存在则继续运行。
- 3.加载接口对应的XML映射文件，并将接口全限定名称转换为路径加上.xml并解析(设置成`<mapper>`的resource)
如Mybatis.simple.mapper.UserMapper会转换成Mybatis/simple/mapper/UserMapper.xml并解析
- 4.处理接口中的注解方法。

---
## 4.返回值单一的select用法
**1)使用id查询用户信息：**
1. 使用JDBC需要写查询语句，并对结果集进行手动处理，将结果集手动映射到对象属性中。
使用MyBatis时只需要在XML中添加一个select元素，写一个SQL，在接口中添加一个方法并且简单配置XML即可。
2. 在UserMapper.java接口中添加一个selectById()方法，返回值为User类：
		package Mybatis.simple.mapper;
		
		import Mybatis.simple.model.User;
		
		public interface UserMapper {
			
			User selectById(Long Id);
		}
3. 然后在UserMapper.xml中配置结果集（resultMap）对应实体类以及SQL语句，配置完的UserMapper.xml如下：
		<?xml version="1.0" encoding="UTF-8"?>
		
		<!DOCTYPE mapper
		  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
		  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
		  
		  <mapper namespace="Mybatis.simple.mapper.UserMapper">
		  
		  <resultMap type="Mybatis.simple.model.User" id="userMap">
		  	<id property="id" column="id"/>
		  	<result property="userName" column="user_name"/>
		  	<result property="userPassword" column="user_password"/>
		  	<result property="userEmail" column="user_email"/>
		  	<result property="userInfo" column="user_info"/>
		  	<result property="headImg" column="head_img"/>
		  	<result property="createTime" column="create_time"/>
		  </resultMap>
		  
		  <select id="selectById" resultMap="userMap">
		  	SELECT * FROM user WHERE id=#{id}
		  </select>
		  </mapper>
select的id属性与接口方法名相同。
select返回值与resultMap的id相同（如果返回值是实体类对象的话）
先不做测试，先对XML与接口命名的规则以及XML中一些标签和属性进行介绍

**2)命名规则，标签与属性的功能与介绍：**
1. 映射XML和接口的命名规则：
	- 只使用XML而不使用接口时，namespace可以是任意不重复的名称。
	- id属性在任何时候不能出现`.`,同一个命名空间下不能有重复的id。
	- 接口方法是可以重载的，但是对应的id值不能重复，所以会出现多个接口方法对应一个select查询的情况，这时可以在同名方法中的其中一个方法加上一个RowBound类型的参数用于实现**分页查询。**（但这是内存分页，一般不用）
2. XML中一些标签和属性的作用：
	- `<select>`:映射查询语句的标签。
	- id：命名空间的唯一标识符，可用来代表（定位）这条语句。
	- resultMap：用于设置返回值的类型和映射的关系
	- `#{id}`：MyBatis SQL中预处理参数的一种方式，大括号的id是传入的参数名。
3. **resultMap的属性：**
`<resultMap>`标签用于配置Java对象和查询结果列的对应关系。resultMap是一种很重要的配置结果映射的方法，必须熟练掌握。
resultMap包含了许多**属性**：
	- id：必填，且唯一，`<select>`标签的resultMap指定的值就是这个id的值。
	- type:必填，配置查询列所对应的java对象类型。
	- expends:选填，继承，可以配置当前的resultMap继承至其他的resultMap,填的是被继承的那个resultMap的id。
	- autoMapping：选填，可选值为true或false,用于配置是否启用非映射字段（没有在resultMap中配置的字段）的自动映射功能（该配置可以覆盖全局的autoMappingBehavior的值)
4. **resultMap的标签：**
	- construtor:配置使用构造方法注入结果，有两个子标签：
		- idArg：id参数，标记结果做为id（唯一值），可以帮助提高整体性能。
		- arg：注入到构造方法的一个普通结果。
	- id：id参数，标记结果做为id（唯一值），可以帮助提高整体性能。
	- result：注入到java对象属性的一个普通结果。
	- association：一个复杂的关联集，许多结果将包成这种类型。
	- collection：复杂类型的集合
	- discriminator：根据结果值来决定使用哪个结果映射。
	- case：就某些值的结果映射。
5. constructor,id,result是常用标签，其他标签后面章节才讲。
6. 标签属性之间的关系：
	- constructor:**通过构造方法注入属性的结果值。**其中的idArg和arg属性的含义和resultMap的id和result相同，只是注入的方式不同。
	- resultMap的id和result标签的属性完全相同，不同的是id标识了主键（或唯一键）字段(唯一键可以有多个)，result则是普通的字段。resultMap是通过set方法注入的。
7. id与result标签包含的属性：
	- column：数据库表的字段名（或别名）。
	- property：映射到字段的实体类属性。(可以映射简单的也可以映射复杂的)
	- javaType：一个java类的完全限定名，也可以是别名。
		- 结果映射到javaBean，MyBatis可以自动判断。
		- 结果映射到HashMap，则需要明确指定javaType属性。
	- jdbcType：字段类型，JDBC类仅仅需要对增删改可为空的字段进行处理。
	这是JDBC的需要，不是MyBatis的需要。
	- typeHandler：可以覆盖默认的类型处理器。属性值为类的完全限定名或者类的别名。

**3)测试接口返回值的定义：**
1. 在`userMapper.java`接口中新增查询所有用户的方法：
		//查询所有用户
		List<User> selectAll();
2. 在对应的`userMapper.xml`中新增查询所有用户的SQL语句：
		<select id="selectAll" resultType="Mybatis.simple.model.User">
		  	SELECT 
		  		id,user_name userName,user_password userPassword,user_email userEmail,user_info userInfo,head_img headImg,create_time createTime
		  	FROM
		  		user
		</select>
3. 注意上述select标签与查询一个记录的select标签的区别：
`<select id="selectById" resultMap="userMap">`
`<select id="selectAll" resultType="Mybatis.simple.model.User">`
resultMap属性使用的是id对应的resultMap标签进行注入。
resultType属性则是指定了实体类名，使用别名进行注入(自动映射)。
4. resultType属性中要使用全限定名，若在核心配置文件中配置了别名则可以使用别名。如果未配置别名可以下载一个MyBatipse插件，具有全限定名提示功能：
Eclipse(或STS)-->【Help】-->【Ecipse Marketplace】-->搜索安装重启即可：
![](http://p5ki4lhmo.bkt.clouddn.com/00019MyBatis%E5%AD%A6%E4%B9%A02-05.jpg)
重启后进入映射文件，resultMap或resultType下按【ALT】+【/】,会出现如下提示（灰色是因为已经配置了别名）：
![](http://p5ki4lhmo.bkt.clouddn.com/00019MyBatis%E5%AD%A6%E4%B9%A02-06.jpg)
5. 接口方法返回值为List<User>而resultType写的是User,如果写List<User>或User[]也没什么大问题。因为返回值为一个值(集合)。
但是**返回值为多个结果时一定要使用List<User>或User[]**，否则会报错：
`TooManyResultsException`。

**4)更简单的映射：**
1. 除了resultMap映射与直接映射外，还有一种更加简单的配置映射。
但是需要数据库字段与实体类属性严格符合**"下划线转驼峰"**的命名规范。
2. MyBtis提供了一个全局属性mapUndercoreToCameCase，配置该属性为true可以将字段的下划线自动映射到对应的驼峰对象属性中。在核心配置文件myBatis中配置如下：
		<settings>
			<setting name="mapUnderscoreToCamelCase" value="true"/>
		</settings>
3. 测试，添加一个查询所有角色的方法：
RoleMapper.java：
		package Mybatis.simple.mapper;
		import java.util.List;
		import Mybatis.simple.model.Role;
		public interface RoleMapper {
			//查询所有角色
			List<Role> selectAll();
		}
RoleMapper.xml：
		<?xml version="1.0" encoding="UTF-8"?>
		<!DOCTYPE mapper
		  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
		  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
		  
		  <mapper namespace="Mybatis.simple.mapper.RoleMapper">
		  	<select id="selectAll" resultType="Role">
		  	SELECT
		  		id,role_name,enabled,create_by,create_time
		  	FROM
		  		role
		  	</select>
		  </mapper>
4. 为了简单还可以将`id,role_name,enabled,create_by,create_time`写成`*`,但是为了效率，一般不用。

---
## 5.接口代理测试：
**1)使用继承修改CountryMapperTest:**
1. 因为会有许多个测试，所以最好提取一个基础测试类，将可以复用的代码写入其中并使其它测试类继承或调用它：
在src/test/java下创建一个Mybatis.simple.base包，创建BaseMapperTest.java，代码如下：
		package Mybatis.simple.base;
		import java.io.IOException;
		import java.io.Reader;
		import org.apache.ibatis.io.Resources;
		import org.apache.ibatis.session.SqlSession;
		import org.apache.ibatis.session.SqlSessionFactory;
		import org.apache.ibatis.session.SqlSessionFactoryBuilder;
		import org.junit.BeforeClass;
	
		public class BaseMapperTest {
			//SqlSession工厂类
			private static SqlSessionFactory sqlSessionFactory;
			
			@BeforeClass
			public static void init(){
				try{
					// 加载核心配置文件
					Reader reader=Resources.getResourceAsReader("mybatis-config.xml");
					//新建工厂对象，相当于JDBCconnect
					sqlSessionFactory=new SqlSessionFactoryBuilder().build(reader);
					//关闭输入流
					reader.close();
				}catch(IOException e){
					e.printStackTrace();
				}
			}
			
			public SqlSession getSqlSession(){
				return sqlSessionFactory.openSession();
			}	
		}
2.修改CountryMapperTest为继承(省略import)：
		package Mybatis.simple.mapper;
		
		public class CountryMapperTest extends BaseMapperTest{
			@Test
			public void testSelectAll(){
				//使用sqlSession实例化一个sqlSession对象,相当于使用预处理语句
				SqlSession sqlSession=getSqlSession();
				try{
					List<Country> countryList=sqlSession.selectList("Mybatis.simple.mapper.CountryMapper.selectAll");
					for(Country coun:countryList){
						System.out.println(coun);
					}
				}finally{
					//关闭sqlSession
					sqlSession.close();
				}
			}
		}
继承至BaseMapperTest,使用getSqlSession方法来获取sqlSession。

**2)测试通过关于用户的两个方法：**
1. 创建User测试类UserMapperTest:
		package Mybatis.simple.mapper;
		
		import java.util.List;
		
		import org.apache.ibatis.session.SqlSession;
		import org.junit.Assert;
		import org.junit.Test;
		
		import Mybatis.simple.base.BaseMapperTest;
		import Mybatis.simple.model.User;
		
		public class UserMapperTest extends BaseMapperTest{
			
			@Test
			public void testSelectById(){
				//获取sqlSession
				SqlSession sqlSession =getSqlSession();
				try{
					//获取UserMapper接口
					UserMapper userMapper =sqlSession.getMapper(UserMapper.class);
					//调用selectById方法
					User user=userMapper.selectById(1L);
					//断言测试user不为空
					Assert.assertNotNull(user);
					//判断userName=admin是否成立
					Assert.assertEquals("admin", user.getUserName());
				}finally{
					sqlSession.close();
				}
			}
			
			@Test
			public void testSelectAll(){
				//获取sqlSession
				SqlSession sqlSession =getSqlSession();
				try{
					//获取UserMapper接口
					UserMapper userMapper =sqlSession.getMapper(UserMapper.class);
					//调用selectALL方法
					List<User> luser=userMapper.selectAll();
					//断言测试luser不为空
					Assert.assertNotNull(luser);
					//判断luser长度是否为大于0
					Assert.assertTrue(luser.size()>0);
				}finally{
					sqlSession.close();
				}
			}
		}
2. 测试方法:
测试本文件所有方法：文件-->右键-->【RUN AS】...
测试某一个方法，如testSelectAll：选中方法名-->右键-->【RUN AS】...
3. 运行测试，测试结果如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00019MyBatis%E5%AD%A6%E4%B9%A02-07.jpg)


---