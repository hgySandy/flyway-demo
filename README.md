# flyway-demo
[TOC]

[flyway-demo 源码](https://github.com/hgySandy/flyway-demo)
## 1. flyway简介

### 1.1 Flyway是什么
  >Flyway是一款开源的数据库版本管理工具，它更倾向于规约优于配置的方式。Flyway可以独立于应用实现管理并跟踪数据库变更，支持数据库版本自动升级，并且有一套默认的规约，不需要复杂的配置，Migrations可以写成SQL脚本，也可以写在Java代码中，不仅支持Command Line和Java API，还支持Build构建工具和Spring Boot等，同时在分布式环境下能够安全可靠地升级数据库，同时也支持失败恢复等。
### 1.2  Flyway如何运行
  >Flyway对数据库进行版本管理主要由Metadata表和6种命令完成，Metadata主要用于记录元数据，每种命令功能和解决的问题范围不一样。
[Flyway命令行介绍](https://flywaydb.org/documentation/commandline/)
### 1.3  Flyway中sql脚本的命名规则
  
  - sql脚本的命名规则组成
    - 前缀: 默认是 V.
    - 版本: 圆点或者下划线在你想要添加的地方添加.
    - 分隔符: 默认为 __ (两个下划线)，将版本与描述分开。
    - 描述:用下划线或空格隔开的文本。
    - 后缀: 默认是 .sql.
  
  - Flyway的sql脚本命名示例：
```
  V1.0001__some_description.sql 
  V1_0001__another_description.sql 
  V002_1_5__my_new_script.sql 
  V15__some_other_script.sql 
```

  
### 1.4 数据库版本控制表schema_version
  - schema_version表结构如下：
  
  |column name    | column type  |
  |:--------------|:-------------|
  |installed_rank | integer      |
  |version        | varchar(50)  |
  |description    | varchar(200) |
  |type           | varchar(20)  |
  |script         | varchar(1000)|
  |checksum       | integer      |
  |installed_by   | varchar(100) |
  |installed_on   | timestamp    |
  |execution_time | integer      |
  |success        | boolean      |


## 2. flyway集成springboot

### 2.1 配置pom.xml

```
#片段一

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <dependency>
        <groupId>org.flywaydb</groupId>
        <artifactId>flyway-core</artifactId>
        <version>4.0</version>
    </dependency>

```

```
#片段二
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

```

### 2.2 配置application.properties

```

spring.datasource.url=jdbc:mysql://localhost:3306/flyway_demo
spring.datasource.username=root
spring.datasource.password=12345
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5Dialect

spring.jpa.hibernate.ddl-auto=none
spring.jpa.show-sql=true

```
### 2.3 准备数据库脚本,并测试

#### 2.3.1 添加数据库初始脚本V1__create_table.sql
- 添加src/main/resources/db/migration/V1__create_table.sql脚本，内容如下：
```
//创建user_sql表
create table user_sql(
 user_id int(4) not null primary key auto_increment,
 name char(20) not null,
 sex int(4) not null default '0',
 degree double(16,2)
 );

```
- 启动命令
```
//如启动不成功，可先执行此命令
mvn clean install
//启动服务
mvn spring-boot：run
```
- 启动服务前的数据库截图

![2018-3-15-截图](http://img.pinbot.me:8080/uploads/2018/3/15/blob_1521083379939.png "blob_1521083379939.png")
- 启动服务后的数据库截图

![2018-3-15-截图](http://img.pinbot.me:8080/uploads/2018/3/15/blob_1521084408833.png "blob_1521084408833.png")
![2018-3-15-截图](http://img.pinbot.me:8080/uploads/2018/3/15/blob_1521084592824.png "blob_1521084592824.png")
![2018-3-15-截图](http://img.pinbot.me:8080/uploads/2018/3/15/blob_1521084997841.png "blob_1521084997841.png")

#### 2.3.2 添加数据库脚本V2__insert-data.sql
- 添加src/main/resources/db/migration/V2__insert-data.sql脚本，内容如下：
```
//插入数据到user_sql表
insert user_sql(name,sex,degree) values('henry','0','120');
insert user_sql(name,sex,degree) values('bill','0','120');
insert user_sql(name,sex,degree) values('eric','0','121');
insert user_sql(name,sex,degree) values('tom','0','120');
insert user_sql(name,sex,degree) values('jerry','0','120');

```
- 启动命令
```
mvn spring-boot：run
```
- 启动服务后的数据库截图
![2018-3-15-截图](http://img.pinbot.me:8080/uploads/2018/3/15/blob_1521085128086.png "blob_1521085128086.png")
![2018-3-15-截图](http://img.pinbot.me:8080/uploads/2018/3/15/blob_1521085169063.png "blob_1521085169063.png")


#### 2.3.3 添加数据库脚本V3__delete-data.sql
- 添加src/main/resources/db/migration/V3__delete-data.sql脚本，内容如下：
```
//删除user_sql表中部分数据
delete from user_sql where name = 'tom';

```
- 启动命令
```
mvn spring-boot：run
```
- 启动服务后的数据库截图
![2018-3-15-截图](http://img.pinbot.me:8080/uploads/2018/3/15/blob_1521085456050.png "blob_1521085456050.png")
![2018-3-15-截图](http://img.pinbot.me:8080/uploads/2018/3/15/blob_1521085495415.png "blob_1521085495415.png")



#### 2.3.4 添加数据库脚本V4__update-data.sql
- 添加src/main/resources/db/migration/V4__update-data.sql脚本，内容如下：
```
//更新user_sql表中部分数据
update  user_sql set name = 'james' where degree = 121;

```
- 启动命令
```
mvn spring-boot：run
```
- 启动服务后的数据库截图
![2018-3-15-截图](http://img.pinbot.me:8080/uploads/2018/3/15/blob_1521085857190.png "blob_1521085857190.png")
![2018-3-15-截图](http://img.pinbot.me:8080/uploads/2018/3/15/blob_1521085887885.png "blob_1521085887885.png")



#### 2.3.5 添加数据库脚本V5__add-column.sql
- 添加src/main/resources/db/migration/V5__add-column.sql脚本，内容如下：
```
//添加字段到表
alter table user_sql add age int not null default 18;
```
- 启动命令
```
mvn spring-boot：run
```
- 启动服务后的数据库截图
![2018-3-15-截图](http://img.pinbot.me:8080/uploads/2018/3/15/blob_1521086125059.png "blob_1521086125059.png")
![2018-3-15-截图](http://img.pinbot.me:8080/uploads/2018/3/15/blob_1521086161338.png "blob_1521086161338.png")





## 3. flyway-demo目录结构

```
|____src
| |____main
| | |____resources
| | | |____db
| | | | |____migration  # flyway sql脚本默认目录
| | | | | |____V1__create_table.sql
| | | | | |____V5__add-column.sql
| | | | | |____V3__delete-data.sql
| | | | | |____V2__insert-data.sql
| | | | | |____V4__update-data.sql
| | | |____application.properties #配置文件
| | |____java
| | | |____com
| | | | |____flyway
| | | | | |____Migrations.java #springboot 启动类 
|____pom.xml # maven 配置文件
|____README.md

```
## 4. flyway拓展阅读

[flyway官网](https://flywaydb.org/)

[快速掌握和使用Flyway](https://blog.waterstrong.me/flyway-in-practice/)
