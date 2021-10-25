# 数据库持续集成

## 场景描述
  通常在项目开始时会针对数据库进行全局设计，但在开发产品新特性过程中，难免会遇到需要更新数据库Schema的情况，比如：添加新表，添加新字段和约束等，这种情况在实际项目中也经常发生。那么，当开发人员完成了对数据库更的SQL脚本后，如何快速地在其他开发者机器上同步？并且如何在测试服务器上快速同步？以及如何保证集成测试能够顺利执行并通过呢？

到各测试服务器上手动执行SQL脚本费时费神费力的，干嘛不自动化呢，当然，对于高级别和PROD环境，还是需要DBA手动执行的。

## Why database migrations?

我们以为开发过程中的DB是这样的

![image](https://user-images.githubusercontent.com/74898931/138644138-fdbdee11-3167-400d-a6cd-ac236a9cfead.png)

实际上我们的DB是这样的

![image](https://user-images.githubusercontent.com/74898931/138644206-39c3fc5a-4ded-4e6f-bb27-b8e0daccdf7b.png)

各自在自己分支开发的时候还好，但是合并代码以及上线后数据库统一就成了问题了。我们总不能改一点数据库就把sql语句拿着去改各个服务器的DB吧（万一漏掉哪个不是很蛋疼）

## 解决方案

Spring Boot为两款流行的数据库迁移库提供了自动配置支持。

- [Flyway](https://flywaydb.org/)
  - 原理：Flyway是一个非常简单的开源数据库迁移库，使用SQL来定义迁移脚本。它的理念是，每个脚本都有一个版本号，Flyway会顺序执行这些脚本，让数据库达到期望的状态。它也会记录已执行的脚本状态，不会重复执行。Flyway脚本就是SQL。让其发挥作用的是其在Classpath里的位置和文件名。Flyway脚本都遵循一个命名规范，含有版本号。例如：V2_test.sql。所有Flyway脚本的名字都以大写字母V开头，随后是脚本的版本号。后面跟着两个下划线和对脚本的描述。Flyway脚本需要放在相对于应用程序Classpath根路径，在src/main/resources/db/migration路径下。在应用程序部署并运行起来后，Spring Boot会检测到Classpath里的Flyway，自动配置所需的Bean。Flyway会依次查看/db/migration里的脚本，如果没有执行过就运行这些脚本。每个脚本都执行过后，向schema_version表里写一条记录。应用程序下次启动时，Flyway会先看schema_version里的记录，跳过那些脚本。
  - 优点：SQL用起来便捷顺手。
  - 缺点：无法跨平台使用。
  - 最佳实践: [Spring Boot 2 实战：使用 Flyway 管理你数据库的版本变更](https://segmentfault.com/a/1190000020850220)
- [liquibase](http://www.liquibase.org)
  - 原理：支持XML、YAML和JSON格式的迁移脚本。默认情况下，Bean会在/db/changelog（相对于Classpath根目录）里查找db.changelog-master.yaml文件。Liquibase变更集都集中在一个文件里。changeset命令后的那行有一个id属性，要对数据库进行后续变更。可以添加一个新的changeset，只要id不一样就行。此外，id属性也不一定是数字，可以包含任意内容。应用程序启动时，Liquibase会读取db.changelog-master.yaml里的变更集指令集，与之前写入databaseChangeLog表里的内容做对比，随后执行未运行过的变更集。
  - 优点：
    - 可以跨平台使用
    - 可以回滚操作
  - 缺点：迁移脚本需要花费精力去维护
  - 最佳实践: [Spring Boot使用Liquibase最佳实践](https://segmentfault.com/a/1190000016641122)
