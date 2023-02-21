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
>    ![avatar](![avatar](https://github.com/xiaoxu137/blog.git.io/blob/main/jpg/JDBC%E5%9F%BA%E7%A1%80/mulu.png))
>
> - 基本介绍
>   - commons-dbutils是Apache组织提供的一个开源的JDBCg工具类库，它是对JDBC的封装，使用dbutils可以极大简化jdbc编码的工作量
>
> - DbUtils
>   - QuertRunner类：该类封装了sql的执行，是线程安全的，可以实现增删改查批处理
>   - 使用QueryRunner类实现查询
>   - ResultSetHandler接口：该接口用于处理java.sal.ResultSet,将数据按要求转换为另一种形式
> - 常用类
>   - ArrayHandler:把结果的第一行数据转换成对象数组
>   - ArrayListHandler:把结果集中的每一行数据都转换成一个数组，再存放到List中
>   - BeanHandler:将结果集中的第一行数据封装到一个对应的JavaBean实例中
>   - BeanListHandler:将结果集中的每一行数据都封装到一个对应的JavaBean实例中，存放到List里
>   - ConlumnListHandler:将结果集中某一列的数据存放到List中
>   - KeyedHandler(name):将结果集中的每行数据都封装到Map里，key是列名，value是对应的值
>   - MapListHandler:将结果集中的每一行数据都封装到一个Map里，然后再存放到List

