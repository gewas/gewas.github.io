---
layout: post
title: 《Mybatis 入门篇》 05 懒加载与缓存以及注解开发
---

上一篇，讲了一些多表操作相关的内容。本篇中，会着重介绍懒加载与 mybatis 的二级缓存，最后再提一下使用注解开发的注意事项。

---

### mybatis 中的加载时机（查询的时机）

在多表查询时，如果每次都把关联表的信息全部查出来，会影响性能。  
比如在用户与账户的例子中：

- 一个用户有上百的账户，我只是查用户信息，这时我们根本不需要把账户信息一道查出来。但是如果我们后面用到了，也能再去加载出来。
- 查询上述用户的某一个账户时，通常我们又是需要查询用户信息的。

这里就有了两种（通常情况）经典实践场景：

- **一对多、多对多：懒加载**
- **一对一、多对一：急加载**

**懒加载**，在真正使用数据时才发起查询，不使用时不查询，按需加载。  
**急加载**，不论是否需要使用，都去查询。

#### mybatis 实现懒加载

讲解技术点时，我们不考虑什么时最佳实践，一对一，一对多的懒加载我们都看看：

- 首先，mybatis 默认是关闭了懒加载的，查看官方文档，我们在主配置文件中配置：

  ```xml
  <!--配置参数-->
  <setting>
      <!--开启Mybatis支持延迟加载-->
      <setting name="lazyLoadingEnabled" value="true"/>
      <setting name="aggressiveLazyLoading" value="false"></setting>
  </setting>
  ```

- **一对一**

  - 在账户的 mapper 文件中修改

    ```xml
    <!-- 定义封装account和user的resultMap -->
    <resultMap id="accountUserMap" type="account">
      <id property="id" column="id"></id>
      <result property="uid" column="uid"></result>
      <result property="money" column="money"></result>
      <!-- 一对一的关系映射：配置封装user的内容
      select属性指定的内容：查询用户的唯一标识：
      column属性指定的内容：用户根据id查询时，所需要的参数的值
      -->
      <association property="user" column="uid" javaType="user" select="com.itheima.dao.IUserDao.findById"></association>
    </resultMap>

    <!-- 查询所有 -->
    <select id="findAll" resultMap="accountUserMap">
      select * from account
    </select>
    ```

    可以看出我们修改了 sql 语句，同时在 resultMap 中将 user 配置了 select 属性，配合 uid 字段就可以调用 IUserDao.findById 查询出来 User 信息。所以我们要去给用户添加 findById 方法。

  - 在用户 mapper 文件中：

    ```xml
    <!-- 根据id查询用户 -->
    <select id="findById" parameterType="INT" resultType="user">
      select * from user where id = #{uid}
    </select>
    ```

这样，我们就实现了一对一场景的延迟加载。

- **一对多**

  - 在用户 mapper 中

    ```xml
    <!-- 定义User的resultMap-->
    <resultMap id="userAccountMap" type="user">
        <id property="id" column="id"></id>
        <result property="username" column="username"></result>
        <result property="address" column="address"></result>
        <result property="sex" column="sex"></result>
        <result property="birthday" column="birthday"></result>
        <!-- 配置user对象中accounts集合的映射 -->
        <collection property="accounts" ofType="account" select="com.itheima.dao.IAccountDao.findAccountByUid" column="id"></collection>
    </resultMap>

    <!-- 查询所有 -->
    <select id="findAll" resultMap="userAccountMap">
        select * from user
    </select>
    ```

  - 在账户 mapper 中

    ```xml
    <!-- 根据用户id查询账户列表 -->
    <select id="findAccountByUid" resultType="account">
        select * from account where uid = #{uid}
    </select>
    ```

这样，我们就实现了一对多场景的延迟加载。

### mybatis 的一级/二级缓存

使用缓存，是为了在不必要的情况，利用空间换时间，大大提升性能。  
缓存不是万金油，需要分场景：

- 适用场景：
  - 经常查询且数据不经常改变。如：省市区行政区域通用数据。
  - 数据正确与否对结果影响不大。
- 不适用场景：
  - 改变频繁的数据。
  - 对数据正确性要求很高。如：商品库存、银行汇率。

#### mybatis 的一级缓存

它是指 SqlSession 对象的缓存。  
它会缓存查询结果的**数据对象**，当我们再次查询相同数据时，mybatis 就会先去看看一级缓存是否有这个数据的对象，有就直接返回。  
当调用 SqlSession 对象的修改、添加、删除、commit、close，就会清空一级缓存。

#### mybatis 的二级缓存

它是指 SqlSessionFactory 对象的缓存。由同一个 SqlSessionFactory 对象创建的 SqlSession 对象共享 SqlSessionFactory 对象的二级缓存。  
它会缓存查询结果的**纯数据**，当 SqlSession 再次查询相同数据时，mybatis 就会先去看看二级缓存是否有这个数据，有就直接构建成对象返回给 SqlSession。

- 启用步骤

  - 主配置中添加配置：

    ```xml
    <settings>
        <!-- 这个其实默认配置是开启的，也就是说，可以不用写，但是写了可以给程序员看到你的意图 -->
        <setting name="cacheEnabled" value="true"/>
    </settings>
    ```

  - mapper 中添加标签：

    ```xml
    <!--开启user支持二级缓存-->
    <cache/>
    ```

  - mapper 的具体方法添加`useCache`属性：

    ```xml
    <!-- 根据id查询用户，注意增加了useCache属性 -->
    <select id="findById" parameterType="INT" resultType="user" useCache="true">
        select * from user where id = #{uid}
    </select>
    ```

### mybatis 的基于注解开发

相较于 xml 配置式的方式，基于注解开发起来更快更爽，但是耦合也更高。

由于注解的方式比较直观，这里就只说几点需要注意的地方。

- 基于注解开发的话，就不能让 mybatis 找到相同 mapper 的 xml 文件，即便你没打算使用 xml 方式，mybatis 在扫描时还是会出现冲突，导致出错。

- 使用@Insert @Update @Select @Delete 4 个注解来实现 CRUD，下面给几个示例找找感觉：

  - `@Select("select * from user")`
  - `@Insert("insert into user(username,address,sex,birthday)values(#{username},#{address},#{sex},#{birthday})")`
  - `@Update("update user set username=#{username},sex=#{sex},birthday=#{birthday},address=#{address} where id=#{id}")`
  - `@Delete("delete from user where id=#{id} ")`
  - `@Select("select * from user where id=#{id} ")`
  - `@Select("select * from user where username like #{username} ")`
  - `@Select("select * from user where username like '%${value}%' ")`
  - `@Select("select count(*) from user ")`

- 使用`@Results`注解实现 resultMap，解决属性名与字段名不匹配的问题

  ```java
  @Results(id="userMap",value={
            @Result(id=true,column = "id",property = "userId"),
            @Result(column = "username",property = "userName"),
            @Result(column = "address",property = "userAddress"),
            @Result(column = "sex",property = "userSex"),
            @Result(column = "birthday",property = "userBirthday"),
            @Result(property = "accounts",column = "id",
                    many = @Many(select = "com.test.dao.IAccountDao.findAccountByUid",
                                fetchType = FetchType.LAZY))
    })

  @Results(id="accountMap",value = {
            @Result(id=true,column = "id",property = "id"),
            @Result(column = "uid",property = "uid"),
            @Result(column = "money",property = "money"),
            @Result(property = "user",column = "uid",one=@One(select="com.test.dao.IUserDao.findById",fetchType= FetchType.EAGER))
    })
  ```

  而且，在向上面这样定义过 Results 了以后，别的方法也可以方便的通过 id 引用：`@ResultMap("userMap")`

  同时，代码中还展示了一对多、多对一，以及懒加载、急加载的配置。

- 开启二级缓存

  - 首先，主配置文件中：

  ```xml
  <!--配置开启二级缓存-->
    <settings>
        <setting name="cacheEnabled" value="true"/>
    </settings>
  ```

  - 然后，在 Dao 接口类上加注解：

  ```java
  @CacheNamespace(blocking = true)
  public interface IUserDao {
    ...
  }
  ```

## 更多详细配置，可以查看[官方文档](https://mybatis.org/mybatis-3/zh/index.html)

---

好了，《Mybatis 入门篇》的内容到此结束。希望你已经上手了 MyBatis 框架。虽然使用注解的方式写着代码真的很爽，但是耦合度的确也随之上升，这就偏离了使用 mybatis 利用 xml 配置写 sql 以解耦代码与 sql 的意图了。如果想很爽的写注解，那还不如去使用 Spring Data Jpa。
