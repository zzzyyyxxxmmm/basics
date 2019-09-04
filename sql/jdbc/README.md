# JDBC 源码阅读

## 实现
```java
@Test
    public void testJDBC() throws SQLException, ClassNotFoundException {
        Class.forName("com.mysql.cj.jdbc.Driver");
        Connection connection=DriverManager.getConnection("jdbc:mysql://localhost:3306/test?serverTimezone=UTC","root","root");
        Statement statement=connection.createStatement();
        ResultSet resultSet=statement.executeQuery("select * from student");
        while(resultSet.next()){
            String name=resultSet.getString("name");
            System.out.println(name);
        }
        connection.close();
    }
```
首先看一下我们是怎么调用mysql实现的jdbc的

### 第一行：```Class.forName("com.mysql.cj.jdbc.Driver");``` Load class Driver

In Driver class:
```java
static {
        try {
            java.sql.DriverManager.registerDriver(new Driver());
        } catch (SQLException E) {
            throw new RuntimeException("Can't register driver!");
        }
    }
```
It contains a static block, it will be excuted when class is loaded

Then in class DriveManager:
```java
// List of registered JDBC drivers
private final static CopyOnWriteArrayList<DriverInfo> registeredDrivers = new CopyOnWriteArrayList<>();
registerDriver(Driver driver){
    if (driver != null) registeredDrivers.addIfAbsent(new DriverInfo(driver, da));
}
```

What is Driver info?
```java
class DriverInfo {
    final Driver driver;
    DriverAction da;
}
```

So, It just put the driver into an arraylist called registeredDrivers

It seems we finish tracing the first lines

### second line
```java
Connection connection=DriverManager.getConnection("jdbc:mysql://localhost:3306/test?serverTimezone=UTC","root","root");
```

The getConnnection method will encapsulate the user and password in java.util.properties, then call the following method:

```java
   private static Connection getConnection(
        String url, java.util.Properties info, Class<?> caller) throws SQLException {
            /*
            normally, the classloader is Appclassloader as we expect
            */
        ClassLoader callerCL = caller != null ? caller.getClassLoader() : null;
        if (callerCL == null) {
            callerCL = Thread.currentThread().getContextClassLoader();
        }

        if (url == null) {
            throw new SQLException("The url cannot be null", "08001");
        }

        println("DriverManager.getConnection(\"" + url + "\")");

        ensureDriversInitialized();     //As what it said, don't dive into it.

        // Walk through the loaded registeredDrivers attempting to make a connection.
        // Remember the first exception that gets raised so we can reraise it.
        SQLException reason = null;

        for (DriverInfo aDriver : registeredDrivers) {
            // If the caller does not have permission to load the driver then
            // skip it.
            if (isDriverAllowed(aDriver.driver, callerCL)) {
                try {
                    println("    trying " + aDriver.driver.getClass().getName());
                    Connection con = aDriver.driver.connect(url, info); //get connection here，最终还是要通过driver调用connect的，所以driver是必须创建的
                    if (con != null) {
                        // Success!
                        println("getConnection returning " + aDriver.driver.getClass().getName());
                        return (con);
                    }
                } catch (SQLException ex) {
                    if (reason == null) {
                        reason = ex;
                    }
                }

            } else {
                println("    skipping: " + aDriver.getClass().getName());
            }

        }

        // if we got here nobody could connect.
        if (reason != null)    {
            println("getConnection failed: " + reason);
            throw reason;
        }

        println("getConnection: no suitable driver found for "+ url);
        throw new SQLException("No suitable driver found for "+ url, "08001");
    }


}
```

```java
ConnectionUrl conStr = ConnectionUrl.getConnectionUrlInstance(url, info);
            switch (conStr.getType()) {
                case SINGLE_CONNECTION:
                    return com.mysql.cj.jdbc.ConnectionImpl.getInstance(conStr.getMainHost());

                case LOADBALANCE_CONNECTION:
                    return LoadBalancedConnectionProxy.createProxyInstance((LoadbalanceConnectionUrl) conStr);

                case FAILOVER_CONNECTION:
                    return FailoverConnectionProxy.createProxyInstance(conStr);

                case REPLICATION_CONNECTION:
                    return ReplicationConnectionProxy.createProxyInstance((ReplicationConnectionUrl) conStr);

                default:
                    return null;
            }
```

ok, here we create a instance of connection. For more info about these types, please refer to [multi-host-connections](!https://dev.mysql.com/doc/connector-j/5.1/en/connector-j-multi-host-connections.html).

Normally, we will call com.mysql.cj.jdbc.ConnectionImpl.getInstance(conStr.getMainHost()); This method will finally return a instance of ConnectionImpl.class

### Third Line Statement statement=connection.createStatement();
```java
@Override
    public java.sql.Statement createStatement(int resultSetType, int resultSetConcurrency) throws SQLException {

        StatementImpl stmt = new StatementImpl(getMultiHostSafeProxy(), this.database);
        stmt.setResultSetType(resultSetType);
        stmt.setResultSetConcurrency(resultSetConcurrency);

        return stmt;
    }
```

这里涉及一些参数，应该也是一些其他框架实现其特殊功能的方法

ResultSet.TYPE_FORWORD_ONLY 结果集的游标只能向下滚动。 

ResultSet.TYPE_SCROLL_INSENSITIVE 结果集的游标可以上下移动，当数据库变化时，当前结果集不变。 

ResultSet.TYPE_SCROLL_SENSITIVE 返回可滚动的结果集，当数据库变化时，当前结果集同步改变。 

参数 int concurrency 

ResultSet.CONCUR_READ_ONLY 不能用结果集更新数据库中的表。 

ResultSet.CONCUR_UPDATETABLE 能用结果集更新数据库中的表。 

```java
Statement statement=connection.createStatement(ResultSet.TYPE_FORWARD_ONLY,ResultSet.CONCUR_UPDATABLE);
ResultSet resultSet=statement.executeQuery("select * from student");
while(resultSet.next()){
    String name=resultSet.getString("name");
    resultSet.updateString("name","wang");
    resultSet.updateRow();
}


 @Override
public java.sql.Statement createStatement(int resultSetType, int resultSetConcurrency) throws SQLException {

    StatementImpl stmt = new StatementImpl(getMultiHostSafeProxy(), this.database);
    stmt.setResultSetType(resultSetType);
    stmt.setResultSetConcurrency(resultSetConcurrency);

    return stmt;
}
```
如果是默认的话，这里就会报错

总之就是创建了一个StatementImpl对象，

#  ResultSet resultSet=statement.executeQuery("select * from student");



