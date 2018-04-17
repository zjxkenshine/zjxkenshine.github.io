---
title: MyBatis学习笔记（七）:使用存储过程与枚举类等
date: 2018-04-04 22:25:24
tags: MyBatis
categories: J2EE框架

---
## 0.学习准备
1. 学习资料：
书本：《MyBatis从入门到精通》
视频：某培训班视频
资料：[MyBatis中文文档](http://www.mybatis.org/mybatis-3/zh/index.html)

---
## 1.存储过程简介及创建
1. 存储过程在数据库中很常见，MyBatis中的存储过程调用方法与MySQL中的差不多。一般的存储过程调用都比较复杂。
创建几个不同类型的存储过程用于测试。
具体的存储过程使用方法在M
2. 创建第一个存储过程：
根据用户id查询用户信息：
		-- 第一个存储过程，根据用户id查询其他用户信息
		DROP PROCEDURE IF EXISTS select_user_by_id;
		delimiter $$
		CREATE PROCEDURE select_user_by_id(
			IN userId BIGINT,
			OUT userName VARCHAR(50),
			OUT userPassword VARCHAR(50),
			OUT userEmail VARCHAR(50),
			OUT userInfo TEXT,
			OUT headImg BLOB,
			OUT createTime DATETIME
		)
		BEGIN
			-- 查询
			SELECT user_name,user_password,user_email,user_info,head_img,create_time
			INTO userName,userPassword,userEmail,userInfo,headImg,createTime
			FROM user
			WHERE id=userId;
		END
		$$
		delimiter ;
添加结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00028MyBatis%E5%AD%A6%E4%B9%A07-01.jpg)
3. 创建第二个存储过程：
实现分页查找。
根据用户名和分页参数查询数据总数和分页查询的结果集。
		-- 第二个存储过程，使用用户名分页查询
		DROP PROCEDURE IF EXISTS select_user_page;
		delimiter $$
		CREATE PROCEDURE select_user_page(
			IN userName VARCHAR(50),
			IN _offset BIGINT,
			IN _limit BIGINT,
			OUT total BIGINT
		)
		BEGIN
			-- 查询数据总数
			SELECT COUNT(*) INTO total
			FROM user
			WHERE user_name LIKE CONCAT('%',userName,'%');
			-- 分页查询
			SELECT * FROM user
			WHERE user_name LIKE CONCAT('%',userName,'%')
			LIMIT _offset,_limit;
		END $$
		delimiter ;
添加结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00028MyBatis%E5%AD%A6%E4%B9%A07-02.jpg)
4. 创建第三个存储过程：
保存用户信息和角色关联信息。
		-- 第三个存储过程,保存用户信息和角色关联信息
		DROP PROCEDURE IF EXISTS `insert_user_and_roles`;
		delimiter $$
		CREATE PROCEDURE `insert_user_and_roles`(
			OUT userId BIGINT,
			IN userName VARCHAR(50),
			IN userPassword VARCHAR(50),
			IN userEmail VARCHAR(50),
			IN userInfo TEXT,
			IN headImg BLOB,
			OUT createTime DATETIME,
			IN roleIds VARCHAR(200)
		)
		BEGIN
			-- 设置当前时间
			SET createTime=NOW();
			-- 插入用户数据
			INSERT INTO `user`(user_name,user_password,user_email,user_info,head_img,create_time)
			VALUES(userName,userPassword,userEmail,userInfo,headImg,createTime);
			-- 返回自增主键
			SELECT LAST_INSERT_ID() INTO userId;
			-- 插入用户和角色的关系表
			-- 在前后插入，方便使用instr正确匹配
			SET roleIds=CONCAT(',',roleIds,',');
			-- 执行插入(使用select子查询)
			INSERT INTO user_role(user_id,role_id)
			SELECT userId,id FROM role
			WHERE INSTR(roleIds,CONCAT(',',id,',')) >0;
		END
		$$
		delimiter ;
这里的`select...from role WHERE INSTR(roleIds,CONCAT(',',id,',')) >0;`的`>0`加不加效果都一样。这句话相当于：
		SELECT...FROM role WHERE id IN(roleIds);
也相当于：
		SELECT...FROM WHERE roleIds LIKE concat('%,',id,',%');
执行结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00028MyBatis%E5%AD%A6%E4%B9%A07-03.jpg)

5. 创建第四个存储过程：
删除用户信息。
		-- 第四个存储过程，删除用户信息及角色关联信息
		DROP PROCEDURE IF EXISTS `delete_user_by_id`;
		delimiter $$
		CREATE PROCEDURE `delete_user_by_id`(IN userId BIGINT)
		BEGIN
			DELETE FROM `user` WHERE id =userId;
			DELETE FROM user_role WHERE user_id=userId;
		END $$
		delimiter ;
添加结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00028MyBatis%E5%AD%A6%E4%B9%A07-04.jpg)

6. 无聊的一个小测试，原来Like可以用来实现in的效果：
		SET @usenames='测试用户1,测试用户,测试用户2';
		SELECT * FROM `user` WHERE @usenames LIKE CONCAT('%',user_name,'%');
测试结果:
![](http://p5ki4lhmo.bkt.clouddn.com/00028MyBatis%E5%AD%A6%E4%B9%A07-05.jpg)

---
## 2.存储过程的具体使用
### 第一，二个存储过程使用测试
**1)第一个存储过程：**
1. 第一个存储过程：根据用户id查询用户信息
创建接口方法：
		//第一个存储过程：使用id查询用户数据
		void selectUserById(User user);
**注意：**
	- 存储过程的出参并不是返回值，而是修改传入的参数
	- 这个存储过程没有返回值，所以使用的是void
2. 创建xml映射方法：
		<select id="selectUserById" statementType="CALLABLE" useCache="false">
			{call select_user_by_id(
				#{id,mode=IN},
				#{userName,mode=OUT,jdbcType=VARCHAR},
				#{userPassword,mode=OUT,jdbcType=VARCHAR},
				#{userEmail,mode=OUT,jdbcType=VARCHAR},
				#{userInfo,mode=OUT,jdbcType=VARCHAR},
				#{headImg,mode=OUT,jdbcType=BLOB,javaType=_byte[]},
				#{createTime,mode=OUT,jdbcType=TIMESTAMP}
			)}
		</select>
**注意：**
	- 在调用存储过程的方法时，需要把`statementType`属性的值改为CALLABLE。
	- 存储过程不支持MyBatis的二级缓存，为了避免出错，将`useCache`改为false。
	- 在使用参数过程中，除了必须的属性名外，还要指定参数的mode(模式)：
	有IN,OUT,和INOUT三种选项。MySQL使用IN入参时内置了jdbcType，不用指定，但是OUT,INOUT时需要指定jdbcType
	- 注意headImg字段的配置。
	MyBatis映射的类中不推荐使用基本类型，BLOB默认对应的是Byte类型，但是一般都是用byte[]数组存值，所以javaType设为'_byte[]'
	- javaType中，基本类型需要使用_设置，如javaType=byte代表的是Byte封装类，而_byte才代表byte基本类型。
3. 创建测试方法：
		//第一个存储过程的测试
		@Test
		public void testSelectUserById(){
			SqlSession sqlSession =getSqlSession();
			try{
				UserMapper userMapper=sqlSession.getMapper(UserMapper.class);
				User user =new User();
				user.setId(1L);
				//调用存储过程并出参
				userMapper.selectUserById(user);
				Assert.assertEquals("admin", user.getUserName());
				System.out.println(user);
			}finally{
				sqlSession.close();
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00028MyBatis%E5%AD%A6%E4%B9%A07-06.jpg)
4. 使用这种类型（没有返回，使用出参获取信息）的存储过程时需要注意的问题。
	- 使用出参方式有两种方式接收出参的值：**javaBean和Map方式。**
		- javaBean方式接收出参时必须保证所有的出参参数在JavaBean中都有对应的属性存在，否则会报异常提示没有该属性的setter方法
		- 使用Map方式则不需要保证所有的出参都有对应的属性。
	- MyBatis中写的出参要与数据库存储过程中的出参类型相符合，否则也会出错。

**2)第二个存储过程：**
1. 根据用户名和分页参数查询数据总数和分页查询的结果集。
创建接口方法：
		//第二个存储过程：实现分页查询
		/**
		 * @param:
		 * userName
		 * pageNum
		 * pageSize
		 * total
		 * @return
		 */
		List<User> selectUserPage(Map<String,Object> params);
因为参数不只是一个对象内的，所以使用Map做参数，就不用给user对象添加额外的属性了。
而且这个存储过程中有了返回值（输出值，并非函数的那种返回值），所以接口方法返回值设为`List<User>`;
2. 创建XML方法:
		<select id="selectUserPage" statementType="CALLABLE" useCache="false" resultMap="ResultMapWithBLOBs">
			{call select_user_page(
				#{userName,mode=IN},
				#{offset,mode=IN},
				#{limit,mode=IN},
				#{total,mode=OUT,jdbcType=BIGINT}
			)}
		</select>
和第一个存储过程的最大区别就是有返回值，所以需要配置resultMap标签。
使用Map作为出参对象时，如果map中没有这个key，则会自动添加一对的键值对。
同样需要设置`statementType="CALLABLE" useCache="false"`。
3. 创建测试方法：
		//第二个存储过程测试
		@Test
		public void testSelectUserPage(){
			SqlSession sqlSession =getSqlSession();
			try{
				UserMapper userMapper=sqlSession.getMapper(UserMapper.class);  //获取userMapper
				Map<String,Object> params=new HashMap<String,Object>();  //创建参数map
				params.put("userName","测试" );
				params.put("offset", 2);
				params.put("limit", 2);
				//注意并没有键total的对象
				List<User> userList =userMapper.selectUserPage(params);
				System.out.println("总数为"+params.get("total"));
				System.out.println("本页用户为:");
				for (User user : userList) {
					System.out.println(user);
				}
			}finally{
					sqlSession.close();
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00028MyBatis%E5%AD%A6%E4%B9%A07-07.jpg)

### 第三，四个存储过程使用测试
因为这两个存储过程实现的效果刚好相反，所以书上将两个方法放到一起测试：
第三个存储过程：添加用户信息和角色关联信息
第四个存储过程：删除用户信息和角色关联信息
1. 添加两个接口方法：
		//第三个存储过程：添加角色和角色关联信息
		int insertUserAndRoles(@Param("user")User user,@Param("roleIds")String roleIds);
		//第四个存储过程:删除角色和角色关联信息
		int deleteUserById(Long id);
使用了Param传递了多个参数，在xml中配置user中的属性就得使用user.属性名来获取。
2. 添加两个XML方法:
		<insert id="insertUserAndRoles" statementType="CALLABLE"> 
			{call insert_user_and_roles(
				#{user.id,mode=OUT,jdbcType=BIGINT},
				#{user.userName,mode=IN},
				#{user.userPassword,mode=IN},
				#{user.userEmail,mode=IN},
				#{user.userInfo,mode=IN},
				#{user.headImg,mode=IN},
				#{user.createTime,mode=OUT,jdbcType=TIMESTAMP},
				#{roleIds,mode=IN}
			)}
		</insert>
		<delete id="deleteUserById" statementType="CALLABLE">
			{call delete_user_by_id(#{id,mode=IN})}
		</delete>
MyBtis的缓存配置是针对查询的，所以其他方法就不用配置useCache属性了（也没有这个属性）。
3. 第三存储过程测试：
测试方法：
		//第三个存储过程测试
		@Test
		public void testInsertUserAndRoles(){
			SqlSession sqlSession =getSqlSession();
			try{
				UserMapper userMapper=sqlSession.getMapper(UserMapper.class);  //获取userMapper
				//创建需要添加的对象
				User user = new User();
				user.setUserName("测试用户007");
				user.setUserInfo("这是一个特工");
				user.setUserEmail("007@uk.com");
				user.setUserPassword("6666");
				user.setHeadImg(null);
				//插入用户信息
				userMapper.insertUserAndRoles(user, "1,2,5");
				Assert.assertNotNull(user.getId());      //测试回传的参数
				Assert.assertNotNull(user.getCreateTime());
				//使用上一节的映射查询所有用户及角色权限的信息
				User user1=userMapper.selectUserWithRoleListByUserId(user.getId());  //查询user
				//断言测试user1是否为空
				Assert.assertNotNull(user1);
				//输出user1
				System.out.println(user1);
				//输出所有角色及权限
				for(Role role:user1.getRoleList()){
					System.out.println("角色名："+role.getRoleName());
					System.out.print("拥有权限:");
					for(Privilege privilege:role.getPrivilegeList()){
						System.out.print(" "+privilege.getPrivilegeName()+",");
					}
					System.out.println();
				}
			}finally{
					sqlSession.rollback();  //回滚
					sqlSession.close();
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00028MyBatis%E5%AD%A6%E4%B9%A07-08.jpg)
4. 第四个存储过程测试：
测试方法：
		//第四个存储过程测试
		@Test
		public void testDeleteUserById(){
			SqlSession sqlSession =getSqlSession();
			try{
				UserMapper userMapper=sqlSession.getMapper(UserMapper.class);  //获取userMapper
				int result=userMapper.deleteUserById(1014L);
				Assert.assertEquals(1, result);  //是否删除一条数据
				//再次查询
				User user=userMapper.selectUserWithRoleListByUserId(1014L);  //查询user
				//查询结果应该为空
				Assert.assertNull(user);
			}finally{
				sqlSession.rollback();  //回滚
				sqlSession.close();
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00028MyBatis%E5%AD%A6%E4%B9%A07-09.jpg)

### 游标参数的简单介绍
1. MySQL不支持MyBatis的游标参数，但是MySQL是支持游标的（详见Mysql学习笔记八），mysql的游标没有参数。
2. 由于我多oracle不太了解，所以这部分以后再深入学习
3. 游标参数的简单使用就是在存储过程内创建游标，给该游标传入参数（一般是一个集合）时，jdbcType指定为CURSOR,而javaType指定为ResultSet就可以了。

---
## 3. 使用枚举类或者自定义的类处理器
### 使用MyBatis提供的枚举类处理器
**1)测试准备：**
1. 为role表的enabled属性创建枚举类
因为这个字段字段只有两个可选值0和1，如果像这种字段的可选值多了之后，对值的校验就变的复杂了，所以需要使用枚举类来解决。
2. 创建枚举类：
		package mbg.type;
		
		public enum Enabled {
			disabled, //禁用，索引为0
			enabled; //启用，索引为1
		}
枚举类除了本身的字面外，还可以通过枚举的ordinal()方法获取枚举值的索引。
修改role类的enabled属性如下：
		private Enabled enabled;
		public Enabled getEnabled() {
			return enabled;
		}
		public void setEnabled(Enabled enabled) {
			this.enabled = enabled;
		}


**2)当数据库为对应字段为Enum类或者VARCHAR时：**
1. 首先修改Role的enabled字段：
		ALTER TABLE role MODIFY enabled enum('disabled','enabled') AFTER role_name;
但是需要注意，数据库字段的枚举的索引是从1开始的，而java的枚举类的索引是从0开始的。
2. 修改自动生成的xml方法：将jdbcType改为VARCHAR
![](http://p5ki4lhmo.bkt.clouddn.com/00028MyBatis%E5%AD%A6%E4%B9%A07-11.jpg)
3. 添加测试类方法：
在RoleMapper.xml中添加如下方法：
		//测试枚举类1,数据库字段也使用ENUM
		@Test
		public void testRoleEnabledEnum(){
			SqlSession sqlSession =getSqlSession();
			try{
				RoleMapper roleMapper=sqlSession.getMapper(RoleMapper.class);
				//按Id查询
				Role role=roleMapper.selectByPrimaryKey(1L);
				System.out.println(role);
				//按Id更新
				role.setEnabled(Enabled.disabled);
				int result=roleMapper.updateByPrimaryKey(role);
				Assert.assertEquals(1, result);
				Role role1=roleMapper.selectByPrimaryKey(1L); //再次查询
				System.out.println(role1);
			}finally{
				sqlSession.rollback();
				sqlSession.close();
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00028MyBatis%E5%AD%A6%E4%B9%A07-10-1.jpg)
4. 此时java或者数据库的索引都是用表面值进行操作。
当然也可以向数据库的字段传入索引值，但是这需要配置核心配置文件（下面会介绍），或者修改Role类的索引使其存的是索引，并且要修改java索引与数据库字段的Enum类索引相匹配。这里就不介绍了。
数据库字段使用VARCHAR时效果和使用ENUM相同，只是比较浪费时间。
5. java枚举类使用的默认：
这个处理器默认使用的类处理器是`org.apache.ibatis.type.EnumTypeHandler`,这时在使用java的枚举类的时候，使用的就是字面值，对应Enabled而言便是"enabled"及"disabled"字符串。

**3)当数据库为对应字段为Int类时：**
1. 将数据库的字段修改回INT:
		ALTER TABLE role MODIFY enabled INT AFTER role_name;
但是还需要手动修改值，因为转过来的是1和2，而不是0和1。
将RoleMapper.xml自动生成的代码也修改过来：
![](http://p5ki4lhmo.bkt.clouddn.com/00028MyBatis%E5%AD%A6%E4%B9%A07-12.jpg)
测试代码不变。
2. 这时如果要实现和上述一样的效果时，就需要注意一点了：
在查询时，将数据库的int值转换为Java的枚举类值；
在保存，更新或者作为查询条件时将java枚举类值转换为数据库中的int值。
3. 由于默认的类处理器使用的是枚举类的自面值进行匹配，所以一定会出错，这时就需要另一个类处理器了：`org.apache.ibatis.type.EnumOrdinalTypeHandler`处理器。它的效果时使用枚举类的索引进行处理。
4. 需要在核心配置文件中配置如下代码：
		<typeHandlers>
			<typeHandler javaType="mbg.type.Enabled" handler="org.apache.ibatis.type.EnumOrdinalTypeHandler"/>
		</typeHandlers>
在typeHandlers标签中通过javaType属性指定需要处理的java类，使用handler指定需要使用的类型处理器。更多的类型处理器可以查看[MyBatis中文文档](http://www.mybatis.org/mybatis-3/zh/index.html)。
5. 运行测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00028MyBatis%E5%AD%A6%E4%B9%A07-13.jpg)

### 使用自定义的类处理器
**1)为什么要使用自定义的类处理器：**
1. 有很多时候，枚举类的值既不是枚举的字面值，也不是枚举的索引值，而是自己额外指定的值。这时就需要使用自定义的类处理器了。
2. 修改枚举类如下：
		package mbg.type;
		
		public enum Enabled {
			enabled(1), //启用，值为1
			disabled(0); //禁用，值为0
			
			private final int value;
			private Enabled(int value){
				this.value=value;
			}
			public int getValue(){
				return value;
			}
		}
此时使用的value,既不是索引值也不是字面值，而是（）内指定的值，这种用法也很多。

**2)简单的实现自定义类处理器：**
1. 在mbg.type类下新建EnabledTypeHandler类，代码如下：
		package mbg.type;
		
		import java.sql.CallableStatement;
		import java.sql.PreparedStatement;
		import java.sql.ResultSet;
		import java.sql.SQLException;
		import java.util.HashMap;
		import java.util.Map;
		import org.apache.ibatis.type.JdbcType;
		import org.apache.ibatis.type.TypeHandler;
		
		public class EnabledTypeHandler implements TypeHandler<Enabled> {
			
			private final Map<Integer, Enabled> enabledMap=new HashMap<Integer, Enabled>();
			//默认无参构造方法
			public EnabledTypeHandler(){
				// TODO Auto-generated constructor stub
				for(Enabled enabled:Enabled.values()){
					enabledMap.put(enabled.getValue(), enabled);
				}
			}
		
			@Override
			public void setParameter(PreparedStatement ps, int i, Enabled parameter,
					JdbcType jdbcType) throws SQLException {
				// TODO Auto-generated method stub
				ps.setInt(i, parameter.getValue());
			}
		
			@Override
			public Enabled getResult(ResultSet rs, String columnName)
					throws SQLException {
				// TODO Auto-generated method stub
				Integer value=rs.getInt(columnName);
				return enabledMap.get(value);
			}
		
			@Override
			public Enabled getResult(ResultSet rs, int columnIndex) throws SQLException {
				// TODO Auto-generated method stub
				Integer value=rs.getInt(columnIndex);
				return enabledMap.get(value);
			}
		
			@Override
			public Enabled getResult(CallableStatement cs, int columnIndex)
					throws SQLException {
				// TODO Auto-generated method stub
				Integer value=cs.getInt(columnIndex);
				return enabledMap.get(value);
			}
		}
2. 这个方法实现了TypeHandler接口，并且针对四个接口方法对Enabled类型进行转换。
3. TypeHandler实现类除了默认的无参构造方法还有一个隐含的带有一个Class参数的构造方法：
		public EnabledTypeHandler(Class<?> type){
			this();
		}
针对特定的接口处理类型时，这个构造方法可以写出通用的类型处理器，就像前面的两个枚举类型的处理器一样。
4. 还需要配置mybatis-config.xml核心配置文件，修改如下：
		<typeHandlers>
			<typeHandler javaType="mbg.type.Enabled" handler="mbg.type.EnabledTypeHandler"/>
		</typeHandlers>
再次运行测试类进行测试：
![](http://p5ki4lhmo.bkt.clouddn.com/00028MyBatis%E5%AD%A6%E4%B9%A07-14.jpg)
5. 这只是简单的类处理器实现，复杂的类处理器可以查件`org.apache.ibatis.type`包下的源码。

---
## 4. 对java8日期(JSR-310)的支持
1. MyBatis从3.4.0版本开始增加了对java8日期(JSR-310)的支持。
详情可以查看:
<https://github.com/mybatis/typehandlers-jsr310>
2. MyBatis3.4.0及以上版本只需要在Maven的pom.xml添加以下依赖即可：
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis-typehandlers-jsr310</artifactId>
			<version>1.0.2</version>
		</dependency>
3. MyBatis3.4.0以前的版本需要在此基础上添加类处理器的配置：
		<typeHandlers>
			<typeHandler handler="org.apache.ibatis.type.InstantTypeHandler"/>
			<typeHandler handler="org.apache.ibatis.type.LocalDateTypeHandler"/>
			<typeHandler handler="org.apache.ibatis.type.LocalTimeTypeHandler"/>
			<typeHandler handler="org.apache.ibatis.type.LocalDateTimeTypeHandler"/>
			<typeHandler handler="org.apache.ibatis.type.OffsetTimeTypeHandler"/>
			<typeHandler handler="org.apache.ibatis.type.ZonedDateTimeTypeHandler"/>
			<typeHandler handler="org.apache.ibatis.type.YearTypeHandler"/>
			<typeHandler handler="org.apache.ibatis.type.MonthTypeHandler"/>
		</typeHandlers>
4. 上面的typeHandler都没有指定javaType，因为这些类处理器继承了BaseTypeHandler<T>类,而BaseTypeHandler<T>继承了TypeReference<T>类，MyBatis会对继承了TypeReference<T>的类进行处理，获取泛型T作为javaType的值。
5. 但是不知道为什么我的MyBatis配置完typeHandler之后就显示unsupportedVersionError,可能MyBatis3.3.0也支持吧。这种问题以后熟了再解决吧。

---

