# 简介

MyBatis是java开发web应用时常使用的**持久层框架**，用于简化JDBC的开发

# 将Mybaits框架整合到SpringBoot中

## 创建工程

- 创建springboot工程，并勾选Mybatis相关的依懒
  - MyBatis Freamwork
  - MySQL Driver
- 在数据库中创建好项目相关的表
- 在springboot中创建好相关表的实体类，并写好setter、getter和toString方法

## 配置Mybatis与数据库的连接信息

在springboot中的**application.properties**中配置好如类名、url、用户名、密码登配置信息

![image-20240119161504712](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20240119161504712.png)

## 编写SQL

### 基于注解

根据Mybatis的规范，需要定一个Mapper接口来操作表，如User表对应UserMapper接口，这需要通过**@Mapper**注解标识:

```java
@Mapper
public interface UserMapper {
  	// @Select注解表名是一个查询语句
		@Select("sql语句")
		// 定义方法
		public List<User> list();
}
```

此后当**list()**方法被调用，**@Select**注解中的SQL语句就会被执行，并将返回的结果写入到方法的返回值`List<User>`中

通过**@Mapper**对象，程序会在运行时**自动生成其修饰的接口的实现类对象**(代理对象)，并且将该对象交给IOC容器管理，成为其中的一个bean，故**不需要再编写实现类**

且在整合到Springboot中时，**由于@Mapper注解已经能标识是Dao层组件，因此不必再使用@Repository注解**

在需要拿到SQL的结果时，创建的接口的对象并调用对应方法并接收即可

编写测试类:

```java
@SpringBootTest
class MyBatisStudyApplicationTests {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testListUser() {
        List<User> userList = userMapper.list();
        userList.stream().forEach(user -> {
            System.out.println(user);
        });
    }
}
```

### 基于XML

见之后的篇幅

## 配置SQL提示

选中SQL语句，右键 -> 显示上下文操作 -> 语言注入设置 -> MySQL

在IDE中配置数据源，才会在写SQL时有表名的提示



# JDBC

JDBC(Java DataBase Connectivity)，是由SUN公司指定的一套使用Java操作关系型数据库的API，即接口

各个关系型数据库厂商的产品，如果希望可以允许Java去操作它，则需要提供实现JDBC的jar包，是**数据库驱动**的一种

由此，对于用户而言只需要操作JDBC的接口去满足需求即可，而真正去操作数据库的是驱动jar包中的实现类



## 使用JDBC操作数据库

使用JDBC操作数据的代码量是比较大的，以下代码实现了和上述使用注解编写SQL并操作数据库一样的效果:

```java
public void testJDBC() throws Exception {
    // 1. 注册驱动
    Class.forName("com.mysql.cj.jdbc.Driver");

    // 2. 获取链接对象
    String url = "jdbc:mysql://localhost:3306/mybatis";
    String userName = "root";
    String password = "sf20020107";
    Connection connection = DriverManager.getConnection(url, userName, password);

    // 3. 获取执行SQL的对象Statement，执行SQL，返回结果
    String sql = "select * from user";
    Statement statement = connection.createStatement();
    ResultSet resultSet = statement.executeQuery(sql);

    // 4. 封装结果数据
    List<User> userList = new ArrayList<>();
    while (resultSet.next()) {
        // 每个字段都需要独立解析
        int id = resultSet.getInt("id");
        String name = resultSet.getString("name");
        Short age = resultSet.getShort("age");
        Short gender = resultSet.getShort("gender");
        String phone = resultSet.getString("phone");

        User user = new User(id, name, age, gender, phone);
        userList.add(user);
    }

    // 5. 释放资源
    statement.close();
    connection.close();
}
```

可见编码实现的过程和获取驱动 -> IDE配置 -> 执行SQL -> 获取封装对象的过程类似，MyBatis对于这一过程做了很好的封装

其具体的提升部分为:

- 将配置信息硬编码由硬编码改为写入在配置文件`application.properties`中

  ```properties
  # 驱动类名称
  spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
  # 数据库连接的url
  spring.datasource.url=jdbc:mysql://localhost:3306/mybatis
  # 连接数据库的用户名
  spring.datasource.username=root
  # 连接数据库的密码
  spring.datasource.password=xxxxxx
  ```

- 省去了繁杂的结果封装过程，通过注解写入的SQL在执行完成后会直接将结果写入到接口定义的方法的返回值中

  ```java
  public interface UserMapper {
      @Select("select * from user")
      public List<User> list();
  }
  ```

- 采用了数据库连接池的设计，替代了JDBC中每执行一次SQL任务都需要创建一个连接对象，并执行完后就释放资源这种较为浪费资源的方式

# 数据库连接池

为了提升客户端与数据库服务端之间的连接对象的利用率，mybatis采用了数据库连接池的设计

在连接初始化时，就在一个容器中创建多个连接对象，当有SQL任务需要执行时，则在容器取出一个连接对象，当任务完成之后，把连接对象再返还给连接池，由此避免在有任务时需要执行耗时的创建连接的操作，提升了系统的响应速度

- 容器负责连接(Connection对象)的分配与管理
- 容器中的连接是可以被复用的
- 当一个连接的空闲时间(即没有执行任何任务的时间)超过了设置的**最大空闲时间**，则会被主动释放，由此**避免**因为没有释放连接而引起的**数据库连接泄露**

## 标准接口: DataSource

Sun公司规定了Java语言的数据库连接池结果是DataSource，由数据库厂商们实现

主要的功能实现提供获取链接的方法: `Connection getConnection() throws SQLException`

SpringBoot中默认的连接池产品是**Hikari**，其具备较好的性能

阿里开源的**Druid**也是目前最好的数据库连接池技术之一

## 切换连接池

如果希望使用的是Druid，可以在`pom.xml`中配置依赖

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.2.8</version>
</dependency>
```

 # 工具包: lombok

lombok是为了解决在编写实体类时，解决去生成各属性的getter和setter方法以及toString方法而造成的代码臃肿的问题

lombok是一个java类库，能通过注解的方式自动生成构造器、getter/setter、equals、hashcode、toString等反方法，并可以自动化生成日志变量，用于提效

| 注解                       | 含义                                                         |
| :------------------------- | :----------------------------------------------------------- |
| `@Getter`                  | 自动生成字段的 getter 方法。                                 |
| `@Setter`                  | 自动生成字段的 setter 方法。                                 |
| `@ToString`                | 自动生成 toString 方法，用于打印对象的字符串表示。           |
| `@EqualsAndHashCode`       | 自动生成 equals 和 hashCode 方法，用于对象的比较和哈希计算。 |
| `@NoArgsConstructor`       | 自动生成无参构造函数。                                       |
| `@AllArgsConstructor`      | 自动生成包含所有字段的构造函数。                             |
| `@RequiredArgsConstructor` | 自动生成包含标记了 `@NonNull` 的字段的构造函数。             |
| `@Data`                    | 是 `@ToString`、`@EqualsAndHashCode`、`@Getter` 和 `@Setter` 的组合注解。 |

引入lomback只需在创建项目/模块时勾选`Developer Tools`下的Lombok并等待下载完成即可

或手动在`pom.xml`文件中添加:

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

值得注意的是，对于在数据库中采用蛇形命名的字段，在编写实体类时需转换为小驼峰式命名

Lombok在ide中使用是需要`lombok`插件的，但在较新版本的ide中都自动附带了该插件，若没有的话可以去插件商店手动下载

# 打开Mybatis日志(打开预编译SQL)

该功能在mybatis中默认是关闭的，如需打开，在`application.properties`中增加Mybatis的配置项:

```properties
# 配置mybatis的日志信息，指定输出到控制台
mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```

至此可使mybatis执行的SQL输出到控制台中:

```shell
==>  Preparing: delete from emp where id = ?
==> Parameters: 17(Integer)
<==    Updates: 0
```

该SQL即参数是Mybatis**要发送给数据库执行的，而不是已经执行的**，因此是预编译SQL

通过这种参数化的SQL，无论客户端传的是什么字符，都会被视为一个整体，由此解决了**SQL注入**的问题

# 基础操作

## 参数占位符

- `#{}`: **是参数传递**，是预编译SQL下读取的占位符，能有效防止SQL注入，需要注意的是**{}内一定要有参数**，否则会报错
- `${}`: **是拼接操作**，直接将参数拼接到SQL语句中，**存在SQL注入问题**，一般仅用在对表名、列表进行动态设置时使用

## Delete

如果采用使用注解的方式，根查询操作类似，在mapper接口中定义删除的方法上加上`@Delete`注解即可

```java
@Mapper
public interface EmpMapper {
    @Delete("delete from emp where id = #{id}")
    void deleteById(int id);
}
```

对于mybaits，其参数的占位符是**#{}**，

且如果传递的参数只有一个，占位符的形参名可以是任意的

对于`@Delete`注解，其操作时有返回值的，是一个返回影响行数的int类型的值，因此在接口中定义方法时，也可以将返回值类型设为int，可以用在一些判断删除是否成功的场景中

测试:

```java
@Autowired
private EmpMapper empMapper;

@Test
public void testDelete() {
    empMapper.deleteById(17);
}
```

## Insert

一般插入由于涉及到的字段较多，故与其传入多个参数，**使用直接传入一个对象的方式会更加简洁**，采用注解的方式:

```java
@Insert("insert into emp(username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time)" +
        "values (#{username}, #{password}, #{name}, #{gender}, #{image}, #{job}, #{entrydate}, #{deptId}, #{createTime}, #{updateTime}")
void insert(Emp emp);
```

值得注意的是，诸如`create_time`、`update_time`这样在数据库中是蛇形命名的字段，在传入参数时需要转换为对应的驼峰式命名

测试:

```java
@Test
public void testInsert() {
    Emp emp = new Emp();
    emp.setUsername("Tom");
    emp.setName("汤姆");
    emp.getGender((short)1);
		...
}
```

### 主键返回

由于在项目中表之间常常有关联关系，因此在某张表的数据添加成功后，**常常会需要添加成功的数据的ID**，**用于去维护和其他表之间的关联关系**

@insert注解无法返回已经参入的对象的信息，获取主键可以通过**@Options**注解

```java
@Options(keyProperty = "id", useGeneratedKeys = true)
```

- KeyProperty: 指定返回的主键封装到的哪一属性中
- useGeneratedKeys: `true`时代表要返回被操作的对象的主键值

其会自动将生成的主键值，赋给对象的id属性

```java
@Options(keyProperty = "id", useGeneratedKeys = true)
@Insert("insert into emp(username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time)" +
        "values (#{username}, #{password}, #{name}, #{gender}, #{image}, #{job}, #{entrydate}, #{deptId}, #{createTime}, #{updateTime}")
void insert(Emp emp);
```

至此，可以通过对象的get方法获取到主键值:

```java
empMapper.insert(emp);
emp.getId();
```

## Update

常常根据主键去修改数据，使用注解:

```java
@Update("update emp set username = #{username}, name = #{name}, gender = #{gender}, image = #{image}, " +
        "job = #{job}, entrydate = #{entrydate}, dept_id = #{deptId}, update_time = #{updateTime} where id = #{id} ")
void update(Emp emp);
```

@Update注解返回的值是影响的行数

## Select

通过使用注解的方式，直接用`@Select`修饰对应的方法即可

当查询操作的方法被调用时，往往需要一个对象去承载返回值，这一点是和删、增、改不同的

```java
@Select("SELECT * from emp where id = #{id}")
Emp query(Integer id);
```

### 封装查询结果

当数据库返回查询结果后，若实体类中的属性名和与数据库中的字段名一致，**则mybatis会自动完成结果到对象的封装**，否则诸如在数据库中是蛇形命名和在实体中是驼峰命名这种情况则不会自动封装

为了解决不能自动封装的情况，可以:

1. 在注解的SQL中给字段**起别名**，使得别名与实体类属性名一致

   ```java
   @Select("SELECT create_time createTime, update_time updateTime from emp where id = #{id}")
   ```

2. 通过`@Results`与`@Result`**注解手动标注**字段名到属性名的映射

   ```java
   @Results({
           @Result(column = "create_time", property = "createTime"),
           @Result(column = "update_time", property = "updateTime")
   })
   @Select("SELECT * from emp where id = #{id}")
   Emp query(Integer id);
   ```

   其中column是指明数据库中的字段名，property是实体类中的属性名

3. 开启mybatis中的**驼峰命名自动映射开关**

   在`application.properties`中配置mybatis中该功能的选项值为`true`:

   ```properties
   mybatis.configuration.map-underscore-to-camel-case=true
   ```

   通过这种方式，则不需要对原代码进行任何修改

   需要注意的是，若使用该方式，需要严格保证数据库中的命名是蛇形，而实体类中的属性是驼峰

### 使用like时的注意事项

再注解中写SQL使用到like时，会出现占位符在`''`中的情况，而在参数化的SQL中，?不能出现在`''`中，因此在使用like时，应使用`${}`占位符拼接即可

不过，使用以上这种方法会面临SQL注入的风险，因此可以将换占位符的这种解决方法换为使用MySQL中的函数**concat**

**Concat 的作用是将多个字符串拼接成一个字符串**

```java
@Select("SELECT * from emp where name like concat('%', #{name}, '%') and ...")
```

# 通过XML文件映射SQL

## XML映射定义规范

使用xml需要遵循一些规范才能生效:

- xml映射文件的**名称**必须和**Mapper**接口名称**一致**，并且将xml映射文件的Mapper接口放置在**相同包**下

- xml映射文件的namespace属性为Mapper接口**全限定名**一致，全限类名是指路径+文件名，如`com.mybatiscrud.mapper.EmpMapper`

- xml映射文件中的sql语句和id与Mapper接口中的**方法名**一致，**id是sql语句标签必有的一个属性**，并保持**返回类型**一致

  sql标签还有一个属性`resultType`，用于在返回的**类型是单一**的情况时指定好返回的类型

值得注意的是，再resource中创建目录，分隔符并非是`.`，而是`/`

在编写xml文件时，需要先引入mybatis的约束:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
```

示例:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.mybatiscrud.mapper.EmpMapper">
    <select id="list" resultType="com.mybatiscrud.pojo.Emp">
        SELECT id, username. password, name, gender, image, job,
               entrydate, dept_id, create_time, update_time from emp
        where
               <if test="name != null">
                   name like concat('%', #{name}, '%')
               </if>
               <if test="gender != null">
                   and gender = #{gender}
               </if>
               <if test="begin != null and end != null">
                   and entrydate between #{begin} and #{end}
               </if>
        order by update_time desc
    </select>
</mapper>
```

对应的EmpMapper.java中的对应的方法:

```java
public List<Emp> list(@Param("name") String name, @Param("gender") String gender, @Param("begin") LocalDate begin, @Param("end") LocalDate end);

```

注意使用`@Param`注解修饰形参

## MybatisX插件

mybatisX是一款帮助用户高效使用mybatis的插件，其主要功能有:

- 快速定位，比如xml文件中的SQL所对应的方法



## 使用XML映射SQL语句的适用场景

如果是较为**简单**的SQL语句，使用**注解**会使得代码更加简洁

但如果是较为复杂的SQL语句，若放到注解中会十分不利于阅读，因此建议采用XML映射SQL语句



# Mybatis动态SQL

随着用户的输入或外部条件变化而变化的SQL被称为动态SQL

## if标签

比如，对于很多系统的很多模块的查询功能，几乎都需要能根据**任意参数、任意数量的参数**去查询，如果仅写一条固定的覆盖全字段的SQL，这样在输入参数并不覆盖全字段的情况下返回的结果大概率为NULL，是不符合期望的

因此，Mybatis提供了一个标签`if`，**使用test属性写明条件判断的表达式**，用于设定SQL的变化条件，比如:

```xml
<mapper namespace="com.mybatiscrud.mapper.EmpMapper">
    <select id="query" resultType="com.mybatiscrud.pojo.Emp">
        SELECT id, username. password, name, gender, image, job,
               entrydate, dept_id, create_time, update_time from emp
        where
               <if test="name != null">
                   name like concat('%', #{name}, '%')
               </if>
               <if test="gender != null">
                   and gender = #{gender}
               </if>
               <if test="begin != null and end != null">
                   and entrydate between #{begin} and #{end}
               </if>
        order by update_time desc
    </select>
</mapper>
```

上述例子中，保证了name、gender、begin、end这些输入参数在不为NULL时才参与SQL的条件构建，避免了其中任意一个参数为NULL导致查询结果为NULL以及在更新时将未指定的字段的值更新为NULL的情况

## where标签

where标签的作用主要有两点:

- 根据**其包裹的if标签中的表达式是否有一个成立**而决定**是否生成最终执行的SQL中的where关键字**，比如，如果所有条件都不成立，则不会生成where关键字，否则会生成
- 自动的去除各条件间冗余的**and**，如果这些and不去除，也就是如果仅使用SQL中的where关键字的话，可能会造成最终要执行的SQL出现语法错误

```xml
<mapper namespace="com.mybatiscrud.mapper.EmpMapper">
    <select id="list" resultType="com.mybatiscrud.pojo.Emp">
        SELECT id, password, name, gender, image, job,
               entrydate, dept_id, create_time, update_time from emp
        <where>
            <if test="name != null">
                name like concat('%', #{name}, '%')
            </if>
            <if test="gender != null">
                and gender = #{gender}
            </if>
            <if test="begin != null and end != null">
                and entrydate between #{begin} and #{end}
            </if>
            order by update_time desc  
        </where>
    </select>
</mapper>
```

## set标签

和Where标签的用处类似，使用于在写Update语句时，**自动的适配个字段之间的`,`**，使得SQL符合语法

```xml
<update id="update">
    update emp
    <set>
        <if test="username != null"> username = #{username}</if>
        <if test="name != null"> name = #{name}</if>
    </set>
</update>
```

```xml
<update id="update">
    update emp
        <set>
               username = #{username}, name = #{name}, gender = #{gender},update_time = #{updateTime},
            <if test="image != null">
                image = #{image},
            </if>
            <if test="deptId != null">
                dept_id = #{deptId},
            </if>
            <if test="entrydate != null">
                entrydate = #{entrydate},
            </if>
            <if test="job != null">
                job = #{job}
            </if>
        </set>
        where id = #{id}
</update>
```

## foreach标签

用于循环遍历某个集合的元素，常用于有`in`关键字的场景中，比如批量操作

其有四个主要属性:

- Collection: 指定要遍历的集合，即是客户端传来的集合
- Item: 遍历的元素名
- Separator: 分割符，即指定个元素之间的用什么分割，比如在in关键字中是`,`
- Open: 在开始遍历之前，拼接到SQL语句中的字符串，比如在in关键字中是`(`
- Close: 与open是相对的，在遍历结束之后拼接到SQL中的字符串

xml文件示例:

```xml
<delete id="deleteByIds">
    delete from emp where id in
    <foreach collection="ids" item="id" separator="," open="(" close=")">
        #{id}
    </foreach>
</delete>
```

Mapper:

```java
public void deleteByIds(List<Integer> ids);
```

## sql&include标签

sql&include是为了提升SQL在xml文件中的复用性而创造的

比如，在查询和更新的SQL中，都可能会涉及到全字段，如果在不同的标签中把全部的字段写两遍，复用性是不高的

可以使用sql标签，将一段**固定的**SQL设置成一个类似变量的角色:

```xml
<sql id="selectAllField">
    select id, username, password, name, gender, image, job, entrydate,
           dept_id, create_time, update_time from emp
</sql>
```

之后通过include标签来指定SQL放入的位置，两者通过`id`属性来建立联系，比如可以通过这种方法修改之前的更新操作的SQL:

```xml
<select id="list" resultType="com.mybatiscrud.pojo.Emp">
    <include refid="selectAllField"/>
    <where>
        <if test="name != null">
            name like concat('%', #{name}, '%')
        </if>
        <if test="gender != null">
            and gender = #{gender}
        </if>
        <if test="begin != null and end != null">
            and entrydate between #{begin} and #{end}
        </if>
        order by update_time desc
    </where>
</select>
```

其中include标签自闭和即可



# Mybatis分页查询插件 - PageHelper

正常的分页查询流程是

1. 接收请求的页数以及页面容量
2. 在Dao层计算起始索引: (请求的页数- 1) * 页面容量
3. 查询总记录数，以及查询返回的数据库对象列表
4. 返回状态码、msg、总记录数，查询的对象列表

通过pageHelper简化分页查询操作:

1. 引入依赖

   ```xml
   <dependency>
       <groupId>com.github.pagehelper</groupId>
       <artifactId>pagehelper-spring-boot-starter</artifactId>
       <version>1.4.2</version>
   </dependency>
   ```

2. 简化mapper接口，该插件会自动进行分页

   ```java
   @Select("select * from emp")
   List<Emp> list();
   ```

   替代了:

   ```java
   @Select("select count(*) from emp")
   Long count();
   
   @Select("select * from emp limit #{arg0}, #{arg1}")
   List<Emp> pageQuery(Integer start, Integer pageSize);
   ```

3. 改变service逻辑

   ```java
   public PageBean pageQuery(Integer page, Integer pageSize) {
   
       // 配置分页信息
       PageHelper.startPage(page, pageSize);
       // 查询请求结果
       List<Emp> rows = empMapper.list();
       // 强转page对象
       Page<Emp> p = (Page<Emp>) rows;
       // 返回page对象中的total和result
       return new PageBean(p.getTotal(), p.getResult());
   }
   ```

   代替了:

   ```java
   public PageBean pageQuery(Integer page, Integer pageSize) {
       Long total = empMapper.count();
   
       List<Emp> rows = empMapper.pageQuery((page - 1) * pageSize, pageSize);
   
       return new PageBean(total, rows);
   }
   ```

   

# 常见报错

## Parameter 'xxx' not found. Available parameters are [arg1, arg0, param1, param2]

当你的参数类型为实体类型时，可以使用  #{实体属性名} 。

**当你的参数类型为基本类型时，如（Integer，String ，Boolean 等），使用 #{arg0},#{arg1}……**

当你的方法拥有多种参数时，parameterType属性也可以不写（其实基本上都可以不写）

