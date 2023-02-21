# MySQL事务

## 概念

### 什么是事务？

> 事务是用于保证数据一致性，它由一组相关的dml语句组成，改组的dml语句，要么全部成功，要么全部失败。

### 事务和锁

> 当执行事务操作时，MySQL会在表上加锁，防止其他用户改表的数据，这对用户来说是非常重要的

### MySQL数据库控制台事务的几个重要操作

> 1. 如果不开始事务，默认情况下，dml操作自动提交，不可以回滚
>
> 2. 如果开始一个事务，没有创建保存点，可以执行rollback，默认回到事务开始的状态
>
> 3. mysql的事务机制需要innodb的存储引擎还可以使用，myisam不好用
>
> 4. start transaction开始一个事务
>
> 5. savepoint 保存点名 设置保存点
>
> 6. rollback to  保存点名 回退事务
>
> 7. commit  提交事务，所有操作全部生效，不能回退
>
> 8. 回退事务：首先要明白保存点(savepoint),保存点是事务中的点，用于取消部分事务，当结束事务时(commit),会自动删除该事物所定义的所有保存点，当执行回退事务时，通过指定保存点，可以回退到指定的点
>
> 9. 提交事务：使用commit语句可以提交事务，当执行了commit语句之后，会确认事务的变化，结束事务，删除保存点、释放锁、数据生效。当使用commit语句结束事务之后，其他会话(连接)将可以查看到事物变化后的新数据，所有数据正式生效
>
> 10. 代码介绍
>
>     > ```mysql
>     > SELECT	* FROM test;
>     > -- 开始事务
>     > START TRANSACTION;
>     > -- 设置保存点a
>     > SAVEPOINT a;
>     > INSERT INTO test
>     > VALUES
>     > 	( "小徐", "1223", "555" )
>     > 	;-- 设置保存点b
>     > SAVEPOINT b;
>     > INSERT INTO test
>     > VALUES
>     > 	( "小白", "2213", "666" );
>     > 	-- 回退到b
>     > ROLLBACK TO b;
>     > -- 回退到a
>     > ROLLBACK TO a;
>     > -- 回退到原点
>     > ROLLBACK;
>     > -- 提交事务
>     > COMMIT;
>     > 
>     > -- 注意，不可以先回退到a再回退到b，可以直接回退到a
>     > ```

## MySQL事务隔离级别

### 事务隔离级别介绍

> 概念：MySQL隔离级别定义了事务于事务之间的隔离程度
>
> 多个连接开启各自事务操作数据库中的数据时，数据库系统主要负责隔离操作，以保证各个连接在获取数据时的准确性
>
> 如果不考虑隔离性，可能会引发如下问题
>
> 1. 脏读：当一个事务读取另一个事务尚未提交的修改时，产生脏读
>
> 2. 不可重复读：同一查询在同一事务中多次进行，由于其他提提交事务所做的修改或删除，每次返回不同的结果集，此时发生不可重复读
>
> 3. 幻读：同一查询在同一事务中多次进行，由于其他提交事务所做的插入操作，每次返回不同的结果集，此时发生幻读
>
> 4. ![avatar](https://github.com/xiaoxu137/blog.git.io/blob/main/jpg/JDBC%E5%9F%BA%E7%A1%80/%E9%9A%94%E7%A6%BB%E5%9B%BE.png)
>
> 5. ```mysql
>    -- 隔离级别常用指令
>    
>    -- 查看当前会话隔离级别
>    SELECT @@tx_isolation
>    -- 查看系统当前隔离级别
>    SELECT @@GLOBAL.tx.tx_isolation
>    -- 设置当前会话隔离级别
>    SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
>    -- 设置系统当前隔离级别
>    SET GLOBAL TRANSACTION ISOLATION LEVEL REPEATABLE READ;
>    -- mysql默认的事务隔离级别是repeatable read 一般情况下，没有特殊需求没有必要修改(因为这个级别可以满足绝大部分项目需求)
>    ```
>
>    

