# SpringBoot概述

Spring全家桶的核心是Spring Framework，很多项目是基于其开发而成的，而SpringBoot(boot:引导)是为了方便入门者学习所开发出的可用于快速开发web应用的框架

对于Spring Framework，具备较为繁琐的依赖配置，容易出现版本号冲突等问题，而SpringBoot简化了这一过程，使得开发者可以更加专注于实现业务

# 搭建第一个SpringBoot应用Demo

java目录下存放源码，resource目录下存放配置文件

1. 创建项目(新模块) -> Spring Initializr -> 配置SDK等 -> 创建 -> 选择版本 -> 勾选自动加入的依赖

2. 在`java/com.xxxx`目录下创建Controller层，编写请求处理类以及返回的方法:

   ```java
   public class HelloController {
       public String hello() {
           System.out.println("hello spring!");
           return "hello spring!";
       }
   }
   ```

3. 加上请求处理类标识注解和Map映射注解:

   ```java
   package com.springstudydemo.controller;
   
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   @RestController
   public class HelloController {
       @RequestMapping("/hello")
       public String hello() {
           System.out.println("hello spring!");
           return "hello spring!";
       }
   }
   ```

4. 本地访问测试`localhost:8080/hello`

# Tomcat

web服务器对HTTP协议的操作进行了高度的封装，省去了繁杂的解析HTTP报文等操作的过程，使得开发web程序更加便捷，主要的功能是"提供网上信息游览服务"

Tomcat是一个开源免费的轻量级web服务器，使用Java语言开发，支持Servlet/JSP少量JavaEE规范，可以简化web服务的部署操作

Tomcat也被称为web容器、Servlet容器，Servlet程序需要依赖于Tomcat才能运行

## 下载与启动Tomcat

1. 官网 -> download -> 下载core下的tar.gz包并解压 -> 终端进入bin目录

2. 授权bin目录下的操作:

   ```shell
   sudo chmod 755 *.sh
   ```

3. 启动Tomcat

   ```shell
   sudo sh ./startup.sh
   ```

   关闭则是shutdown

4. 访问`localhost:8080`测试，需注意端口是否被占用

乱码问题: 

Conf -> logging.properties -> java.util.logging.ConsoleHandler.encoding = GBK

配置端口:

Conf -> server.xml 修改:

```xml
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443"
               maxParameterCount="1000"
               />
```

## 在Tomcat部署web服务

将编码并构建完成的项目放置到webapps目录下

# Spring内嵌Tomcat

Spring在自己的脚手架中内嵌了Tomcat，在Spring项目构建时，会通过maven下载Tomcat的相关依赖项，使得项目启动时就跑在Tomcat中

Gin、Echo、Iris等Web框架在Go语言中可以类比于Java中的Tomcat，它们都是用于构建和运行Web应用程序的服务器组件

# Servlet

Servlet在Spring中被称为**核心控制器**或**前端控制器**，其是请求到达服务器时所最先访问的接口，其会解析请求要访问的接口并将请求分发到应接收的接口中处理，最后把处理的结果返回

在Go语言中，可以将Java中的Servlet类比于HTTP处理函数，它们都是用于接收和处理HTTP请求的核心组件。

Servlet有两个核心对象:

- HttpServletRequest: 获取与解析HTTP请求
- HttpServletResponse: 设置与响应HTTP请求



# Controller中的路径设定简化

常常对于某个模块，由此操作的内容基本是同一张表，故在路径命名上往往有相同的前缀，可以把这一部分前缀指定到类的注解中，以简化代码:

比如，简化前的类是:

```java
@Slf4j
@RestController
public class DeptController {

    @Autowired
    private DeptService deptService;

    @GetMapping("/depts")
    public Result list() {
        log.info("查询全部部门数据");

        // 调用service层对象方法处理请求
        List<Dept> deptList = deptService.list();

        return Result.success(deptList);
    }

    @DeleteMapping("/depts/{id}")
    public Result deleteDept(@PathVariable Integer id) {
        log.info("删除id为 " + id + " 的部门");

        Integer code = deptService.deleteDept(id);

        return code == 0? Result.error("删除部门失败") : Result.success();
    }

    @PostMapping("/depts")
    public Result insertDept(@RequestBody Dept dept) {
        log.info("添加名称为 " + dept.getName() + " 的部门");

        Integer code = deptService.insertDept(dept);

        return code == 0? Result.error("添加部门失败") : Result.success();
    }
}
```

简化后:

```java
@Slf4j
@RestController
@RequestMapping("/depts")
public class DeptController {

    @Autowired
    private DeptService deptService;

    @GetMapping
    public Result list() {
        log.info("查询全部部门数据");

        // 调用service层对象方法处理请求
        List<Dept> deptList = deptService.list();

        return Result.success(deptList);
    }

    @DeleteMapping("/{id}")
    public Result deleteDept(@PathVariable Integer id) {
        log.info("删除id为 " + id + " 的部门");

        Integer code = deptService.deleteDept(id);

        return code == 0? Result.error("删除部门失败") : Result.success();
    }

    @PostMapping
    public Result insertDept(@RequestBody Dept dept) {
        log.info("添加名称为 " + dept.getName() + " 的部门");

        Integer code = deptService.insertDept(dept);

        return code == 0? Result.error("添加部门失败") : Result.success();
    }
}
```



# SpringBoot中接收请求参数

## 简单参数

### 使用HttpServletRequest

HttpServletRequest是javax包下servlet(java extension)的类

HttpServletRequest对象代表客户端的请求，当客户端通过HTTP协议访问服务器时，HTTP请求头中的所有信息都封装在这个对象中，开发人员通过这个对象的方法，可以获得客户这些信息。

request就是将请求文本封装而成的对象，所以通过request能获得请求文本中的所有内容，**请求头、请求方式、请求体、请求行**

```java
@RestController
public class RequestController {
    @RequestMapping("/demo")
    public String ParaDemo01(HttpServletRequest request) {
        // 获取请求参数
        String name = request.getParameter("name");
        String ageStr = request.getParameter("age");

        int age = Integer.parseInt(ageStr);
        System.out.println(name + " " + age);

        return "ok";
    }
}
```

通过HttpServletRequest获取的参数均是String类型的，需要手动进行类型转换，且显式的通过方法接收参数

### SpringBoot方法

SpringBoot对于简单参数的接收进行了封装，使得在编写响应的方法时可以更简便

```java
@RequestMapping("/demospring")
public String ParaDemo02(String name, Integer age) {
    System.out.println(name + " " + age);
    return "ok";
}
```

通过这种方式可以省去**类型转换**和显式接收的代码，但需要**保证前端传来的参数名称必须和方法接收的形参名一致**，如此参数名不一致，**接收的值会是形参列表指定类型的零值**

该方式也封装了**不同的请求方式**，即对于不同的请求方式(如GET/POST)均可以通过这一个方法拿到参数

#### @RequestParam注解

为了防止参数不一致的情况发送，SpringBoot提供了`@RequestParam`注解，用来完成接收参数和方法形参的映射

```java
@RequestMapping("/demospring")
public String ParaDemo02(@RequestParam(name = "name") String userName, Integer age) {
    System.out.println(userName + " " + age);
    return "ok";
}
```

对于`@RequestParam`注解有一个参数`required`其默认值是true，即默认是要求客户端、前端必须要传入此参数，否则会返回**400**状态码，若要求改参数非必须传入，可以将其值设置为false

```java
@RequestMapping("/demospring")
public String ParaDemo02(@RequestParam(name = "name", required = false) String userName, Integer age) {
    System.out.println(userName + " " + age);
    return "ok";
}
```

该注解还可以帮助高效的设置默认值，比如在分页查询这样的场景:



## 实体(类)参数

实体参数即在如参数量过大的情景下，我们可以创建一个参数类去接收参数，比如:

```java
public class User {
    private String name;
    private Integer age;
}
```

```java
@RequestMapping("/demospring")
public String ParaDemo03(User user) {
    System.out.println(user);
    return "ok";
}
```

一般可以将实体类创建在和controller同级的目录entity下

对于简单实体，即保证传入参数是某个实体的基本数据类型的属性，同样**需要保证传入的参数名称和实体类中的属性名的名称是一样的**

对于复杂实体，则前端、客户端需要在传入参数时指明具体实体类下的属性，比如在`User`中，加入一个`Address`类型的属性:

```java
public class Address {
    private String province;
    public String city;

    public String getProvince() {
        return province;
    }

    public void setProvince(String province) {
        this.province = province;
    }

    @Override
    public String toString() {
        return "Address{" +
                "province='" + province + '\'' +
                ", city='" + city + '\'' +
                '}';
    }

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }
}

```

```java
public class User {
    private String name;
    private Integer age;

    private Address address;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Address getAddress() {
        return address;
    }

    public void setAddress(Address address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", address=" + address +
                '}';
    }
}
```

则请求参数(GET请求)应为:

```http
http://localhost:8080/entity?name=ryan&age=18&address.province=北京&address.city=北京
```

## 数组集合参数

在如勾选框这样的场景，前端可能会传入一个数组，比如:

```http
http://localhost:8080/array?hobby=java&hobby=golang&hobby=mysql
```

在服务端可以直接将接收到的参数放入一个数组中

### 数组接收

SpringBoot封装好了数组类型的接收，类似接收简单参数的方式，我们直接在形参列表定义好数组的类型即可

```java
@RequestMapping("/array")
public String ParaDemo04(String[] hobby) {
    System.out.println(Arrays.toString(hobby));
    return "ok~";
}
```

需要注意的是也需要**保证形参名和传入的参数名称一致**

### 集合接收

前端传入的数据容器类型默认是数组，因此如果希望直接用集合去接收多个数据，**可以使用`@RequestParam`注解绑定**:

```java
@RequestMapping("/collection")
public String ParaDemo04(@RequestParam List<String> hobby) {
    System.out.println(hobby);
    return "ok~";
}
```

## 日期参数

接收参数时仍需借助注解完成接收，通过`@DateTimeFormat`注解，可以在其提供的`pattern`参数中设定接受的日期的格式

```java
@RequestMapping("/dateparam")
public String ParaDemo05(@DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss") LocalDateTime time) {
    System.out.println(time);
    return "copy that";
}
```

传入的参数:

```http
http://localhost:8080/dateparam?time=2024-01-01 10:11:12
```

输出:

```shell
2024-01-01T10:11:12
```

## JSON参数

由于JSON格式的数据往往承载着较多数量的参数，故一般会创建一个实体去接收它

在接收JSON格式的参数时，也要遵循Spring封装后的规则，即需要**保证JSON数据的键名和形参主体的属性名一致**，同时**需要用`@RequestBody`注解修饰**

```java
@RequestMapping("/json")
public String ParaDemo06(@RequestBody User user) {
    System.out.println(user);
    return "copy that";
}
```

传入的参数:

```json
{
    "name": "ryan",
    "age": 18,
    "address": {
        "province": "北京",
        "city": "北京"
    }
}
```

输出:

```shell
User{name='ryan', age=18, address=Address{province='北京', city='北京'}}
```

## 路径参数

路径参数是指参数需要在URL中获取，且不同于GET请求有`?`标识

此时，请求的路径是**动态的**，因此不能再使用传统的在`@RequestMapping()`注解中写死URL的方法

可以通过在URL中将会动态变化的参数的位置用`{}`包裹，并在处理的方法的形参列表中用`@PathVariable`注解绑定该动态变化的参数:

```java
@RequestMapping("/path/{id}")
public String ParaDemo07(@PathVariable Integer id) {
    System.out.println(id);
    return "copy that";
}
```

传入的参数:

```http
http://localhost:8080/path/10
```

如果需要传入的路径参数有多个，用`/`分割即可

```java
@RequestMapping("/path/{id}/{name}")
public String ParaDemo07(@PathVariable Integer id, @PathVariable String name) {
    System.out.println(id);
    System.out.println(name);
    return "copy that";
}
```

# 日期格式化

对于实体类的日期属性，当从数据库中查询到数据后，返回给前端的是一个数组对象，由此可能会造成前端的显示不符合日期格式

解决方式1: 用注解**@JsonFormat**修饰时间属性，并指定日期格式

```java
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
private LocalDateTime createTime;
```

解决方式2: 扩展SpringMVC消息转换器

```java
// 通过重写父类WebMvcConfigurationSupport的方法，扩展Spring MVC的消息转换器
@Override
protected void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
    log.info("扩展消息转换器...");
    // 创建消息转换器
    MappingJackson2CborHttpMessageConverter mappingJackson2CborHttpMessageConverter = new MappingJackson2CborHttpMessageConverter();
    // 为消息转换器设置一个对象转换器，该对象转换器的能力是将java对象和JSON之间做序列化和反序列化
    mappingJackson2CborHttpMessageConverter.setObjectMapper(new JacksonObjectMapper());
    // 将消息转换器加入到配置容器中
    // 配置容器中内置了8个消息转换器，如果不配置索引，则默认是在最后，是不会生效的，故将其配置到第一位优先使用
    converters.add(0, mappingJackson2CborHttpMessageConverter);
}
```

其中，关于日期格式化的操作定义在自定义的对象转换器JacksonObjectMapper中:

```java
public class JacksonObjectMapper extends ObjectMapper {

    public static final String DEFAULT_DATE_FORMAT = "yyyy-MM-dd";
    //public static final String DEFAULT_DATE_TIME_FORMAT = "yyyy-MM-dd HH:mm:ss";
    public static final String DEFAULT_DATE_TIME_FORMAT = "yyyy-MM-dd HH:mm";
    public static final String DEFAULT_TIME_FORMAT = "HH:mm:ss";

    public JacksonObjectMapper() {
        super();
        //收到未知属性时不报异常
        this.configure(FAIL_ON_UNKNOWN_PROPERTIES, false);

        //反序列化时，属性不存在的兼容处理
        this.getDeserializationConfig().withoutFeatures(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);

        SimpleModule simpleModule = new SimpleModule()
                .addDeserializer(LocalDateTime.class, new LocalDateTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT)))
                .addDeserializer(LocalDate.class, new LocalDateDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT)))
                .addDeserializer(LocalTime.class, new LocalTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT)))
                .addSerializer(LocalDateTime.class, new LocalDateTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT)))
                .addSerializer(LocalDate.class, new LocalDateSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT)))
                .addSerializer(LocalTime.class, new LocalTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT)));

        //注册功能模块 例如，可以添加自定义序列化器和反序列化器
        this.registerModule(simpleModule);
    }
}
```

# SpringBoot中响应请求

SpringBoot封装的响应请求的功能核心在`@ResponseBody`中，其一般修饰在Cotroller的方法或类上，作用是:

- 如果返回值是基本数据类型，将方法的返回值直接响应到游览器上，比如:

  ```java
  @RequestMapping("/hello")
  public String hello() {
      System.out.println("hello has been visited");
      return "hello~";
  }
  ```

- 如果返回值是实体对象/集合，**则自动会转换为JSON格式并响应**，比如:

  ```java
  @RequestMapping("/getAddr")
  public Address getAddr() {
      Address address = new Address();
      address.setProvince("北京");
      address.setCity("北京");
  
      return address;
  }
  ```

  ```java
  @RequestMapping("/listAddr")
  public List<Address> listAddr() {
      List<Address> list = new ArrayList<>();
  
      Address address01 = new Address();
      address01.setProvince("广东");
      address01.setCity("深圳");
      list.add(address01);
  
      Address address02 = new Address();
      address02.setProvince("浙江");
      address02.setCity("杭州");
      list.add(address02);
  
      return list;
  }
  ```

`@RestController` = `@Controller` + `@ResponseBody`，因此一般使用`RestController`去修饰Controller即可

一般的响应格式是: code + msg + data

# IOC(控制反转)&DI(依赖注入)

## Web应用常遵循的三层架构

- controller: 控制层，接收前端、客户端的请求以及请求参数，并在将参数经过业务逻辑处理后响应
- service: 业务逻辑层，实现具体的业务逻辑
- dao: 数据访问层(data access object)，也称为持久层，实现与数据库的交互操作

且一般遵循的是**面向接口编程**的范式，以实现代码的解耦等，同时也一般遵循**高内聚低耦合**的设计原则

比如:

controller层:

```java
@RestController
public class EmpController {
    private EmpService empService = new EmpServiceImp();

    @RequestMapping("/listEmp")
    public Result list() {
        List<Emp> empList = empService.listEmp();
        return Result.success(empList);
    }
}
```

service层:

```java
public class EmpServiceImp implements EmpService {
    private EmpDao empDao = new EmpDaoImp();
    @Override
    public List<Emp> listEmp() {
        List<Emp> empList = empDao.listEmp();

        // stream()的功能类似于iterator，但只提供一次遍历，是JDK8提供了方便对于集合进行计算操作的功能
        empList.stream().forEach(emp -> {
            String gender = emp.getGender();
            if ("1".equals(gender)) {
                emp.setGender("男");
            } else {
                emp.setGender("女");
            }

            String job = emp.getJob();
            if ("1".equals(job)) {
                emp.setGender("讲师");
            } else if ("2".equals(job)){
                emp.setGender("班主任");
            } else {
                emp.setGender("就业指导");
            }
        });

        return empList;
    }
}
```

dao层:

```java
public class EmpDaoImp implements EmpDao  {
    @Override
    public List<Emp> listEmp() {
        String file = Objects.requireNonNull(this.getClass().getClassLoader().getResource("emp.xml")).getFile();
        System.out.println(file);
        return XmlParserUtils.parse(file, Emp.class);
    }
}
```

## IOC

IOC(Inversion Of Control)，是使**对象的创建控制权**由程序自身**转移到外部(容器)的思想**，实现可以**动态的**运行的依懒层中不同的依赖类的功能，由此可用于模块间的解耦，提升代码的维护便捷性

比如，上述的例子的service层中创建了一个dao层依懒类的对象，如果希望依懒的时dao层中的另一个对象，就需要修改service层的代码，这就是一种耦合

如果希望将一个类交给IOC容器管理成为其中的一个bean，则用`@Component`注解修饰它

```java
@Component // 将当前类交给spring IOC容器管理，成为IOC容器中的一个bean
public class EmpServiceImp implements EmpService {
    @Autowired // 运行时ICO容器会为其修饰的属性提供对应类型的bean对象并赋值给属性
    private EmpDao empDao;
    @Override
    public List<Emp> listEmp() {
        List<Emp> empList = empDao.listEmp();

        // stream()的功能类似于iterator，但只提供一次遍历，是JDK8提供了方便对于集合进行计算操作的功能
        empList.stream().forEach(emp -> {
            String gender = emp.getGender();
            if ("1".equals(gender)) {
                emp.setGender("男");
            } else {
                emp.setGender("女");
            }

            String job = emp.getJob();
            if ("1".equals(job)) {
                emp.setGender("讲师");
            } else if ("2".equals(job)){
                emp.setGender("班主任");
            } else {
                emp.setGender("就业指导");
            }
        });

        return empList;
    }
}
```

由此，如果我们希望更新某一层中的依赖类，**只需用`@componet`修饰更新后的依赖类即可完成和上层属性的绑定**，不用修改上层的代码，完成层级之间的解耦

### @component的衍生注解

根据主流的controller-service-dao层架构，@component有跟其相关的三个衍生注解，能在其对应的层中实现和@component同样的功能，且能提高代码的可读性:

| 注解          | 功能说明                                                     |
| :------------ | :----------------------------------------------------------- |
| `@Component`  | 通用的注解，用于标识一个类为组件类。被 `@Component` 注解标记的类将被自动扫描并注册为Bean对象，可用于自动装配和依赖注入。 |
| `@Repository` | 基于 `@Component` 注解的衍生注解，用于标识一个类为数据访问组件（例如 DAO）。通常在持久层使用，提供对数据库的访问。 |
| `@Service`    | 基于 `@Component` 注解的衍生注解，用于标识一个类为业务逻辑组件（例如 Service）。通常在服务层使用，处理业务逻辑、事务管理等。 |
| `@Controller` | 基于 `@Component` 注解的衍生注解，用于标识一个类为控制器组件（例如 MVC Controller）。通常在表现层使用，接收用户请求、处理请求并返回响应。 |

### Bean组件扫描

需要注意的是，声明为bean对象的注解必须要**被组件扫描注解@ComponentScan扫描到才能生效**，但由于启动类声明注解`@SpringBootApplication`中已经包含了`@ComponentScan`注解，其作用范围是**启动类所在包及其子包**，故一般不用显式注明该注解，若不再默认的作用范围内，可以:

- 在启动类上增加注解`@ComponentScan`，并在其中写入要扫描的包名
- 把包放回到默认作用范围内，这也是默认的架构规范
- 值得注意的是，如果使用了`@ComponentScan`，则默认的当前包及其子包不会生效，需要手动添加到`@ComponentScan`中

## DI

DI(Dependency Injection)，容器为应用程序提供运行时所依赖的资源，称之为依赖注入

在Spring中，通过`@Autowired`注解即可自动完成依懒组件的匹配和注入，会将其修饰的属性所依赖的类型的bean对象从容器中取出并赋值:

```java
@Component
public class EmpServiceImp implements EmpService {
    @Autowired // 运行时ICO容器会为其修饰的属性提供对应类型的bean对象并赋值给属性
    private EmpDao empDao;
    @Override
    public List<Emp> listEmp() {
        List<Emp> empList = empDao.listEmp();

        // stream()的功能类似于iterator，但只提供一次遍历，是JDK8提供了方便对于集合进行计算操作的功能
        empList.stream().forEach(emp -> {
            String gender = emp.getGender();
            if ("1".equals(gender)) {
                emp.setGender("男");
            } else {
                emp.setGender("女");
            }

            String job = emp.getJob();
            if ("1".equals(job)) {
                emp.setGender("讲师");
            } else if ("2".equals(job)){
                emp.setGender("班主任");
            } else {
                emp.setGender("就业指导");
            }
        });

        return empList;
    }
}
```

### 依懒匹配机制

在容器中取出bean对象的默认匹配机制是**类型相同则匹配**，且会在**编译层面上支持这一匹配机制**，如果在IOC容器中对于同一类型有多个bean，则不会通过编译，因此该匹配机制确保结果一定是一对一的

#### @Primary注解

使用`@Primary`可以使得spring在编译层面上支持在IOC容器中有多个同一类型的bean，**被其修饰的bean会是在匹配时被选中的bean**

#### @Qualifier注解

使用`@Qualifier`注解可以通过bean对象的value值在上层指定其被修饰的属性所匹配的bean

```java
@RestController
public class EmpController {
    @Qualifier("empServiceImp")
    @Autowired
    private EmpService empService;

    @RequestMapping("/listEmp")
    public Result list() {
        List<Emp> empList = empService.listEmp();
        return Result.success(empList);
    }
}

```

#### @Resource注解

如果采用了`@Autowared`注解即在语言上表面了交给spring去自动匹配对应的bean对象，因此如果在加上手动匹配的`@Qualifier`会显得有些语义不搭和冗余，如果已经确定希望手动指定注解，推荐使用`@Resource`注解按照名称进行注入

```java
@RestController
public class EmpController {
    @Resource(name = "empServiceImp")
    private EmpService empService;

    @RequestMapping("/listEmp")
    public Result list() {
        List<Emp> empList = empService.listEmp();
        return Result.success(empList);
    }
}
```

但`@Resource`并不是spring提供的，而是javax，**其匹配根据的是名称，而spring中根据的是类型**

## Bean对象

IOC容器中创建、管理的对象，称之为bean

bean对象有一个属性`value`，含义是其名称，应用在bean对象匹配机制中，默认是**类名的首字母小写形式**，也可以在注解声明时修改，如:

```java
@Repository(value = "daoImp") // 将当前类交给spring IOC容器管理，成为IOC容器中的一个bean
public class EmpDaoImp implements EmpDao  {
    @Override
    public List<Emp> listEmp() {
        String file = Objects.requireNonNull(this.getClass().getClassLoader().getResource("emp.xml")).getFile();
        System.out.println(file);
        return XmlParserUtils.parse(file, Emp.class);
    }
}
```

### Bean管理

再默认情况下，Spring项目启动时就会创建好IOC容器(也称为Spring容器)，其会把bean都创建好放到IOC容器中，如果希望手动获取这些bean，可以通过IOC容器提供的以下方式:

1. 根据name获取bean

   ```java
   Object getBean(String name)
   ```

2. 根据类型获取bean

   ```java
   <T> T getBean(Class<T> requiredType)
   ```

3. 根据name和类型获取bean(带类型转换)

   ```java
   <T> T getBean(String name, Class<T> requiredType)
   ```

以上三个方法都是spring提供的`ApplicationContext`类的方法，当需要通过上述方法手动获取Bean对象时，需先创建一个`ApplicationContext`的对象，并完成注入

```java
@Autowired
private ApplicationContext applicationContext; // IOC容器对象
```

三种获取方式:

```java
public void getBeanPractice() {
    // 根据名称获取
    DeptController deptController = (DeptController) applicationContext.getBean("deptController");

    // 根据类型获取
    DeptController deptController1 = applicationContext.getBean(DeptController.class);

    // 根据名称及类型获取
    DeptController deptController2 = applicationContext.getBean("deptController", DeptController.class);
}
```

### Bean的作用域

通过注解`@Scope`可以设置组件的作用域，以下是五种作用域值

| 作用域         | 说明                                                         |
| :------------- | :----------------------------------------------------------- |
| singleton      | 单例模式，每个 Spring 容器同名称的bean只存在一个实例。是默认的作用域。 |
| prototype      | 原型模式，每次请求时都会创建一个新的 bean 实例。             |
| request        | 在每个 HTTP 请求中创建一个新的 bean 实例。仅适用于 Web 应用程序。 |
| session        | 在每个 HTTP 会话中创建一个新的 bean 实例。仅适用于 Web 应用程序。 |
| global session | 在全局 HTTP 会话中创建一个新的 bean 实例。仅适用于 Web 应用程序。 |

在**单例模式**中，即使通过不同的方式手动获取了bean对象，那么操作的也是**同一块内存地址**

以下是设置为非单例的实例:

```java
@Scope("prototype")
@Slf4j
@RestController
@RequestMapping("/depts")
public class DeptController {

    @Autowired
    private DeptService deptService;

    @GetMapping
    public Result list() {
        log.info("查询全部部门数据");

        // 调用service层对象方法处理请求
        List<Dept> deptList = deptService.list();

        return Result.success(deptList);
    }
}
```

在非单例模式下，每次获取bean对象都是一个新对象

在实际开发中，一般的bean都是单例的

### 延迟初始化

bean对象的初始化(即将bean放入到IOC容器中)默认是在项目启动即容器启动时就会完成的，如果希望延迟初始化，可以在希望延迟初始化的组件上加上`@Lazy`注解，由此可以使得在执行bean对象被调用的代码时才会完成bean的初始化

```java
@Lazy
@Slf4j
@RestController
@RequestMapping("/depts")
public class DeptController {

    @Autowired
    private DeptService deptService;

    @GetMapping
    public Result list() {
        log.info("查询全部部门数据");

        // 调用service层对象方法处理请求
        List<Dept> deptList = deptService.list();

        return Result.success(deptList);
    }
}
```

### 第三方bean

如果希望将一个第三方的依懒(jar包)中的某个类(组件)也交给IOC容器管理，使其成为一个bean对象，此时无法使用类似`@Component`这样的本地注解去标识bean对象，可以使用`@Bean`注解来完成这样的需求

1. 可以将引入第三方Bean的方法写在启动类中，比如有一个第三方jar包中的类SAXReader:

   ```java
   // 声明第三方bean
   @Bean // 将当前方法的返回值对象交给IOC容器管理，成为一个bean
   public SAXReader saxReader() {
       return new SAXReader();
   }
   ```

   但这是不被推荐的，因为这样有损启动类的纯粹性

2. 创建一个配置类，用于管理第三方bean对象，这是推荐的方式

   ```java
   @Configuration
   public class CommonConfig {
   
       // 声明第三方bean
       @Bean // 将当前方法的返回值对象交给IOC容器管理，成为一个bean
       public SAXReader saxReader() {
           return new SAXReader();
       }
   }
   ```

对于第三方bean的名称可以通过@Bean的name或者value属性来设定，**第三方bean的默认名称是方法名**

如果第三方的bean需要进行依懒注入，如果是本地的组件，则直接在方法中形参列表加入需要注入的依赖即可，spring会自动完成装配操作

```java
@Configuration
public class CommonConfig {

    // 声明第三方bean
    @Bean // 将当前方法的返回值对象交给IOC容器管理，成为一个bean
    public SAXReader saxReader(DeptService deptService) {
        System.out.println(deptService);
        return new SAXReader();
    }
}
```

# 文件上传

是指将本地图片、视频、音频等文件上传到服务器，供其他用户游览或下载的过程 

spring web提供了MultipartFile类用于文件上传与存储的操作

## 本地存储

```java
@Slf4j
@RestController
public class UploadController {
    @PostMapping("/upload")
    public Result upload(MultipartFile image) throws IOException {
        // 获取文件名
        String fileName = image.getOriginalFilename();
        // 将上传的文件转存到服务器中
        image.transferTo(new File("/Users/steaksunflower/Java/SpringBootProjectDemo/tlias/tlias/src/main/java/com/tlias" + fileName));

        return Result.success();
    }
}
```

### uuid

长度为32位，有4个"-"作为分隔符，共5部分，保证唯一

如果不同用户上传了名称相同的文件，先上传的用户的文件会被后上传的文件覆盖掉，因此，应保证文件名唯一

可以使用uuid+文件扩展名来实现该需求

```java
@Slf4j
@RestController
public class UploadController {
    @PostMapping("/upload")
    public Result upload(MultipartFile image) throws IOException {
        // 获取文件名
        String fileName = image.getOriginalFilename();
        // 获取文件拓展名
        int index = fileName.lastIndexOf(".");
        String extendName = fileName.substring(index);
        // 拼接uuid与扩展名
        String uuidName = UUID.randomUUID().toString() + extendName;
        // 将上传的文件转存到服务器中
        image.transferTo(new File("/Users/steaksunflower/Java/SpringBootProjectDemo/tlias/tlias/src/main/java/com/tlias" + fileName));

        return Result.success();
    }
}
```

## 文件大小限制

springboot中默认的文件大小限制是1M，如果需要修改该限制，更增加以下配置:

```properties
# 配置单个文件的最大上传大小为10MB
spring.servlet.multipart.max-file-size=10MB
# 配置单次请求最大上传大小(允许一次上传多文件，文件总和大小的限制)
spring.servlet.multipart.max-request-size=100MB
```

# 参数化配置

在项目中很多配置信息并不是固定的，比如数据库的配置信息，如果在项目中写死那么每次数据库配置更新时需要更改代码，造成耦合，因此应当将配置写在配置文件中，比如springboot自带的application.properties

springboot提供了注解`Value()`来进行装配注入，value中通过`${}`圈出配置信息的key即可

Application.properties文件:

```properties
aliyun.oss.endpoint=https://oss-cn-hangzhou.aliyuncs.com
aliyun.oss.accessKeyId=xxxx
aliyun.oss.accessKeySecret=xxxx
aliyun.oss.bucketName=web-tlias
```

工具类:

```java
@Component
public class AliOSSUtils {
    @Value("${aliyun.oss.endpoint}")
    private String endPoint;
    @Value("${aliyun.oss.accessKeyId}")
    private String accessKeyId;
    @Value("${aliyun.oss.accessKeySecret}")
    private String accessKeySecret;
    @Value("${aliyun.oss.bucketName}")
    private String bucketName
}
```

## @configurationProperties

使用`@configurationProperties`注解可以代替`@value`注解自动完成注入，但需要保证:

- 为需要注入配置的类**提供getter/setter方法**，如果使用了lombok，即加上`@data`注解即可
- 将需要注入配置的类交给IOC容器管理，**成为其中的一个Bean对象**，即需加上`@Componet`注解
- 指定好改类中各配置属性的key的**前缀**，比如在上述例子中的前缀是`aliyun.oss`

```java
@Data
@Component
@ConfigurationProperties(prefix = "aliyun.oss")
public class AliOSSUtils {
    private String endPoint;
    private String accessKeyId;
    private String accessKeySecret;
    private String bucketName;
}

```

### spring-boot-configuration-processor依赖

若增加以下依懒，则会在配置文件中写入配置信息时，提示被@configurationProperties修饰的类中的属性，仅仅是提示的功能

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
</dependency>
```

## yml

springboot除了提供.proerties文件的配置方式之外，还提供了使用.yml和.yaml文件配置的方式

其也是key-value的形式，不过具备更清晰的层级信息，目前springboot主流的配置文件也是yml

### 基本语法

- 大小写敏感
- value前必须有空格，作为分隔符
- 使用缩进来表述层级关系
- 同层的元素应在缩进上对齐

### 数据格式

- 对象/map集合

  ```yml
  user:
    name: ryan
    age: 18
  ```

- 数组/List/Set集合

  ```yml
  hobby:
    - java
    - golang
    - python
  ```

### 和.properties的对比

```properties
#驱动类名称
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
#数据库连接的url
spring.datasource.url=jdbc:mysql://localhost:3306/tlias
#连接数据库的用户名
spring.datasource.username=root
#连接数据库的密码
spring.datasource.password=sf20020107

#配置mybatis的日志, 指定输出到控制台
mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl

#开启mybatis的驼峰命名自动映射开关 a_column ------> aCloumn
mybatis.configuration.map-underscore-to-camel-case=true

# 配置单个文件的最大上传大小为10MB
spring.servlet.multipart.max-file-size=10MB
# 配置单次请求最大上传大小(允许一次上传多文件，文件总和大小的限制)
spring.servlet.multipart.max-request-size=100MB
```

Yml:

```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/tlias
    username: root
    password: sf20020107
  servlet:
    multipart:
      max-file-size: 10MB
      max-request-size: 100MB

mybatis:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    map-underscore-to-camel-case: true
```

# 登录校验

登录成功则标志在后续请求中都会附加一个用于通过权限校验的**信息标记**，这同时也要求所有的To c接口都应有**拦截请求**并校验的操作

## 会话

**一个客户端**与服务器进行**一次或多次数据交互的过程**称之为一次会话，结束的标志是有任意一方断开了连接

由此衍生出了一种维护游览器状态的方法，即**会话跟踪**，服务器应**识别**多个请求是否来自于同一游览器/客户端，以便在同一客户端的多次请求中能够**共享数据**

常见的会话跟踪实现有三种:

- **客户端**: cookie，后续请求后会一直携带，是HTTP所原生支持的
  - 请求头cookie，是在cookie被设置后一直携带着的字段
  - 响应头set-cookie，是服务端发送给客户端之后所携带的cookie的值
- **服务端**: session
- **令牌**

### 跨域

url的三要素是**协议、域名、端口**，当请求的url与当前所处的url在这三者中有任何一个是不同的，则是一次跨域请求

如今大部分项目都是前后端分离开发、分离部署在不同的服务器上，因此用户的一次请求会异步的请求前端服务器与后端服务器，由此造成了跨域，**因此跨域是不可避免的**

## cookie

```java
@Slf4j
@RestController
public class SessionController {
    // 设置cookie
    @GetMapping("/c1")
    public Result cookie1(HttpServletResponse response) {
        // 设置cookie值
        response.addCookie(new Cookie("login_name", "ryan"));
        return Result.success();
    }

    // 请求cookie
    @GetMapping("/c2")
    public Result cookie2(HttpServletRequest request) {
        // 获取所有cookie
        Cookie[] cookies = request.getCookies();
        for (Cookie cookie : cookies) {
            if (cookie.getName().equals("login_name")) {
                System.out.println("login_name: " + cookie.getValue());
            }
        }

        return Result.success();
    }
}
```

优点: Http原生支持，操作方便

缺点: 

- 移动端APP无法使用cookie，只能保存在游览器中
- cookie存储在客户端，是不安全的，且用户可以自行禁用cookie
- cookie不能进行**跨域**，不适用于如今的服务架构

## session

```java
@Slf4j
@RestController
public class SessionController {
    // 设置session
    @GetMapping("/s1")
    public Result session1(HttpSession response) {
        // 往HttpSession中存储数据
        response.setAttribute("login_name", "ryan");
        return Result.success();
    }

    // 从HttpSession中获取值
    @GetMapping("/s2")
    public Result session2(HttpServletRequest request) {
        // 获取所有session
       HttpSession session = request.getSession();

       Object loginName = session.getAttribute("login_name");

       return Result.success(loginName);
    }
}
```

优点: 存储在服务端，较为安全

缺点: 

- 如今的微服务架构下，每个服务都有多个副本，其中一个副本生成的session并不会在另一副本适用，即**不适用于当下的主流架构**

## 令牌

令牌是为实现在当下主流的微服务架构下的会话跟踪而应运而生的技术，其主要解决的问题是:

- 支持PC端、移动端
- 解决**集群环境**下的认证问题
- 不需要在服务端存储任何数据，减轻服务器端的存储压力

令牌的本质是一个字符串

### JWT

JWT(JSON Web Token)，定义了一种简洁的、自包含的格式，用于通信双方以**json数据**格式安全的传输信息，由于**数字签名**的存在，使得这些信息时可靠的

它由三部分组成:

1. Header: 标明令牌的类型、签名算法等，如: {"alg": "HS256", "type": "JWT"}
2. Payload(有效荷载): 携带一些自定义信息，默认信息等，如: {"id" : "1", "username": "Tom"}
3. Signature: 功能是**防止Token被篡改**，确保安全性，是由header、payload并加入指定密钥，通过指定签名算法计算而来，即数字签名

### 生成与解析JWT

引入依赖包:

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```

使用依赖包中的API构建JWT令牌:

```java
public void testGenJWT() {
    // 设置传递的参数
    HashMap<String, Object> claims = new HashMap<>();
    claims.put("id", 1);
    claims.put("name", "ryan");

    // 构建JWT令牌: 选取加密算法与传入加密信息、设置过期时间，ryan是令牌的名称
    String jwt = Jwts.builder().signWith(SignatureAlgorithm.HS256, "ryan").setClaims(claims)
            .setExpiration(new Date(System.currentTimeMillis() + 3600*1000))
            .compact();
    System.out.println(jwt);
  // eyJhbGciOiJIUzI1NiJ9.eyJuYW1lIjoicnlhbiIsImlkIjoxLCJleHAiOjE3MDc2ODQ4OTh9.HiM1b4eLhmgHzP_nC8RrJQcuN7AE6EyhQU0QM_dW0Lk

}
```

生成的内容，可以在**jwt.io**进行解码

解析JWT内容:

```java
public void testParseJWT(){
    Claims claims = Jwts.parser().setSigningKey("ryan").parseClaimsJws("eyJhbGciOiJIUzI1NiJ9.eyJuYW1lIjoicnlhbiIsImlkIjoxLCJleHAiOjE3MDc2ODQ4OTh9.HiM1b4eLhmgHzP_nC8RrJQcuN7AE6EyhQU0QM_dW0Lk\n")
            .getBody();
    System.out.println(claims);
}
```

# Filter

Filter过滤器与Servlet、Listener共成为JavaWeb三大组件，过滤器可以把对资源的请求拦截下来，从而实现一些通用的功能，比如登录校验、统一编码处理、敏感字符处理等

使用Filter的步骤是:

1. 定义Filter

   定义一个类，**实现Filter接口**，并重写其所有方法

2. 配置Filter

   Filter类加上**@WebFilter注解**，通过urlPatterns参数配置拦截资源的路径，并**在启动类加上@ServletComponentScan注解**开启Servelt组件支持

```java
package com.tlias.filter;

import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import java.io.IOException;

@WebFilter(urlPatterns = "/*")
public class DemoFilter implements Filter {

    // 在web服务启动时创建filter对象时自动调用，只会调用一次用于初始化
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        Filter.super.init(filterConfig);
    }

    // 过滤的逻辑
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        // 放行前的逻辑

        // 放行，允许请求访问其目标资源
        filterChain.doFilter(servletRequest, servletResponse);

        // 放行后的逻辑(请求已经访问完了目标资源)
        
    }

    // 再服务器关闭时销毁filter对象时调用，只会调用一次
    @Override
    public void destroy() {
        Filter.super.destroy();
    }
}

```

```java
@ServletComponentScan // 开启对servlet组件的支持
@SpringBootApplication
public class TliasApplication {

    public static void main(String[] args) {
        SpringApplication.run(TliasApplication.class, args);
    }
}
```

## 过滤路径

一般有三种方式:

- 过滤具体路径

  如/login，则只有访问/login的请求会被标识该urlPatterns值的filter拦截

- 目录过滤

  如/emps/*，则访问/emps目录下的所有资源都会被拦截

- 过滤所有

  /*

## 过滤器链

在web应用中，可以**配置多个filter**，由此形成过滤器链，即有多层filter

根据注解配置的filter，**层级关系是由过滤器类的类名的首字母的自然顺序来确定的**，如A开头的类其访问顺序则在最前

## 用Filter实现登录校验

1. 获取请求的URL
2. 判断请求的URL是否是类似/login这样不需要令牌的资源，如果是，则放行
3. 获取请求头中的令牌
4. 判断令牌是否存在，如果不存在，跳转到登录界面或返回登录错误信息
5. 解析令牌，如果解析失败，返回错误信息
6. 放行

```java
@Slf4j
@WebFilter(urlPatterns = "/*")
public class LoginCheckFilter implements Filter {

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        // 强转
        HttpServletRequest req = (HttpServletRequest) servletRequest;
        HttpServletResponse resp = (HttpServletResponse) servletResponse;

        // 获取请求url
        String requestURI = req.getRequestURI();
        log.info("请求的url: " + requestURI);

        // 判断是否是登录操作
        if (requestURI.contains("/login")) {
            log.info("登录操作，放行");
            filterChain.doFilter(servletRequest, servletResponse);
            return;
        }

        // 获取token(token的具体名称需看接口问档)
        String jwt = req.getHeader("token");
        if (!StringUtils.hasLength(jwt)) {
            log.info("请求头token为空");
            Result error = Result.error("NOT_LOGIN");
            // 手动转换把对象转为JSON
            String nogLogin = JSONObject.toJSONString(error);
            // 写入响应
            resp.getWriter().write(nogLogin);
            return;
        }

        // 解析token
        try {
            JWTGenerator.parseJWT(jwt);
        } catch (Exception e) {
            // jwt解析失败
            e.printStackTrace();
            log.info("请求的令牌解析失败");
            
            Result error = Result.error("NOT_LOGIN");
            // 手动转换把对象转为JSON
            String nogLogin = JSONObject.toJSONString(error);
            // 写入响应
            resp.getWriter().write(nogLogin);
            return;
        }
        
        // 放行
        log.info("放行");
        filterChain.doFilter(servletRequest, servletResponse);
    }
}
```

# Interceptor

是一种动态拦截方法调用的机制，类似于过滤器。是**spring框架中**提供的用来动态**拦截controller方法**的执行

其能拦截请求，并在被指定的方法的调用前后去执行预先根据业务设定的代码

使用interceptor的步骤是:

1. 定义拦截器，实现HandlerInterceptor接口，并重写所有的方法

   ```java
   @Component
   public class LoginCheckInterceptor implements HandlerInterceptor {
       @Override // 指定方法运行前运行，返回true则放行，返回false则不放行
       public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
           return HandlerInterceptor.super.preHandle(request, response, handler);
       }
   
       @Override // 指定方法运行完后运行
       public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
           HandlerInterceptor.super.postHandle(request, response, handler, modelAndView);
       }
   
       @Override // 视图渲染完毕后运行
       public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
           HandlerInterceptor.super.afterCompletion(request, response, handler, ex);
       }
   }
   ```

2. 编写配置类，注册拦截器

   ```java
   @Configuration //配置类
   public class WebConfig implements WebMvcConfigurer {
       @Autowired
       private LoginCheckInterceptor loginCheckInterceptor;
       @Override
       public void addInterceptors(InterceptorRegistry registry) {
           // 注册拦截器，拦截所有资源
           registry.addInterceptor(loginCheckInterceptor).addPathPatterns("/**");
           WebMvcConfigurer.super.addInterceptors(registry);
       }
   }
   ```

## 拦截路径

配置拦截的资源路径主要可通过**addPathPatterns()**和**excludePathPatterns()**两个方法去设定

- **addPathPatterns**: 指定需要拦截的资源
- **excludePathPatterns**: 指定不需要拦截的资源

比如:

```java
  registry.addInterceptor(loginCheckInterceptor).addPathPatterns("/**").excludePathPatterns("/login");
```

对于路径的通配符，有以下规则:

- /*: 一级路径，能匹配的是/xxx，而不能匹配/xxx/1
- /**: 所有路径
- /xxx/*: /xxx下的以及路径
- /xxx/**: /xxx下的任意级路径

## 执行流程

![image-20240216160406394](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20240216160406394.png)

DispatcherServlet是前端控制器设计模式的实现，提供Spring Web MVC的集中访问点，而且负责职责的分派，而且与Spring IoC容器无缝集成，从而可以获得Spring的所有好处

## 和Fliter的区别

- 接口规范不同: Filter需实现Filter接口，而拦截器需实现HandlerInterceptor接口
- 拦截范围不同: 过滤器Filter会拦截所有的资源，而**interceptor只会拦截spring环境中的资源**

# 全局异常处理

在项目的运行过程中，不可避免的会遇到异常现象，在出现异常时，服务器返回给前端的响应是不符合规范或约定的

如果给每一个controller的方法都使用try-catch包裹，会使代码变的臃肿，冗余度高

对于异常的处理，在spring中可以通过**@RestControllerAdvice(@ControllerAdvice + @ResponseBody)**注解定义一个全局异常处理器类，并通过**@ExceptionHandler注解**修饰进行异常处理的方法

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(Exception.class)
    public Result ex(Exception ex) {
        // 输出异常堆栈信息
        ex.printStackTrace();
        return Result.error("an exception has occur");
    }
}
```

如果创建了全局异常处理，那么就可以在项目中任何出现意料之外的地方抛出自定义异常

# spring事务管理

spring中的事务是解决基于现实情况所提出的需求，比如在删除一个部门时，应同时将该部门下的所有员工在数据库中删除，这两个操作应包裹在一个事务中执行，以维护数据的完整性

再spring中可以通过**@Transactional注解**来注明和包裹一个事务，其可以修饰在service层的方法上，或在类上、接口上

如果修饰在类上，则会使**该类中的所有方法都会以事务的形式去执行**，如果修饰在接口上，则该接口的所有实现都会以事务的形式去执行

其本质是将被注解修饰的逻辑交给spring进行事务管理，在方法执行前开启事务，在成功执行完毕后提交事务，再出现异常后回滚事务

```java
@Override
@Transactional
public Integer deleteDept(Integer id) {
    // 根据id删除部门
    deptMapper.deleteDept(id);

    // 模拟异常，异常发生后会回滚
    int i = 1/ 0;

    // 根据部门id删除该部门下的员工
    empMapper.deleteByDeptId(id);

    return deptMapper.deleteDept(id) == 1 ? 1 : 0;
}
```

## rollbackFor

rollbackFor是@Transactional注解中的重要属性之一，用于**控制出现何种类型的异常时回滚事务**

在默认情况下，只有RuntimeException异常才会被回滚，可以通过指定rollbackFor属性来回滚所有的异常

```java
@Override
@Transactional(rollbackFor = Exception.class)
public Integer deleteDept(Integer id) {
    // 根据id删除部门
    deptMapper.deleteDept(id);

    // 模拟异常，异常发生后会回滚
    int i = 1/ 0;

    // 根据部门id删除该部门下的员工
    empMapper.deleteByDeptId(id);

    return deptMapper.deleteDept(id) == 1 ? 1 : 0;
}
```

## propagation

propagation是@Transactional注解中用于**控制事务传播行为**的属性

事务的传播行为是指，当一个事务方法在被另一个事务方法调用时，

| 属性值        | 含义                                                         |
| :------------ | :----------------------------------------------------------- |
| REQUIRED      | 如果当前存在事务，则加入当前事务；如果当前没有事务，则创建一个新事务。这是最常用的选项，**默认值**。 |
| SUPPORTS      | 如果当前存在事务，则加入当前事务；如果当前没有事务，则以非事务方式执行。 |
| MANDATORY     | 如果当前存在事务，则加入当前事务；如果当前没有事务，则抛出异常。 |
| REQUIRES_NEW  | 创建一个新事务，如果当前存在事务，则将当前事务挂起，创建新事务 |
| NOT_SUPPORTED | 以非事务方式执行操作，如果当前存在事务，则将当前事务挂起。   |
| NEVER         | 以非事务方式执行操作，如果当前存在事务，则抛出异常。         |
| NESTED        | 如果当前存在事务，则在嵌套事务内执行；如果当前没有事务，则创建一个新事务。嵌套事务是独立提交或回滚的子事务，它可以独立于父事务存在。 |

对于REQUIRES_NEW，其常见的应用场景是用finally包裹的语句，比如插入日志，即无论上文是否出现异常都应被执行

## 开启事务管理日志

再配置文件中加入:

```yml
logging:
  level:
    org.springframework.jdbc.support.JdbcTranscationManager: debug
```

开启后可以看到事务的提交、回滚信息

# AOP

AOP(Aspect Oriented Programming)，面向切面编程，可以理解为面向特定的方法编程，是一种编程范式，**其关注的是一些共性的功能，核心是抽取**

**动态代理是面向切面编程最主流的实现**，而SpringAOP是Spring框架的高阶技术，其目的是在管理bean对象的过程中，主要通过底层的动态代理机制，对特定的方法进行编程

AOP机制是为**方法**服务的，**通过它可以给一个方法赋能**，其可以有类似filter、interceptor在方法执行前、后去做额外的事的能力，比如可以通过AOP机制创建一个为各方法统计耗时的切面，**其最大的意义是使得代码的复用性提高**

其使用步骤是:

1. 引入AOP的依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-aop</artifactId>
   </dependency>
   ```

2. 编写切面类，需要交给IOC容器管理，即需要加上**@Component**注解，同时加上**@Aspect**注解来表明这是一个切面类，并通过**@Around注解**来修饰方法，以指明方法的作用范围

比如:

```java
@Component
@Aspect
@Slf4j
public class TimeAspect {
    // 指定作用的方法
    @Around("execution(* com.tlias.service.*.*(..))") // 切入点表达式，作用范围是service下所有的接口和方法
    public Object recordTime(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        // 记录执行时的时间点
        long begin = System.currentTimeMillis();

        // 调用原始方法，返回值即原始方法的返回值
        Object result = proceedingJoinPoint.proceed();

        // 记录执行后的时间点
        long end = System.currentTimeMillis();
        log.info("method " + proceedingJoinPoint.getSignature() + " cost {}ms", end - begin);

        return result;
    }
}
```

通过AOP，可以更加便捷的实现记录操作日志、记录方法的传入参数和输出、权限控制、事务管理(spring的事务管理的底层实现即是通过AOP)等等

其主要优点是:

- 代码无侵入即可对方法赋能
- 减少重复代码，降冗余
- 提高开发效率
- 维护较为方便

## 核心概念

1. **切面（Aspect）：** 切面是横切关注点的模块化单元。它是一个跨越多个对象的通用功能的组合，例如日志记录、性能监测、事务管理等。切面通过定义切入点和通知来描述它们的行为。**其在概念上等同于通知+切入点**
2. **切入点（Pointcut）：** 切入点定义了**在应用程序中哪些位置去执行切面逻辑**。它指定了在何处和何时执行切面的操作。切入点可以使用表达式或者模式匹配来选择连接点（Join Point）。
3. **连接点（Join Point）：** 连接点是在应用程序执行过程中可以插入切面逻辑的一个点。它可以是方法调用、方法执行、异常抛出等。切入点定义了连接点的集合。**所有的方法都可以是连接点**
4. **通知（Advice）：** 通知是切面在特定连接点执行的代码。它定义了在连接点的何时执行什么样的操作。常见的通知类型包括前置通知（Before）、后置通知（After）、返回通知（After-returning）、异常通知（After-throwing）和环绕通知（Around）。**即对切入点起作用的共性方法**
5. **目标对象（Target Object）：** 目标对象是应用程序中被一个或多个切面所通知的对象。**它是切面所赋能的目标**。

## 通知

### 通知类型

| 通知类型                     | 说明                                                         |
| :--------------------------- | :----------------------------------------------------------- |
| 前置通知（@Before）          | 在目标方法执行之前执行的通知。可以在方法执行前进行一些准备工作或参数校验等操作。 |
| 后置通知（@After）           | 在目标方法执行之后（无论是否发生异常）执行的通知。可以进行一些清理工作或资源释放等操作。 |
| 返回通知（@After-returning） | 在目标方法成功执行并返回结果后执行的通知。可以获取目标方法的返回值并进行相应处理。如果方法在执行过程中发生了异常，则通知不会被执行 |
| 异常通知（@After-throwing）  | 在目标方法抛出异常后执行的通知。可以捕获目标方法抛出的异常并进行相应的异常处理。 |
| 环绕通知（@Around）          | 包围目标方法的通知，在目标方法的前后都执行特定逻辑。可以控制目标方法的执行，包括修改输入参数、拦截方法的执行、修改返回结果等。 |

对于@Around注解，**需要通过`proceedingJoinPoint.proceed()`手动的执行目标方法，并将目标方法的返回值返回，即需要将通知的返回值类型设置为Object**

### 通知执行顺序

当有多个切面的切入点都匹配到了同一个目标方法，则在目标方法运行时，会有多个通知被运行，由此涉及到了通知执行的先后顺序

默认的执行顺序是与**类名的字母自然排序有关**，字母排名靠前的切面在目标方法前的通知方法时会先执行，而在目标方法后的通知方法会后执行

也可以手动的通过**@Order注解**来控制顺序，其规则是:

- 目标方法前的通知方法: 数字小的先执行
- 目标方法后的通知方法: 数字小的后执行

```java
@Component
@Aspect
@Slf4j
@Order(1)
public class TimeAspect {
    @Pointcut
    public void pc(){}

    // 指定作用的方法
    @Around("pc()") // 切入点表达式
    public Object recordTime(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        // 记录执行时的时间点
        long begin = System.currentTimeMillis();

        // 调用原始方法，返回值即原始方法的返回值
        Object result = proceedingJoinPoint.proceed();

        // 记录执行后的时间点
        long end = System.currentTimeMillis();
        log.info("method " + proceedingJoinPoint.getSignature() + " cost {}ms", end - begin);

        return result;
    }
}
```

## 切入点表达式

切入点表达式的作用是指定哪些方法会被通知赋能

通过**@Pointcut注解**切入点表达式也是可以被抽取的，以减少代码的冗余，并且被抽取出的表达式在其他切面也是可用的，只要不超过修饰其的访问修饰符的范围

```java
@Component
@Aspect
@Slf4j
public class TimeAspect {
    @Pointcut
    public void pc(){}

    // 指定作用的方法
    @Around("pc()") // 切入点表达式
    public Object recordTime(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        // 记录执行时的时间点
        long begin = System.currentTimeMillis();

        // 调用原始方法，返回值即原始方法的返回值
        Object result = proceedingJoinPoint.proceed();

        // 记录执行后的时间点
        long end = System.currentTimeMillis();
        log.info("method " + proceedingJoinPoint.getSignature() + " cost {}ms", end - begin);

        return result;
    }
}
```

切入点表达式常见的有两种匹配方法:

- execution(...): 根据方法的签名来匹配
- @annotation(...): 根据注解匹配

### exection

exection主要根据方法的返回值、包名、类名、方法名、方法参数等信息来匹配

```
exection(方法修饰符(可省略) 返回值 包名.类名(可省略) 方法名(方法参数) throws 异常(可省略))
```

其中，方法参数需写**全类名**，包名.类名不建议省略，以免造成因匹配的范围过大而使得通知被重复执行

比如:

```java
execution(public Integer com.tlias.service.impl.DeptServiceImpl.deleteDept(java.lang.Integer))
```

#### 通配符

exection有两种通配符:

- *: 单个独立的任意符号，能表达任意返回值、包名、类名、方法名、任意类型的一个参数，**也可以表达包、类、方法名的一部分**

  比如`execution(* com.*.service.*.update(*))`

  代表作用域是任意返回类型、com包下的任意包下的service下的任意类或接口下的形参列表只有一个参数的update方法

  如果是`update*(*)`，则是任意名称以update开头的方法都会被匹配到

- ..: 多个连续的任意符号，可以表达任意层级的包，或任意类型、任意个数的参数

  比如`execution(* com.*.service.*.update(..))`，则有任意形参列表的update方法即在前置限制条件下的所有update方法都会被匹配到

如果要匹配当前项目中所有的方法，可用`"execution(* *(..))"`

当想要匹配多个毫无共性的方法时，可在execution中通过`||`连接，比如`execution(* com.*.service.*.update(..) || * com.*.service.*.update(*))`，&&和！等逻辑运算符也可以使用

切入点的方法尽量去匹配接口，由此可以增强扩展性，同时建议方法名在命名时如果是同一类操作的方法应有相同的前缀或后缀

### @annotation

@annotation常用于匹配标识有特定注解的方法，类似设定一个标签，被标签修饰的方法都会在作用域中

比如，先创建一个自定义注解:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MyLog {
}
```

用该自定义注解去修饰希望作用的方法:

```java
@MyLog
@Override
public void insert(Emp emp) {
    emp.setCreateTime(LocalDateTime.now());
    emp.setUpdateTime(LocalDateTime.now());

    empMapper.insert(emp);
}
```

在通知中通过自定义的注解来指明该目标方法:

```java
@Component
@Aspect
@Slf4j
@Order(1)
public class TimeAspect {

    // 指定作用的方法
    @Around("@annotation(com.tlias.service.impl.EmpServiceImpl.MyLog)")// 切入点表达式
    public Object recordTime(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        // 记录执行时的时间点
        long begin = System.currentTimeMillis();

        // 调用原始方法，返回值即原始方法的返回值
        Object result = proceedingJoinPoint.proceed();

        // 记录执行后的时间点
        long end = System.currentTimeMillis();
        log.info("method " + proceedingJoinPoint.getSignature() + " cost {}ms", end - begin);

        return result;
    }
}
```

其拥有更灵活的匹配方式，希望加入到作用域的方法直接用自定义的注解修饰即可

exection和@annotation也可以联合使用:

```java
@Pointcut("execution(* com.sky.mapper.*.*(..)) && @annotation(com.sky.annotation.AutoFill)")
```



## 连接点

所有的方法都可以是连接点，再Spring中抽象了连接点到JoinPoint中，**通过它可以获得方法执行时的相关信息**，如类名、方法名、方法参数等

需要注意的是:

- 对@Around通知类型，获取连接点信息只能使用**ProceedingJoinPoint**
- 对于其他四种通知类型，获取连接点只能使用**JoinPoint**，它是ProceedingJoinPoint的父类型

```java
@Around("@annotation(com.tlias.service.impl.EmpServiceImpl.MyLog)")// 切入点表达式
public Object recordTime(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
    String className = proceedingJoinPoint.getTarget().getClass().getName(); // 获取目标类名
    Signature signature = proceedingJoinPoint.getSignature(); // 获取目标方法签名
    String methodName = proceedingJoinPoint.getSignature().getName(); // 获取目标方法名
    Object args = proceedingJoinPoint.getArgs(); // 获取目标方法运行参数
    Object res = proceedingJoinPoint.proceed(); // 执行原始方法，获取返回值，只在@Around时需要手动操作

    return res;
}
```

## 应用实例 - 公共字段自动填充

对于类似create_time、create_user、update_time、update_user等各表都有的公共字段，可以将填充这些信息的代码放到切面中，以提升代码的复用性

定义注解，用于划分通知作用范围

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface AutoFill {
    // 数据库操作类型 UPDATE INSERT
    OperationType value();
}
```

切面:

```java
@Aspect
@Component
@Slf4j
public class AutoFillAspect {
    // 切入点
    @Pointcut("execution(* com.sky.mapper.*.*(..)) && @annotation(com.sky.annotation.AutoFill)")
    public void autoFillPointCut(){}

    // 前置通知
    @Before("autoFillPointCut()")
    public void autoFill(JoinPoint joinPoint) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        log.info("进行公共字段的自动填充");
        // 获取方法签名对象
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        // 获取注解对象
        AutoFill autoFill = signature.getMethod().getAnnotation(AutoFill.class);
        // 获取操作类型
        OperationType value = autoFill.value();


        // 获取操作传入的参数
        Object[] args = joinPoint.getArgs();
        // 处理异常情况
        if (args == null || args.length == 0) {
            return;
        }

        Object entity = args[0];
        // 公共字段
        LocalDateTime now = LocalDateTime.now();
        Long currentId = BaseContext.getCurrentId();

        if (value == OperationType.INSERT) {
            Method setCreateTime = entity.getClass().getDeclaredMethod("setCreateTime", LocalDateTime.class);
            Method setCreateUser = entity.getClass().getDeclaredMethod("setCreateUser", Long.class);
            Method setUpdateTime = entity.getClass().getDeclaredMethod("setUpdateTime", LocalDateTime.class);
            Method setUpdateUser = entity.getClass().getDeclaredMethod("setUpdateUser", Long.class);

            // 反射赋值
            setCreateTime.invoke(entity, now);
            setCreateUser.invoke(entity, currentId);
            setUpdateTime.invoke(entity, now);
            setUpdateUser.invoke(entity, currentId);
        } else {
            Method setUpdateTime = entity.getClass().getDeclaredMethod("setUpdateTime", LocalDateTime.class);
            Method setUpdateUser = entity.getClass().getDeclaredMethod("setUpdateUser", Long.class);

            // 反射赋值
            setUpdateTime.invoke(entity, now);
            setUpdateUser.invoke(entity, currentId);
        }
    }
}
```

# SpringBoot配置

## 配置优先级

配置优先级是指当有多份不同类型的配置文件时，生效的优先级

springboot中支持三种格式的配置文件:

- application.properties(最高)
- application.yml(次高)
- application.yaml(最低)

主流的是application.yml

## 命令行参数&java系统属性配置

可以通过命令行去设置参数，比如:

```shell
java -Dserver.port=9000 -jar tlias-web-management -0.0.1-SNAPSHOT.jar --server.port=10010
```

其中:

- `-Dserver.port`: 是java系统属性，可以通过这种方法配置参数
- `--server.port`: 是命令行属性，也可以通过这种方式配置参数



# 起步依赖&自动配置

SpringBoot与SpringFrameWork的最大不同在于**起步依赖**和**自动配置(如bean的声明、配置)**，从而大大简化了繁琐的配置过程

## 起步依赖原理

springboot中的一个显著的简化工作是将原本较为繁杂的如AOP、springmvc、servlet的依赖等简化为一个web开发的起步依赖:`org.springframework.boot:spring-boot-starter-web:x.x.x`

**其主要原理是使用到了maven的依赖传递**，即如果A依赖了B依赖了C，那么在引入依赖A时会自动的引入B和C

## 自动配置原理

springboot的自动配置是指当spring容器启动后，一些未被手动表示的配置类、bean对象就自动的加入到了IOC容器中，从而不需要手动去声明，简化了配置操作

这些未被标识的bean往往是一些通用的工具，是springboot所自带的，比如google开发的用于将对象转换为JSON的工具包`Gson`，可以在需要使用其时直接注入

```java
@SpringBootTest
class TliasApplicationTests {

    @Autowired
    private Gson gson;

    public void testGson() {
        String json = gson.toJson(Result.success());
        System.out.println(json);
    }
}
```

### 第三方bean对象的导入

实现自动配置，可以通过:

1. 手动配置`@ComponentScan`的扫描范围

   将`@ComponentScan`注解加入到启动类上实现组件扫描，并指明要扫描的范围，对于第三方已经创建好的bean，如果没有被扫描到则不能完成注入

2. 使用`@Import`导入

   可以在启动类上使用`@Import`注解导入需要的第三方bean，则此时不必用`@Component`注解修饰第三方类

   - 导入普通类

     ```java
     @Import({TokenParser.class}) // 导入普通类，交给IOC容器管理
     @SpringBootApplication
     public class TliasApplication {
         public static void main(String[] args) {
             SpringApplication.run(TliasApplication.class, args);
         }
     }
     ```

   - 导入配置类，一个配置类中往往会通过`@Bean`注解而创建多个bean

     ```java
     @Import({HeaderConfig.class}) // 导入普通类，交给IOC容器管理
     @SpringBootApplication
     public class TliasApplication {
         public static void main(String[] args) {
             SpringApplication.run(TliasApplication.class, args);
         }
     }
     ```

   - 实现ImportSelector接口

     ```java
     public class MyImportSelector implements ImportSelector {
         @Override
         public String[] selectImports(AnnotationMetadata importingClassMetadata) {
             // 返回全类名
             return new String[]{"com.example.HeaderConfig"};
         }
     }
     ```

     可以自定义一个ImportSelector接口的实现类，并重写selectImports方法，该方法要求返回的是要导入的第三方bean的**全类名的数组集合**，通过这种方式，可以实现将要导入的bean写在文件中，并在实现类中读取该文件封装成字符串数组并返回，从而一次性实现多个第三方bean的配置

     此时在启动类中，导入实现类的类对象即可

     ```java
     @Import({MyImportSelector.class}) // 导入普通类，交给IOC容器管理
     @SpringBootApplication
     public class TliasApplication {
         public static void main(String[] args) {
             SpringApplication.run(TliasApplication.class, args);
         }
     }
     ```

3. 使用第三方依赖提供的注解

   可以把需要导入哪些对象的工作交给第三方，其常常会提供注解`@EnableXxx`，其往往会使用`@Import`作为元注解

   ```java
   import org.springframework.context.annotation.Import;
   
   import java.lang.annotation.ElementType;
   import java.lang.annotation.Retention;
   import java.lang.annotation.RetentionPolicy;
   import java.lang.annotation.Target;
   
   
   @Retention(RetentionPolicy.RUNTIME)
   @Target(ElementType.TYPE)
   @Import(MyImportSelector.class)
   
   public @interface EnableHeaderConfig {
   }
   ```

   此时，在启动类中只需要使用第三方提供的注解修饰即可

   ```java
   @EnableHeaderConfig
   @SpringBootApplication
   public class TliasApplication {
       public static void main(String[] args) {
           SpringApplication.run(TliasApplication.class, args);
       }
   }
   ```

   这也是spring所推荐的导入方式



# 工具包

## BeanUtils

BeanUtils提供了很多便捷的提效和去冗余的方法，比如`copyProperties(src, dst)`，可以两个类的字段数量不相同时但**字段名相同**时直接进行属性的拷贝，从而省去重复性编写setXxx的时间

常用于在service层调用mapper层接口时，需创造一个pojo对象即方法mybaitis操作，将controller接收的DTO对象中的字段值赋值到pojo对象中



# ThreadLocal

ThreadLocal并不是一个Thread，而是Thread的局部变量

ThreadLocal为每个线程提供一份单独的存储空间，具有线程隔离的效果，只有线程内才能获取到对应的值，线程外则不可访问

ThreadLocal **会为每个线程单独开辟一块只有该线程能操作的内存空间**。每个线程都有自己独立的 ThreadLocal 变量副本，该变量存储在线程的 Thread 对象中。当一个线程访问 ThreadLocal 变量时，实际上是在访问自己线程内部的副本，而不是访问其他线程的副本。

**ThreadLocal 内部使用 ThreadLocalMap 数据结构**，该结构以 ThreadLocal 对象作为键，以线程特定的值作为值，实现了线程与变量的绑定关系。每个线程在访问 ThreadLocal 变量时，会通过当前线程的 Thread 对象来获取该线程的 ThreadLocalMap，然后在 ThreadLocalMap 中查找相应的 ThreadLocal 变量副本。

由于每个线程都有自己的 ThreadLocal 变量副本，所以每个线程对 ThreadLocal 变量的操作都是独立的，互不干扰。这样可以在多线程环境中实现线程安全的数据隔离，避免了线程间的数据竞争和并发访问的问题。

需要注意的是，**使用 ThreadLocal 时需要注意内存泄漏的问题**。如果在使用完 ThreadLocal 后没有及时清理或移除对应的变量副本，会导致变量一直存在于内存中，无法被垃圾回收，从而造成内存泄漏。因此，**建议在不再使用 ThreadLocal 变量时调用 `remove()` 方法将其从当前线程中清除。**

| 方法                                                         | 含义                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| `void set(T value)`                                          | 设置当前线程的 ThreadLocal 变量的值为指定值。每个线程都有自己独立的 ThreadLocal 变量副本。 |
| `T get()`                                                    | 返回当前线程的 ThreadLocal 变量的值。如果当前线程没有设置该变量，则返回 null。 |
| `void remove()`                                              | 移除当前线程的 ThreadLocal 变量。这样做可以避免 ThreadLocal 变量的内存泄漏问题。 |
| `protected T initialValue()`                                 | 在首次调用 `get()` 方法时，如果 ThreadLocal 变量还没有设置值，则会调用该方法来设置初始值。可以通过继承 ThreadLocal 并重写该方法来定义自定义的初始值。 |
| `static <S> ThreadLocal<S> withInitial(Supplier<? extends S> supplier)` | 返回一个带有初始值供应商的新的 ThreadLocal 实例。初始值供应商是一个函数式接口，用于提供 ThreadLocal 变量的初始值。 |