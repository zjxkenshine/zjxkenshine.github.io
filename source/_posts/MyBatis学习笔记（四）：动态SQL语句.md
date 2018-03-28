---
title: MyBatis学习笔记（四）：动态SQL语句
date: 2018-03-26 15:46:47
tags: MyBatis
categories: Java框架

---
## 0.学习准备
1. 学习资料：
书本：《MyBatis从入门到精通》
视频：某培训班视频
资料：[MyBatis官方文档](http://www.mybatis.org/mybatis-3/zh/index.html)

---
## 1.if的用法：
**1)动态SQL语句：**
1. 使用标签简化JDBC对字符串拼接的操作。
MyBatis3之前有很多的标签，现在的MyBatis采用和OGNL:Object Graphic Navigation Language(对象图导航语言)表达式消除了许多标签。（使用方法在最后）
2. 目前的MyBatis在XML中支持的标签：
	- if
	- choose(when,otherwise)
	- trim ，where,set
	- foreach
	- bind

**2)WHERE条件中使用if：**
1. if标签经常用于where，update标签，也可以搭配insert使用。
2. 需求：
实现高级的查询用户的功能：
若只输入用户名，根据用户名进行模糊查询
若只输入邮箱名，根据邮箱进行完全匹配
若同时输入用户和邮箱，使用这两个条件查询匹配的用户
3. 在UserMapper.java接口中添加方法：
		//where中的if标签，根据不同条件来查询User
		List<User> selectByUser(User user);
添加UserMapper.xml的方法：
		  <select id="selectByUser" resultType="User">
		  	SELECT 
		  		id,user_name,user_password,user_email,user_info,head_img,create_time
		  	FROM User
		  	WHERE 1
		  	<if test="userName!=null and userName!=''">
		  		and user_name like concat('%',#{userName},'%')
		  	</if>
		  	<if test="userEmail!=null and userEmail!=''">
		  		and user_email=#{userEmail}
		  	</if>
		  </select>
`where 1`可以保证没有没有用户名和邮箱的时候SQL能正常执行。
4. if标签中有一个必填属性test,里面填的是一个符合OGNL要求的判断表达式：
返回值可以为true和false（推荐），除此之外还可以返回0代表false，返回其他值都为true。
有如下的规则：
		property =/!= null:适用于所有类型用于判断是否为空
		property =/!= '':仅适用于字符串类型，判断是否为空字符串
		and和or:与和或
注意：**test属性中property是实体类属性名而不是字段名。**
5. 新增测试类方法：
		//where内使用if标签动态sql
		@Test
		public void testSelectByUser(){
			//获取sqlSession
			SqlSession sqlSession =getSqlSession();
			try{
				//获取UserMapper接口
				UserMapper userMapper =sqlSession.getMapper(UserMapper.class);
				//创建查询对象
				User user=new User();
				
				//情况一：仅有用户名
				user.setUserName("ad");
				//调用selectByUser方法
				List<User> luser=userMapper.selectByUser(user);
				//判断luser长度是否为大于0
				Assert.assertTrue(luser.size()>0);
				
				//情况二：仅有邮箱
				user.setUserName(null);
				user.setUserEmail("6666@00");
				//调用selectByUser方法
				List<User> luser2=userMapper.selectByUser(user);
				//判断luser长度是否大于0
				Assert.assertTrue(luser2.size()>0);
				
				//情况二：仅有邮箱
				user.setUserName("测试");
				//调用selectByUser方法
				List<User> luser3=userMapper.selectByUser(user);
				//判断luser长度是否大于0
				Assert.assertTrue(luser3.size()>0);
			}finally{
				sqlSession.close();
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00024MyBatis%E5%AD%A6%E4%B9%A04-01.jpg)

**3)update中使用if：**
1. 需求：
实现用户信息的更新：
只更新有变化的字段，不能将已有值的字段修改为null或空字符串。
2. 创建接口方法：
		//if根据条件动态更新
		int updateUserById(User user);
新增xml方法：
		  <update id="updateUserById">
		  	UPDATE user SET
		  	<if test="userName!=null and userName!=''">
				user_name=#{userName},
			</if>
			<if test="userPassword!=null and userPassword!=''">
				user_password=#{userPassword},
			</if>
			<if test="userEmail!=null and userEmail!=''">
				user_email=#{userEmail},
			</if>
			<if test="userInfo!=null and userInfo!=''">
				user_info=#{userInfo},
			</if>
			<if test="headImg!=null">
				head_img=#{headImg,jdbcType=BLOB},
			</if>
			<if test="createTime!=null">
				create_time=#{createTime,jdbcType=TIMESTAMP},
			</if>
			id=#{id}
			WHERE id=#{id}
		  </update>
注意where之前的`id=#{id}`也是为了没有任何更新时程序不出错：
		UPDATE user SET id=#{id} WHERE id=#{id}
也可以最大限度保证程序不出错。
后面还有Where标签与set标签也可以解决这个问题。
3. 新建测试类：
		//测试动态if更新
		@Test
		public void testUpdateUserById(){
			//获取sqlSession
			SqlSession sqlSession =getSqlSession();
			try{
				UserMapper userMapper =sqlSession.getMapper(UserMapper.class);		//获取UserMapper接口
				User user=userMapper.selectById(6L);			//从数据库中查询一个user对象
				Assert.assertEquals("测试用户3", user.getUserName());		//测试是否是想要的那个数据
				user.setUserName("大老板");				//修改用户名
				user.setUserEmail("7777@11.11");			//修改邮箱
				int result=userMapper.updateUserById(user);		//更新数据(返回值为SQL影响条数)
				Assert.assertEquals(1, result);			//是否只更新一条数据
				user=userMapper.selectById(6L);		//查询修改后的数据
				Assert.assertEquals("大老板", user.getUserName());		//验证修改后的名字
			}finally{
				sqlSession.rollback();
				sqlSession.close();
			}
		}
使用rollback回滚，不对数据库进行更新。
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00024MyBatis%E5%AD%A6%E4%B9%A04-02.jpg)

**4)Insert中使用if动态插入：**
1. 需求：
数据库插入数据时，如果某一字段的参数不为空，那么传入该值，如果某一字段参数为空则不传入，使用默认值进行插入。
2. 创建UserMapper.java接口方法：
		//if动态添加并返回主键
		int insertByIf(User user);
创建xml方法：
		<insert id="insertByIf" useGeneratedKeys="true" keyProperty="id">
			INSERT INTO user 
			VALUES(
			null
			<if test="userName!=null and userName!=''">
			,#{userName}
			</if>
			<if test="userName==null or userName==''">
			,default
			</if>
			<if test="userPassword!=null and userPassword!=''">
			,#{userPassword}
			</if>
			<if test="userPassword==null or userPassword==''">
			,default
			</if>
			<!--其他属性相同-->
			)
		</insert>
一般在列的地方添加if,values相应的地方也要添加if,这是不用写字段列表的一种写法，但是同样需要两对if标签。
3. 新建测试方法：
		//新增用户(jdbc返回主键),if动态插入
		@Test
		public void testInsertByIf(){
			//获取sqlSession
			SqlSession sqlSession =getSqlSession();
			try{
				//获取UserMapper接口
				UserMapper userMapper =sqlSession.getMapper(UserMapper.class);
				User user=new User();					//创建一个user对象：
				user.setUserName("测试用户4");
				user.setUserPassword("66666");
				user.setUserEmail("");
				user.setUserInfo("小猫");
				user.setHeadImg(null);					//本来是存图片的
				user.setCreateTime(new Date());
				int result=userMapper.insertByIf(user);
				Assert.assertEquals(1, result);			//是否插入一条数据
				Assert.assertNotNull(user.getId());			//id是否为Null
				System.out.println("id="+user.getId());
			}finally{
				//sqlSession.commit();
				sqlSession.rollback();  //回滚
				sqlSession.close();
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00024MyBatis%E5%AD%A6%E4%B9%A04-03.jpg)

---
## 2.choose的用法（when,otherwise）：
**1)if标签在insert中使用时的缺陷：**
1. 无法实现if...else...这种结构。只有if...结构
2. 想要实现这样的逻辑就需要使用choose标签了.
choose标签中有`<when>`和`<otherwise>`两个子标签：
**一个choose中至少要有一个when，0个或一个otherwise**。
3. 当然也可以通过choose来实现上面的动态插入。
4. 使用choose when otherwise时一定要严谨，避免某些特殊值导致SQL错误


**2）choose使用测试--动态查询：**
1. 需求：
查询用户信息：
id有值用id查，id没值用用户名查(完全匹配)，用户名也没值则返回查询失败。
2. 添加UserMapper接口方法：
		//使用choose实现动态查询
		User selectByIdOrName(User user);
添加UserMapperXML方法：
		<!-- choose实现动态查询 -->
		<select id="selectByIdOrName" resultType="User">
			SELECT 	id,user_name,user_password,user_email,user_info,head_img,create_time
			FROM User
			WHERE 1
			<choose>
			<when test="id!=null">
				and id=#{id}
			</when>
			<when test="userName!=null">
				and user_name=#{userName}
			</when>
			<otherwise>
				and 0
			</otherwise>
			</choose>
		</select>
注意otherwise标签中的写法，当以的条件都不满足时返回0，代表不会查出任何值。如果不加则会查出所有的值。
3. 添加测试类方法：
		//测试choose实现动态查询
		@Test
		public void testSelectByIdOrName(){
			//获取sqlSession
			SqlSession sqlSession =getSqlSession();
			try{
				//获取UserMapper接口
				UserMapper userMapper =sqlSession.getMapper(UserMapper.class);
				User user =new User();  //创建查询实体类
				User que_user0 =userMapper.selectByIdOrName(user);  //测试啥都没有时
				Assert.assertNull(que_user0);		//判断是否查出空值
				user.setId(5L);   //测试情况一：只有Id
				User que_user =userMapper.selectByIdOrName(user);  //查询
				Assert.assertEquals("测试用户2", que_user.getUserName());  //测试是不是得到对的值
				user.setId(null);         //测试情况2：只有用户名
				user.setUserName("测试用户3"); 
				User que_user2 =userMapper.selectByIdOrName(user);  //查询
				Assert.assertTrue(que_user2.getId()==6L); 		//测试是不是得到对的值
				user.setId(1001L);				//测试情况3：都有
				user.setUserName("测试用户1"); 
				User que_user3 =userMapper.selectByIdOrName(user);  //查询
				Assert.assertEquals("测试用户", que_user3.getUserInfo());
			}finally{
				sqlSession.close();
			}
		}
测试结果:
![](http://p5ki4lhmo.bkt.clouddn.com/00024MyBatis%E5%AD%A6%E4%B9%A04-04.jpg)

---
## 3.where,set,trim的用法
where和set都属于trim的一种具体用法。
**1)where的用法：**
1. 作用：**如果该标签包含的元素有返回值，就插入一个where；如果where后面的字符串是以AND或OR开头的，就将它们删除。**
2. 参考if在where中的用法：
需求与那个例子相同：（对比方法，尤其是xml的方法）
新建接口方法：
		//where标签结合if标签实现动态查询
		List<User> selectByUser2(User user);
新建xml方法：
		<select id="selectByUser2" resultType="User">
			SELECT 	id,user_name,user_password,user_email,user_info,head_img,create_time
			FROM User
			<where>
			<if test="userName!=null and userName!=''">
				 user_name like concat('%',#{userName},'%')
			</if>
			<if test="userEmail!=null and userEmail!=''">
				and user_email=#{userEmail}
			</if>
			</where>
		</select>
不用在where后写1=1了，它会自动去除。
3. 测试方法:
与testSelectByUser()相同，把方法换一下就好了。
测试结果:
![](http://p5ki4lhmo.bkt.clouddn.com/00024MyBatis%E5%AD%A6%E4%B9%A04-05.jpg)

**2)set的用法：**
1. 作用：**如果set标签包含的元素有返回值则插入一个set；如果set后面的字符串是以逗号结尾的就讲这个逗号删除。**
2. 测试set的方法：
参考前面if在update中的用法：需求相同(对比前面的xml方法)
新增接口方法：
		//if配合set动态更新
		int updateUserById2(User user);
新增xml方法：
		<update id="updateUserById2">
		  	UPDATE user 
			<set>
		  	<if test="userName!=null and userName!=''">
				user_name=#{userName},
			</if>
			<if test="userPassword!=null and userPassword!=''">
				user_password=#{userPassword},
			</if>
			<if test="userEmail!=null and userEmail!=''">
				user_email=#{userEmail},
			</if>
			<if test="userInfo!=null and userInfo!=''">
				user_info=#{userInfo},
			</if>
			<if test="headImg!=null">
				head_img=#{headImg,jdbcType=BLOB},
			</if>
			<if test="createTime!=null">
				create_time=#{createTime,jdbcType=TIMESTAMP},
			</if>
			id=#{id}
			</set>
			where id=#{id}
		</update>
这里的set中的`id=#{id}`最好还是加上，因为<set>标签中什么都没有会报异常，所以这样必然存在的赋值很有存在的必要。

3. 测试类和update中使用if的差不多。
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00024MyBatis%E5%AD%A6%E4%B9%A04-06.jpg)

**3)trim的用法：**
1. where,set标签的功能都可以用trim标签来实现。
而且这三个标签的底层都是由TrimSqlNode来实现的。
2. **trim标签的属性**：
	- prefix：当trim标签内包含内容时,会给内容增加prefix指定的前缀。
	- prefixOverrides：当trim标签内包含内容时，会把内容中匹配的前缀字符串去掉。
	- suffix：当trim标签内包含内容时,会给内容增加suffix指定的后缀。
	- suffixOverrides：当trim标签内包含内容时，会把内容中匹配的后缀字符串去掉。
3. 用trim简单实现where标签：
		<trim prefix='WHERE' prefixOverrides="AND |OR ">
		...
		</trim>
实际上where的`prefixOverrides`还有许多情况，AND和OR后面还有换行符，空格等。
用trim简单实现set标签：
		<trim prefix='SET' suffixOverrides=",">
		...
		</trim>

---
## 4.foreach标签的用法



---