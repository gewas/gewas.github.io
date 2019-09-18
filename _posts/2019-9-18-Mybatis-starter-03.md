---
layout: post
title: 《Mybatis 入门篇》 03 基本使用
---

上一篇，主要是分析了一些 Mybatis 必要的类以及原理。本篇内容就很轻松，主要是 Mybatis 的使用

---

## mybatis 基本使用

- mybatis 的默认配置中，是没有开启自动提交的，所以如果在增删改操作后没手动提交，是会自动回滚事务的，即操作失效。只需添加一句：

  ```java
  //提交事务
  sqlSession.commit();
  ```

- JUnit 单元测试的使用技巧：  
  在我们测试时，如果不采取一定的手段，会在每个@Test 注解的方法中写大量的样板代码，我们可以利用@Before 和@After 两个注解，让 JUnit 为我们自动完成这些操作。

  ```java
  /**
   * 测试mybatis的crud操作
   */
  public class MybatisTest {

      private InputStream in;
      private SqlSession sqlSession;
      private IUserDao userDao;

      @Before//用于在测试方法执行之前执行
      public void init()throws Exception{
          //1.读取配置文件，生成字节输入流
          in = Resources.getResourceAsStream   ("mybatisConfig.xml");
          //2.获取SqlSessionFactory
          SqlSessionFactory factory = new    SqlSessionFactoryBuilder().build(in);
          //3.获取SqlSession对象
          sqlSession = factory.openSession();
          //4.获取dao的代理对象
          userDao = sqlSession.getMapper(IUserDao.class);
      }

      @After//用于在测试方法执行之后执行
      public void destroy()throws Exception{
          //提交事务
          sqlSession.commit();
          //6.释放资源
          sqlSession.close();
          in.close();
      }

      ...
  ```

## mybatis 的单表 crud

### 增

```java
/**
 * 保存用户
 * @param user
 */
void saveUser(User user);
```

```xml
<!-- 保存用户 -->
<!-- 此处的parameterType只写了user，是因为在主配置中配置了别名，在后续小节中说明-->
<insert id="saveUser" parameterType="user">
    <!-- 配置插入操作后，获取插入数据的id。keyproperty对应实体类的属性名，keyColumn对应表中的字段名，order="AFTER"表示是执行完插入语句后再执行这条语句-->
    <selectKey keyProperty="userId" keyColumn="id" resultType="int" order="AFTER">
        select last_insert_id();
    </selectKey>
    insert into user(username,address,sex,birthday)values(#{userName},#{userAddress},#{userSex},#{userBirthday});
</insert>
```

### 删

```java
/**
 * 根据Id删除用户
 * @param userId
 */
void deleteUser(Integer userId);
```

```xml
<delete id="deleteUser" parameterType="java.lang.Integer">
        delete from user where id = #{uid}
</delete>
```

### 改

```java
    /**
     * 更新用户
     * @param user
     */
    void updateUser(User user);
```

```xml
<!-- 更新用户 -->
<!-- 此处的parameterType只写了USER，是因为在主配置中配置了别名，在后续小节中说明-->
<update id="updateUser" parameterType="USER">
    update user set username=#{userName},address=#{userAddress},sex=#{userAex},birthday=#{userBirthday} where id=#{userId}
</update>
```

### 查

```java
    /**
     * 根据id查询用户信息
     * @param userId
     * @return
     */
    User findById(Integer userId);

    /**
     * 根据名称模糊查询用户信息
     * @param username
     * @return
     */
    List<User> findByName(String username);

    /**
     * 查询总用户数
     * @return
     */
    int findTotal();

    /**
     * 根据queryVo中的条件查询用户
     * @param vo
     * @return
     */
    List<User> findUserByVo(QueryVo vo);
```

```java
public class QueryVo {
    private User user;
    // 省略getter和setter
}
```

```xml
    <!-- 根据id查询用户 -->
    <select id="findById" parameterType="INT" resultMap="userMap">
        select * from user where id = #{uid}
    </select>

    <!-- 根据名称模糊查询 -->
    <select id="findByName" parameterType="string" resultMap="userMap">
        <!--使用#{name}，使用的是PreparedStatement占位符的方式传参，可以防SQL注入。如果想模糊匹配，就需要传入参数带%，比如：%王%-->
        select * from user where username like #{name}

        <!--使用'%${value}%'，使用的是Statement字符串拼接的方式传参，有安全隐患。由于已经预写%，只需要传递具体内容即可支持模糊查询-->
        <!-- select * from user where username like '%${value}%'-->
   </select>

    <!-- 获取用户的总记录条数 -->
    <select id="findTotal" resultType="int">
        select count(id) from user
    </select>

    <!-- 根据queryVo的条件查询用户 -->
    <select id="findUserByVo" parameterType="com.test.domain.QueryVo" resultMap="userMap">
        select * from user where username like #{user.username}
    </select>
```

## mybatis 的参数和返回值

- 当方法只有一个参数且该参数为基本类型或包装类时，mybatis 对 sql 中参数占位符的名字没有具体要求，叫什么都可以。参考 2.1.2 中的示例。

- **OGNL 表达式**  
  Object Graphic Navigation Language，对象图导航语言。  
  它通过对象的取值方法获取数据，写法上省略了 get。  
  例：获取用户名  
   类方法调用写法：user.getUsername();  
   OGNL 表达式写法：user.username  
  mybatis 中可以直接写 username 而不需要加 user.是因为在 parameterType 中已经指定了类，所以不需要写对象名。

- parameterType 可接受的值

  - **基本类型与包装类**
  - **POJO 对象**  
    mybatis 使用 ognl 表达式解析对象字段的值，#{}或\${}括号中的值是 pojo 的属性名。
  - **POJO 的包装对象**  
    当需要进行复杂的查询业务时，传入的查询 pojo 对象中还会包含别的 pojo。在上方 2.1.4 的 findUserByVo 有示例。这种由多个对象组成一个对象作为查询条件的方式广泛应用在实际开发中。

- 返回结果集相关说明

  - 结果集字段与实体属性能一一对应：
    - 一切正常
  - 结果集字段与实体属性不能对应上：

    - 如果不做处理，就会出现 orm 不能自动映射
    - 在 windows 下 mysql 数据库不区分字段大小写，所以做到一些神奇的映射，例：字段 username，属性 userName。但是在 Linux 下 mysql 是严格区分大小写的。
    - 所以需要我们把不能一一对应的字段-属性给对应上就能解决问题：

      - 方案一、sql 语句中使用 as 更改结果集中的字段名，从而一一对应，进而解决问题。**但是这样做很低效，且不科学**
      - 方案二、在 mapper 的 xml 中，增加 resultMap 映射关系配置：

        ```xml
        <resultMap id="userMap" type="com.test.domain.User">
            <!-- 主键字段的对应 -->
            <id property="userId" column="id"></id>
            <!--非主键字段的对应-->
            <result property="userName" column="username"></result>
            <result property="userBirthday" column="birthday"></result>
        </resultMap>
        ```

        然后在方法配置中，将 resultType 更换成 resultMap：

        ```xml
        <select id="findByName" parameterType="string" resultMap="userMap">
          select * from user where username like #{name}
        </select>
        ```

        这样，findByName 方法就能将结果按照 userMap 配置的规则映射到`com.test.domain.User`类对象中了。可以看出的是，该方案需要多解析一道 xml，所以性能肯定较方案一低，但是大大提升了开发效率。

## mybatis 的 DAO 实现类

mybatis 支持我们自己编写 DAO 实现类，**虽然通常我们不这么做**。其基本步骤：

1. 读取 mybatis 主配置文件生产输入流
2. 使用 SqlSessionFactoryBuilder 传入主配置输入流，获取 SqlSessionFactory
3. factory.openSession()获取 session
4. 使用 session 对象执行 statement，这里的 statement 是定位具体执行某个方法的声明，例：`com.test.dao.IUserDao.findAll`
5. 然后就可以正常使用，得到结果
6. 最后要记得释放资源

## mybatis 配置的细节

- 主配置引用外部 properties 配置文件

  - 在 resources 目录添加数据库配置文件 jdbcConfig.properties：

    ```properties
    jdbc.driver=com.mysql.jdbc.Driver
    jdbc.url=jdbc:mysql://localhost:3306/dbname
    jdbc.username=root
    jdbc.password=1234
    ```

  - 在 mybatis 主配置文件中:

    ```xml
    <!-- properties可以使用两个属性来配置文件路径：
    一、是常用的resource属性： 在本例中，resource="jdbcConfig.propeerties"
    二、url属性，使用url来定位配置文件路径：在本例中已展示 -->
    <properties url="file:///D:/path/to/project/src/main/resources/jdbcConfig.properties">
    </properties>

    <!--配置环境-->
    <environments default="mysql">
        <!-- 配置mysql的环境-->
        <environment id="mysql">
            <!-- 配置事务 -->
            <transactionManager type="JDBC"></transactionManager>

            <!--配置连接池-->
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"></property>
                <property name="url" value="${jdbc.url}"></property>
                <property name="username" value="${jdbc.username}"></property>
                <property name="password" value="${jdbc.password}"></property>
            </dataSource>
        </environment>
    </environments>
    ```

  - 同样的，主配置文件中 Mapper 标签也支持用 resource 属性与 url 属性来引用配置文件，方便我们灵活的配置。

- 配置别名，方便引用实体类

  - mybatis 主配置文件中可以添加`<typeAliases>`标签来配置别名信息，使用`<typeAlias>`或`<package>`标签配置精确到具体某个类或者包路径下所有类的别名。配置了别名可以方便的以大小写不敏感的方式引用。

    ```xml
    <typeAlias type="com.test.domain.User" alias="user"></typeAlias>
    <package name="com.test.domain"></package>
    ```

    注意，如果使用 package 标签配置，则实体的别名默认为其类名。

- mappers 标签内使用 package 标签来按包指定 DAO 接口

  ```xml
  <mappers>
    <!--<mapper resource="com/test/dao/IUserDao.xml"></mapper>-->
    <!-- package标签是用于指定dao接口所在的包,当指定了之后就不需要再写mapper以及resource或class了 -->
    <package name="com.itheima.dao"></package>
  </mappers>
  ```
