---
title: MyBatis学习笔记（四）：动态SQL语句
date: 2018-03-26 15:46:47
tags: MyBatis
categories: J2EE框架

---
## 0.学习准备
1. 学习资料：
书本：《MyBatis从入门到精通》
视频：某培训班视频
资料：[MyBatis中文文档](http://www.mybatis.org/mybatis-3/zh/index.html)

---
## 1.if的用法
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
## 2.choose的用法（when,otherwise）
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
**1)使用foreach的场景：**
1. 在SQL中会有一些操作会用到集合，如`in (a,b,c)`还有some，all等。
()中的值可以直接使用${}来获取但是有[SQL注入](https://baike.baidu.com/item/SQL%E6%B3%A8%E5%85%A5%E6%94%BB%E5%87%BB/4766224?fr=aladdin)的风险。
想要避免SQL注入就需要使用#{},这时就得使用foreach标签配合了。
2. foreach标签可以对数组、Map或实现了Iterable接口（set,List等）的对象进行遍历。数组在遍历时会转换为List。
所以foreach循环的遍历分为两类：Iterable类和Map类。

**2)foreach实现in集合：**
1. foreach实现in集合是最简单和最常见的一种情况。
2. 需求：
根据用户id的集合查询所有符合id的用户。
3. 新建UserMapper接口方法：
		//foreach标签实现动态查询user列表
		List<User> selectByIdList(List<Long> idList);
新建UserMapper.xml方法：
		<select id="selectByIdList" resultType="User">
			SELECT 	id,user_name,user_password,user_email,user_info,head_img,create_time
			FROM User
			<where> id in
			<foreach collection="list" open="(" close=")" separator="," item="id" index="i">
				#{id}
			</foreach>
			</where>
		</select>
新建测试类：
		//foreach动态标签实现in集合
		@Test
		public void testSelectByIdList(){
			//获取sqlSession
			SqlSession sqlSession =getSqlSession();
			try{
				UserMapper userMapper =sqlSession.getMapper(UserMapper.class);		//获取UserMapper接口
			//	Long a[]={1L,5L,6L};					//创建参数合集
				List<Long> idlist=new ArrayList<Long>();
				idlist.add(1L);
				idlist.add(5L);
				idlist.add(6L);
				List<User> users =userMapper.selectByIdList(idlist);
				Assert.assertEquals(3, users.size());    //是否查询到了三条数据	
			}finally{
				sqlSession.close();
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00024MyBatis%E5%AD%A6%E4%B9%A04-07.jpg)

**3)foreach标签包含的属性：**
`<foreach collection="list" open="(" close=")" separator="," item="id" index="i">`
- collection：必填，值为要迭代循环的类型名（属性名）。有很多种情况。
- item：值为从迭代对象中取到的每一个值。(名字可以任取但是需要统一)
- index：填索引名字。在集合或数组情况下值为当前的索引值。跌代对象是Map类似时值为Map的key(键值)。（这个名字可以随便起，一般用i，使用的时候只会用值）
- open：整个循环的开始字符串。
- close：整个循环的结束字符串。
- separator：每次循环之间的分隔符。

**4)不同情况下collection的属性指定：**
1. 先查看DefaultSqlSession默认处理类的源码(MyBatis)：
		private Object wrapCollection(final Object object) {
		  if (object instanceof Collection) {	//如果是collection集合类
		    StrictMap<Object> map = new StrictMap<Object>();
		    map.put("collection", object);		//先添加collection为map的key
		    if (object instanceof List) {		//如果是list类型
		      map.put("list", object);		//再添加一个list为key
		    }
		    return map;
		  } else if (object != null && object.getClass().isArray()) {		//不为空且为数组类型
		    StrictMap<Object> map = new StrictMap<Object>();
		    map.put("array", object);		//添加一个key为array
		    return map;
		  }
		  return object;
		}
2. 如果参数是List类型使用：`collection="list"`
如果参数是Array类型使用：`collection="array"`；
这是默认的使用值，也可以**使用@param注解自定义**：
接口方法像这样写：
		List<User> selectByIdList(@Param("idlist")List<Long> idList);
在xml中就可以使用`collection="idlist"`，或者param1:
![](http://p5ki4lhmo.bkt.clouddn.com/00024MyBatis%E5%AD%A6%E4%B9%A04-08.jpg)
数组同样也可以这样做。
3. 有两个以上参数时:
		List<User> selectByIdList(@Param("idlist")List<Long> idList,@Param("666")long id);
需要使用@param为每个参数指定名字，colletion的值为@param为参数起的名字。(和上面一样)
如果有多个参数都需要循环遍历，那么就需要使用多组foreach标签。
4. 参数是一个Map类型时：
如果是Map参数中包含了一个需要循环的对象，如：Map<Integer,List>中有一个map(6,idlist)，那么需要这样写：`collection="6"`。
如果是想要遍历这个Map本身，取出所有的键值对，可以使用默认值：`collection="_parameter"`，不过更推荐使用@param给这个map指定名字。
5. 参数是自定义类对象中一个需要循环遍历的属性：
直接指定对象的属性名即可：`collection="对象名.属性名"`
需要遍历的是对象本身，则指定为对象名(使用@param注解指定)。
多层嵌套则使用多个小数点取出。

**4)foreach标签实现批量插入：**
1. 数据库支持批量插入的话就可以使用foreach来批量插入，
批量插入是SQL-92特性，目前支持的数据库有：Mysql,DB2,SQL Sever2008以上版本，SQLite3.711及以上版本，H2等。（Oracal不支持）
批量插入的语法：
		Insert into 表名（字段列表） values(值列表),（值列表）...
可以看出后面的值列表是循环的，所以可以使用foreach实现。
2. 添加接口方法：
		//foreach标签实现动态批量插入
		int insertUserList(List<User> userlist);
添加xml方法：
		<insert id="insertUserList">
			insert into user
			values
			<foreach collection="list" item="user" separator=",">
				(
				#{user.id},#{user.userName},#{user.userPassword},#{user.userEmail},#{user.userInfo},#{user.headImg,jdbcType=BLOB},#{user.createTime,jdbcType=TIMESTAMP}
				)
			</foreach>
		</insert>
注意，item指定了遍历出来的值名字为user,后面需要使用这个值(user)内部的属性时就需要使用user.属性来获取。
3. 新增测试类方法：
		//foreach动态标签实现批量插入
		@Test
		public void testInsertUserList(){
			//获取sqlSession
			SqlSession sqlSession =getSqlSession();
			try{
				UserMapper userMapper =sqlSession.getMapper(UserMapper.class);		//获取UserMapper接口
				List<User> users =new ArrayList<User>();		//创建需要添加的User列表
				for(int i=0;i<3;i++){
					User user=new User();
					user.setUserName("测试用户");
					user.setUserEmail("123456");
					user.setUserInfo("批量新增用户");
					user.setCreateTime(new Date());
					users.add(user);
				}
				int result = userMapper.insertUserList(users);
				Assert.assertEquals(users.size(),result);    //测试是否成功添加
			}finally{
				sqlSession.rollback;
				sqlSession.close();
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00024MyBatis%E5%AD%A6%E4%B9%A04-09.jpg)
4. 批量返回主键(仅支持MyBatis3.3.1及以上版本，数据库最好使用Mysql)：
书上只写了JDBC的方式，而且需要3.3.1版本及以上的支持。
只需要在xml的insert标签上添加属性即可：
		<insert id="insertUserList" useGeneratedKeys="true" keyProperty="id">
useGeneratedKeys与keyProperty属性，然后执行插入后参数对象的所有user的id都有了返回的key值，可以手动遍历查看。

**5)foreach标签实现动态Update更新：**
1. 参数对象是Map,遍历取出所有的键值对并以此对user进行更新。
2. 新建接口方法：
		//foreach标签实现动态更新
		int updateByMap(Map<String,Object> userMap);
新增xml文件方法：
		<update id="updateByMap">
			update user
			<set>
				<foreach collection="_parameter" item="val" index="key" separator=",">
					${key}=#{val}
				</foreach>
			</set>
			where id=#{id}
		</update>
如果这样只要在map中存入一个id值就不用在where前写id=#{id}了。
3. 新建测试方法：
		//foreach动态更新
		@Test
		public void testUpdateByMap(){
			SqlSession sqlSession =getSqlSession();
			try{
				UserMapper userMapper =sqlSession.getMapper(UserMapper.class);		//获取UserMapper接口
				Map<String,Object> map=new HashMap<String, Object>();
				map.put("id",5L);
				map.put("user_name", "hahaha");
				map.put("user_info", "666");
				int result =userMapper.updateByMap(map);
				Assert.assertEquals(1,result);  //是不是更新一条数据
				User user=userMapper.selectById((Long) map.get("id"));
				Assert.assertEquals(user.getUserName(),(String)map.get("user_name"));  //测试是否更新成功
			}finally{
				sqlSession.rollback();
				sqlSession.close();
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00024MyBatis%E5%AD%A6%E4%B9%A04-10.jpg)

---
## 5.bind的用法
1. 问题引入：
在使用模糊查询时一般都是使用的concat()来连接like后面的内容，如：
		<select id="selectByUser" resultType="User">
			SELECT 	id,user_name,user_password,user_email,user_info,head_img,create_time
			FROM User
			WHERE 1
			<if test="userName!=null and userName!=''">
				and user_name like concat('%',#{userName},'%')
			</if>
		</select>
但是concat在Mysql和Oracle数据库中的使用有很大的不同：
concat在Mysql中支持多个参数，但是在oracle中仅支持两个参数。
为了转换数据库时不会出错，就需要使用bind标签。
2. bind标签的作用：
使用OGNL表达式创建一个人变量并将其绑定到上下文中。
3. 简单例子：
将上述例子的含模糊查询的部分修改如下：
		<if test="userName!=null and userName!=''">
			<bind name="nameLike" value="'%',#{userName},'%'"/>
			and user_name like #{nameLike}
		</if>
4. 属性说明：
name：必填，变量名
value：必填，OGNL表达式，变量值
5. 使用bind不仅可以方便更换数据库，也可以防止SQL注入。
但是bind并不能完全解决更换数据库的问题。

---
## 6.多数据库支持
**1)databaseIdProvider配置介绍**
1. MyBatis可以根据不同的数据库厂商执行不同的语句，这种机制是基于映射语句中的databaseId属性的。
2. Mybatis会加载所有带和不带databaseId属性值的语句，如果同时找到了带和不带databaseId的语句，那么后者会被忽略。
3. 支持多厂商特性，需要在Mybatis核心配置文件中加入如下配置：
（注意需要加在Mapper映射配置的上方）
		<databaseIdProvider type="DB_VENDOR"></databaseIdProvider>
4. DB_VENDOR是使用DatabaseMetData#getDatabaseProductName()方法返回的值来设置的，但是这个值非常长，所以一般需要设置属性别名：
		<databaseIdProvider type="DB_VENDOR">
			<property name="SQL Sever" value="sqlsever"/>
			<property name="DB2" value="db2"/>
			<property name="Oracle" value="oracle"/>
			<property name="MySQL" value="mysql"/>
			<property name="H2" value="h2"/>
			<!-- 其他数据库 -->
		</databaseIdProvider>
这样配置后databaseId会被设置成第一个能匹配的数据库的属性键对应的值（value），没有匹配到则会被设置成null。
5. **匹配测策略：**
DatabaseMetData#getDatabaseProductName()返回的值中包含`<property name="" value=""/>`中的name属性值即可，将value赋值给databaseId。
6. 数据库产品名是由所选择的当前数据库的JDBC指定的（实现DatabaseMetData接口指定）。而是使用getDatabaseProductName()就能得到数据库产品名。

**2）多数据库的用法：**
1. 配置了databaseIdProvider后，一些映射文件的标签中的databaseId属性就有作用了：
	- select
	- insert
	- delete
	- update
	- selectKey
	- sql
2. 整条SQL语句的自动匹配（以MySQL和Oracle为例子）：
		<select id="selecttest01" databaseId="mysql" resultType="User">
				select * from user where
				user_name like concat('%',#{userName},'%')
		</select>
		<select id="selecttest01" databaseId="oracle" resultType="User">
				select * from user where
				user_name like	'%'||#{userName}||'%'
		</select>
3. Mysql和Oracle中的大部分SQL语句都是差不多的，所以可以使用if标签配合默认的上下文中的_databaseId参数实现部分匹配，可以省去很多不必要的SQL语句。
4. 部分匹配：
		<select id="selecttest02" resultType="User">
				select * from user
		<where>
		<if test="_databaseId='mysql'">
			user_name like concat('%',#{userName},'%')
		</if>
		<if test="_databaseId='orcale'">
			user_name like	'%'||#{userName}||'%'
		</if>
		</where>
		</select>

---
## 7.OGNL的简单用法
1. OGNL简介：
OGNL（Object-Graph Navigation Language的简称），对象图导航语言，它是一门表达式语言，除了用来设置和获取Java对象的属性之外，另外提供诸如集合的投影和过滤以及lambda表达式等。解决了java和页面之间数据不匹配的问题。
2. MyBatis常用的OGNL表达式：
	- e1 or e2
	- e1 and e2
	- e1 == e2 或者 e1 eq e2
	- e1 != e2 或者 e1 neq e2
	- e1 lt e2   -- 小于（大于：gt）
	- e1 lte e2    -- 小于等于(大于等于：gte)
	- e1+e2、e1*e2、e1/e2、e1-e2、e1%e2
	- !e或not e    -- 非，反
	- e.method(args)		--调用对象方法
	- e.property		-- 对象属性值(可多层嵌套)
	- e1[e2]		-- 按索引取值(List,Map和数组)
	- @class@method(args)		--调用类的静态方法
	- @class@filed		--调用类的静态字段
3. 最后俩表达式常用于简化一些校验或者进特殊的操作。
		@类名@静态方法名	/	@类名@静态属性名
4. 更多关于OGNL的操作可以参考博客：
【1】[OGNL表达式原理及使用](https://blog.csdn.net/yxflovegs2012/article/details/53227889)
【2】[OGNL表达式语言详解](https://blog.csdn.net/yu102655/article/details/52179695)
【3】[MyBatis中的OGNL教程](https://blog.csdn.net/isea533/article/details/50061705)

---
## 8.动态SQL中的可插拔脚本语言
1. MyBatis 从 3.2 开始支持可插拔脚本语言，这允许你插入一种脚本语言驱动，并基于这种语言来编写动态SQL查询语句。
可以通过实现LanguageDriver接口实现。
更多的操作以后再学吧。
2. 详见[MyBatis中文文档](http://www.mybatis.org/mybatis-3/zh/index.html)

---

