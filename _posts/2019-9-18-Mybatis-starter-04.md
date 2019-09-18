---
layout: post
title: 《Mybatis 入门篇》 04 多表操作
---

上一篇，主要是介绍了 Mybatis 的使用方法。本篇，我们着重讲讲多表操作。

---

### mybatis 的连接池

**连接池**就是一个提供可用链接的有限资源容器，它必须是线程安全的，有着队列的 FIFO 特性。感觉上类似于线程池。连接池于系统时就初始化完毕，可以降低获取链接消耗的时间。

mybatis 配置连接池的位置在 mybatis 主配置 xml 中的 environment 标签的`<datasource>`子标签中，可以配置数据源属性`type`，有 3 种类型：

- **POOLED**，采用传统 javax.sql.DataSource 规范，mybatis 有规范的实现类 PooledDataSource，用了池的思想来管理链接。
- **UNPOOLED**，同上也遵循了规范，有实现类 UnpooledDataSource，但是没有用池的思想管理链接，每次都是新建链接。
- **JNDI**，用服务器提供的 JNDI 技术实现，来获取 DataSource 对象，不同服务器能拿到的 DataSource 不一样。**注意**：如果不是 web 或 maven 的 war 工程，是不能使用的。我们常用的 tomcat 服务器，采用的是 dbcp 连接池。

**PooledDataSource**获取链接的核心逻辑：

1. 先去**空闲链接集合**中看看有没有链接可用，没有就进行下一步，否则直接返回集合中的第 0 个
2. 如果无空闲链接，就判断当前活动链接数是否达到限制，否则 new 一个 pooledConnection，扔进**活动连接集合**中
3. 如果已达活动连接数限额，就在**活动连接集合**中拿最老的那个链接处理后返回

我们开发中自然要优先选择**POOLED**类型的数据源使用。

### mybatis 的事务控制及涉及的方法

#### 什么是事务？

- 事务(Transaction)，是并发控制的基本单位。它是一个操作序列，这些操作要么全部执行完毕，要么都不执行。

#### 事务的四大特性 ACID

- **Atomicity，原子性**。原子性，取其不可分割之意。这说明一个事务是最基本的操作，没有中间过程状态。事务要么执行完毕，要么出错回滚。
- **CONSISTENCY，一致性**。一个事务必须保护执行前后数据的完整性一致。
- **ISOLATION，隔离性**。一个事务的执行过程中，不受到其他事务的影响。
- **DURABILITY，持久性**。一个被完成的事务，其效果应该是持久化的。

#### 不考虑隔离性会产生的 3 个问题

- **脏读**。事务 A 读到了事务 B 未提交的数据。
- **幻读**。事务 A 读到了事务 B 已提交的数据，导致多次查询结果不一致。
- **不可重复读**。事务 A 读到了事务 B 已提交的数据，导致一个事务中的多次查询结果不一致。

#### 事务的四种隔离级别

- **read uncommitted**。事务 A 能读到事务 B 改变但未提交的数据。脏读、幻读、不可重复读都能发生。
- **read committed**。事务 A 能读到事务 B 改变且已提交的数据。避免了脏读，幻读、不可重复读可能发生。
- **repeatable read**。事务 A 执行过程中读到的数据与事务 A 开始执行那一刻的数据保持一致。避免了不可重复读、脏读，幻读可能发生。
- **serializable**。事务串行化执行，不会出现上面 3 个问题。

#### mybatis 中的事务操作

mybatis 的事务通过 sqlSession 对象的 commit 方法和 rollback 方法实现事务的提交与回滚。但最终都是在操作`java.sql.Connection`对象的 commit 与 rollback。

在执行 sqlSessionFactory.openSession 方法时，可以传入参数`true`来激活自动提交。但是我们更推荐手动控制事务的提交与回滚。

### mybatis 的动态语句

在 mapper.xml 配置文件中的方法描述里，可以使用`<where>`、`<if>`、`<foreach>`等标签，实现一些复杂的操作。

- 使用给定的用户信息查找用户（sql 语句中有一个 `where 1=1`）：

  ```xml
  <select id="findUserByCondition" resultMap="userMap" parameterType="user">
      select * from user where 1=1
      <if test="userName != null">
        and username = #{userName}
      </if>
      <if test="userSex != null">
          and sex = #{userSex}
      </if>
  </select>
  ```

- 上面的例子还可以替换成`<where>`标签的形式，结构会清晰一些：

  ```xml
  <select id="findUserByCondition" resultMap="userMap" parameterType="user">
        select * from user
        <where>
            <if test="userName != null">
                and username = #{userName}
            </if>
            <if test="userSex != null">
                and sex = #{userSex}
            </if>
        </where>
  </select>
  ```

- 根据传入参数对象中的 ids 集合，查询对象（例子中还使用了`<include>`来**复用 sql** 语句）：

  ```xml
  <!-- 先用sql标签定义复用的sql语句，后面才能include，通常用来定义从表中抽取的具体字段们，而不是示例的* -->
  <sql id="defaultUser">
        select * from user
  </sql>
  <!-- 根据queryvo中的Id集合实现查询用户列表 -->
  <select id="findUserInIds" resultMap="userMap" parameterType="queryvo">
    <include refid="defaultUser"></include>
    <where>
        <if test="ids != null and ids.size()>0">
            <foreach collection="ids" open="and id in (" close=")" item="uid" separator=",">
                #{uid}
            </foreach>
        </if>
    </where>
  </select>
  ```

  - 由于用到了`<sql>`和`<include>`来抽取复用 sql，就需要注意一点：在`<sql>`标签中的 sql 语句最好不要加`;`，mapper 的 xml 文件中 sql 语句都不要加`;`是一个良好的实践。

### mybatis 的多表查询

#### 表之间的关系

- **一对多**
- **多对一**
- **一对一**
- **多对多**

例子：

- 一个用户可以下多个订单，多个订单属于一个用户。
  - 用户->订单：一对多
  - 订单->用户：多对一
- 一个人只有一个省份证号，一个身份证号也只对应一个人
  - 人-身份证号：一对一
- 一个学生可以被多个老师教，一个老师可以教多个学生
  - 学生-老师：多对多

特例：用户-订单关系中，单拿出一个订单看，它只属于一个用户。所以 MyBatis 就把多对一看成了一对一。

#### mybatis 多表示例

##### 用户与账户：一对一，一对多

一个用户可以有多个账户，一个账户只属于一个用户。

- 步骤：

  - 建两张表：用户表、账户表
    - 让用户表与账户表间有一对多的关系，需要在在账户表中添加用户 id 外键
  - 建两个实体类：用户、账户
    - 让用户与账户两个实体类能体现一对多关系
  - 建两个配置文件
    - 用户 mapper.xml
    - 账户 mapper.xml
  - 实现配置：
    - 查询用户时，同时得到其所有的账户信息
    - 查询账户时，同时得到其所属用户的信息

- 查询 Account 同时得到 User，一对一实现：

  - 在 IAccountDao.xml 的 mapper 配置中：

    ```xml
    <!-- 定义封装account和user的resultMap -->
    <resultMap id="accountUserMap" type="account">
        <id property="id" column="aid"></id>
        <result property="uid" column="uid"></result>
        <result property="money" column="money"></result>
        <!-- 一对一的关系映射：配置封装user的内容，这里的javaType在没配置别名时要写全限定类名canonical class name-->
        <association property="user" column="uid" javaType="user">
            <id property="id" column="id"></id>
            <result property="username" column="username" ></result>
            <result property="address" column="address" ></result>
            <result property="sex" column="sex" ></result>
            <result property="birthday" column="birthday" ></result>
        </association>
    </resultMap>

    <!-- 查询所有 -->
    <select id="findAll" resultMap="accountUserMap">
        select u.*,a.id as aid,a.uid,a.money from account a , user u where u.id = a.uid;
    </select>
    ```

  - 其实有经验的话，大致也能猜出 Account 实体类中的属性了

    ```java
    public class Account implements Serializable {

      private Integer id;
      private Integer uid;
      private Double money;
      private User user;
      // 省略了setter/getter
    }
    ```

- 查询 User 同时得到 List\<Account\>，一对多实现：

  - IUserDao.xml 配置中：

    ```xml
    <!-- 定义User的resultMap-->
    <resultMap id="userAccountMap" type="user">
        <id property="id" column="id"></id>
        <result property="username" column="username"></result>
        <result property="address" column="address"></result>
        <result property="sex" column="sex"></result>
        <result property="birthday" column="birthday"></result>
        <!-- 配置user对象中accounts集合的映射 这里的ofType如果没使用别名，要写全限定类名canonical class name-->
        <collection property="accounts" ofType="account">
            <id column="aid" property="id"></id>
            <result column="uid" property="uid"></result>
            <result column="money" property="money"></result>
        </collection>
    </resultMap>

    <!-- 查询所有 -->
    <select id="findAll" resultMap="userAccountMap">
        select u.*,a.id as aid,a.uid,a.money from user u left outer join account a on u.id = a.uid
    </select>
    ```

##### 用户与角色：多对多

一个用户可以有多个角色，一个角色也可以分配给多个用户。

- 步骤：

  - 建两张表：用户表、角色表
    - 让用户表与账户表间有多对多的关系，需要使用中间表，中间表包含各自的主键，在中间表中是外键。
  - 建两个实体类：用户、角色
    - 让用户与角色两个实体类能体现多对多关系
  - 建两个配置文件
    - 用户 mapper.xml
    - 角色 mapper.xml
  - 实现配置：
    - 查询用户时，同时得到其所有的角色信息
    - 查询角色时，同时得到其所有的用户信息

- 查询角色带用户

  ```xml
    <!--定义role表的ResultMap-->
    <resultMap id="roleMap" type="role">
        <id property="roleId" column="rid"></id>
        <result property="roleName" column="role_name"></result>
        <result property="roleDesc" column="role_desc"></result>
        <collection property="users" ofType="user">
            <id column="id" property="id"></id>
            <result column="username" property="username"></result>
            <result column="address" property="address"></result>
            <result column="sex" property="sex"></result>
            <result column="birthday" property="birthday"></result>
        </collection>
    </resultMap>

    <!--查询所有,多对多主要是在sql语句上的变化,这里使用了 t_role -> t_user_role -> t_user两次左外连接-->
    <select id="findAll" resultMap="roleMap">
       select u.*,r.id as rid,r.role_name,r.role_desc from role r
        left outer join user_role ur  on r.id = ur.rid
        left outer join user u on u.id = ur.uid
    </select>
  ```

- 查询用户带角色

  ```xml
    <!-- 定义User的resultMap-->
    <resultMap id="userMap" type="user">
        <id property="id" column="id"></id>
        <result property="username" column="username"></result>
        <result property="address" column="address"></result>
        <result property="sex" column="sex"></result>
        <result property="birthday" column="birthday"></result>
        <!-- 配置角色集合的映射 -->
        <collection property="roles" ofType="role">
            <id property="roleId" column="rid"></id>
            <result property="roleName" column="role_name"></result>
            <result property="roleDesc" column="role_desc"></result>
        </collection>
    </resultMap>

    <!-- 查询所有 -->
    <select id="findAll" resultMap="userMap">
        select u.*,r.id as rid,r.role_name,r.role_desc from user u
         left outer join user_role ur  on u.id = ur.uid
         left outer join role r on r.id = ur.rid
    </select>
  ```

### 补充：JNDI

在使用 tomcat 容器部署 war 包服务时，需要使用 JNDI，走的是另一套配置方，交由 tomcat 容器通过 JNDI 来管理我们的数据源等信息，必须经过 tomcat 获取，才能成功拿到数据源。

#### 配置方式

- 在 webapp/META-INF/context.xml 中

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <Context>
  <!--
  <Resource
  name="jdbc/test_mybatis"						数据源的名称
  type="javax.sql.DataSource"						数据源类型
  auth="Container"								数据源提供者
  maxActive="20"									最大活动数
  maxWait="10000"									最大等待时间
  maxIdle="5"										最大空闲数
  username="root"									用户名
  password="1234"									密码
  driverClassName="com.mysql.jdbc.Driver"			驱动类
  url="jdbc:mysql://localhost:3306/dbname"	连接url字符串
  />
  -->
  <Resource
  name="jdbc/test_mybatis"
  type="javax.sql.DataSource"
  auth="Container"
  maxActive="20"
  maxWait="10000"
  maxIdle="5"
  username="root"
  password="1234"
  driverClassName="com.mysql.jdbc.Driver"
  url="jdbc:mysql://localhost:3306/dbname"
  />
  </Context>
  ```

- 在 resources 的 mybatis 主配置文件中

  ```xml
  <environments default="mysql">
        <!-- 配置mysql的环境 -->
        <environment id="mysql">
            <!-- 配置事务控制的方式 -->
            <transactionManager type="JDBC"></transactionManager>
            <!-- type更换成了JNDI，也注意内部属性的内容的变化 -->
            <dataSource type="JNDI">
                <property name="data_source" value="java:comp/env/jdbc/test_mybatis"/>
            </dataSource>
        </environment>
    </environments>
  ```
