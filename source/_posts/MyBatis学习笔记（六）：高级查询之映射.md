---
title: MyBatis学习笔记（六）：MyBatis高级查询--映射
date: 2018-04-01 21:08:08
tags: MyBatis
categories: Java框架

---
## 0.学习准备
1. 学习资料：
书本：《MyBatis从入门到精通》
视频：某培训班视频
资料：[MyBatis中文文档](http://www.mybatis.org/mybatis-3/zh/index.html)

---
## 1.高级结果映射简介：
1. 前面章节的基本的增删改查及动态SQL语句已经可以满足绝大部分的需求了。
但是MyBatis还有一些更高级的操作，可以提高性能或者简化操作。
2. 在MyBatis中经常要处理一对一或者一对多的关系。如前面的rbac表中的用户与角色，角色与权限之间都是一对多的关系。
3. 通常的做法是分别查询数据然后再组合在一起。
在大型的系统上，由于分库分表，这种方式可以减少表之间的关联查询，方便对系统进行扩展。
但是在一般的企业级应用里，使用MyBatis的高级结果映射可以轻松地处理一对一以及一对多的关系。

---
## 2.一对一映射
1. 首先假设一个用户只能拥有一个角色，并且修改数据表user_role。使用一对一映射实现查询用户时同时获取用户的角色信息。一对一映射不需要考虑是否存在重复的数据。
![](http://p5ki4lhmo.bkt.clouddn.com/00027MyBatis%E5%AD%A6%E4%B9%A06-01.jpg)
2. 实现一对一映射有四种途径：
自动映射处理，resultMap配置一对一映射，resultMap中的association标签配置一对一映射，association标签嵌套查询（多条SQL语句）。

### 配置一对一映射的三种基本途径
**1）使用自动映射处理一对一的关系：**
1. 准备，首先使用MBG自动生成相关代码。
在bgm.model包User类中添加一个Role对象属性及相关get,set方法：
		...
		// 角色信息
		private Role role;
		public Role getRole() {
			return role;
		}
		public void setRole(Role role) {
			this.role=role;
		}
		...
并在Role类以及Model类添加toString方法。
2. 自动映射其实就是使用配置或者别名的方式将查询的结果自动匹配到对应的字段上。
简单的映射就是user_name映射userName，支持复杂映射(多层嵌套)，如role.role_name会映射到role.roleName。实现原理是MyBatis会在user中寻找Role类的属性，如果存在Role类属性就创建一个role对象并将值赋给role.roleName。
3. 创建接口方法：
		//一对一映射，根据id查询用户及相关角色信息
		User selectUserAndRoleByUserId(Long id);
在UserMapper.xml中创建XML方法如下：
		<select id="selectUserAndRoleByUserId" resultType="User">
			SELECT
				u.id,
				user_name userName,
				user_password userPassword,
				user_email userEmail,
				user_info userInfo,
				head_img headImg,
				u.create_time createTime,
				r.id "role.id",
				role_name "role.roleName",
				enabled "role.enabled",
				create_by "role.createBy",
				r.create_time "role.createTime"
			FROM user u
			left join user_role ur on u.id=ur.user_id
			left join role r on ur.role_id=r.id
			WHERE u.id=#{id}  
		</select>
	- 查询时字段只要是在数据源中唯一就不需要加`别名.`的前缀来约束，但是为了更加严谨还是加上吧（这里没有加）。
	- 如果配置了核心配置文件的自动映射的话，将上述的别名都去掉也可以，会自动创建role并匹配赋值。
	- 使用左连接则左边部分全部保留，而未被匹配的部分会被置空，而右边未匹配上左边的则直接删除。
使用右连接则刚好相反，全连接则只保留能互相匹配的。
4. 创建测试类UserMapperTest并添加如下方法：
		//测试一对一映射的别名实现
		@Test
		public void testSelectUserAndRoleByUserId(){
			SqlSession sqlSession =getSqlSession();
			try{
				UserMapper userMapper =sqlSession.getMapper(UserMapper.class);		//获取UserMapper接口
				User user=userMapper.selectUserAndRoleByUserId(5L);  //查询user
				//断言测试user与role是否为空
				Assert.assertNotNull(user);
				Assert.assertNotNull(user.getRole());
			}finally{
				sqlSession.close();
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00027MyBatis%E5%AD%A6%E4%B9%A06-02.jpg)
BLOB类型的数据就会这样子显示，text类型的数据也会显示为blob，反正输出的数据是正确的。
5. 向上述这种将一此查询的结果映射到一个或者多个对象的方式称为**关联的结果集映射**。
这种方式的好处是减少数据库查询次数，减轻数据库压力。缺点是要写很复杂的结果集映射，且会给服务器带来压力。
当一定会使用到嵌套结果且嵌套SQL执行速度很快时可以使用关联结果集映射。

**2）使用resultMap配置一对一的映射**
1. 添加接口方法：
		//一对一映射，resultMap实现
		User selectUserAndRoleByUserId2(Long id);
在XML映射文件中添加resultMap配置：
		<resultMap type="User" id="userRoleMap">
			<id column="id" property="id" />
			<result column="user_name" property="userName" />
			<result column="user_password" property="userPassword" />
			<result column="user_email" property="userEmail" />
			<result column="create_time" jdbcType="TIMESTAMP" property="createTime" />
			<result column="user_info" jdbcType="LONGVARCHAR" property="userInfo" />
			<result column="head_img" jdbcType="LONGVARBINARY" property="headImg" />
			<!-- role相关属性 -->
			<result column="role_id" property="role.id" />
			<result column="role_name" property="role.roleName" />
			<result column="enabled" property="role.enabled" />
			<result column="create_by" property="role.createBy" />
			<result column="create_time" property="role.createTime" jdbcType="TIMESTAMP"/>
		</resultMap>
添加XML映射文件方法：
		<select id="selectUserAndRoleByUserId2" resultMap="userRoleMap">
			SELECT
				u.id,
				user_name,
				user_password,
				user_email,
				user_info,
				head_img,
				u.create_time,
				r.id as role_id,
				r.role_name,
				r.enabled,
				r.create_by,
				r.create_time
			FROM user u
			left join user_role ur on u.id=ur.user_id
			left join role r on ur.role_id=r.id
			WHERE u.id=#{id}
		</select>
注意和自动映射相比需要额外的配置，且需要将resultType改为resultMap,改完后别名就已经没有用了，所以去掉了。
重复的字段（如id）最好加上别名并在resutMap中使用别名来配置。
2. 测试方法自动映射的一模一样，只是将方法名换一下。
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00027MyBatis%E5%AD%A6%E4%B9%A06-03.jpg)
3. resultMap的配置非常麻烦。但是如果是用MBG自动生成的代码会自动生成当前对象的resultMap(有BLOB或TEXT会分为两个):
![](http://p5ki4lhmo.bkt.clouddn.com/00027MyBatis%E5%AD%A6%E4%B9%A06-04.jpg)
所以只要继承至上述的id为`ResultMapWithBLOBs`的resultMap，然后再加上Role的映射就可以了：
		<resultMap extends="ResultMapWithBLOBs" type="User" id="userRoleMap">
			<!-- role相关属性 -->
			<result column="role_id" property="role.id" />
			<result column="role_name" property="role.roleName" />
			<result column="enabled" property="role.enabled" />
			<result column="create_by" property="role.createBy" />
			<result column="create_time" property="role.createTime" jdbcType="TIMESTAMP"/>
		</resultMap>
测试结果和上面的一样。
4. 虽然使用了继承，但是还是比较麻烦的。但是起码修改的时候比原来方便了，而且代码量也减少了。不过我还是比较喜欢自动映射。
 
**3)association标签配置一对一：**
1. association标签是resultMap的子标签
在resultMap的代码中进行修改，为了方便使用association的某些属性的使用，给查询出的role表的属性都取上别名:
		<select id="selectUserAndRoleByUserId2" resultMap="userRoleMap">
			SELECT
				u.id,
				user_name,
				user_password,
				user_email,
				user_info,
				head_img,
				u.create_time,
				r.id as role_id,
				r.role_name role_role_name,
				r.enabled role_enabled,
				r.create_by role_create_by,
				r.create_time role_create_time
			FROM user u
			left join user_role ur on u.id=ur.user_id
			left join role r on ur.role_id=r.id
			WHERE u.id=#{id}
		</select>
resultMap中这样写：
		<resultMap extends="ResultMapWithBLOBs" type="User" id="userRoleMap">
			<association property="role" columnPrefix="role_" javaType="Role">
				<result column="id" property="id" />
				<result column="role_name" property="roleName" />
				<result column="enabled" property="enabled" />
				<result column="create_by" property="createBy" />
				<result column="create_time" property="createTime" jdbcType="TIMESTAMP"/>
			</association>
		</resultMap>
对比上面的ResultMap,column的前缀都去掉了，property中也不用加role.了。执行的结果和只使用resultMap标签时是一样的。
2. association标签的第二种使用方法：
可以配置一个已经存在的resultMap映射（有一个resultMap属性）。
在自动生成的RoleMapper.xml中有一个Role的基础映射：
		<resultMap id="BaseResultMap" type="mbg.model.Role">
			<id column="id" jdbcType="BIGINT" property="id" />
			<result column="role_name" jdbcType="VARCHAR" property="roleName" />
			<result column="enabled" jdbcType="INTEGER" property="enabled" />
			<result column="create_by" jdbcType="BIGINT" property="createBy" />
			<result column="create_time" jdbcType="TIMESTAMP" property="createTime" />
		</resultMap>
association引入外部的resultMap时，MyBatis会自动给这个resultMap加上当前命名空间的前缀，变为`mbg.mapper.UserMapper.BaseResultMap`，而UserMap中自动生成的resultMap的id也为BaseResultMap，但是并不产生冲突。
所以可以直接引入使用（要用全限定名称）：
		<resultMap extends="ResultMapWithBLOBs" type="User" id="userRoleMap">
			<association property="role" columnPrefix="role_" resultMap="mbg.mapper.RoleMapper.BaseResultMap" />
		</resultMap>
也可以直接在UserMapper下新建一个Role的resultMap,然后将id写入resultMap属性即可（不用写全限定名）。
测试结果和前面也一样：
![](http://p5ki4lhmo.bkt.clouddn.com/00027MyBatis%E5%AD%A6%E4%B9%A06-03.jpg)
3. association标签配置一对一时的属性：
	- `property`：对应实体类中的属性名，必填
	- `javaType`：属性对应的Java类型
	- `resultMap`：使用现有的resultMap而不需要额外配置
	- `columnPrefix`：前缀，配置后子标签可以省略前缀

### association标签的嵌套查询
**1)关联的嵌套查询：**
1. 上面的三种情况都是通过"关联的嵌套结果映射"来实现一对一查询。**关联的嵌套结果映射**就是通过一次SQL查询根据表或指定的属性映射到不同的对象中。
2. 关联的嵌套查询：
利用简单的SQL执行多次查询转换为我们需要的结果。
和根据业务逻辑手动执行多次SQL相似。
3. association关于嵌套查询的常用属性：
	- `select`：填写另一个映射查询的Id,Mybatis会额外执行这个查询获取嵌套对象的结果。
	- `column`：列名（或别名）,将朱主查询中列的结果作为嵌套查询的参数。
	配置方式为`column={prop1=col1,prop2=col2...}`,prop1和prop2将作为嵌套查询的参数。
	- `fetchType`：数据加载的方式，可选值为lazy和eager,分别为延迟加载了积极加载。会覆盖全局配置：`lazyLoadingEnabled`。

**2)使用关联的嵌套查询实现上面的user查询**
1. 创建接口方法：
		//一对一映射,association嵌套实现
		User selectUserAndRoleByUserId3(Long id);
创建xml方法：
		<select id="selectUserAndRoleByUserId3" resultMap="userRoleMapSelect">
			SELECT
				u.id,
				user_name,
				user_password,
				user_email,
				user_info,
				head_img,
				u.create_time,
				ur.role_id
			FROM user u
			left join user_role ur on u.id=ur.user_id
			WHERE u.id=#{id}  
		</select>
2. 查看RoleMapper.xml自动生成的代码有一个按主键查询的方法：
		<select id="selectByPrimaryKey" resultMap="BaseResultMap">
			select 
			<include refid="Base_Column_List" />
			from role
			where id = #{id}
		</select>
然后使用这个方法的id在UserMapper.xml中创建resultMap：
		<resultMap type="User" extends="ResultMapWithBLOBs" id="userRoleMapSelect">
			<association property="role" column="{id=role_id}" select="mbg.mapper.RoleMapper.selectByPrimaryKey"></association>
		</resultMap>
`selectByPrimaryKey`查询的参数id时通过`column="{id=role_id}"`指定,如果想要指定多个参数，可以使用逗号隔开，如`column="{id=role_id,name=role_name}"`
3. 添加测试方法：
		//测试一对一映射的嵌套实现
		@Test
		public void testSelectUserAndRoleByUserId3(){
			SqlSession sqlSession =getSqlSession();
			try{
				UserMapper userMapper =sqlSession.getMapper(UserMapper.class);		//获取UserMapper接口
				User user=userMapper.selectUserAndRoleByUserId3(1001L);  //查询user
				//断言测试user与role是否为空
				Assert.assertNotNull(user);
				Assert.assertNotNull(user.getRole());
				System.out.println(user);
			}finally{
				sqlSession.close();
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00027MyBatis%E5%AD%A6%E4%B9%A06-05.jpg)
4. 基本执行流程：
Test-->接口方法-->xml查询方法-->resultMap--(传参)-->Role子查询-->其他配置

**3)延迟加载的使用（未配置时）：**

**4)正确配置后的延迟加载的使用：**

---


