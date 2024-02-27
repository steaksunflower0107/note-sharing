# SQL通用语法

- sql不区分大小写
- sql可以单行或多行书写
- 注释
  - 单行注释: `-- `或`#`  
  - 多行注释: `/*  */`

# SQL分类

| 类别                | 描述                                                         |
| :------------------ | :----------------------------------------------------------- |
| 数据定义语言（DDL） | 用于定义和管理数据库中的对象，如表、视图、索引等。DDL包括创建、修改和删除数据库对象的命令。 |
| 数据操作语言（DML） | 用于对数据库中的数据进行操作，如查询、插入、更新和删除数据。DML命令用于从表中检索、插入、修改和删除数据。 |
| 数据控制语言（DCL） | 用于控制数据库访问权限和安全性的语言。DCL命令用于授予或撤销用户的访问权限，以及管理数据库对象的安全性。 |
| 事务控制语言（TCL） | 用于管理数据库中的事务处理。TCL命令用于开始、提交或回滚事务，以及设置事务的隔离级别。 |

# DDL

## 数据库操作

以下的databases也可以被替换为schema

### 查询数据库

#### show databases;

查询所有的已创建的数据库

#### select database();

查询当前选定的数据库

### 使用数据库

#### use 数据库名;

切换到给定数据库

### 创建数据库

#### create database [if not exists] 数据库名;

在给定名称不存在的情况下，创建新数据库

加上`if not exists`，可以保证该sql的执行不保存

### 创建表

```mysql
create table 表名 (
	字段1 字段类型 [约束] [comment 字段1注释]
	...
) [comment 表注释]
```

比如:

```mysql
create table tb_user (
    id int primary key comment 'ID',
    username varchar(20) not null comment '用户名',
    name varchar(10) comment '名称',
    age int comment '年龄' default 18,
    gender char(1) comment '性别'
) comment '用户表'
```

### 约束

| 约束类型         | 含义                                                         |
| :--------------- | :----------------------------------------------------------- |
| `NOT NULL`       | 约束字段的值不能为空，即该字段必须包含非空值。               |
| `UNIQUE`         | 约束字段的值在整个表中必须唯一，不允许重复。                 |
| `PRIMARY KEY`    | 将一个或多个字段定义为主键，主键唯一标识表中的每一行数据，并且不允许为空。 |
| `FOREIGN KEY`    | 用于建立表与表之间的关系，定义了一个字段与另一个表的主键或唯一键之间的引用关系。 |
| `CHECK`          | 定义字段的取值范围或条件，确保字段的值满足指定的条件。       |
| `DEFAULT`        | 为字段指定默认值，如果插入数据时未提供该字段的值，则将使用默认值。 |
| `AUTO_INCREMENT` | 用于自动生成唯一的、递增的数值，常用于主键字段的定义。       |

### 删除

#### drop database [if exists] 数据库名

在给定名称存在的情况下，删除数据库

### 数据类型

#### 数值类型

| 类型                   | 描述                                                       |
| :--------------------- | :--------------------------------------------------------- |
| `INT`                  | 整数类型，用于存储整数值。(-2^31, 2^31-1)                  |
| `BIGINT`               | 长整数类型，用于存储较大范围的整数值。(-2^63, 2^63-1)      |
| `SMALLINT`             | 短整数类型，用于存储较小范围的整数值。(-32768, 32767)      |
| `TINYINT`              | 微整数类型，用于存储非常小的整数值。(-128,127)             |
| `FLOAT`                | 单精度浮点数类型，用于存储小数值。                         |
| `DOUBLE`               | 双精度浮点数类型，用于存储更高精度的小数值。               |
| `DECIMAL` 或 `NUMERIC` | 定点数类型，用于存储精确的小数值，可以指定精度和小数位数。 |
| `BIT`                  | 位类型，用于存储位值（0 或 1）。                           |

如果存储的是无符号的数据，在数值类型后加`unsigned`即可

对于浮点数的精度问题，需要在声明表时指定其总位数与小数位数，如`double(5,2)`

#### 字符串类型

| 类型         | 描述                     | 最大长度              | 占用空间               |
| :----------- | :----------------------- | :-------------------- | :--------------------- |
| `CHAR`       | 固定长度字符串           | 最多 255 bytes        | 实际长度 + 1 字节      |
| `VARCHAR`    | 可变长度字符串           | 最多 65535 bytes      | 实际长度 + 1 或 2 字节 |
| `TINYTEXT`   | 非常短的可变长度字符串   | 最多 255 bytes        | 实际长度 + 1 字节      |
| `TEXT`       | 短的可变长度字符串       | 最多 65535 bytes      | 实际长度 + 2 字节      |
| `MEDIUMTEXT` | 中等长度的可变长度字符串 | 最多 16777215 bytes   | 实际长度 + 3 字节      |
| `LONGTEXT`   | 长的可变长度字符串       | 最多 4294967295 bytes | 实际长度 + 4 字节      |

#### 日期类型

| 类型         | 描述           | 能表示的时间范围                           | 日期格式              |
| ------------ | -------------- | ------------------------------------------ | --------------------- |
| **DATE**     | 日期类型       | 1000-01-01 到 9999-12-31                   | `YYYY-MM-DD`          |
| TIME         | 时间类型       | '-838:59:59' 到 '838:59:59'                | `HH:mm:ss`            |
| **DATETIME** | 日期和时间类型 | 1000-01-01 00:00:00 到 9999-12-31 23:59:59 | `YYYY-MM-DD HH:mm:ss` |
| TIMESTAMP    | 时间戳类型     | 1970-01-01 00:00:01 到 2038-01-19 03:14:07 | `YYYY-MM-DD HH:mm:ss` |
| YEAR         | 年份类型       | 1901 到 2155                               | `YYYY`                |

## 表操作

### 查询表

#### show tables;

查询当前数据库的所有表

#### desc 表名;

查询表结构

#### show create table 表名;

查询建表的sql语句

### 修改表

#### alter table 表名 add 字段名 类型(长度) [comment 注释] [约束];

添加字段

#### alter table 表名 modify 字段名 新数据类型(长度)l

修改字段类型

#### alter table 表名 change 旧字段名 新字段名 类型(长度) [comment 注释] [约束];

修改字段名和字段类型

#### alter table 表名 drop column 字段名;

删除字段

#### rename table 表名 to 新表名;

修改表名

#### drop table [if exists] 表名;

删除表，删除表时表中的数据也全部会被删除

# DML

## 添加数据(insert)

### insert into 表名 (字段名1, 字段名2, ...) values (值1, 值2, ...);

指定字段添加数据(一条数据)

```mysql
insert into tb_emp (username, gender, create_time, update_time) values ('ryan', 1, now(), now());
```

**指定字段的顺序和插入的值的顺序需要一一对应**

### insert into 表名 values (值1, 值2, ...);

全部字段添加数据(一条数据)

```mysql
insert into tb_emp (id, username, password, gender, image, job, entry_time, create_time, update_time)
    values (null, 'sunflower', '123', 2, '1.jpg', 1, '2002-01-01', now(), now());
```

### insert into 表名 (字段名1, 字段名2) values (值1, 值2), (值1, 值2)

批量添加数据(指定字段) (多条数据)

```mysql
insert into tb_emp (username, gender, create_time, update_time) values
    ('curry', 1, now(), now()),
    ('steak', 1, now(), now());
```

### insert into 表名 values (值1, 值2, ...) (值1, 值2, ...);

批量添加数据(全部字段)(多条数据)

## 更新数据(update)

### update 表名 set 字段名1 = 值1, 字段名2 = 值2, ... [where 条件];

修改数据

**条件更新:**

```mysql
update tb_emp set username = 'james', update_time = now() where username = 'curry';
```

**全表更新:**

```mysql
update tb_emp set entry_time = '2023-10-20', update_time = now();
```

## 删除数据(delete)

### delete from 表名 [where 条件]

删除数据

**条件删除:**

```mysql
delete from tb_emp where username = 'james';
```

**删除全表数据:**

```mysql
delete from tb_emp
```

# DQL

## 语法概览

```mysql
select 字段列表 from 表名列表 where 条件列表 group by 分组字段列表 having 分组后条件列表 order by 排序字段列表 limit 分页参数
```

## 基本查询

### select 字段1, 字段2, 字段3 from 表名;

查询多个字段

```mysql
select name, entrydate from tb_emp;
```

### select * from 表名;

查询表中所有数据

### select 字段1 [as 别名1], 字段2 [as 别名2] from 表名;

设置别名并查询多个字段

```mysql
select name as '姓名', entrydate as '入职日期' from tb_emp;
```

### select distinct 字段列表 from 表名;

在去除重复记录的情况下查询多个字段

```mysql
select distinct job from tb_emp;
```

## 条件查询(where)

### is null

判断某个值是否为空

```mysql
select * from tb_emp where job is [not] null;
```

### <>

`<>` == `!=`

### between ... and ...

在某个范围之间，是闭区间，类似`xx >= xxx and xx <= xxx`

```mysql
select * from tb_emp where entrydate between '2000-01-01' and '2010-01-01';
```

### in ()

把某个字段的值限制在某个集合之内，类似`xx = x and xx = xx and xx = xxx`

```mysql
select * from tb_emp where job in (2,3,4);
```

### like

模糊匹配，有两个占位符

- `_`: 匹配单个字符
- `%`: 匹配任意个字符

比如，在员工表中查询name为两个字的员工的数据:

```mysql
select * from tb_emp where name like '__';
```

在员工表中查询姓张的员工的数据:

```mysql
select * from tb_emp where name like '张%';
```

## 分组查询(group by)

```mysql
select 字段列表 from 表名 [where 条件] group by 分组字段名 [having 分组后的过滤条件]
```

select后能跟的元素只有

- **分组字段**
- **聚合函数**

### 聚合函数(count、max、min、avg、sum)

将某一列的数据作为一个整体，进行纵向计算

语法: `select 聚合函数(字段列表) from 表名`

| 聚合函数 | 描述                                   |
| -------- | -------------------------------------- |
| COUNT    | 计算结果集中行的数量（不包括NULL值）。 |
| MAX      | 返回结果集中某列的最大值。             |
| MIN      | 返回结果集中某列的最小值。             |
| AVG      | 计算结果集中某列的平均值。             |
| SUM      | 计算结果集中某列值的总和。             |

需要注意的是，**Count等聚合函数不会统计NULL值，所以用在非空的字段上才有意义**

```mysql
select max(entry_date) from tb_emp;
select min(entry_date) from tb_emp;
select count(id) from tb_emp;
```

对于Count，如果是单纯的想知道某张表中的数据量，可以直接使用`Count(常量)`或`Count(*)`获取

在统计数据量时，`Count(*)`的写法是被推荐的，因为MySQL对此在底层上做了优化处理，有更好的执行效率

```mysql
select count(1) from tb_emp;
select count(*) from tb_emp;
```

`avg`和`sum`的用法和`min`与`max`一致

### 按某个字段分组

比如，统计某张员工表中男性、女性各有多少人:

```mysql
select gender, count(*) from tb_emp group by gender;
```

### 对分组后字段的条件查询

比如，某张表按照职位进行分组，筛选出职位的员工数大于2的职位:

```mysql
select job, count(*) from tb_emp where entrydate <= '2015-01-01' group by job having count(*) > 2;
```

值得注意的是:

- 在进行了分组之后，查询的字段一般为聚合函数和分组字段，因为查询其他就丧失了分组的意义，在ide中执行也会报错
- 执行顺序: where > 聚合函数 > having

### having和where的区别

- **执行时机不同**: where是**分组前**的条件过滤，不满足where条件的数据不会参数分组，而having是**分组后**的条件过滤
- **可判断的条件不同**: having可以对聚合函数进行判断，而where不可以

## 排序查询(order by)

```mysql
... order by 字段1 排序方式1, 字段2 排序方式2...;
```

### 排序方式

- **ASC**: 升序(默认值)

  ```mysql
  select * from tb_emp order by entrydate;
  ```

- **DESC**: 降序

  ```mysql
  select * from tb_emp order by entrydate DESC;
  ```

### 多字段排序

在对多个字段进行排序时，字段列表中的字段，在靠前的字段排序生效后，对于排序后值相同的数据，靠后的字段才会生效

比如，按照入职日期降序排列，如果值相同，则按照更新时间升序排列:

```mysql
select * from tb_emp order by entrydate, update_time DESC;
```

## 分页查询(limit)

```mysql
... limit 起始索引, 查询记录数;
```

### 起始索引

-  起始索引的默认值是0

- 起始索引**所对应的行时不会被选中的**

- 起始索引(页码)公式:

  ```shell
  起始索引 = (页码 - 1) * 每页展示的数量
  ```



## 流程控制

### if表达式

```mysql
if(条件表达式, true取值, false取值);
```

可以应用在某个枚举值转换为对应的含义时，用于使得呈现的统计信息更加直观，比如在统计各性别数量时:

```mysql
select if(gender = 1, '男性员工', '女性员工') as '性别', count(*) from tb_emp group by gender;
```

### 多分支- case ... when ... then ... else ... end

if表达式仅适用于两分支的情况，如果有多个分支，可使用**case**关键字，通过**end**关键字标识结束

```mysql
case 表达式 when 值1 then 结果1 when 值2 then 结果2 ... else ... end
```

比如，对于员工角色常使用枚举数字存储在数据库中，如果统计各个职位上的员工数，可以使用:

```mysql
select
    (case job when 1 then '班主任' when 2 then '讲师' when 3 then '学工主管' when 4 then '教研主管' else '未分配职位' end) as 职位,
    count(*) as 数量
    from tb_emp
    group by job;
```

## 多表查询

多表查询即查询范围是在不同表中的查询

### select * from 表列表

以笛卡尔积(集合间所有的组合情况 A * B)的方式，组合多个表中所有的情况(包括无效数据)并展现，各个表中的字段都会横向呈现出来

```mysql
select * from tb_emp, tb_dept;
```

#### 消除无效数据

消除无效的数据可以使用字段匹配的方法，即限定两表之间的有关联的字段的值的关系，比如:

```mysql
select * from tb_emp, tb_dept where tb_emp.dept_id = tb_dept.id;
```

### 连接查询

连续查询即在多表查询的情况下，消除掉无效的数据的一种方法

#### 内连接

查询A、B...等集合相交部分的数据，即**交集**(A ∩ B)， 比如连接查询员工表和部门表，查询出每个员工的信息及其所属的部门，不包括不属于任何部门的员工(即交集外的部分)

内查询分为隐式内连接与显式内连接:

- 隐式内连接

  ```mysql
  select 字段列表 from 表1, 表2 where 条件...;
  ```

  也是上述**消除无效数据**中举例使用的方法

  ```mysql
  select tb_emp.name, tb_dept.name from tb_emp, tb_dept where tb_emp.dept_id = tb_dept.id;
  ```

- 显式内连接

  ```mysql
  select 字段列表 from 表1 [inner] join 表2 on 连接条件...;
  ```

  在语义上表名了查询访问是有哪些表，除此之外是用**join ... on ...**替代了**from 表列表 where ...**

  ```mysql
  select tb_emp.name, tb_dept.name from tb_emp inner join tb_dept on tb_emp.dept_id = tb_dept.id;
  ```

在多表查询时，为了省去写入可能较为繁杂的表名的时间，可以给表名起别名:

```mysql
select e.name, d.name from tb_emp as e inner join tb_dept as d on e.dept_id = d.id;
```

值得注意的是，当**起了别名之后原表名不能再使用**

#### 外连接

适用于查询某张表的全部信息即另一张表与其的交集部分的场景

外连接又分左外连接和右外连接:

- 左外连接: 查询**左表所有的数据**(包括两张表**交集部分**的数据)，即**交集+ A - A ∩ B**

  ```mysq;
  select 字段列表 from 表1 left [outer] join 表2 on 连接条件...;
  ```

  **left join 左边的表即是左表**

  ```mysql
  select e.name, d.name from tb_emp e left join tb_dept d on e.dept_id = d.id;
  ```

  比如上述sql中**tb_emp**的信息会全部加入到查询结果中

- 右外连接: 查询**右表所有的数据**(包括两张表**交集部分**的数据)，即**交集+ B - A ∩ B**

  ```mysql
  select 字段列表 from 表1 right [outer] join 表2 on 连接条件...;
  ```

  **right join 右边的表即是右表**

  ```mysql
  select e.name, d.name from tb_emp e right join tb_dept d on e.dept_id = d.id;
  ```

可见左外连接和右外连接时可以相互替换的，只需将表放在关键字的左、右边即是左表、右表，这也仅是帮助在需要保留一张表的信息的情况下加入和另一张表的交集的信息的场景，**在一般的项目开发中使用左连接即可**

### 子查询

又称为嵌套查询，即在SQL语句中嵌套select/insert/update/delete语句，常见的是select

```mysql
select * from 表1 where 字段1 = (select 字段 1 from 表2 ...);
```

#### 标量子查询

子查询返回的结果为单个值(数字、字符串、日期等)，常用的操作符是 `=、<>、>、>=、<、<=`

需要保证的是子查询返回的结果一定是**单个值**，比如查询教研部下的员工信息:

```mysql
select * from tb_emp where tb_emp.dept_id = (select id from tb_dept where name = '教研部');
```

查询入职时间比姓名为方东白的员工晚的员工信息:

```mysql
select * from tb_emp where entrydate > (select entrydate from tb_emp where name = '方东白');
```

#### 列子查询

子查询返回的结果为一列，但可以是多行，常用的操作符是`in、not in`等

适用于子查询的结果有多个值的情况，故使用`in`等关键字去匹配多个值

比如查询教研部、咨询部下的员工信息:

```mysql
select * from tb_emp where tb_emp.dept_id in (select id from tb_dept where name = '教研部' or name = '咨询部');
```

#### 行子查询

子查询返回的结果为一行，但可以是多列，常用的操作符是`=、<>、in、not in`

比如查询职位和入职日期均和姓名为韦一笑相同的员工信息，根据标量子查询的经历，可以写出:

```mysql
select * from tb_emp where entrydate = (select entrydate from tb_emp where name = '韦一笑') and job = (select job from tb_emp where name = '韦一笑');
```

但无疑这有些冗余，可以将入职日期和职位组合起来，简化成:

```mysql
select * from tb_emp where (entrydate, job) = (select entrydate, job from tb_emp where name = '韦一笑');
```

#### 表子查询

子查询返回的结果是多行多列，即一张临时表，常见的操作符是`in`

比如查询入职日期在2006-01-01之后的员工信息及其部门名称这个需求，我们可以分解为:

1. 在员工表中查询入职入职日期在2006-01-01之后的员工信息
2. 再在1中返回的表中连接查询部门表获取各员工所属的部门名称

可见，1中返回的是一张表，需要再临时表中进行后续的查询，子查询可写为:

```mysql
select temp.*, d.name as '部门名称' from (select * from tb_emp where entrydate > '2006-01-01') as temp, tb_dept d where temp.dept_id = d.id;
```

# 多表操作

在表结构的设计上，可大致分为3种:

- 一对多(多对一)
- 多对多
- 一对一

## 一对多

如**部门 -> 员工**之间的关系

"多"的一方由于只能对应"一"个，故可称为**子表**，反之。"一"的一方可称为**父表**

### 建立约束

约束是为了**保证各表之间的数据完整性**，比如有关联关系的两张表，通过对两表之间的纽带字段建立约束，可以防止建立约束的数据只在其中一张表有而另一张表中不存在的情况(如被删除或未被建立)

一般是通过**指定外键**的方式来建立两表之间的约束，在一对多的关系中，**外键建立在子表中**

### 物理外键

通过关键字**constraint**，有两种方式来添加物理外键:

- 在创建表时添加外键

  ```mysql
  create table 表名 (
  		字段名 字段类型,
  		...
  		[constraint] [外键名称] foreign key(外键字段名) references 主(父)表(字段名)
  );
  ```

- 在表已经被创建后建立外键:

  ```MySQL
  alter table 表名 add constraint 外键名称 foreign key(外键字段名) references 主(父)表(字段名);
  ```

需要注意的是，在被约束的列中，**只有在两表中均存在的值才会被约束**

如果使用物理外键，则会有一些缺点:

- 影响增、删、改的效率，因为当外键创立后存储引擎会花时间检查外键的关系
- 物理外键仅适用于单节点数据库，不适用于分布式、集群场景
- 容易引发数据库的死锁问题，消耗性能

### 逻辑外键

相对于在表设计中加入物理外键，逻辑外键是通过**在编写业务逻辑的代码中解决表之间的关联问题**，也是现在主流项目开发中所推荐使用的

## 一对一

如用户和其身份证的关系

一对一的表连接关系往往用在**表的拆分中**，即横向的拆分表的字段，比如可以将基础字段放到一张表中，其他字段放到另外一张表中，其中基础字段和其他字段的**划分依据**往往是**访问的频率大小**

### 建立约束

由于只是水平的拆分，故仅需**在任意一方建立一个外键，并设置为UNIQUE，关联另一方的主键即可**

比如，建立个人基本信息表和其他信息表(拆分出去的)之间的关联:

个人基本信息表:

```mysql
create table tb_user(
    id int unsigned  primary key auto_increment comment 'ID',
    name varchar(10) not null comment '姓名',
    gender tinyint unsigned not null comment '性别, 1 男  2 女',
    phone char(11) comment '手机号',
    degree varchar(10) comment '学历'
) comment '用户信息表';
```

其他信息表:

```mysql
create table tb_user_card(
    id int unsigned  primary key auto_increment comment 'ID',
    nationality varchar(10) not null comment '民族',
    birthday date not null comment '生日',
    idcard char(18) not null comment '身份证号',
    issued varchar(20) not null comment '签发机关',
    expire_begin date not null comment '有效期限-开始',
    expire_end date comment '有效期限-结束',
    user_id int unsigned not null unique comment '用户ID',
    constraint fk_user_id foreign key (user_id) references tb_user(id)
) comment '用户信息表';
```

其中是通过```constraint fk_user_id foreign key (user_id) references tb_user(id)```，将水平拆分的两个表连接到了一起，且关联的字段`user_id`是非空且唯一的

## 多对多

如学生和课程之间的关系

此时，由于对于任一张表中的任一字段，相对于另一张表的中的值，都是一对多的关系，这就不符合了数据库的第一范式

由此，需要再建立一张**中间表**去建立两表之间的关联关系

比如，再学生于课程之间的关联关系中，可以将两表的主键都提炼到一张表中，分别为**student_id**和**course_id**，由此在表中描绘多对多的关系，但仅就这样的表是没有约束作用的，还**需要在中间表中建立外键来建立两表之间的约束**

### 建立约束

学生表:

```mysql
create table tb_student(
    id int auto_increment primary key comment '主键ID',
    name varchar(10) comment '姓名',
    no varchar(10) comment '学号'
) comment '学生表';
```

课程表:

```mysql
create table tb_course(
   id int auto_increment primary key comment '主键ID',
   name varchar(10) comment '课程名称'
) comment '课程表';
```

中间表:

```mysql
create table tb_student_course(
   id int auto_increment comment '主键' primary key,
   student_id int not null comment '学生ID',
   course_id  int not null comment '课程ID',
   constraint fk_courseid foreign key (course_id) references tb_course (id),
   constraint fk_studentid foreign key (student_id) references tb_student (id)
)comment '学生课程中间表';
```

在查询时，以内连接为例，设定的连接应是两表与中间表中的字段的连接

比如查询商务套餐A中包含了哪些菜品，展示出套餐名称、价格、包含的菜品名称、价格、份数(中间表中的字段)

```mysql
select s.name, s.price, d.name, d.price, sd.copies
from setmeal s,
     dish d,
     setmeal_dish sd
where s.id = sd.setmeal_id
  and d.id = sd.dish_id
  and s.name = '商务套餐A';
```

# 事务

事务是一系列操作的集合，这组集合内的行为要么同时成功，要么同时失败

事务控制主要有三个关键字:

- 开启事务

  ```mysql
  start transaction; / begin
  ```

- 提交事务

  ```mysql
  commit;
  ```

- 回滚事务

  ```mysql
  rollback
  ```

比如:

```mysql
start transaction;

delete from tb_dept where id = 3;

delete from tb_emp where dept_id = 3;

commit;
```



# 数据库调优

## 索引

MySQL中的查询操作是全表扫描直到扫描到匹配项，**是线性的**，而创建索引可以百倍、千倍的减少通过索引列查询数据的时间，但是也会占用一定的空间维护索引，索引是一种数据结构，**是树形的**，可以极大程度上减少比较的次数，是一种空间换时间的理念，且由于建立索引即对该列中的值进行了排序，故降低了在查找时数据排序的成本，因此也会降低CPU的消耗

同时，**建立索引也可能是个比较耗时的操作**，这和表的数据量呈正相关，不过一般索引是在创建表时即创建完成的，且由于数据已经是有序的了，故**再进行insert、delete、update操作时会更加耗时**

### 创建索引

```mysql
create [unique] index 索引名 on 表名(列名);
```

创建唯一索引时才需要加入**unique**

比如为员工表的name字段创建一个普通索引:

```mysql
create index idx_emp_name on tb_emp(name);
```

### 查看索引

```mysql
show index from 表名;
```

比如查看员工表的索引信息:

```mysql
show index from tb_emp;
```

得到结果如下:

```shell
tb_emp,0,PRIMARY,1,id,A,17,,,"",BTREE,"","",YES,
tb_emp,0,username,1,username,A,17,,,"",BTREE,"","",YES,
tb_emp,1,idx_emp_name,1,name,A,17,,,"",BTREE,"","",YES,
```

可见有数据库默认创建的主键索引(性能最高)、由于username字段是一个有唯一约束的字段，故会默认添加一唯一索引(这也是唯一约束即**unique**关键字的本质)、手动创建的name索引

### 删除索引

```mysql
drop index 索引名 on 表名;
```

