---
layout: post
title: SpringBoot Mybatis
categories: [SpringBoot]
description: Listener 有助于完成其他的功能
keywords: SpringBoot,Java
---

# SpringBoot Mybatis
MyBatis 是支持定制化SQL、存储过程以及高级映射的优秀的持久层框架.MyBatis 避免了几乎所有的JDBC 代码和手动设置参数以及获取结果集.把单身狗很好的解放出来.

## 任务目标
SpringBoot 整合 Mybatis.

## 样例实现

### 1.Maven依赖

1. 连接mysql的必要依赖mysql-connector-java
2. MyBatis的核心依赖mybatis-spring-boot-starter
3. 不引入spring-boot-starter-jdbc依赖，是由于mybatis-spring-boot-starter中已经包含了此依赖

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.1.1</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.21</version>
</dependency>
```

### 2.application.properties配置

```
spring.datasource.url=jdbc:mysql://localhost:3306/wangl
spring.datasource.username=mysql
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

### 3.在Mysql中创建表和对应的实体类
在Mysql中创建User表，包含id(BIGINT)、name(INT)、age(VARCHAR)字段。同时，创建映射对象User

```java
public class User {
    private Long id;
    private String name;
    private Integer age;
}
```

### 4.注解的映射操作
创建User映射的操作UserMapper，为了后续单元测试验证，实现插入和查询操作

```java
@Mapper
public interface UserMapper {
    @Select("SELECT * FROM USER WHERE NAME = #{name}")
    User findByName(@Param("name") String name);
    @Insert("INSERT INTO USER(NAME, AGE) VALUES(#{name}, #{age})")
    int insert(@Param("name") String name, @Param("age") Integer age);
}
```

### 5.插入数据测试

- 测试逻辑：插入一条 `name=wl，age=25 `的记录，然后根据name=wl查询，并判断 `age` 是否为 `25`
- 测试结束回滚数据，保证测试单元每次运行的数据环境独立

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = Application.class)
public class ApplicationTests {
    @Autowired
    private UserMapper userMapper;
    @Test
    @Rollback
    public void findByName() throws Exception {
        userMapper.insert("wl", 25);
        User u = userMapper.findByName("wl");
        Assert.assertEquals(20, u.getAge().intValue());
    }
}
```

代码详见:[Chapter2_1](https://github.com/ITriangle/SpringBoot)


## 注解说明

### 1.参数传递
变量传递的形式: `#{变量名 [,数据类型]}`

- @Param: @Param中定义的name对应了SQL中的#{name}，age对应了SQL中的#{age}。

```java
@Insert("INSERT INTO USER(NAME, AGE) VALUES(#{name}, #{age})")
int insert(@Param("name") String name, @Param("age") Integer age);
```

- Map:Map赋值时直接用名称匹配,需要指明变量类型

```java
Map<String, Object> map = new HashMap<>();
map.put("name", "wk");
map.put("age", 28);
userMapper.insertByMap(map);

@Insert("INSERT INTO USER(NAME, AGE) VALUES(#{name,jdbcType=VARCHAR}, #{age,jdbcType=INTEGER})")
int insertByMap(Map<String, Object> map);
```

- 对象:实例对象同样可以作为参数传递,毕竟这个才是亲生的啊!类型也不需要指定.

```java
@Insert("INSERT INTO USER(NAME, AGE) VALUES(#{name}, #{age})")
int insertByUser(User user);
```

### 2.CURD
频率高的增删改查

```java
public interface UserMapper {
    @Select("SELECT * FROM user WHERE name = #{name}")
    User findByName(@Param("name") String name);

    @Insert("INSERT INTO user(name, age) VALUES(#{name}, #{age})")
    int insert(@Param("name") String name, @Param("age") Integer age);

    @Update("UPDATE user SET age=#{age} WHERE name=#{name}")
    void update(User user);

    @Delete("DELETE FROM user WHERE id =#{id}")
    void delete(Long id);
}
```

### 3.获取返回结果集

增、删、改 对于返回值较少,但是查询不一样.并且会涉及多表,所有返回结果集会比较复杂.当然,Mybatis也考量了这点,提供了 `@Result` 注解用做字段和类属性的绑定.

```java
@Results({
    @Result(property = "name", column = "name"),
    @Result(property = "age", column = "age")
})
@Select("SELECT name, age FROM user")
List<User> findAll();
```

`@Result`中的`property`属性对应`User`对象中的成员名，`column`对应`SELECT`出的字段名。在该配置中故意没有查出`id`属性，只对`User`对应中的`name`和`age`对象做了映射配置，这样可以通过下面的单元测试来验证查出的id为null，而其他属性不为`null`.

```java
@Test
@Rollback
public void testUserMapper() throws Exception {
    List<User> userList = userMapper.findAll();
    for(User user : userList) {
        Assert.assertEquals(null, user.getId());
        Assert.assertNotEquals(null, user.getName());
    }
}
```

## 参考
1. [mybatis](http://www.mybatis.org/mybatis-3/zh/java-api.html)