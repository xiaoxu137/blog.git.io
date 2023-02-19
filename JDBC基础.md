# JDBC基础

## 1.通过代码实现连接数据库

```java
public static void connection1(){
        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
            String url="jdbc:mysql://localhost:3306/xiaoxu?useSSL=false&serverTimezone=UTC";//在MySQL8.0之后必须写时区
            String user="root";
            String password="123456";
            Connection connection= DriverManager.getConnection(url,user,password);
            System.out.println("连接成功");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
```

## 2.通过配置文件获取连接数据库

```java
public static void connection2() {
        Properties properties = new Properties();
        try {
            properties.load(new FileInputStream("src\\jdbcStudy\\mysql.properties"));
            String user = properties.getProperty("user");
            String password = properties.getProperty("password");
            String url = properties.getProperty("url");
            String driver = properties.getProperty("driver");
            Class.forName(driver);
            Connection connection = DriverManager.getConnection(url, user, password);
            System.out.println("连接成功");
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
```

```properties
url=jdbc:mysql://localhost:3306/xiaoxu?useSSL=false&serverTimezone=UTC
user=root
password=123456
driver=com.mysql.cj.jdbc.Driver
#properties存储方式为键值对  通过获取properties的key值来获取其value
```

目录结构

 ![image-20230218105125296](C:\Users\CW\AppData\Roaming\Typora\typora-user-images\image-20230218105125296.png)

## 3.数据库的增删改查

```Java
  public static void connection2() {
        /**
         * 简单的sql注入问题
         * Select * from admin where name="admin" and password="123456";
         *  此时数据存在
         *  当将name修改为   1 'or'
         *  password修改为 or '1'='1'
         *  这时的查找结果仍然存在
         *  这就是主要不使用Statement的主要原因
         *  使用preparedStatement可以有效防止注入
         */
        Properties properties = new Properties();
        try {
            properties.load(new FileInputStream("src\\jdbcStudy\\mysql.properties"));
            String user = properties.getProperty("user");
            String password = properties.getProperty("password");
            String url = properties.getProperty("url");
            String driver = properties.getProperty("driver");
            Class.forName(driver);
            Connection connection = DriverManager.getConnection(url, user, password);
            //sql语句  这里使用了占位符
            String sql = "Select name,password from test where name=? and password=?";
            //创建PreparedStatement对象
            PreparedStatement preparedStatement = connection.prepareStatement(sql);
            //给占位符赋值
            preparedStatement.setString(1, "xiaoxu");
            preparedStatement.setString(2, "123456");
            //执行查询  注意：查询使用的是executeQuery();
            //如果是(add,update,delete)使用的则是executeUpdate()
            //上面已经给preparedStatement对象赋值  所以不需要填写sql
            ResultSet resultSet = preparedStatement.executeQuery();
            if (resultSet.next()) {
                System.out.println("登录成功");
            } else {
                System.out.println("登录失败");
            }
            //关闭连接
            resultSet.close();
            preparedStatement.close();
            connection.close();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    /**
     * 数据库的增删改操作
     */
    public static void dml() {
        Properties properties = new Properties();
        try {
            properties.load(new FileInputStream("src\\jdbcStudy\\mysql.properties"));
            String user = properties.getProperty("user");
            String password = properties.getProperty("password");
            String url = properties.getProperty("url");
            String driver = properties.getProperty("driver");
            Class.forName(driver);
            Connection connection = DriverManager.getConnection(url, user, password);

            //数据库的增加
            String add="insert into test values(?,?,?)";
            //数据库的删除
            String delete="delete from test where name=?";
            //数据库的修改
            String update ="update test set password =? where name=?";
            //依次问号赋值并且创建preparedStatement对象即可
            PreparedStatement preparedStatement=connection.prepareStatement(add);
            preparedStatement.setString(1,"小王");
            preparedStatement.setString(2,"1234565");
            preparedStatement.setString(3,"444");
           int addRow=preparedStatement.executeUpdate();//返回受影响的行数
            if (addRow>0){
                System.out.println("执行成功");
            }
            else {
                System.out.println("执行失败");
            }
             //关闭连接
            resultSet.close();
            preparedStatement.close();
            connection.close();
        } catch (SQLException e) {
            e.printStackTrace();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
```

## 4.JDBC API导图

![](D:\DeskTop\JDBC API.jpg)