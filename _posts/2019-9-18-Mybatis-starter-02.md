---
layout: post
title: 《Mybatis 入门篇》 02 简易原理
---

上一篇中，简要说了一下 mybatis 的基本使用步骤。本篇中，主要会说一些基本使用流程中每一步中的原理。

---

## 简单分析

1. 根据主配置文件可以得到**连接信息**，也就能注册驱动，获取 connection。同时也知道了 mapper 配置的位置。

1. 在 mapper 配置文件中可以得到**映射信息**，由 sql 语句可以得到 preparedStatement，从而执行得到 ResultSet，配合 resultType 可以反射操作，最终实现 ORM。  
   **补充说明：**
   Mapper（映射信息对象）中必须包含两个信息：一、是要执行的 sql 语句；二、是方法要返回的类型的全限定类名，好让框架完成 ORM 封装。然后 Mapper 信息被框架加载后，按以下的 key-value 形式保存在内存中。

   |            string             |     Mapper      |
   | :---------------------------: | :-------------: |
   | com.test.dao.IUserDao.findAll | Mapper 信息对象 |

1. 根据 Dao 接口动态代理创建 Dao 的代理对象

   - 代理对象使用和被代理对象相同的类加载器
   - 代理对象要实现和被代理对象相同的接口
   - 如何代理：需要实现 InvocationHandler 接口，我们自己提供方法内容，实现去执行对应的 sql。

1. 分析基本步骤中所使用到的几个类

   1. Resources，它提供了加载本地配置文件到 InputStream 的方法。

   2. Mapper，它是映射信息实体类，只包含两个属性：

      ```java
      private String sql;
      private Stirng resultType;//实体类的全限定类名
      ```

   3. Configuration，它是我们简易框架的配置类，包含以下属性：

      ```java
      private String driver;
      private Stirng url;
      private String username;
      private String password;
      private Map<String, Mapper> mappers;//映射信息的map
      ```

   4. XMLConfigLoader，它使用 DOM 解析工具解析主配置文件的输入流，提供返回 Configuration 对象的方法。

   5. SqlSessionFactoryBuilder，它通过传入的主配置文件输入流创建 SqlSessionFactory。在这个过程中，它会使用 4 中的类解析主配置文件，生成 Configuration 对象，供 SqlSessionFactory 使用。

   6. SqlSessionFactory 及其实现类 DefaultSqlSessionFactory，DefaultSqlSessionFactory 持有一个 Configuration 对象。并且提供一个返回 SqlSession 的 openSession 方法，SqlSession 对象也持有 Configuration。

   7. SqlSession 及其实现类 DefaultSqlSession，它使用 Configuration 中的 driver，url，username，password 完成加载驱动，获取 Connection。它还提供获取 Mapper 代理对象的方法和关闭 session 的方法。以下给出获取 Mapper 代理的方法实现：

      ```java
      /**
       * 用于创建代理对象
       * @param daoInterfaceClass dao的接口字节码
       * @param <T>
       * @return
       */
      @Override
      public <T> T getMapper(Class<T> daoInterfaceClass) {
          return (T) Proxy.newProxyInstance(daoInterfaceClass.getClassLoader(),
                  new Class[]{daoInterfaceClass},new MapperProxy(configuration.getMappers(),     connection));
      }
      ```

   8. MapperProxy，实现 InvocationHandler 接口，在 invoke 方法中实现查找具体 mapper 并递交给 Exector 执行。

   9. Executor，它是 sql 语句的最终执行者。获取 PreparedStatment，执行并获取 ResultSet，以及 ORM 封装进实体类。

1. 总结：以上为简易的 mybatis 基本原理分析。简单概括起来：**加载配置信息，创建 Mapper 代理对象，实现查询功能。**
