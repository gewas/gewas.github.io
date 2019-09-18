---
layout: post
title: 《Mybatis 入门篇》 01 简易上手
---

说起 Java Web 开发哪家强，这 SSM 组合可真是耳熟能详。所以准备整理一个 Mybatis 入门系列的文章。

---

## 什么是框架

框架是软件开发中的技术层面上的一套解决方案（高可复用），不同的框架解决不同的问题。  
框架封装了很多技术细节，开发者可以使用简单方式实现业务，提高开发效率。  
框架本身是商业半成品，只有加上业务，才能成为商业产品。

## 经典的三层架构

展示层：展示数据  
业务层：处理业务需求  
持久层：和数据库交互

## 持久层技术解决方案

- JDBC
- Spring 的 Jdbctemplate
  - 它是对 JDBC 的简易封装
- Apache 的 DBUtils

  - 和 Spring 的 JdbcTemplate 类似，是 JDBC 的简易封装

以上这些**都不是框架**

- JDBC 是规范
- Spring 的 JdbcTemplate 和 Apache 的 DBUtils 都只是工具类

## JDBC

### JDBC 的三大部件

- Connection

  - PreparedStatement
  - ResultSet

### JDBC 的基本步骤

1. 加载注册驱动

   ```java
   Class.forName("com.mysql.jdbc.Driver");
   ```

2. 通过 url 获取 Connection

   ```java
   connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/dbname?charsetEncoding=utf-8", "root", "password");
   ```

3. 编写 sql

   ```java
   // ? 表示参数占位符
   String sql = "select * from user where username = ?";
   ```

4. 获取预处理 statement

   ```java
   preparedStatement = connection.preparedStatement(sql);
   // 设置参数，sql语句中的参数序号从1开始
   preparedStatement.setString(1, "张三");
   ```

5. 执行并得到结果集

   ```java
   resultSet = preparedStatement.executeQuery();
   ```

6. 遍历结果集

   ```java
   while (resultSet.next()){
      String id = resultSet.getString("id");
      String username = resultSet.getString("username");
   }
   ```

7. 释放资源，按顺序释放 resultSet、preparedStatement、connection

## MyBatis 介绍

mybatis 是一个优秀的基于 Java 的持久层框架，内部封装了 jdbc 的诸多操作细节，开发者只需要关注 sql 语句本身，省去了很多样板代码。  
它使用了 ORM(Object Relational Mapping) 把结果集封装到了实体类。

## mybatis 环境搭建的基本步骤

1. 首先，我们使用 maven 搭建项目

2. 导入 mybatis 的 maven 依赖 GAV，至少需要 mybatis 本身和 mysql-connector-java

   ```xml
    <dependencies>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>x.y.z</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>x.y.z</version>
        </dependency>
    </dependencies>
   ```

3. 创建实体类

   ```java
   package com.test.domain;

   import java.io.Serializable;
   import java.util.Date;

   public class User implements Serializable {

      private Integer id;
      private String username;
      private Date birthday;
      private String gender;
      private String address;

      // 省略了getter和setter没列出来
   }
   ```

4. 创建 dao 接口

   ```java
   package com.test.dao;

   import com.test.domain.User;
   import java.util.List;

   /**
    * 用户的持久层接口
    */
   public interface IUserDao {

      /**
       * 查询所有操作
       * @return
       */
      List<User> findAll();
   }
   ```

5. resources 目录下创建 mybatis 的主配置文件 mybatisConfig.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE configuration
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <!-- mybatis的主配置文件 -->
   <configuration>
       <!-- 配置环境 -->
       <environments default="mysql">
           <!-- 配置mysql的环境-->
           <environment id="mysql">
               <!-- 配置事务的类型-->
               <transactionManager type="JDBC"></transactionManager>
               <!-- 配置数据源（连接池） -->
               <dataSource type="POOLED">
                   <!-- 配置连接数据库的4个基本信息 -->
                   <property name="driver" value="com.mysql.jdbc.Driver"/>
                   <property name="url" value="jdbc:mysql://localhost:3306/         dbname"/>
                   <property name="username" value="root"/>
                   <property name="password" value="password"/>
               </dataSource>
           </environment>
       </environments>

       <!-- 指定映射配置文件的位置，映射配置文件指的是每个dao独立的配置文件 -->
       <mappers>
           <mapper resource="com/test/dao/IUserDao.xml"/>
       </mappers>
   </configuration>
   ```

6. 在 resources 目录下创建上述路径中的 mapper 映射配置文件

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE mapper
           PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="com.test.dao.IUserDao">
       <!--配置查询所有，这对应了dao中的方法信息-->
       <select id="findAll" resultType="com.test.domain.User">
           select * from user
       </select>
   </mapper>
   ```

## mybatis 配置注意事项

1. 创建 IUserDao.java 和 IUserDao.xml 时，是照顾以前的编码习惯，所以命名为 Dao。但是 MyBatis 中也把持久层的操作接口和映射文件命名为 Mapper。按开发者的习惯不同，我们都需要知道 Dao 和 Mapper 是代表一个意思。

2. mybatis 的映射配置文件位置必须和 dao 接口的包结构相同。

3. 映射配置文件的 mapper 标签 namespace 属性的值必须是 dao 接口的全限定类名。

4. 映射配置文件的操作配置，其 id 属性的值必须是 dao 接口的方法名。

**按照以上 2. 3. 4.条配置后，开发中就不必去写 dao 的实现类，将由 mybatis 框架动态代理生成代理类供开发者使用。**

## mybatis 入门案例

### 基本使用步骤

```java
//1.读取配置文件
InputStream in = Resources.getResourceAsStream("mybatisConfig.xml");
//2.创建SqlSessionFactory工厂
SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
SqlSessionFactory factory = builder.build(in);
//3.使用工厂生产SqlSession对象
SqlSession session = factory.openSession();
//4.使用SqlSession创建Dao接口的代理对象
IUserDao userDao = session.getMapper(IUserDao.class);
//5.使用代理对象执行方法
List<User> users = userDao.findAll();
for(User user : users){
    System.out.println(user);
}
//6.释放资源
session.close();
in.close();
```

要记得在 mapper 配置文件中的操作标签设定 resultType 属性，其值为返回类型的全限定类名。这样才能实现 ORM 操作。

### 基于注解开发的注意事项

- 把 IUserDao.xml 移除（mapper 配置与注解不可共存），在 Dao 接口的方法上使用@select 注解，并指定 sql 语句
- 在 mybatisConfig.xml 中 mapper 标签使用 class 属性，其值为 Dao 接口的全限定类名

---

好了，第一篇就先到此为止。下一篇，我会分析一下 mybatis 基本使用流程中的原理。
