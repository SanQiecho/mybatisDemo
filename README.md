![IMG_2346](https://github.com/user-attachments/assets/65344607-5520-4b1d-93b4-00283073b4e4)
根据上图表结构，利用mybatis框架，实现1、根据id值查询User信息；2、根据username和password查询User信息，3、增加一行User表记录；
这个项目是java上课老师讲的基础内容
测试

要实现一个基于 Maven 的项目，通过 MyBatis 框架完成对数据库的操作，包括根据 ID 查询用户信息、根据用户名和密码查询用户信息，以及添加用户记录，可以按照以下步骤实现：

---

### 1. 创建 Maven 项目
1. **初始化 Maven 项目**  
   使用以下命令初始化项目：
   ```bash
   mvn archetype:generate -DgroupId=com.example -DartifactId=MyBatisExample -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
   ```
2. **引入依赖**  
   在 `pom.xml` 文件中引入以下依赖：
   ```xml
   <dependencies>
       <!-- MyBatis -->
       <dependency>
           <groupId>org.mybatis</groupId>
           <artifactId>mybatis</artifactId>
           <version>3.5.13</version>
       </dependency>

       <!-- MyBatis-Spring (如果使用Spring整合) -->
       <dependency>
           <groupId>org.mybatis.spring</groupId>
           <artifactId>mybatis-spring</artifactId>
           <version>2.0.7</version>
       </dependency>

       <!-- MySQL驱动 -->
       <dependency>
           <groupId>mysql</groupId>
           <artifactId>mysql-connector-java</artifactId>
           <version>8.0.34</version>
       </dependency>

       <!-- Log4j (用于日志记录) -->
       <dependency>
           <groupId>org.apache.logging.log4j</groupId>
           <artifactId>log4j-api</artifactId>
           <version>2.20.0</version>
       </dependency>
       <dependency>
           <groupId>org.apache.logging.log4j</groupId>
           <artifactId>log4j-core</artifactId>
           <version>2.20.0</version>
       </dependency>
   </dependencies>
   ```

---

### 2. 创建数据库表结构
假设你的 `User` 表结构如下：
```sql
CREATE TABLE User (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    password VARCHAR(50) NOT NULL,
    email VARCHAR(100),
    phone VARCHAR(20)
);

INSERT INTO User (username, password, email, phone)
VALUES ('testuser', 'password123', 'test@example.com', '1234567890');
```

---

### 3. 配置 MyBatis 环境
1. **创建 MyBatis 配置文件 `mybatis-config.xml`**
   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
       <environments default="development">
           <environment id="development">
               <transactionManager type="JDBC"/>
               <dataSource type="POOLED">
                   <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                   <property name="url" value="jdbc:mysql://localhost:3306/testdb"/>
                   <property name="username" value="root"/>
                   <property name="password" value="password"/>
               </dataSource>
           </environment>
       </environments>
       <mappers>
           <mapper resource="com/example/mapper/UserMapper.xml"/>
       </mappers>
   </configuration>
   ```

2. **创建 Mapper 接口**
   ```java
   package com.example.mapper;

   import com.example.model.User;
   import org.apache.ibatis.annotations.Param;

   public interface UserMapper {
       // 根据 ID 查询用户
       User getUserById(@Param("id") int id);

       // 根据用户名和密码查询用户
       User getUserByUsernameAndPassword(@Param("username") String username, @Param("password") String password);

       // 插入新用户
       int insertUser(User user);
   }
   ```

3. **创建 Mapper 映射文件 `UserMapper.xml`**
   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper
           PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="com.example.mapper.UserMapper">

       <!-- 根据 ID 查询用户 -->
       <select id="getUserById" parameterType="int" resultType="com.example.model.User">
           SELECT * FROM User WHERE id = #{id}
       </select>

       <!-- 根据用户名和密码查询用户 -->
       <select id="getUserByUsernameAndPassword" parameterType="map" resultType="com.example.model.User">
           SELECT * FROM User WHERE username = #{username} AND password = #{password}
       </select>

       <!-- 插入新用户 -->
       <insert id="insertUser" parameterType="com.example.model.User">
           INSERT INTO User (username, password, email, phone)
           VALUES (#{username}, #{password}, #{email}, #{phone})
       </insert>

   </mapper>
   ```

---

### 4. 创建实体类
```java
package com.example.model;

public class User {
    private int id;
    private String username;
    private String password;
    private String email;
    private String phone;

    // Getters and Setters
    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getPhone() {
        return phone;
    }

    public void setPhone(String phone) {
        this.phone = phone;
    }
}
```

---

### 5. 测试功能
```java
package com.example;

import com.example.mapper.UserMapper;
import com.example.model.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

public class Main {
    public static void main(String[] args) throws IOException {
        // 加载 MyBatis 配置文件
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        // 获取 SqlSession
        try (SqlSession session = sqlSessionFactory.openSession()) {
            UserMapper userMapper = session.getMapper(UserMapper.class);

            // 根据 ID 查询用户
            User userById = userMapper.getUserById(1);
            System.out.println("User by ID: " + userById);

            // 根据用户名和密码查询用户
            User userByUsernameAndPassword = userMapper.getUserByUsernameAndPassword("testuser", "password123");
            System.out.println("User by Username and Password: " + userByUsernameAndPassword);

            // 插入新用户
            User newUser = new User();
            newUser.setUsername("newuser");
            newUser.setPassword("newpassword");
            newUser.setEmail("newuser@example.com");
            newUser.setPhone("9876543210");
            int result = userMapper.insertUser(newUser);
            session.commit(); // 提交事务
            System.out.println("Inserted rows: " + result);
        }
    }
}
```

---

### 6. 运行结果
运行程序后，可以看到以下输出：
- 查询的用户信息。
- 插入操作返回的影响行数。
- 插入后的用户可以通过数据库验证。

完成后，你的 Maven 项目就可以通过 MyBatis 框架与数据库交互了。
