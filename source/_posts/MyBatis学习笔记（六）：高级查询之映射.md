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
4. 主要有一对一，一对多和鉴别器映射。

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
	- `fetchType`：数据加载的方式，可选值为lazy和eager,分别为延迟加载和积极加载。会覆盖全局配置：`lazyLoadingEnabled`。

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
一个参数时column也可以写成这样：`column="role_id"`此时不需要使用id统一也可以。
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

**3)延迟加载的使用：**
1. 普通方式带来的效率问题：
普通方式是先查询嵌套查询的子查询：
			select 
			<include refid="Base_Column_List" />
			from role
			where id = #{id}
查到结果后再赋值给User对象中的Role对象，但是如果一直不使用这个role对象的话，那么这部分的资源就会白白浪费，也会增加额外的查询。如果嵌套查询中有N条语句(或者有N个需要嵌套查询的结果)，就会增加N条额外的查询。非常影响效率。
2. 延迟加载的解决方案：
将fetchType属性设置为lazy之后，不会立即执行嵌套的语句，只有当执行getRole()方法使用role对象时才会执行嵌套的语句。
3. 但是**实现这样的延迟加载效果需要配置核心配置文件**:
		<settings>
			...
			<setting name="aggressiveLazyLoading" value="false"/>
		</settings>
`aggressiveLazyLoading`属性默认为true,当属性时true时对任意延迟属性的调用都会使延迟加载的对象加载（如调用setter方法时）。设为false则是按需加载，只有在getter时才会加载。
4. 修改上述测试的代码：
resultMap:
		<resultMap type="User" extends="ResultMapWithBLOBs" id="userRoleMapSelect">
			<association property="role" column="{id=role_id}" fetchType="lazy" select="mbg.mapper.RoleMapper.selectByPrimaryKey"></association>
		</resultMap>
测试类添加这条语句：
		System.out.println("调用user.getRole()");
		Assert.assertNotNull(user.getRole());
测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00027MyBatis%E5%AD%A6%E4%B9%A06-06.jpg)
5. 关于延迟加载的一个问题：
有时候延迟加载会得到数据，有是有延迟加载会报错的原因。
MyBatis的延迟加载是通过动态代理调用Sqlsession去执行嵌套的SQL语句的，MyBatis与其它框架集成时SqlSession交给了框架来管理，因此当对象调用超出SqlSession的生命周期时就会出错。

**4)延迟加载用法完善：**
1. 上述延迟加载已经满足我们的基本需求。但是有时候会有特殊情况：
在触发某些方法时将所有的数据都加载进来。
2. 不用修改任何配置就可以实现：**lazyLoadTriggerMethods**
这个参数的含义是调用配置中的方法时就会加载全部的数据。默认值为"equals,clone,hashCode,toString"
所以要调用其中一个方法（包括getRole）就可以加载全部数据了。
3. 以toString方法为例,在测试类testSelectUserAndRoleByUserId3中添加如下代码：
		System.out.println("调用toString方法");
		System.out.println(user);
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00027MyBatis%E5%AD%A6%E4%B9%A06-07.jpg)

---
## 3.一对多映射
**一对多映射介绍和测试准备：**
1. 一对多本质和一对一没什么区别，只是映射的结果集（同一类的）变成了多个。
一对一映射使用的是association而一对多映射使用的是collection标签。
一对多的嵌套查询，延迟加载等都和一对一相同。
2. 在一对多的关系中，主表的一条记录会对应关联表中的多条记录。rbac中用户与角色，角色与权限都是一对多的关系，这里使用角色--权限来测试。
3. 修改角色表添加如下属性与方法：
		//权限集合
		private List<Privilege> privilegeList;
		public List<Privilege> getPrivilegeList(){
			return privilegeList;
		}
		public void setPrivilegeList(List<Privilege> privilegeList){
			this.privilegeList=privilegeList;
		}
角色—权限表(role_privilege)数据如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00027MyBatis%E5%AD%A6%E4%B9%A06-08.jpg)
4. 实现一对多映射的两种方式：
collection的嵌套结果映射
collection集合的嵌套查询

### collection的嵌套结果映射
**1).获取所有的角色以及对应角色所拥有的权限的实现**
1. 在RoleMapper.java新建接口方法：
		//查询所有的角色（带权限列表）
		List<Role> selectAllRoleWithPrivilege();
创建XML方法主体(select标签)：
		<select id="selectAllRoleWithPrivilege" resultMap="roleListPrivilege">
			SELECT 
				r.id,r.role_name,r.enabled,r.create_by,r.create_time,.
				p.id pri_id,p.privilege_name pri_privilege_name,p.privilege_url pri_privilege_url
			FROM
				role r
			LEFT JOIN role_privilege rp ON r.id=rp.role_id
			LEFT JOIN privilege p ON rp.privilege_id=p.id
		</select>
2. ResultMap标签的写法和一对一的类似。
基本的写法非常复杂，就是将所有字段都一一配置。
使用MGB自动生成的RoleMapper.xml代码中有一个基本的resultMap配置：
		<resultMap id="BaseResultMap" type="mbg.model.Role">
			<id column="id" jdbcType="BIGINT" property="id" />
			<result column="role_name" jdbcType="VARCHAR" property="roleName" />
			<result column="enabled" jdbcType="INTEGER" property="enabled" />
			<result column="create_by" jdbcType="BIGINT" property="createBy" />
			<result column="create_time" jdbcType="TIMESTAMP" property="createTime" />
		</resultMap>
所以resultMap可以这样简写：
		<resultMap extends="BaseResultMap" type="Role" id="roleListPrivilege">
			<collection property="privilegeList" columnPrefix="pri_" javaType="Privilege">
				<id property="id" column="id"/>
				<result column="privilege_name" property="privilegeName"/>
				<result column="privilege_url" property="privilegeUrl" />
			</collection>
		</resultMap>
继续查看自动生成的PrivilegeMapper.xml方法，里面有一个BaseResultMap：
		<resultMap id="BaseResultMap" type="mbg.model.Privilege">
			<id column="id" jdbcType="BIGINT" property="id" />
			<result column="privilege_name" jdbcType="VARCHAR" property="privilegeName" />
			<result column="privilege_url" jdbcType="VARCHAR" property="privilegeUrl" />
		</resultMap>
所以最后的resultMap可以简化成这样:
		<resultMap extends="BaseResultMap" type="Role" id="roleListPrivilege">
			<collection property="privilegeList" columnPrefix="pri_" resultMap="mbg.mapper.PrivilegeMapper.BaseResultMap">
			</collection>
		</resultMap>
3. 新建测试类RoleMapperTest.java，并添加测试方法：
		public class RoleMapperTest extends BaseMapperTest{
			@Test
			public void testSelectAllRoleWithPrivilege(){
				SqlSession sqlSession =getSqlSession();
				try{
					RoleMapper roleMapper=sqlSession.getMapper(RoleMapper.class);
					//查询
					List<Role> roleList=roleMapper.selectAllRoleWithPrivilege();
					System.out.println("共有角色数量："+roleList.size());
					for(Role role:roleList){
						System.out.println("角色名："+role.getRoleName());
						System.out.print("拥有权限:");
						for(Privilege privilege:role.getPrivilegeList()){
							System.out.print(" "+privilege.getPrivilegeName()+",");
						}
						System.out.println();
					}
				}finally{
					sqlSession.close();
				}
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00027MyBatis%E5%AD%A6%E4%B9%A06-09.jpg)

**2)实现根据UserId查询该用户的权限：**
1. 将user-role修改为一对多：
![](http://p5ki4lhmo.bkt.clouddn.com/00027MyBatis%E5%AD%A6%E4%B9%A06-10.jpg)
修改User类将Role改为ListRole：
		...
		// 角色信息
		private List<Role> roleList;
		public List<Role> getRoleList() {
			return roleList;
		}
		public void setRole(List<Role> roleList) {
			this.roleList=roleList;
		}
		...
添加UserMapper接口方法：
		//一对多映射,collection实现
		User selectUserWithRoleListByUserId(Long id);
添加UserMapper.xml方法：
		<select id="selectUserWithRoleListByUserId" resultMap="userListRole">
			  SELECT
				u.id,
				u.id,
				u.user_name,
				u.user_password,
				u.user_email,
				u.user_info,
				u.head_img,
				u.create_time,
				r.id as role_id, 
				r.role_name role_role_name,
				r.enabled role_enabled,
				r.create_by role_create_by,
				r.create_time role_create_time,
				p.id role_pri_id,
				p.privilege_name role_pri_privilege_name,
				p.privilege_url role_pri_privilege_url
			FROM user u
			LEFT JOIN user_role ur ON u.id=ur.user_id
			LEFT JOIN role r ON ur.role_id=r.id
			LEFT JOIN role_privilege rp ON r.id=rp.role_id
			LEFT JOIN privilege p ON rp.privilege_id=p.id
			WHERE u.id=#{id}  
		</select>
注意privilege的字段的别名，**需要在原来的别名前加上role_前缀**。
添加ResultMap:
因为已经配置了Role的resultMap，只要直接引用roleListPrivilege就可以了：
		  <resultMap type="User" extends="BaseResultMap" id="userListRole">
		  	<collection property="roleList" columnPrefix="role_" resultMap="mbg.mapper.RoleMapper.roleListPrivilege"></collection>
		  </resultMap>
2. 创建测试类：
		//测试一对多映射的collection非嵌套实现
		@Test
		public void testSelectUserWithRoleListByUserId(){
			SqlSession sqlSession =getSqlSession();
			try{
				UserMapper userMapper =sqlSession.getMapper(UserMapper.class);		//获取UserMapper接口
				User user=userMapper.selectUserWithRoleListByUserId(1L);  //查询user
				//断言测试user是否为空
				Assert.assertNotNull(user);
				//角色数是否>0
				Assert.assertTrue(user.getRoleList().size()>0);
				//输出角色及权限
			//	System.out.println(user);
				for(Role role:user.getRoleList()){
					System.out.println("角色名："+role.getRoleName());
					System.out.print("拥有权限:");
					for(Privilege privilege:role.getPrivilegeList()){
						System.out.print(" "+privilege.getPrivilegeName()+",");
					}
					System.out.println();
				}
			}finally{
				sqlSession.close();
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00027MyBatis%E5%AD%A6%E4%B9%A06-11.jpg)
3. 一个映射的配置不仅可以被嵌套的配置引用，其自身也可以使用。复杂的映射都是由这种简单的映射配置组成的。
要配置一个复杂的映射，一定要从最基础（最内层）开始配置，每增加一些配置就进行相应的测试，这样容易发现和解决问题。
4. association标签可以和collection标签互相组合或者嵌套从而得到任何想要的数据。

**3)MyBatis的处理规则及注意事项：**
1. MyBatis在处理结果的时候，会判断结果是否*相同*，如果相同则只保留第一个结果。
2. 相同的含义（相同的判断）：
如果配置了id标签，则id相同的结果会被合并（不相同的部分为一对多的多合并成列表）；
如果没有配置id标签则会将所有字段一一比较，若每个字段都相同则只保留第一个，如果有一个字段不相同则不按相同处理（不相同）。
3. 注意：
如果在嵌套结果中配置了id标签,但是查询语句中并没有查询id属性配置的列,那么所有的id都为null,这时最后查出来的嵌套结果集就只有一条数据。
所以在配置id的时候，一定要查询相对的列。
4. id属性：
`<id column="" property="" />`
中不止可以配置id字段，也可以配置其他任何字段，但是如果不是唯一的字段会使一些不同的字段莫名其妙的合并。
如果未配置id属性则要比较所有的字段，这是一个平方级别的比较，比配置id属性的效率要低得多。

### collection集合的嵌套查询
collection集合的嵌套查询和association的集合嵌套查询一模一样，延迟加载的方式也相同。
虽然看似相同，但是实际测试的时候踩了很多坑，一定要注意。

**1)使用集合嵌套查询实现上面的例子：分析**
1. 首先这是一个两层的嵌套，一定要自下而上实现，并且每一层的方法都可以单独使用。最后的结果都是以前一个方法为基础的。
2. 一定要自下而上的连接关联表，而不是自上而下连接关联表。
`privilege left/inner/right JOIN role_privilege`
`role r LEFT JOIN user_role ur ON r.id=ur.role_id`
而一对一时的那种情况就是自上而下。一对多千万不能自上而下，否则会出现很多意外的合并与错误（亲测）。

**2)实现上面的例子：**
1. PriviligeMapper.xml的代码：
		<resultMap id="BaseResultMap" type="Privilege">
			<id column="id" jdbcType="BIGINT" property="id" />
			<result column="privilege_name" jdbcType="VARCHAR" property="privilegeName" />
			<result column="privilege_url" jdbcType="VARCHAR" property="privilegeUrl" />
		</resultMap>
		
		<sql id="Base_Column_List">
			id, privilege_name, privilege_url
		</sql>
		
		<select id="selectByPrimaryKey2" resultMap="BaseResultMap">
			select 
			<include refid="Base_Column_List" />
			from privilege
			INNER JOIN role_privilege rp ON rp.privilege_id=id
			where rp.role_id = #{id}
		</select>
2. RoleMapper.xml的代码：
		<resultMap id="BaseResultMap" type="mbg.model.Role">
			<id column="id" jdbcType="BIGINT" property="id" />
			<result column="role_name" jdbcType="VARCHAR" property="roleName" />
			<result column="enabled" jdbcType="INTEGER" property="enabled" />
			<result column="create_by" jdbcType="BIGINT" property="createBy" />
			<result column="create_time" jdbcType="TIMESTAMP" property="createTime" />
		</resultMap>

		<resultMap extends="BaseResultMap" type="Role" id="roleListPrivilegeMap">
			<collection property="privilegeList" column="id" fetchType="lazy" select="mbg.mapper.PrivilegeMapper.selectByPrimaryKey2">
			</collection>
		</resultMap>

		<select id="selectRoleWithPrivilegeByPrimaryKey" resultMap="roleListPrivilegeMap">
			select 
				 	r.id,r.role_name,r.enabled,r.create_by,r.create_time
			from role r
			LEFT JOIN user_role ur ON r.id=ur.role_id
			where ur.user_id = #{id}
		</select>
3. UserMapper.xml代码：
		<resultMap id="BaseResultMap" type="mbg.model.User">
			<id column="id" property="id" />
			<result column="user_name" jdbcType="VARCHAR" property="userName" />
			<result column="user_password" jdbcType="VARCHAR" property="userPassword" />
			<result column="user_email" jdbcType="VARCHAR" property="userEmail" />
			<result column="create_time" jdbcType="TIMESTAMP" property="createTime" />
		</resultMap>
		<resultMap extends="BaseResultMap" id="ResultMapWithBLOBs" type="mbg.model.User">
			<result column="user_info" jdbcType="LONGVARCHAR" property="userInfo" />
			<result column="head_img" jdbcType="LONGVARBINARY" property="headImg" />
		</resultMap>
		
		<resultMap type="User" extends="ResultMapWithBLOBs" id="userListRoleMap">
			<collection property="roleList" column="id" select="mbg.mapper.RoleMapper.selectRoleWithPrivilegeByPrimaryKey"></collection>
		</resultMap>
		
		<select id="selectUserWithRoleListByUserId2" resultMap="userListRoleMap">
			SELECT
				id,
				user_name,
				user_password,
				user_email,
				user_info,
				head_img,
				u.create_time
			FROM user u
			WHERE u.id=#{id} 
		</select>
注意这里的顶层查询没有任何的子查询。
4. 添加接口方法：
		//一对多映射,collection嵌套实现
		User selectUserWithRoleListByUserId2(Long id);
添加测试类方法：
		//测试一对多映射的collection嵌套实现
		@Test
		public void testSelectUserWithRoleListByUserId2(){
			SqlSession sqlSession =getSqlSession();
			try{
				UserMapper userMapper =sqlSession.getMapper(UserMapper.class);		//获取UserMapper接口
				User user=userMapper.selectUserWithRoleListByUserId2(1L);  //查询user
				//断言测试user是否为空
				Assert.assertNotNull(user);
				//角色数是否>0
				Assert.assertTrue(user.getRoleList().size()>0);
				//输出角色及权限
				System.out.println(user);
				for(Role role:user.getRoleList()){
					System.out.println("角色名："+role.getRoleName());
					System.out.print("拥有权限:");
					for(Privilege privilege:role.getPrivilegeList()){
						System.out.print(" "+privilege.getPrivilegeName()+",");
					}
					System.out.println();
				}
				}finally{
				sqlSession.close();
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00027MyBatis%E5%AD%A6%E4%B9%A06-12.jpg)
5. MyBtis的处理规则（合并规则）以后还是得更深入的了解才行。

---
## 4.鉴别器映射
**1)鉴别器映射的简介及标签介绍：**
1. 有时候一个单独的数据库查询会返回很多不同数据类型的结果集。
discriminator（鉴别器）标签就是来处理这种情况的。
相当于Java中的switch...case...选择语句
2. discriminator标签的两个常用属性：
	- `column`：设置要进行鉴别比较的列（字段）。
	- `javaType`：用于指定列的类型，保证使用相同的数据类型进行比较。
3. discriminator标签可以包含一个或者多个case标签，case标签有三个属性：
	- `value`：与column列比较的值。
	- `resultMap`：当column值等于case的value值时用于指定可以配置使用resultMap指定的映射，优先级高于resultType。
	- `resultType`：当column值等于case的value值时用于指定可以配置使用resultType指定的映射
4. case标签下可以使用所有resultMap标签可以使用的子标签及属性，用法也一样。
但是有一点很重要的区别：
resultMap标签配置了部分属性，剩余的属性会自动进行映射。
case标签配置了部分属性，剩余的属性不会自动映射，设为空。
5. **鉴别器是一种很少使用的方式**，使用之前一定要完全掌握。未完全掌握前要避免使用。

**2)鉴别器映射的测试：**
1. 需求：
按照用户Id查询，当角色的enable属性为1，可用时，查询该角色的权限；角色不可用时，只显示角色的基本信息，不显示权限信息。
2. 在RoleMapper.xml添加配置：
		<resultMap type="Role" id="roleListPrivilegeMapChoose">
			<discriminator javaType="int" column="enabled">
				<case value="1" resultMap="roleListPrivilegeMap"></case>
				<case value="0" resultMap="BaseResultMap"></case>
			</discriminator>
		</resultMap>
并修改方法如下：
![](http://p5ki4lhmo.bkt.clouddn.com/00027MyBatis%E5%AD%A6%E4%B9%A06-13.jpg)
3. 其余方法都不变，修改数据库，将Role表的普通用户的enabled改为0。
运行测试方法testSelectUserWithRoleListByUserId2(),测试结果为：
![](http://p5ki4lhmo.bkt.clouddn.com/00027MyBatis%E5%AD%A6%E4%B9%A06-14.jpg)

---
## 5.SQL语句的封装
1. 查看MBG自动生成的代码时，会发现有一个SQL标签，于封装SQL语句，和动态标签Bind的使用差不多。
2. 例子：
		<resultMap id="BaseResultMap" type="Privilege">
			<!--略-->
		</resultMap>
		
		<sql id="Base_Column_List">
			id, privilege_name, privilege_url
		</sql>
		
		<select id="selectByPrimaryKey2" resultMap="BaseResultMap">
			select 
			<include refid="Base_Column_List" />
			from privilege
			...
		</select>
3. sql标签可以通过如下标签来调用：
`<include refid="SQL标签的ID" />`
4. 熟练使用sql标签可以复用很多代码，节省代码量。

---
