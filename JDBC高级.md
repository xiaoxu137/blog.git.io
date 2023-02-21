# JDBC高级

## 事物

> 1. 基本介绍
>
> > - JDBC程序中为了让多个sql语句作为一个整体执行，需要使用事务
> > - 调用Connection的setAutoCommit(false)可以取消自动提交事务
> > - 在所有的sql语句都执行成功后，调用commit()方法提交事务
> > - 在某个操作失败或出现异常时，调用rollback();方法回滚事务
>
> 2. 代码
>
>    ```java
>        public void Transaction(){
>            Connection connection=MysqlConnect.getConn(); //默认情况下，connection自动提交
>            try {
>                //将connection设置不自动提交
>                connection.setAutoCommit(false);
>                /**
>                 * sql语句
>                 * 执行sql
>                 * 如果没有抛出异常
>                 * 则进行事务提交
>                 */
>                connection.commit();
>            } catch (SQLException e) {
>                try {
>                    //如果发生异常，就撤销执行的sql，默认回滚到事物开始的状态
>                    connection.rollback();
>                } catch (SQLException e1) {
>                    e1.printStackTrace();
>                }
>                e.printStackTrace();
>            }
>        }
>    ```

## 批处理

> 1. 基本介绍
>
> > - 当需要成批插入或者更新记录时，可以采用Java的批量更新机制，这一机制允许多条语句一次性提交给数据库批量处理，通常情况下比单独处理更有效率
> > - JDBC的批量处理语句主要包括一下几种方法：
> >   1. addBatch():添加需要批量处理的SQL语句或参数
> >   2. executeBatch():执行批处理包的语句
> >   3. clearBatch():清空批处理包的语句
> >
> > - JDBC连接MySQL时，如果使用批处理功能，需要在url中添加参数？&rewriteBatchedStatements=true
> > - 批处理往往和PreparedStatement一起搭配使用，可以既减少编译次数，有减少运行次数，效率有很大提高
>
> 2. 代码
>
>    ```java
>      public  void bacth() {
>            //添加五千条数据
>            Properties properties = new Properties();
>            try {
>                properties.load(new FileInputStream("src\\jdbcStudy\\mysql.properties"));
>                String user = properties.getProperty("user");
>                String password = properties.getProperty("password");
>                String url = properties.getProperty("url");
>                String driver = properties.getProperty("driver");
>                Class.forName(driver);
>                Connection connection = DriverManager.getConnection(url, user, password);
>                String sql = "insert into test values(?,?,?)";
>                PreparedStatement preparedStatement = connection.prepareStatement(sql);
>                long start=System.currentTimeMillis();
>                for (int i = 0; i < 5000; i++) {
>                    preparedStatement.setString(1, "小徐");
>                    preparedStatement.setString(2, "123456");
>                    preparedStatement.setString(3, String.valueOf(i));
>                    preparedStatement.addBatch();
>                    if ((i+1)% 1000 == 0) {
>                        preparedStatement.executeBatch();//满足1000条数据则进行清空
>                        preparedStatement.clearBatch();
>                    } else {
>          
>                    }
>                }
>                long end=System.currentTimeMillis();
>                System.out.println("用时"+(end-start));//耗时108
>          
>            } catch (SQLException e) {
>                e.printStackTrace();
>            } catch (FileNotFoundException e) {
>                e.printStackTrace();
>            } catch (IOException e) {
>                e.printStackTrace();
>            } catch (ClassNotFoundException e) {
>                e.printStackTrace();
>            }
>        }
>    ```
>
>    3. 源码分析
>
>       > 第一次创建时会创建一个ArrayList集合里面存放了一个elementDate的对象数组，在这个数组里面会存放相应的sql语句，当elementDate满后，会按照1.5倍进行扩容，批处理会减少我们发送sql语句的网络开销，而且减少编译次数，效率可以提高

## 连接池

> - 传统方式连接弊端分析
>   - 传统的JDBC数据库连接使用DriverManager来获取，每次向数据库建立连接的时候都需要将Connection加载到内存中，再验证ip地址，用户名，密码。需要数据库的时候就需要向数据库要求一个，频繁进行数据库连接操作将占用很多系统资源，容易造成服务器崩溃
>   - 每一次数据库连接，使用后都需要断开，如果程序出现异常而未能关闭，将导致数据库内存泄漏，最终导致重启数据库
>   - 传统获取连接的方式，不能控制创建的连接数量，如果连接过多，也可能会导致内存泄漏，MySQL崩溃
>   - 解决传统开发中的数据库连接问题，可以采用数据库连接池技术(Connection pool)
>
> - 数据库连接池基本介绍
>
>   - 预先在缓冲池中放入一定数量的连接，当需要建立数据库连接时，只需要从“缓冲池”中取出一个，使用完毕之后再放回去
>   - 数据库连接负责分配，管理和释放数据库连接，它允许应用程序重复使用一个现有的数据库连接，而不是重新建立一个
>   - 当应用程序向连接池请求的连接数超过最大的连接数量时，这些请求将被加入到等待队列中
>
> - C3P0的使用
>
>   ```java
>   import java.io.FileInputStream;
>   import java.sql.Connection;
>   import java.util.Properties;
>
>   public class C3p0Study {
>       //第一种方式
>       public void testC3p01() throws Exception {
>           //创建数据源对象
>           ComboPooledDataSource comboPooledDataSource =new ComboPooledDataSource();
>           //通过配置文件获取相关的信息
>           Properties properties = new Properties();
>           properties.load(new FileInputStream("src\\jdbcStudy\\mysql.properties"));
>           String user = properties.getProperty("user");
>           String password = properties.getProperty("password");
>           String url = properties.getProperty("url");
>           String driver = properties.getProperty("driver");
>
>           //给数据源comboPooledDataSource设置相关参数
>           //注意：连接管理是comboPooledDataSource来管理
>           comboPooledDataSource.setDriverClass(driver);
>           comboPooledDataSource.setUser(user);
>           comboPooledDataSource.setPassword(password);
>           comboPooledDataSource.setJdbcUrl(url);
>
>           //设置初始化连接数
>           comboPooledDataSource.setInitialPoolSize(10);
>           //设置最大连接数
>           comboPooledDataSource.setMaxPoolSize(50);
>           //这个方法就是从DateSource接口实现
>           Connection connection = comboPooledDataSource.getConnection();
>           System.out.println("连接成功");
>           //关闭连接
>           connection.close();
>       }
>       //第二种方式  使用配置文件模版来完成
>       //将c3p0提供的c3p0.config.xml拷贝到src目录下
>       //该文件指定了连接数据库和连接池的相关参数
>       @Test
>       public void testC3p02() throws Exception{
>           ComboPooledDataSource comboPooledDataSource = new ComboPooledDataSource("xiaoxu");
>           Connection connection=comboPooledDataSource.getConnection();
>           System.out.println("连接成功");
>           connection.close();
>       }
>   }
>   ```
>
>   ```xml
>   <c3p0-config>
>       <named-config name="xiaoxu">
>           <property name="driverClass">com.mysql.cj.jdbc.Driver</property>
>           <property name="jdbcUrl">jdbc:mysql://localhost:3306/xiaoxu?serverTimezone=UTC</property>
>           <property name="user">root</property>
>           <property name="password">123456</property>
>           <!--每次增长的连接数-->
>           <property name="AcquireIncrement">3</property>
>           <!--初始连接数-->
>           <property name="InitialPoolSize">3</property>
>           <!--最大连接数-->
>           <property name="MaxPoolSize">15</property>
>           <!--最小连接数-->
>           <property name="MinPoolSize">3</property>
>           <!--可连接的最多的命令对象数-->
>           <property name="MaxStatements">5</property>
>           <!--每个连接对象可连接的最多命令对象数-->
>           <property name="MaxStatementsPerConnection">2</property>
>       </named-config>
>
>   </c3p0-config>
>   ```
>
> - 德鲁伊的基本使用
>
>   ```java
>   public class Druid {
>       //引入Druid jar包
>       //加入配置文件druid.properties
>       @Test
>       public void testDruid() throws Exception{
>           Properties properties=new Properties();
>           properties.load(new FileInputStream("src\\druid.properties"));
>           //创建一个指定参数的数据库连接池 Druid的连接池
>           DataSource dataSource = DruidDataSourceFactory.createDataSource(properties);
>           Connection connection = dataSource.getConnection();
>           System.out.println("连接成功");
>           connection.close();
>       }
>   }
>   ```
>   
>   ```properties
>   #注意：datasource有可能不需要加，jdk11需要低版本不知道
>   datasource.driverClassName=com.mysql.cj.jdbc.Driver;
>   url=jdbc:mysql://localhost:3306/xiaoxu?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai&useSSL=false
>   username=root
>   password=123456
>   initialSize=10
>   minIdle=5
>   maxActive=20
>   #最大等待时间
>   maxWait=5000
>   ```
>   
>   ![avatar](https://github.com/xiaoxu137/blog.git.io/blob/main/jpg/JDBC%E5%9F%BA%E7%A1%80/jar.png)
>
> - 德鲁伊封装连接和关闭数据库的方法
>
>   ```java
>   import javax.sql.DataSource;
>   import java.io.FileInputStream;
>   import java.sql.Connection;
>   import java.sql.ResultSet;
>   import java.sql.SQLException;
>   import java.sql.Statement;
>   import java.util.Properties;
>     
>   public class JDBCUtilsByDruid {
>       private static DataSource dataSource;
>       static {
>           Properties properties = new Properties();
>           try {
>               properties.load(new FileInputStream("src\\druid.properties"));
>               dataSource = DruidDataSourceFactory.createDataSource(properties);
>           }
>           catch (Exception e){
>               e.printStackTrace();
>           }
>       }
>       //编写getConnection方法
>       public static Connection getConnection() throws SQLException {
>           return dataSource.getConnection();
>       }
>       //关闭连接，在数据库连接池的技术中，close并不是真的断掉连接
>       //而是把使用的Connection对象放回连接池
>       public static void close(ResultSet resultSet, Statement statement, Connection connection){
>           try {
>               if (resultSet!=null){
>                   resultSet.close();
>               }
>               if (statement!=null){
>                   resultSet.close();
>               }
>               if (connection!=null){
>                   connection.close();
>               }
>           } catch (SQLException e) {
>              throw new RuntimeException();
>           }
>       }
>  ```
>   

## Apache-DBUtils

> - 问题引出
>
>   - 关闭connection后，resultSet结果集无法使用
>
>   - resultSet不利于数据的管理
>
>   - 示意图
>
>    ![avatar](![avatar](https://github.com/xiaoxu137/blog.git.io/blob/main/jpg/JDBC%E5%9F%BA%E7%A1%80/%E5%BC%95%E5%87%BA.png)
>
> - 基本介绍
>   - commons-dbutils是Apache组织提供的一个开源的JDBCg工具类库，它是对JDBC的封装，使用dbutils可以极大简化jdbc编码的工作量
>
> - DbUtils
>
>   - QuertRunner类：该类封装了sql的执行，是线程安全的，可以实现增删改查批处理
>   - 使用QueryRunner类实现查询
>   - ResultSetHandler接口：该接口用于处理java.sal.ResultSet,将数据按要求转换为另一种形式
>
> - 常用类
>
>   - ArrayHandler:把结果的第一行数据转换成对象数组
>
>   - ArrayListHandler:把结果集中的每一行数据都转换成一个数组，再存放到List中
>
>   - BeanHandler:将结果集中的第一行数据封装到一个对应的JavaBean实例中
>
>   - BeanListHandler:将结果集中的每一行数据都封装到一个对应的JavaBean实例中，存放到List里
>
>   - ConlumnListHandler:将结果集中某一列的数据存放到List中
>
>   - KeyedHandler(name):将结果集中的每行数据都封装到Map里，key是列名，value是对应的值
>
>   - MapListHandler:将结果集中的每一行数据都封装到一个Map里，然后再存放到List
>
>   - 代码实现
>
>     ```java
>     package jdbcStudy;
>     
>     
>     import com.mysql.cj.jdbc.JdbcConnection;
>     import org.apache.commons.dbutils.QueryRunner;
>     import org.apache.commons.dbutils.handlers.BeanHandler;
>     import org.apache.commons.dbutils.handlers.BeanListHandler;
>     import org.junit.Test;
>     
>     import java.sql.Connection;
>     import java.sql.SQLException;
>     import java.util.List;
>     
>     //使用apache-DBUtils工具类+druid完成对表的curd的操作
>     
>     /**
>      * 遇到的问题
>      * 1.当时是将SqlTableTest写在DbUtils同一个类中，导致找SqlTableTest.class找不到主类
>      * 2.在JDBCUtilsByDruid类中没有给DataSource赋值
>      * 3.SqlTableTest类中的字段名需要和数据库中的字段名一致，负责会返回null
>      */
>     public class DbUtils {
>         @Test
>         public void testQueryMain() throws SQLException {
>             //得到连接
>             Connection connection=JDBCUtilsByDruid.getConnection();
>             //创建QueryRunner
>             QueryRunner queryRunner=new QueryRunner();
>             String sql="select * from test";
>             //query方法就是执行sql语句，得到resultSet封装到ArrayList集合中
>             //返回集合
>             //connection:连接
>             //sql:执行的sql语句
>             //new BeanListHandler<>(SqlTableTest.class)将resultSet添加到SqlTableTest对象中，然后再封装到ArrayList中
>             //底层使用反射机制
>             // “小白”就是给sql语句中的？赋值，可以有多个值，因为是可变参数
>             //底层得到resultSet,会在query中关闭，关闭preparedStatement
>             List<SqlTableTest> list= queryRunner.query(connection,sql,new BeanListHandler<>(SqlTableTest.class));
>             for (SqlTableTest s:list
>                  ) {
>                 System.out.println(s);
>             }
>             JDBCUtilsByDruid.close(null,null,connection);
>         }
>         @Test
>         //返回单个对象
>         public void aSingle() throws SQLException {
>             Connection connection= JDBCUtilsByDruid.getConnection();
>             QueryRunner queryRunner = new QueryRunner();
>             //如果查询不到 返回的值是null
>             String sql="select * from test where name=?";
>             SqlTableTest sqlTableTest = queryRunner.query(connection, sql, new BeanHandler<>(SqlTableTest.class), "小白");
>             System.out.println(sqlTableTest);
>             JDBCUtilsByDruid.close(null,null,connection);
>         }
>         @Test
>        //dml操作
>         public void dml() throws SQLException {
>             Connection connection=JDBCUtilsByDruid.getConnection();
>             String updateSql="update test set name=? where phoneNumber=?";
>             QueryRunner queryRunner = new QueryRunner();
>             //这个方法执行的可以是update也可以是insert和delete
>             //返回值是受影响的行数
>             int updateAffectedRow = queryRunner.update(connection, updateSql, "苍井空", "222");
>             System.out.println(updateAffectedRow>0?"修改成功":"执行失败");
>             String insertSql="insert into test values(?,?,?)";
>             int insertAffectedRow = queryRunner.update(connection, insertSql, "桥本环奈", "1221", "12343");
>             System.out.println(insertAffectedRow>0?"添加成功":"执行失败");
>             String deleteSql="delete  from test where name=?";
>             int deleteAffectedRow=queryRunner.update(connection,deleteSql,"桥本环奈");
>             System.out.println(deleteAffectedRow>0?"删除成功":"执行失败");
>         }
>     }
>     
>     
>     //这个是SqlTableTest类也就是domain层或是javabean
>     package jdbcStudy;
>     
>     public class SqlTableTest {
>         private String name;
>         private String password;
>         private String phoneNumber;
>     
>         public String getName() {
>             return name;
>         }
>     
>         public void setName(String name) {
>             this.name = name;
>         }
>     
>         public String getPassword() {
>             return password;
>         }
>     
>         public void setPassword(String password) {
>             this.password = password;
>         }
>     
>         public String getPhoneNumber() {
>             return phoneNumber;
>         }
>     
>         public void setPhoneNumber(String phoneNumber) {
>             this.phoneNumber = phoneNumber;
>         }
>     
>         public SqlTableTest() {
>         }
>     
>         public SqlTableTest(String name, String password, String phoneNumber) {
>             this.name = name;
>             this.password = password;
>             this.phoneNumber = phoneNumber;
>         }
>     
>         @Override
>         public String toString() {
>             return "SqlTableTest{" +
>                     "name='" + name + '\'' +
>                     ", password='" + password + '\'' +
>                     ", phoneNumber='" + phoneNumber + '\'' +
>                     '}';
>         }
>     }
>     ```

## BasicDao

> - 问题引出
>
>   - apach-dbutils+Druid简化了JDBC开发，但是还有不足
>
>   - sql语句是固定的，不能通过参数传入，通用性不好，需要进行改造，更加方便的执行增删改查
>
>   - 对于select操作，如果有返回值，返回值类型不能固定，需要使用泛型
>
>   - 将来的表很多，业务需求复杂，不能只通过一个Java类实现
>
>   - 引出Basic示意图
>
>      ![avatar](https://github.com/xiaoxu137/blog.git.io/blob/main/jpg/JDBC%E5%9F%BA%E7%A1%80/%E5%BC%95%E5%87%BA.png)
>
> - 基本说明
>
>   - Dao:data access object数据访问对象
>   - 这样的通用类成为BasicDao,是专门和数据库交互的，即完成对数据库(表)的curd操作
>   - 在BasicDao的基础上，实现一张表对应一个Dao，更好的完成功能，例如Test表对象SqlTableTest.java类(javabean)同时对应TestDao.java
>
> - 代码实现
>
>   - 所有包以及功能
>
>     1. 项目名为basicDaoStudy.dao
>     2. untils工具类
>     3. domain  存放javabean
>     4. dao 存放xxxDao和BasicDao
>     5. test写测试类
>
>     ```java
>     package basicDaoStudy.dao.utils;
>     
>     import com.alibaba.druid.pool.DruidDataSourceFactory;
>     
>     import javax.sql.DataSource;
>     import java.io.FileInputStream;
>     import java.sql.Connection;
>     import java.sql.ResultSet;
>     import java.sql.SQLException;
>     import java.sql.Statement;
>     import java.util.Properties;
>     
>     
>     /**
>      * 工具类
>      */
>     public class JDBCUtilsByDruid {
>         private static DataSource dataSource;
>         static {
>             Properties properties = new Properties();
>             try {
>                 properties.load(new FileInputStream("src\\druid.properties"));
>                 dataSource = DruidDataSourceFactory.createDataSource(properties);
>             }
>             catch (Exception e){
>                 e.printStackTrace();
>             }
>         }
>         //编写getConnection方法
>         public static Connection getConnection() throws SQLException {
>             return dataSource.getConnection();
>         }
>         //关闭连接，在数据库连接池的技术中，close并不是真的断掉连接
>         //而是把使用的Connection对象放回连接池
>         public static void close(ResultSet resultSet, Statement statement, Connection connection){
>             try {
>                 if (resultSet!=null){
>                     resultSet.close();
>                 }
>                 if (statement!=null){
>                     resultSet.close();
>                 }
>                 if (connection!=null){
>                     connection.close();
>                 }
>             } catch (SQLException e) {
>                throw new RuntimeException();
>             }
>         }
>     }
>     
>     ```
>
>     ```java
>     package basicDaoStudy.dao.domain;
>     
>     public class SqlTableTest {
>         private String name;
>         private String password;
>         private String phoneNumber;
>     
>         public String getName() {
>             return name;
>         }
>     
>         public void setName(String name) {
>             this.name = name;
>         }
>     
>         public String getPassword() {
>             return password;
>         }
>     
>         public void setPassword(String password) {
>             this.password = password;
>         }
>     
>         public String getPhoneNumber() {
>             return phoneNumber;
>         }
>     
>         public void setPhoneNumber(String phoneNumber) {
>             this.phoneNumber = phoneNumber;
>         }
>     
>         public SqlTableTest() {
>         }
>     
>         public SqlTableTest(String name, String password, String phoneNumber) {
>             this.name = name;
>             this.password = password;
>             this.phoneNumber = phoneNumber;
>         }
>     
>         @Override
>         public String toString() {
>             return "SqlTableTest{" +
>                     "name='" + name + '\'' +
>                     ", password='" + password + '\'' +
>                     ", phoneNumber='" + phoneNumber + '\'' +
>                     '}';
>         }
>     }
>     
>     ```
>
>     ```java
>     package basicDaoStudy.dao.dao;
>     
>     import basicDaoStudy.dao.utils.JDBCUtilsByDruid;
>     import org.apache.commons.dbutils.QueryRunner;
>     import org.apache.commons.dbutils.handlers.BeanHandler;
>     import org.apache.commons.dbutils.handlers.BeanListHandler;
>     import org.apache.commons.dbutils.handlers.ScalarHandler;
>     import org.junit.Test;
>     
>     import java.sql.Connection;
>     import java.sql.SQLException;
>     import java.util.ArrayList;
>     import java.util.List;
>     
>     /**
>      * 开发BasicDAO,是其他DAO的父类
>      */
>     public class BasicDAO<T> { //泛型指定具体类型
>         private QueryRunner queryRunner=new QueryRunner();
>     
>         //开发通用的dml方法，针对任意的表
>     
>         public int update(String sql,Object...parameters){
>             Connection connection=null;
>             try {
>                 connection= JDBCUtilsByDruid.getConnection();
>                 int update=queryRunner.update(connection,sql,parameters);
>                 return update;
>             } catch (SQLException e) {
>                 throw  new RuntimeException();  //将编译异常转为运行异常跑出
>             }finally {
>                 JDBCUtilsByDruid.close(null,null,connection);
>             }
>         }
>     
>         /**
>          *返回多个对象(查询的结果是多行的)，针对任意表
>          * @param sql  sql语句可以存在？
>          * @param clazz   传入一个类的class对象   例如SqlTableTest.class
>          * @param parameters
>          * @return  返回对应的ArrayList集合
>          */
>         public List<T> queryMulti(String sql,Class<T> clazz,Object...parameters){
>             Connection connection=null;
>             try {
>                 connection=JDBCUtilsByDruid.getConnection();
>                 return queryRunner.query(connection, sql, new BeanListHandler<T>(clazz), parameters);
>             } catch (SQLException e) {
>                 throw new RuntimeException();
>             }finally {
>                 JDBCUtilsByDruid.close(null,null,connection);
>             }
>         }
>     
>         /**
>          * 查询单行结果的通用方法
>          * @param sql
>          * @param clazz
>          * @param parameters
>          * @return
>          */
>         public T querySingle(String sql,Class<T> clazz,Object...parameters){
>             Connection connection=null;
>             try {
>                 connection=JDBCUtilsByDruid.getConnection();
>                return queryRunner.query(connection,sql,new BeanHandler<T>(clazz),parameters);
>             } catch (SQLException e) {
>                 throw new RuntimeException();
>             }finally {
>                 JDBCUtilsByDruid.close(null,null,connection);
>             }
>         }
>         public Object queryScalar(String sql,Object...parameters){
>             Connection connection=null;
>             try {
>                 connection=JDBCUtilsByDruid.getConnection();
>                 return queryRunner.query(connection,sql,new ScalarHandler<>(),parameters);
>             } catch (SQLException e) {
>                 throw new RuntimeException();
>             }finally {
>                 JDBCUtilsByDruid.close(null,null,connection);
>             }
>         }
>     }
>     
>     ```
>
>     ```java
>     package basicDaoStudy.dao.dao;
>     
>     import basicDaoStudy.dao.domain.SqlTableTest;
>     
>     public class TestDao extends BasicDAO<SqlTableTest> {
>         //具有BasicDao所有方法
>         //根据业务需求，可以编写特有的方法
>     }
>     
>     ```
>
>     ```java
>     package basicDaoStudy.dao.Test;
>     
>     import basicDaoStudy.dao.dao.TestDao;
>     import basicDaoStudy.dao.domain.SqlTableTest;
>     import org.junit.Test;
>     
>     
>     import java.util.List;
>     
>     public class TestDaoTest {
>         @Test
>         public void testDao(){
>             TestDao testDao = new TestDao();
>             //查询
>             String sql1="select * from test";
>             List<SqlTableTest> list = testDao.queryMulti(sql1, SqlTableTest.class);
>             //查询单行记录
>             String sql2="select * from test where name=?";
>             SqlTableTest sqlTableTest = testDao.querySingle(sql2, SqlTableTest.class, "小白");
>             System.out.println("单行查询结果"+sqlTableTest);
>             //查询单行单例
>             String sql3="select password from test where name=?";
>             Object password = testDao.queryScalar(sql3, "小白");
>             System.out.println("查询单行单列的结果是"+password);
>             //dml操作
>             String sql="insert into test values(?,?,?)";
>             int i = testDao.update(sql, "小小", "123", "123");
>             if (i>0){
>                 System.out.println("执行成功");
>             }else {
>                 System.out.println("执行失败");
>             }
>         }
>     }
>     ```
>
>     

