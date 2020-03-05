---
layout: post
title: 使用SpringBoot打war包的注意事项并在Tomcat部署
---

一直以来都是使用SpringBoot默认的打包方式，打可执行的jar包，因为SpringBoot有嵌入式的Web容器（Tomcat、Jetty等），也十分方便。
但是jar包多了会为运维同事造成困扰，还是集中打war包，统一在tomcat部署/管理/配置得劲。

---

接下来就按照步骤来修改

## 修改pom文件

* 修改 `<packing>`为`war`

* 移除 Spring Web 的 Tomcat 依赖
  
  ```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
  ```

* 添加servlet-api依赖，因为代码中会用到但是上一步中去调了api 与 tomcat实现，这里要加回来 api
  
  ```xml
        <dependency>
            <groupId>org.apache.tomcat</groupId>
            <artifactId>tomcat-servlet-api</artifactId>
            <version>9.0.31</version>
            <scope>provided</scope>
        </dependency>
  ```

* 添加 war plugin，其中 failOnMissingWebXml 的设置可以避免没有web.xml时报错
  
  ```xml
            <plugin>
                <artifactId>maven-war-plugin</artifactId>
                <configuration>
                    <failOnMissingWebXml>false</failOnMissingWebXml>
                </configuration>
            </plugin>
  ```

## 修改 MainClass

让Application类继承自SpringBootServletInitializer，才可以放入Tomcat使用。

```java
@SpringBootApplication
public class DemoApplication extends SpringBootServletInitializer {

    public static void main(String[] args) throws Exception {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

## 打包

`maven -Dmaven.test.skip=true clean package`

或

`maven -DskipTests=true clean package`

二者等价。

打出来的包通常类似这个名字：`demo-1.0.0-SNAPSHOT.war`

扔到Tomcat之前最好改个名字，这样可以修改追踪访问路径，如果不改现在就要如此访问：`http://localhost:8080/demo-1.0.0-SNAPSHOT/xxx`。

你还可以将名字改成`ROOT`，这样就可以实现这样访问：`http://localhost:8080/xxx`。
