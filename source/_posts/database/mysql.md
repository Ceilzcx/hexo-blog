---
title: mysql
date: 2021-07-15 15:59:33
tags: 数据库
---

## MySQL数据库



### 二、函数

#### 2.1、递归

mysql的递归比较繁琐，需要通过函数实现，返回的是集合，通过 `FIND_IN_SET` 进行操作。

注：递归调用消耗大量时间，在实现中宁可在代码中采用递归，也不要在SQL中使用递归。

```mysql
create function getChildrenOrg(teamId INT)
    returns varchar(4000)
BEGIN
    DECLARE oTemp VARCHAR(4000);
    DECLARE oTempChild VARCHAR(4000);

    SET oTemp = '';
    SET oTempChild = CAST(teamId AS CHAR);

    WHILE oTempChild IS NOT NULL
        DO
            SET oTemp = CONCAT(oTemp,',',oTempChild);
            SELECT GROUP_CONCAT(team_id) INTO oTempChild FROM team WHERE FIND_IN_SET(parent_id, oTempChild) > 0;
        END WHILE;
    RETURN oTemp;
END
```

#### 2.2 NOW
```mysql
select NOW(), NOW(3)
```
`NOW(3)`: 获得当前时间，包括毫秒值


### 三、主从复制

#### 3.1、类型

+ 基于语句的复制：将修改数据的SQL记录到 `binlog` 中。从库读取 `binlog` 写入中继日志 `relayLog` 中，复制日志时，从库会启动一个**工作线程**，然后将其放入数据库。

  优点：只记录修改数据，日志不大，解决IO。

  缺点：数据可能存在不一致，例：同步过程主数据库挂了

+ 基于行数据的复制：只记录行数据的修改。

  缺点：中间过程全部丢失，产生大量日志（例：alter table）

+ 混合复制：一般的复制使用STATEMENT模式保存binlog，对于STATEMENT模式无法复制的操作使用ROW模式保存binlog



#### 3.2、模式

+ 异步复制模式：主服务器启动I/O线程，将数据写到binlog中返回给客户端数据更新成功，不考虑数据是否同步。效率高，但数据一致性存在风险。
+ 同步复制模式：主服务器执行执行完客户端提交的事物后，等待从库执行完后，才返回执行成功。等待过程中，线程被阻塞，性能较慢。
+ 半同步复制模式：master的dump线程通知从库，从库执行完成后，发送ACK，主库接收到一个标志码（ACK）后，返回成功。与同步不同的是，只需要一个从库同步成功就返回成功。



### 四、事务隔离级别

#### 4.1、查看和修改事务隔离级别

```mysql
select @@transaction_isolation;
```

![mysql-select-transaction](mysql-select-transaction.png)

```mysql
update global transaction isolation REPEATABLE READ;
-- global：全局修改
-- session：本次回话修改
```

![mysql-update-transaction](mysql-update-transaction.png)



#### 4.2、隔离级别

+ 读未提交（READ UNCOMMITTED）

  读未提交会读到另一个事务未提交的数据，产生脏读，不可重复读数据。一个事务的update操作可能会影响另一个事务。

  如下图，可以看到第一个事务修改完后，第二个事务直接可以select修改后的值

  ![read-uncommitted-1](read-uncommitted-1.png)

  ![read-uncommitted-2](read-uncommitted-2.png)

+ 读提交（READ COMMITTED）

  解决脏读的问题，出现不可重复读。一个事务可以读取另一个事务提交后的数据。一个事务的update操作可能会影响另一个事务。

  如下图，可以看到第一个事务update后，第二个事务查询的还是原来的数据，等到第一个事务提交后，第二个事务查询的就是第一个事务修改后的数据

  ![read-committed-1](read-committed-1.png)

  ![read-committed-2](read-committed-2.png)
  
+ 可重复读（REPEATABLE READ）

  解决了不可重复读和脏读的问题。但是存在换读的问题（一个事务新增了一条id=1的数据，这时另一个事务也新增了一条id=1的数据并提交，第一个事务select时会发现不存在id=1的数据，报主键冲突的错误）。

  ![repeatable-read-1](repeatable-read-1.png)

  ![repeatable-read-2](repeatable-read-2.png)

+ 串行化（SERIALIZABLE）

  暂无



### 五、日志（binlog）

#### 5.1使用日志进行数据还原

日志未打开，win10在my.ini文件的[mysqld]下面添加

```sql
log_bin=mysql-bin
binlog-format=ROW
server-id=1
```

查找MySQL当前binlog的配置情况，**第一行ON代表日志已经打开**。

```sql
show variables like '%log_bin%';
```

![mysql-binlog-variables](mysql-binlog-variables.png)

查看日志文件的使用情况（日志名称、大小bit、是否加密）

```sql
show binary logs;
```

![mysql-binlog-show-1](mysql-binlog-show-1.png)

查看当前正在使用的日志情况，后续的DDL操作都会记录在当前日志中。

```sql
show master status;
```

![mysql-binlog-show-2](mysql-binlog-show-2.png)

执行一些测试的SQL语句，可以看到创建一个数据库test2，在数据库中创建了一张表test，模拟不小心删除了test2数据库。

![mysql-test-1](mysql-test-1.png)

在binlog中查找之前执行的SQL语句

```sql
show binlog events in 'binlog.000012';
```

![mysql-binlog-show-3](mysql-binlog-show-3.png)

第二列的1300...表示所在的位置（行数），将这几行数据复制到一个sql文件，执行该文件即可恢复数据。

注：根据需求，可以在恢复数据的时候关闭日志。等到数据恢复后，重新打开日志

```sql
set sql_log_bin=0;	#临时关闭日志
set sql_log_bin=1;	#打开日志
```

注意这里是shell，不是MySQL中执行

```shell
mysqlbinlog --start-position=1300 --stop-position=1757 ./binlog.000012 >./bin.sql
```

注：这里运行可能会报 `unknown variable 'default-character-set=utf8'` 的错误。加上运行参数 `--no-default` ，或者修改配置文件。



### 六、索引

#### 6.1 聚簇索引

+ 如果设置了主键，主键为聚簇索引
+ 否则第一个 NOT NULL and UNIQUE的字段为聚簇索引
+ 默认创建一个隐藏的row_id 为聚簇索引

聚簇索引指向（存储）的数据是行记录（页结构）

**InnoDB必须包含一个聚簇索引**

#### 6.2 普通索引

> 二级索引，非聚簇索引

叶子节点存储聚簇索引字段的值

#### 6.3 回表查询

> 先通过普通索引的值定位聚簇索引值，再通过聚簇索引的值定位行记录数据，需要扫描两次索引B+树，它的性能较扫一遍索引树更低。

```sql
// user表包含 id，name，age字段，其中id为主键（聚簇索引），age为普通索引
select * from user where age = 20;
```

通过age的普通索引查询对应的id，然后回表查询id，获得对应的行（两次查询）

#### 6.4 索引覆盖

> 只需要在一棵索引树上就能获取SQL所需的所有列数据，无需回表，速度快。

```sql
select id, age from user where age = 20;
```

**如何实现覆盖索引？**

+ 创建联合索引

  ```sql
  create index idx_age_name on user(`age`,`name`);
  ```

适用范围：全表count、分页查询

索引结构：age和name放在一个节点，和普通的B+Tree一致，比较大小时（**最左匹配原则**：先比较age，再比较name）



### 七、视图

> 作为一张虚拟表，本身不存储数据，作为一条select语句存储在数据字典中
>
> 简化设计，可能提高性能、也可能降低性能



