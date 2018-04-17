---
title: MyBatis学习笔记（三）：注解方式的基本用法
date: 2018-03-24 20:55:32
tags: MyBatis
categories: J2EE框架

---
## 0.学习准备
1. 学习资料：
书本：《MyBatis从入门到精通》
视频：某培训班视频
资料：[MyBatis官方文档](http://www.mybatis.org/mybatis-3/zh/index.html)

2. 本章重点
	- 注解的基本用法

---
## 1.注解方式简介
MyBatis注解的方式就是直接将SQL语句写在接口上。
优点是对于需求比较简单的系统，效率较高。
缺点是每次改变需求都需要重新编译代码。
**一般情况下不会使用注解方式。**

---
## 2.@select注解
**1)使用方法：**
1. 别名方式的@select注解：
在Mybatis.simple.mapper.RoleMapper接口下创建selectById()方法：
		//@select注解，根据id查询Role角色信息
		@Select({"select id,role_name roleName,enabled,create_by createBy,create_time createTime"
				+ "from role where id=#{id}"})
		Role selectById(long id);
无需配置Xml，样就能直接进行测试了。
2. 使用mapUnderscoreToCameCase配置：
就是在核心配置文件中配置，和`<select>`标签的配置一样。
3. 使用resultMap的方式进行配置：
		//resultMap方式进行配置
		@Results({
			@Result(property="id",column="id",id=true),
			@Result(property="roleName",column="role_name"),
			@Result(property="createTime",column="create_time"),
			@Result(property="enabled",column="enabled"),
			@Result(property="createBy",column="create_by"),
		})
		@Select("select id,role_name,enabled,create_by,create_time"
				+ "from role where id=#{id}")
		Role selectById2(long id);
Mybatis3.3.0版本以前，注解定义的@Result不能共用，使用很不方便。
Mybatis3.3.1版本开始添加了一个id注解，可以通过同一个id属性引用同一个@Result的配置。
如：
		@Results(id="roleResultMap",value={aaaaaaaaaaa})
		@select(....)
其他的ResultMap只要这样引用就可以了:
		@Result("roleResultMap")
		@select(....)
4. 添加测试方法：
selectById测试：
		@Test
		public void testSelectById(){
			//获取sqlSession
			SqlSession sqlSession =getSqlSession();
			try{
				//获取RoleMapper接口
				RoleMapper roleMapper =sqlSession.getMapper(RoleMapper.class);
				//调用selectById方法
				Role role=roleMapper.selectById(1L);
				//断言测试role不为空
				Assert.assertNotNull(role);
				//角色名字是否为管理员
				Assert.assertEquals("管理员", role.getRoleName());
			}finally{
				sqlSession.close();
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00021MyBatis%E5%AD%A6%E4%B9%A03-01.jpg)
selectById2测试与测试selectById的方法相同。测试略。

---
## 2.@insert注解
1. 不需要返回主键值：
和xml中的sql完全一样：
		//添加Role对象
		@Insert({"insert into role values(#{id},#{roleName},#{enabled},#{createBy},#{createTime,jdbcType=TIMESTAMP})"})
		int addRole(Role role);
测试类：
		@Test
		public void testAddRole(){
			//获取sqlSession
			SqlSession sqlSession =getSqlSession();
			try{
				//获取RoleMapper接口
				RoleMapper roleMapper =sqlSession.getMapper(RoleMapper.class);
				//创建角色对象
				Role role=new Role();
				role.setRoleName("游客");
				role.setEnabled(1);
				role.setCreateBy(1);
				role.setCreateTime(new Date());
				//调用addRole方法
				int result =roleMapper.addRole(role);
				//断言是否添加成功
				Assert.assertEquals(1,result);
			}finally{
				sqlSession.rollback();
				sqlSession.close();
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00021MyBatis%E5%AD%A6%E4%B9%A03-02.jpg)
为了不影响数据库而选择回滚。
2. 返回自增主键的值(@option标签)：
添加以下接口方法：
		@Insert({"insert into role values(null,#{roleName},#{enabled},#{createBy},#{createTime,jdbcType=TIMESTAMP})"})
		@Options(useGeneratedKeys=true,keyProperty="id")
		int addRole2(Role role);
测试方法同上，测试结果为：
![](http://p5ki4lhmo.bkt.clouddn.com/00021MyBatis%E5%AD%A6%E4%B9%A03-03.jpg)
3. 返回非自增的主键：
添加接口方法indsert3()：
		//添加Role对象返回非自增主键
		@Insert({"insert into role values(null,#{roleName},#{enabled},#{createBy},#{createTime,jdbcType=TIMESTAMP})"})
		@SelectKey(statement="SELECT LAST_INSERT_ID()",keyProperty="id",resultType=Long.class,before=false)
		int addRole3(Role role);
测试方法与上面的相同，就不测试了。
需要注意的是最后的`before=false`这句话，效果等同于`order=after`。
不同的数据库需要配置的order是不一样的。

---
## 3.@Update注解
1. 添加接口方法：
		//根据Id更新
		@Update({"update role set role_name=#{roleName},enabled=#{enabled},create_by=#{createBy},create_time=#{createTime} where id=#{id}"})
		int updateById();
2. 添加测试类方法：
		@Test
		public void testUpdateById(){
			//获取sqlSession
			SqlSession sqlSession =getSqlSession();
			try{
				//获取RoleMapper接口
				RoleMapper roleMapper =sqlSession.getMapper(RoleMapper.class);
				//创建角色对象
				Role role=new Role();
				role.setId(5l);
				role.setRoleName("人力管理");
				role.setEnabled(1);
				role.setCreateBy(1);
				role.setCreateTime(new Date());
				//调用addRole方法	
				int result = roleMapper.updateById(role);
				//断言是否添加成功
				Assert.assertEquals(1,result);
				//查询添加后的结果
				role = roleMapper.selectById(role.getId());
				//断言测试是否添加成功
				Assert.assertEquals("人力管理",role.getRoleName());
			}finally{
				sqlSession.rollback();
				sqlSession.close();
			}
		}
测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00021MyBatis%E5%AD%A6%E4%B9%A03-04.jpg)

----
## 4.@Delete注解
1. 在接口中添加删方法：
		//根据Id删除
		@Delete({"delete from role where id = #{id}"})
		int deleteById(long id);
2. 添加测试方法：
		@Test
		public void testDeleteById(){
			//获取sqlSession
			SqlSession sqlSession =getSqlSession();
			try{
				//获取RoleMapper接口
				RoleMapper roleMapper =sqlSession.getMapper(RoleMapper.class);
				//调用删除方法	
				int result = roleMapper.deleteById(5L);
				//检测是否删除成功
				Assert.assertEquals(1,result);
			}finally{
				sqlSession.rollback();
				sqlSession.close();
			}
		}
3. 测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00021MyBatis%E5%AD%A6%E4%B9%A03-05.jpg)

---
## 5.Provider注解：
1. 除了使用简单的注解SQL外，MyBatis还提供了四种Provider注解：
@SelectProvider,@InsertProvider,@UpdateProvider,@DeleteProvider
同样可以实现增删改查的操作。
2. 通过@SelectProvider来了解它们的基本用法。
首先在src/main/java下创建一个Mabatis.simple.provider包，并在该包内创建RoleProvider()类：
		package Mybatis.simple.provider;
		import org.apache.ibatis.jdbc.SQL;
		public class RoleProvider {
			public String selectById(final long id){
				return new SQL().SELECT("id,role_name,enabled,create_by,create_tim").FROM("role").WHERE("id=#{id}").toString();
			}
		}
使用`new SQL().SELECT().FROM().WHERE()...`可以组装SQL语句。
RoleProvider()类的方法也可以这样写：
		public String selectById(final long id){
			return "SELECT id,role_name,enabled,create_by,create_tim FROM role WHERE id=#{id}";
		}
3. 在RoleMapper中添加接口方法：
		//根据Id查询
		@SelectProvider(type=RoleProvider.class,method="selectById")
		Role selectByProviderId(long id);
@Provider注解有两个必的属性：
type属性指定的包含method属性方法的类，该类必须要有一个空的构造方法，
method属性指定的是方法名，且返回值必须要是String
4. 测试代码：
		@Test
		public void testSelectByProviderId(){
			//获取sqlSession
			SqlSession sqlSession =getSqlSession();
			try{
				//获取RoleMapper接口
				RoleMapper roleMapper =sqlSession.getMapper(RoleMapper.class);
				//调用selectById方法
				Role role=roleMapper.selectByProviderId(1L);
				//断言测试role不为空
				Assert.assertNotNull(role);
				//角色名字是否为管理员
				Assert.assertEquals("管理员", role.getRoleName());
			}finally{
				sqlSession.close();
			}
		}
5. 测试结果：
![](http://p5ki4lhmo.bkt.clouddn.com/00021MyBatis%E5%AD%A6%E4%B9%A03-06.jpg)
6. 注解模式：
注解模式不是主流，高级的用法以后再学习吧。

---