# Mybatis学习文档

Mybatis是持久层框架类似于Hibernate的orm持久层框架

### 1.JDBC访问数据库存在的问题

频繁创建和打开、关闭数据链接，太消耗资源

Sql语句存在硬编码，不利于维护

Sql参数设置硬编码，不利于维护

结果集获取与遍历复杂，存在硬编码，不利于维护，期望查询后返回一个java对象

### 2.Mybatis介绍

前身是apache下的开源项目，2010有aspache software foundation 迁移到了google code ，并且改名为Mybatis，2013年迁移到github。

### 3.Mybatis入门

工程搭建

> 导入jar包
>
> 配置SqlMapConfig.xml
>
> 配置log4j.properties
>
> 配置sql查询的映射文件
>
> 加载映射文件

完成需求

步骤

> 编写sql语句
>
> 配置user映射文件
>
> 编写测试程序

需求

> 根据用户ID查询用户信息
>
> 根据用户名模糊查询用户信息
>
> 插入用户（主键返回，UUID使用）
>
> 修改删除用户

SqlMapConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <!-- 加载属性文件 -->
    <properties resource="log4j.properties">
        <!--properties中还可以配置一些属性名和属性值  -->
        <!-- <property name="jdbc.driver" value=""/> -->
    </properties>
    <!-- 和spring整合后 environments配置将废除-->
    <environments default="development">
        <environment id="development">
        <!-- 使用jdbc事务管理，事务控制由mybatis-->
            <transactionManager type="JDBC" />
        <!-- 数据库连接池，由mybatis管理-->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/pojo?useSSL=false&amp;serverTimezone=Hongkong&amp;characterEncoding=utf-8&amp;autoReconnect=true" />
                <property name="username" value="root" />
                <property name="password" value="manager" />
            </dataSource>
        </environment>
    </environments>
    <!-- 加载 映射文件 -->
    <mappers>
        <mapper resource="mybatis/user.xml"/>
    </mappers>
</configuration>
```

user.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespqce:命名空间，用于隔离sql语句
#{}是占位符相当于jdbc的?
${}是字符串拼接指令，如果入参为普通数据类型括号内部只写value
  -->
<mapper namespace="user">
<!-- id: sql id的唯一标识
parameterType:入参的数据类型
resultType:返回结果的数据类型
  -->
<select id="getUserById" parameterType="int" resultType="com.yhr.mybatis.User">
SELECT ID,username,sex,address FROM usertable WHERE id= #{id2}
</select>
<!--returnType：如果返回结果为集合，只需设置为每一个的数据类型  -->
<select id="getUserByUserName" parameterType="string" resultType="com.yhr.mybatis.User">
SELECT ID,username,sex,address FROM usertable WHERE username like '%${value}%'
</select>
<!--插入用户  -->
<!-- useGeneratedKeys="true" 使用自增 keyProperty="id" 与之前的配套使用 这里只User的主键id -->
<insert id="insertUser" parameterType="com.yhr.mybatis.User" useGeneratedKeys="true" keyProperty="id">
<!-- selectKey:主键返回
	keyProperty：user中的主键类型
	resultType：主键数据类型
	order：指定selectKey何时执行：AFTER|BEFORE
	可以改变为useGeneratedKeys="true" keyProperty="id"
  -->
<!-- <selectKey	keyProperty="id" resultType="int" order="AFTER">
select last_insert_id()
</selectKey> -->
insert into usertable(username,sex,address) values(#{username},#{sex},#{address});
</insert>
<!--  -->
<insert id="insertUserUUID">
<selectKey keyProperty="uuid2" resultType="string" order="BEFORE">
select UUID()
</selectKey>
insert into usertable(username,sex,address,uuid2) values(#{username},#{sex},#{address},#{uuid2});
</insert>
<update id="updateUser" parameterType="com.yhr.mybatis.User" >
update usertable set username=#{username} where id=#{id};
</update>
<delete id="deleteUser" parameterType="com.yhr.mybatis.User">
delete from usertable where id=#{id}
</delete>
</mapper>
```

MybatisTest.java

```java
package com.yhr.mybatis.test;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.jupiter.api.Test;

import com.yhr.mybatis.User;

public class MybaitsTest {
	//@Test
	public void testGetUserById() throws IOException  {
		//创建SqlSessionFactoryBuilder对象
		SqlSessionFactoryBuilder ssfd=new SqlSessionFactoryBuilder();
		//创建核心配置文件输入流
		InputStream inputStream = Resources.getResourceAsStream("SqlMapConfig.xml");
			//通过输入流创建SqlSessionFactory对象
			SqlSessionFactory sqlSessionFactory = ssfd.build(inputStream);
			//创建SqlSession对象
			SqlSession sqlSession = sqlSessionFactory.openSession();
			//执行查询
			User user = sqlSession.selectOne("user.getUserById",1);
			System.out.println(user);
			sqlSession.close();
		
		
		
	}
	//@Test
	public void testGetUserByUserName() throws IOException {
		//创建SqlSessionFactoryBuilder对象
		SqlSessionFactoryBuilder ssfd=new SqlSessionFactoryBuilder();
		//创建核心配置文件输入流
		InputStream inputStream = Resources.getResourceAsStream("SqlMapConfig.xml");
		//通过输入流创建SqlSessionFactory对象
		SqlSessionFactory sqlSessionFactory = ssfd.build(inputStream);
		//创建SqlSession对象
		SqlSession sqlSession = sqlSessionFactory.openSession();
		//执行查询
		List<User> list = sqlSession.selectList("user.getUserByUserName","张" );
		for (User user : list) {
			System.out.println(user);
		}
		sqlSession.close();
		
	}
	//@Test
	public void testInsertUser() throws IOException {
		//创建SqlSessionFactoryBuilder对象
		SqlSessionFactoryBuilder ssfd=new SqlSessionFactoryBuilder();
		//创建核心配置文件输入流
		InputStream inputStream = Resources.getResourceAsStream("SqlMapConfig.xml");
		//通过输入流创建SqlSessionFactory对象
		SqlSessionFactory sqlSessionFactory = ssfd.build(inputStream);
		//创建SqlSession对象
		SqlSession sqlSession = sqlSessionFactory.openSession();
		//执行查询
		User user =new User();
		user.setUsername("ll");
		user.setSex("男");
		user.setAddress("北京");
		sqlSession.insert("user.insertUser", user);
		System.out.println(user);
		//提交事务
		sqlSession.commit();
		//释放资源
		sqlSession.close();
		
	}
	//@Test
	public void testInsertUserUUID() throws IOException {
		//创建SqlSessionFactoryBuilder对象
		SqlSessionFactoryBuilder ssfd=new SqlSessionFactoryBuilder();
		//创建核心配置文件输入流
		InputStream inputStream = Resources.getResourceAsStream("SqlMapConfig.xml");
		//通过输入流创建SqlSessionFactory对象
		SqlSessionFactory sqlSessionFactory = ssfd.build(inputStream);
		//创建SqlSession对象
		SqlSession sqlSession = sqlSessionFactory.openSession();
		//执行查询
		User user =new User();
		user.setUsername("ll");
		user.setSex("男");
		user.setAddress("北京");
		sqlSession.insert("user.insertUserUUID", user);
		System.out.println(user);
		//提交事务
		sqlSession.commit();
		//释放资源
		sqlSession.close();
		
	}
	//@Test
	public void testUpdateUser() throws IOException {
		//创建SqlSessionFactoryBuilder对象
		SqlSessionFactoryBuilder ssfd=new SqlSessionFactoryBuilder();
		//创建核心配置文件输入流
		InputStream inputStream = Resources.getResourceAsStream("SqlMapConfig.xml");
		//通过输入流创建SqlSessionFactory对象
		SqlSessionFactory sqlSessionFactory = ssfd.build(inputStream);
		//创建SqlSession对象
		SqlSession sqlSession = sqlSessionFactory.openSession();
		//执行查询
		User user =new User();
		user.setId(0);
		user.setUsername("adada");
		sqlSession.update("user.updateUser", user);
		//提交事务
		sqlSession.commit();
		//释放资源
		sqlSession.close();
		
	}
	@Test
	public void testDeleteUser() throws IOException {
		//创建SqlSessionFactoryBuilder对象
		SqlSessionFactoryBuilder ssfd=new SqlSessionFactoryBuilder();
		//创建核心配置文件输入流
		InputStream inputStream = Resources.getResourceAsStream("SqlMapConfig.xml");
		//通过输入流创建SqlSessionFactory对象
		SqlSessionFactory sqlSessionFactory = ssfd.build(inputStream);
		//创建SqlSession对象
		SqlSession sqlSession = sqlSessionFactory.openSession();
		//执行查询
		User user =new User();
		user.setId(7);
		sqlSession.delete("user.deleteUser", user);
		System.out.println(user);
		//提交事务
		sqlSession.commit();
		//释放资源
		sqlSession.close();
		
	}
}

```

Mybatis架构图

![img](https://img2018.cnblogs.com/blog/1559799/201812/1559799-20181220102045987-1587141593.png)

### 4.Mybatis Dao 开发方式

Dao需求

> 根据用户ID查询用户信息
>
> 根据用户名查找用户列表
>
> 添加用户

#### 4.1原始Dao开发方法

SqlSession的使用范围

> SqlSession中封装了对数据库的操作如增删改查
>
> SqlSession由SqlSessionFactory进行创建
>
> SqlSessionFactory由SqlSessionFactoryBuilder进行创建

SqlSessionFactoryBuilder

> SqlSessionFactoryBuilder用于创建SqlSessionFactory，一旦创建完成就不需要SqlSessionFactoryBuilder了，因为SqlSession是由SqlSessionFactory创建的。所以可以将SqlSessionFactoryBuilder当作一个工具类使用，最佳的使用范围是方法范围即方法体内局部变量

SqlSessionFactory

> SqlSessionFactory是一个接口，接口中定义了openSession的不同重载方法，SqlSessionFactory的最佳适用范围是整个应用运行期间，一旦创建后可以重复使用，通常以单例模式管理SqlSessionFactory

SqlSession

> 在 MyBatis 中，你可以使用 `SqlSessionFactory` 来创建 `SqlSession`。一旦你获得一个 session 之后，你可以使用它来执行映射了的语句，提交或回滚连接，最后，当不再需要它的时候，你可以关闭 session。使用 MyBatis-Spring 之后，你不再需要直接使用 `SqlSessionFactory` 了，因为你的 bean 可以被注入一个线程安全的 `SqlSession`，它能基于 Spring 的事务配置来自动提交、回滚、关闭 session

1.使用原始user映射文件，不需修改

2.新建个UserDao接口

~~~java
public interface UserDao{
    User getUserById(Integer id);
    List<User> getUserByUserName(String Username);
    void insertUser(User user);
    .....
}
~~~

3.新建个UserDaoImpl接口实现类

~~~java
public class UserDaoImpl implements UserDao{
    @Override
    public User getUserById(Integer id){
        SqlSession sqlsession =SqlsessionFactoryUtils.getSqlSessionFactory().openSession();
        User user=sqlSession.selectOne("user.getUserById",id);
        sqlsession.close();
        return user;
    }
}
~~~

4.使用Dao测试

~~~java
public class UserDaoTest{
    @Test
    public void testGetUserById(){
        UserDao userdao=new UserDaoImpl();
        User user =userdao.getUserById(30);
        System.out.pringln(user);
    }
}
~~~

#### 4.2接口动态代理开发方法

1.动态代理开发规则

> 动态代理开发规则：
>   	1.namespace必须是接口的全路径名
>   	2.接口的方法名必须与Sql id一致
>   	3.接口的入参必须与parameterType类型一致
>   	4.接口的返回值必须与resultType类型一致

2.动态代理开发步骤

UserMapper.xml

与上面user.xml一样

UserMapper

~~~java
public interface UserMapper{
    User getUserById(int id);
    .........
}
~~~

UserMapperTest

~~~java
public class UserMapperTest{
    @Test
    public void testGetUserById(){
         SqlSession sqlsession =SqlsessionFactoryUtils.getSqlSessionFactory().openSession();
        UserMapper usermapper=sqlsession.getMapper(UserMapper,class);
        System.out.pringln(user);
        sqlsession.close();
    }
}
~~~

#### 4.3注解开发

注解开发不易于维护一般适用于简单的项目

注解开发不需要映射文件.xml只需要sqlmapconfig.xml

常用注解

@Select				相当于映射文件中的select标签

@Insert				相当于映射文件中的insert标签

@SelectKey		相当于映射文件中的selectKey标签

@Update			相当于映射文件中的update标签

@Delete				相当于映射文件中的delete标签

@Result				相当于映射文件中的result标签

@One					相当于映射文件中的association标签，用于封装关联JavaBean对象

@Many				相当于映射文件中的collection标签，用于封装关联JavaBean对象集合

使用方法：

创建接口，在方法上添加注解

~~~java
public interface UserDao{
    @Select("select * from userwhere id=#{id}")
    User queryId(Integer id);
    @Delete("delete from user where id=#{id}")
    void delete(Integer id)
}
~~~

在测试类中测试

~~~java
@Test
public void query(){
    User user=userDao.query(53);
     System.out.pringln(user);
}
@Test
public void delete(){
    userDao.delete(53);
    session.commit();
}
~~~

PS：执行dml语句（CURD）一定到提交事务session.commit();

### 5.SqlMapConfig.xml

#### 5.1配置内容

SqlMapConfig.xml中配置的内容和顺序如下：

> properties（属性）
>
> settings（全局配置参数）
>
> typeAliases（类型别名）
>
> typeHandlers（类型处理器）
>
> objectFactory（对象工场）
>
> plugins（插件）
>
> environments（环境集合属性对象）
>
> ​	environment（环境子属性对象）
>
> ​		transactionManager（事务管理）
>
> ​		dataSource（数据源）
>
> mappers（映射器）

#### 5.2properties（属性）

```xml
<!-- 先加载内部标签，在加载外部文件，若外部文件与内部名称相同时，会将外部的值替换掉内部的值 -->
<properties resource="jdbc.properties">
	<property name="jdbc.username" value="root1"/>
    <property name="jdbc.password" value="root1"/>
</properties>
```

#### 5.3typeAliases

Mybatis支持的别名

> 别名			映射的类型
>
> _byte			byte
>
> _long			long
>
> _short		   short
>
> _int				int
>
> _integer		int
>
> _double		double
>
> _float			float
>
> _boolean	  boolean
>
> string			String
>
> byte				Byte
>
> long				Long
>
> short			Short
>
> int				  Integer
>
> integer		  Integer
>
> double			Double
>
> float				Float
>
> boolean		Boolean
>
> date				Date
>
> decimal			BigDecimal
>
> bigdecmial		BigDecimal
>
> map				Map

自定义别名

```xml
<typeAliases>
	<!--单个别名扫描，别名的使用不区分大小写-->
	<typeAlias type="com.yhr.mybatis.pojo.User" alias="user"/>
	<!--别名包扫描器：别名是类的全称，不区分大小写-->
    <package name="com.yhr.mybatis.pojo"/>
</typeAliases>
```

#### 5.4mappers

```xml
<mapper>
    <!--通过resource方法一次加载一个映射文件 -->
    <!-- <mapper resource="mapper/UserMapper.xml"/> -->
    <mapper resource="mybatis/usermapper.xml"/>
    <!--映射文件，class扫描器
		遵循一些规范：需要将mapper接口类名和mapper.xml映射文件名称保持一致，且在一个目录中  上边规范的前提是：使用的是mapper代理方法
-->
    <mapper class="com.yhr.mybatis.mapper.UserMapper"/>
    <!-- 批量加载mapper
        指定mapper接口的包名，mybatis自动扫描包下边所有mapper接口进行加载
        遵循一些规范：需要将mapper接口类名和mapper.xml映射文件名称保持一致，且在一个目录中  上边规范的前提是：使用的是mapper代理方法
         -->
        <package name="cn.itcast.mybatis.mapper"/>
    
</mapper>
```

## 二

### 1.输入映射和输出映射

#### 1.1parameterType(输入类型)

传递简单类型

> #{}占位符			${}进行sql拼接
>
> eg：id= #{id2}		username like '%${value}%'

传递pojo对象

> Mybatis使用ognl表达式解析对象字段的值#{}或者${}括号中的值为pojo属性名称
>
> eg：<...... parameterType="com.yhr.mybatis.User">
>
> insert into usertable(username,sex,address) values(#{username},#{sex},#{address});

传递pojo包装对象

> 开发中通过可以使用pojo传递查询条件
>
> 查询条件可以是综合的查询条件，不仅包括用户查询条件还包括其他的查询条件（比如查询用户信息的时候，将用户购买商品信息也作为查询条件），这时可以使用包装对象传递输入参数。
>
> 包装对象：Pojo类中的一个属性是另外一个pojo
>
> eg：<...... parameterType="com.yhr.mybatis.pojo.QueryVo">
>
> SELECT ID,username,sex,address FROM usertable WHERE username like '%${user.username}%'
>
> QueryVo.java:
>
> public class QueryVo {
> 	
> 	private User user;
> 	
> 	public User getUser() {
> 		return user;
> 	}
> 	
> 	public void setUser(User user) {
> 		this.user = user;
> 	}
>
> }

#### 1.2resultType(输出类型)

输出简单类型

> 需求：查询用户表数据条数
>
> UserMapper.xml
>
> ```xml
> <select id="queryUserCount" resultType="int">
>     Select count(*)from `user`
> </select>
> ```

输出pojo对象

输出pojo列表

#### 1.3resultMap

> resulrType可以指定将查询结果映射为pojo，但需要pojo的属性名和sql查询的列名一致方可映射成功。
>
> 如果sql查询字段名和pojo的属性名不一致，可以通过resultMap将字段名和属性名做一个对应关系，resultMap实质上还需要将查询结果映射到pojo对象中。
>
> resultMap可以实现将查询结果映射为复杂类型的pojo，比如在查询结果映射对象中包括pojo和list实现一对一和一对多查询。
>
> 需求：查询订单表order的所有数据
>
> sql :Select id，user_id,number,createtime,note from `order`
>
> ```xml
> <resultMap type="com.yhr.mybatis.pojo.Order" id="order_list_map">
> 		<!--用于映射主键  -->
> 		<id property="id" column="id"/>
> 		<!-- 普通字段用result映射 -->
> 		<result property="userId" column="user_id"/>
> 		<result property="number" column="number"/>
> 		<result property="createtime" column="createtime"/>
> 		<result property="note" column="note"/>
> 	</resultMap>
> 	<!--使用resultMap  -->
> 	<select id="getOrderListMap" resultMap="order_list_map">
> 		select id,user_id,number,createtime,note from ordertable;
> 	</select>
> ```
>
> 

### 2.动态sql

通过mybatis的各种标签方法实现动态拼接sql

需求：根据姓名和性别查询用户

select id ,username,sex,address  from usertable where sex='男'  and username like '%张%'

#### 2.1 if标签

```xml
<select id="getUserByPojo" parameterType="User" resultType="User">
		select id ,username,sex,address  from usertable 
		where 1=1
		<if test="username!=null and username!=''">and username like '%${username}%'</if>
		<if test="sex!=null and sex!=''">and sex=#{sex}</if> 
	</select>
```

#### 2.2 Where标签

```xml
<select id="getUserByPojo" parameterType="User" resultType="User">
		select id ,username,sex,address  from usertable 
		<!-- where标签自动补上where关键字，同时处理多余的and，用了where标签就不能手动加上where关键字-->
		<where>
		<if test="username!=null and username!=''">and username like '%${username}%'</if>
		<if test="sex!=null and sex!=''">and sex=#{sex}</if> 
		</where>
	</select>
```

#### 2.3 sql片段

```xml
<sql id="user_sql">
		id ,username,sex,address
</sql>
<select id="getUserByPojo" parameterType="User" resultType="User">
		select
		<!-- sql片段使用：refid引用定义好的sql片段 -->
		 <include refid="user_sql"></include>
		from usertable 
		<where>
		<if test="username!=null and username!=''">and username like '%${username}%'</if>
		<if test="sex!=null and sex!=''">and sex=#{sex}</if> 
		</where>
</select>
```

#### 2.4 foreach标签

```xml
<select id="getUserByIds" parameterType="QueryVo" resultType="User">
	select <include refid="user_sql"></include>
	from usertable
	<where>
		<!--foreach 集合标签
			collection:要遍历的集合
			open:循环开始之前输出的内容
			close:循环借宿之后输出的内容
			separator:分隔符
			item:设置循环变量
		  -->
		  <!-- id in(0,1,8,9) -->
		 <foreach collection="ids" open="id in(" item="uid" separator="," close=")">
		 	#{uid}
		 </foreach>
		
	</where> 
	</select>
```

### 3.关联查询

#### 3.1商品订单数据模型

> 订单表									用户表
>
> 一对一：一个订单只有一个用户创建
>
> 一对多：

#### 3.2一对一查询

resultType（必须有数据库关系一样的pojo类）

> 1.建立OrderUser pojo类继承Order类
>
> 2.SQL语句：select o.id ,o.user_id,o.number,o.createtime,o.note,u.username,u.address 
>
> ​					from ordertable o left join usertable u on o.user_id=u.id
>
> 3.编写OrderMapper.xml
>
> 4.编写测试类

resultMap

OrderMapper.xml

```xml
<resultMap type="Order" id="order_user_map">
		<id property="id" column="id"/>
		<!-- 普通字段用result映射 -->
		<result property="userId" column="user_id"/>
		<result property="number" column="number"/>
		<result property="createtime" column="createtime"/>
		<result property="note" column="note"/>
		<!--用于配置一对一关系
		 property：Order里的user属性 
		 javaType:user的数据类型，支持别名
		  -->
		<association property="user" javaType="User">
		<id property="id" column="user_id"/>
		<result property="username" column="username"/>
		<result property="address" column="address"/>
		</association>
	</resultMap>
	<select id="getOrderUserMap" resultMap="order_user_map">
	select o.id ,o.user_id,o.number,o.createtime,o.note,u.username,u.address 
					from ordertable o left join usertable u on o.user_id=u.id
	</select>
```

#### 3.3一对多查询

sql语句

> select u.id ,u.username,u.sex,u.address,o.id,o.number,o.note from usertable u left join ordertable o on u.id=o.user_id

```xml
<resultMap type="User" id="user_order_map">
		<id property="id" column="id"/>
		<result property="username" column="username"/>
		<result property="address" column="address"/>
		<result property="sex" column="sex"/>
		<!-- 一对多关联
			property：User中的orders属性
			ofType：orders的数据类型，支持别名
		 -->
		<collection property="orders" ofType="Order">
			<id property="id" column="oid"/>
			<result property="number" column="number"/>
			<result property="note" column="note"/>
		</collection>
	</resultMap>
	<select id="getUserOrderMap" resultMap="user_order_map">
	select u.id ,u.username,u.sex,u.address,o.id oid,o.number,o.note 
	from usertable u left join ordertable o on u.id=o.user_id
	</select>
```

### 4.Mybatis整合Spring

#### 4.1整合思路

> 1.SqlSessionFactory对象应放到spring容器中作为单例存在
>
> 2.传统Dao的开发方式，应该从spring容器中获得sqlsession对象
>
> 3.Mapper代理形式中，应该从spring容器中直接获得mapper的代理对象
>
> 4.数据库的连接以及数据库连接池事务管理都交给spring来完成

#### 4.2整合步骤

1.创建一个java工程

2.导入jar包mybatis-spring-x.x.x.jar以及其他相关jar包

3.mybatis的配置文件sqlMapConfig.xml

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
</configuration>
~~~

4.编写Spring的配置文件

​	1）数据库连接及连接池

​	2）sqlsessionFactory对象配置搭配spring容器中

​	3）编写Spring的配置文件

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
	http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
	http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd">

	<!-- 配置 读取properties文件 jdbc.properties -->
	<context:property-placeholder location="classpath:jdbc.properties"/>

	<!-- 配置 数据源 -->
	<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
		<property name="driverClassName" value="${jdbc.driver}"/>
		<property name="url" value="${jdbc.url}"/>
		<property name="username" value="${jdbc.username}"/>
		<property name="password" value="${jdbc.password}"/>
	</bean>

	<!-- 配置SqlSessionFactory -->
	<bean class="org.mybatis.spring.SqlSessionFactoryBean">
		<!-- 设置MyBatis核心配置文件 -->
		<property name="configLocation" value="classpath:mybatis/SqlMapConfig.xml"/>
		<!-- 设置数据源 -->
		<property name="dataSource" ref="dataSource"/>
		<!-- 别名包扫描 -->
		<property name="typeAliasesPackage" value="com.yhr.crm.pojo"/>
	</bean>

	<!-- 配置Mapper扫描 -->
	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<!-- 设置Mapper扫描包 -->
		<property name="basePackage" value="com.yhr.crm.mapper"/>
	</bean>
</beans>
~~~

5.复制jdbc.properties配置文件到工程

6.复制log4j.properties配置文件到工程

#### 4.3 开发步骤

编写pojo类

编写dao层	编写mapper接口

~~~java
public interface BaseDictMapper {
	List<BaseDict> getBaseDictByCode(String code);
}

~~~

编写dao层	编写mapper映射文件

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.yhr.crm.mapper.BaseDictMapper">
	<select id="getBaseDictByCode" parameterType="string" resultType="basedict">
		SELECT 	dict_id, 
				dict_type_code, 
				dict_type_name, 
				dict_item_name, 
				dict_item_code, 
				dict_sort, 
				dict_enable, 
				dict_memo 
		FROM 
		crm.base_dict 
		WHERE dict_type_code = #{code }
	</select>
</mapper>
~~~

其他层如service或controller请自己补充

### 5.Mybatis逆向工程

下载逆向工程https://github.com/mybatis/generator/releases/tag/mybatis-generator-1.3.2

向Eclipse中导入此项目

#### 使用步骤-代码方式

导入jar包:

> log4j-xxx.jar
>
> mybatis-xxx.jar
>
> mybatis-generator-core-xxx.jar
>
> mysql-connector-java-xxx.jar

创建generatorConfig.xml配置文件

使用执行java类

把生成的代码copy进项目中

1. 配置文件：generatorConfig.xml

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
	<context id="testTables" targetRuntime="MyBatis3">
		<commentGenerator>
			<!-- 是否去除自动生成的注释 true：是 ： false:否 -->
			<property name="suppressAllComments" value="true" />
		</commentGenerator>
		<!--数据库连接的信息：驱动类、连接地址、用户名、密码 -->
		<jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
			connectionURL="jdbc:mysql://127.0.0.1:3306/taotaostore?useUnicode=true&amp;characterEncoding=UTF8&amp;useSSL=false&amp;serverTimezone=Asia/Shanghai" 
			userId="root"
			password="manager">
		</jdbcConnection>
		<!-- 默认false，把JDBC DECIMAL 和 NUMERIC 类型解析为 Integer，为 true时把JDBC DECIMAL 和 
			NUMERIC 类型解析为java.math.BigDecimal -->
		<javaTypeResolver>
			<property name="forceBigDecimals" value="false" />
		</javaTypeResolver>

		<!-- targetProject:生成PO类的位置 -->
		<javaModelGenerator targetPackage="com.taotao.pojo"
			targetProject=".\src">
			<!-- enableSubPackages:是否让schema作为包的后缀 -->
			<property name="enableSubPackages" value="false" />
			<!-- 从数据库返回的值被清理前后的空格 -->
			<property name="trimStrings" value="true" />
		</javaModelGenerator>
        <!-- targetProject:mapper映射文件生成的位置 -->
		<sqlMapGenerator targetPackage="com.taotao.mapper" 
			targetProject=".\src">
			<!-- enableSubPackages:是否让schema作为包的后缀 -->
			<property name="enableSubPackages" value="false" />
		</sqlMapGenerator>
		<!-- targetPackage：mapper接口生成的位置 -->
		<javaClientGenerator type="XMLMAPPER"
			targetPackage="com.taotao.mapper" 
			targetProject=".\src">
			<!-- enableSubPackages:是否让schema作为包的后缀 -->
			<property name="enableSubPackages" value="false" />
		</javaClientGenerator>
		<!-- 指定数据库表 -->
		<table schema="" tableName="tb_content"></table>
		<table schema="" tableName="tb_content_category"></table>
		<table schema="" tableName="tb_item"></table>
		<table schema="" tableName="tb_item_cat"></table>
		<table schema="" tableName="tb_item_desc"></table>
		<table schema="" tableName="tb_item_param"></table>
		<table schema="" tableName="tb_item_param_item"></table>
		<table schema="" tableName="tb_order"></table>
		<table schema="" tableName="tb_order_item"></table>
		<table schema="" tableName="tb_order_shipping"></table>
		<table schema="" tableName="tb_user"></table>

	</context>
</generatorConfiguration>

~~~

2. 运行java代码

~~~java
import java.io.File;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import org.mybatis.generator.api.MyBatisGenerator;
import org.mybatis.generator.config.Configuration;
import org.mybatis.generator.config.xml.ConfigurationParser;
import org.mybatis.generator.exception.XMLParserException;
import org.mybatis.generator.internal.DefaultShellCallback;
public class GeneratorSqlmap {

	public void generator() throws Exception{

		List<String> warnings = new ArrayList<String>();
		boolean overwrite = true;
		//指定 逆向工程配置文件
		File configFile = new File("generatorConfig.xml"); 
		ConfigurationParser cp = new ConfigurationParser(warnings);
		Configuration config = cp.parseConfiguration(configFile);
		DefaultShellCallback callback = new DefaultShellCallback(overwrite);
		MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config,
				callback, warnings);
		myBatisGenerator.generate(null);

	} 
	public static void main(String[] args) throws Exception {
		try {
			GeneratorSqlmap generatorSqlmap = new GeneratorSqlmap();
			generatorSqlmap.generator();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
~~~

这样就生成了项目所需的pojo和mapper。copy进自己的项目即可

**PS：Mapper.xml文件已经存在时，如果进行重新生成则mapper.xml文件时，内容不被覆盖而是进行内容追加，结果导致mybatis解析失败。**

**解决方法：删除原来已经生成的mapper xml文件再进行生成。**

**Mybatis自动生成的po及mapper.java文件不是内容而是直接覆盖没有此问题。**

#### 使用步骤-maven

pom.xml

~~~xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.cjw</groupId>
	<artifactId>myBatisGenerator</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>war</packaging>
	<properties>
		<mybatis.version>3.2.8</mybatis.version>
		<mysql.version>5.1.36</mysql.version>
		<mysql-connector-java.version>5.1.28</mysql-connector-java.version>
		<log4j.version>1.2.17</log4j.version>
		<!-- Encoding -->
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	</properties>
 
	<dependencies>
		<!-- 数据库驱动包 -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>${mysql-connector-java.version}</version>
		</dependency>
		<!-- mybatis核心包 -->  
        <dependency>  
            <groupId>org.mybatis</groupId>  
            <artifactId>mybatis</artifactId>  
            <version>${mybatis.version}</version>  
        </dependency>
		<dependency>
			<groupId>org.mybatis.generator</groupId>
			<artifactId>mybatis-generator-core</artifactId>
			<version>1.3.2</version>
		</dependency>
		 <!-- 日志文件管理包 -->  
        <dependency>  
            <groupId>log4j</groupId>  
            <artifactId>log4j</artifactId>  
            <version>${log4j.version}</version>  
        </dependency>
	</dependencies>
	<build>
		<pluginManagement>
			<plugins>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-compiler-plugin</artifactId>
					<configuration>
						<source>1.7</source>
						<target>1.7</target>
					</configuration>
					<version>3.2</version>
				</plugin>
				<plugin>
					<groupId>org.mybatis.generator</groupId>
					<artifactId>mybatis-generator-maven-plugin</artifactId>
					<version>1.3.2</version>
					<configuration>
						<!--配置文件的路径 -->
						<configurationFile>src/main/resources/generatorConfig.xml</configurationFile>
						<overwrite>true</overwrite>
					</configuration>
				</plugin>
			</plugins>
		</pluginManagement>
	</build>
</project>
~~~

generatorConfig.xml配置文件

运行java代码