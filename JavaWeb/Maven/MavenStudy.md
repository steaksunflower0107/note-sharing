Maven是Apache开源的用于快速管理和构建java项目的工具

# Maven的主要作用

- 依赖管理

  快捷的获取java项目运行所需要的依赖包(jar包)，通过maven也可以避免版本冲突问题，类似go mod的功能

- 统一的脚手架

  maven提供了标准、统一的项目结构

- 项目构建

  标准跨平台(Linux、Win、Mac)的自动化项目build方式

## 依赖管理

在ide中可以直接勾选`通过Maven构建项目`，则生成的目录下有`pro.xml`文件，在其中的`<dependence>`标签中，加入项目所需的依赖即可

```xml
    <dependencies>
        <dependency>
            <groupId>org.openjfx</groupId>
            <artifactId>javafx-controls</artifactId>
            <version>17.0.2</version>
        </dependency>
        <dependency>
            <groupId>org.openjfx</groupId>
            <artifactId>javafx-fxml</artifactId>
            <version>17.0.2</version>
        </dependency>

        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

## Maven架构


─ src
   ─ main
      ─ java
         └─ 包名
             └─ 源代码文件.java
      ─ resources
          ─ 配置文件
          ─ 静态资源
   ─ test
       ─ java
          ─ 包名
              ─ 测试文件.java
       ─ resources
           ─ 测试资源文件
─ pom.xml

## 标准的项目构建流程

maven提供了java项目从编译到发布的全流程快捷操作指令

ide中提供了完成指令操作入口

### 编译

指令: `complie`

Complie后，生成的文件会存放再当前目录的`target`目录中，其子目录`classes`存放的是.class即字节码文件

### 打包

指令: `package`

package后，生成的打包文件也会存放在`target`目录下

# Maven仓库

maven的核心是pom.xml文件，通过pom.xml文件中的信息，maven可以去不同的仓库获取需要的依赖包

![image-20231230152121697](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20231230152121697.png)

仓库用于存储资源，管理各种jar包

- 本地仓库: 本地的目录中存放maven的资源
- 中央仓库: 由Maven团队维护的全球唯一的仓库，https://repo1.maven.org/maven2，包含世界上几乎所有对外发布的jar包
- 远程仓库(私有): 由企业团队自己搭建的私有仓库

当在pom.xml中给maven指定一个依赖，maven在拉取的查找顺序类似go mod，会遵循本地仓库-> 中央仓库 -> 下载到本地仓库的顺序

如果搭建了私有的远程仓库，则查找顺序变为本地 -> 私服 -> 中央，查找到后都会下载到本地仓库中

# Maven安装

1. 下载tar包并解压: https://maven.apache.org/download.cgi

2. 设置环境

3. 设置环境变量

   ```shell
   sudo vim /etc/profile
   ```

   加入:

   ```shell
   export M2_HOME={maven包的路径}/apache-maven-x.x.x
   export PATH=$PATH:$M2_HOME/bin
   ```

   使更新生效:

   ```shell
   source /etc/profile
   ```

4. 验证

   ```shell
   mvn -version
   ```

   正常情况:

   ```shell
   Apache Maven 3.9.6 (bc0240f3c744dd6b6ec2920b3cd08dcc295161ae)
   Maven home: /Users/steaksunflower/Java/Maven/apache-maven-3.9.6
   Java version: 11.0.1, vendor: Oracle Corporation, runtime: /Library/Java/JavaVirtualMachines/jdk-11.0.1.jdk/Contents/Home
   Default locale: zh_CN_#Hans, platform encoding: UTF-8
   OS name: "mac os x", version: "10.16", arch: "x86_64", family: "mac"
   ```

# IDE集成Maven

全局配置:

关闭所有项目-> 自定义 -> 所有设置

1. 构建 -> 构建工具 -> Maven -> 配置主路径、setting.xml路径和本地仓库路径
2. Maven -> 运行程序 -> 选择对应的JDK
3. 编译器 -> 项目字节码版本 -> 选择2中JDK的版本

# Maven坐标

坐标是maven资源的唯一标识，maven会通过坐标确认要获取的资源

主要组成部分:

- groupId: 资源隶属的组织名称，通常是域名反写
- artifactId: 资源名称，通常是模块的名称
- version: 资源的版本号



# 依赖管理

## 依赖配置

通过`pom.xml`的`dependencies`中写入资源的坐标

```xml
<dependencies>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.2.3</version>
    </dependency>
</dependencies>
```

注意在**加载**后依懒项才会被应用

maven库: https://mvnrepository.com

## 依赖传递

maven中管理的资源具有依赖传递性，即如果我们指定的依赖的jar包依懒其他的jar包，在引入依懒的jar包(直接依赖)时也会引入依赖的jar包所依赖的jar包(间接依懒)

在ide的**图表功能**中，可以清晰的看到依懒包之间的依赖关系

### 移除依赖传递

如果引入的直接依赖的使用过程并不严格需要某个间接依赖，我们可以将间接依懒移除掉，通过`<exclusions>`标签

比如，如果`ch.qos.logback`依赖`junit`，需要移除它:

```xml
<dependencies>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.4.12</version>
        <exclusions>
            <exclusion>
                <artifactId>junit</artifactId>
                <groupId>junit</groupId>
            </exclusion>
        </exclusions>
    </dependency>
</dependencies>
```

## 依赖范围

项目通过Maven所获取的依懒的jar包默认可以以下范围中使用:

- main目录下
- test目录下
- 如果依懒的jar包参与了打包过程(package)，则打包后也可以使用

maven中可以通过`<scope>`去控制依赖生效的范围:

| `<scope>`值       | 在主程序中有效 | 在测试程序中有效 | 在打包（运行）中有效 |
| :---------------- | :------------- | :--------------- | :------------------- |
| `compile`(默认值) | 是             | 是               | 是                   |
| `provided`        | 是             | 是               | 否                   |
| `runtime`         | 否             | 是               | 是                   |
| `test`            | 否             | 是               | 否                   |
| `system`          | 是             | 是               | 否                   |
| `import`          | 否             | 否               | 否                   |

## 生命周期

maven的生命周期是对maven项目的流程规范性的统一

maven中有3套独立的生命周期

- **clean**: 清理工作，清理之前的项目构建所产生的jar包、.class文件等
- **defalut**: 完成maven的核心功能，如编译、测试、打包、安装、部署等
- **site**: 生成报告、发布站点

每一套生命周期中，又会有着很多细分后的阶段，在maven中可以指定运行某套生命周期中的某个阶段，但在和该阶段在同一套的生命周期中其之前的阶段都会被执行

执行某个阶段可以直接在ide中双击，或通过命令行的方式，比如`mvn clean`清理历史文件、`mvn install`将项目加入到本地仓库中

# 继承

当有多个工程使用了同一依赖包，可以把共性的依赖包抽取出来做成一个父工程，并让其他工程继承它，maven中的继承与java中类似，当继承了某个工程则可以直接使用其依赖的配置信息，且是**单继承**的

实现继承可以通过`<parent>..</parent>`标签指定父工程的坐标，如果父工程在本地，可以使用`<relativePath>`标签指定**父工程的pom文件的相对路径**，如果不配置，其默认是会在本地/远程仓库中寻找

实现继承:

1. 创建xxx-parent父模块，通过`<packaging>`标签设置打包方式为pom，对于打包方式分为:

   - jar: 普通模块打包，springboot项目基本都是jar包(内嵌tomcat运行)，可以通过java指令直接运行
   - war: 普通web程序打包，需要部署在外部的tomcat服务器中运行
   - pom: **父工程或聚合工程**，**该模块不写代码**，仅进行依赖管理

   ```xml
   <groupId>com.ryan</groupId>
   <artifactId>tlias-parent</artifactId>\
   <version>1.0-SNAPSHOT</version>
   <packaging>pom</packaging>
   ```

2. 在子工程中继父工程

   ```xml
   <parent>
       <groupId>com.ryan</groupId>
       <artifactId>tlias-parent</artifactId>\
       <version>1.0-SNAPSHOT</version>
   </parent>
   ```

3. 在父工程中加入共性的依赖，并删除子工程中的冗余依赖

如果子工程中配置了和父工程中不同版本的同一个以来，类似与重写，以子工程的为准

## 版本锁定

当多个子工程都依赖同一依赖时，可能会出现版本不统一的情况，此时可以通过在父工程中通过`<dependencyManagement>`标签来锁定某个依赖的版本，此时父工程中并不会下载并加入该依赖，只有在子工程中使用到才会加入，且在子工程中就不需要去限定依赖的`<version>`了

比如，锁定jwt依赖的版本号，父工程:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.9.1</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

子工程:

```
<dependencies>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt</artifactId>
    </dependency>
</dependencies>
```

`<dependencyManagement>`的作用仅是限定版本号，而`<dependencies>`是加入依赖包

### 自定义属性

在maven中，可以在`<properties>`标签中，以类似创建变量的方式创建自定义的属性，属性(标签)名是任意的，并在需要引用的地方使用`${属性名}`引用

```xml
<properties>
    <java.version>11</java.version>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <spring-boot.version>2.6.13</spring-boot.version>
<!--        自定义属性-->
    <lombok.version>1.18.24</lombok.version>   
    <jjwt.version>0.9.0</jjwt.version>
</properties>
```

在dependencies中引用:

```xml
<dependencies>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt</artifactId>
        <version>${jjwt.version}</version>
    </dependency>
</dependencies>
```

# 聚合

当分模块进行开发时，当想要部署项目，即开始打包项目时，由于模块之间存在的依赖关系，想要完成整个项目的打包就需要将每一个在依赖关系中的模块install到本地仓库中，这是一个较为繁琐的操作

maven中提供了聚合的功能，即将多个模块组织成一个整体，**同时进行项目的构建**

实现聚合，需增加一个**聚合工程**，其不具备业务功能，有且只有一个pom文件，也**常常即是父工程**

在父工程中通过标签`<modules>`来指明需要聚合的模块

```xml
<modules>
    <module>../tlias-pojo</module>
    <module>../tlias-web-management</module>
    <module>../tlias-utils</module>
</modules>
```

至此，在需要对项目整体进行操作时，比如打包，在父工程操作即可对整个项目进行操作

![image-20240218202755478](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20240218202755478.png)

# 私服

私服往往是公司内部的服务器，是一种特殊的远程仓库，用来**代理**外部的中央仓库，以方便不同开发不同模块的同学能够彼此同步、资源共享，比如A同学在自己的模块中需要用到B同学的开发的模块，B同学可以将该模块上传到公司的私服中去后，告知A同学下载到本地

一般上传的项目的两种命名:

1. RELEASE(发行版本): 功能趋于稳定，当前更新停止，一般存储在RELEASE仓库中
2. SNAPSHOT(快照版本): 功能不稳定，尚处于开发中的版本，一般存储在SNAPSHOT仓库中

不同的仓库有不同的用户名和米啊