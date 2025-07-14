### [JDBC](https://www.tutorialspoint.com/jdbc/index.htm)

Java Database Conectivity（JDBC）是 Java 访问数据库的 API。

#### 基础用法

1. 注册 JDBC Driver
2. 获取连接
3. 执行操作
4. 关闭连接

```java
// 注册 jdbc driver——方式一
Class.forName("oracle.jdbc.driver.OracleDriver");
/*
注册 jdbc driver——方式二
Driver myDriver = new oracle.jdbc.driver.OracleDriver();
DriverManager.registerDriver( myDriver );
*/
// 获取连接，try-with-resource 自动关闭连接
try(Connection conn = DriverManager.getConnection(DB_URL, USER, PASS);
    Statement stmt = conn.createStatement();
    ResultSet rs = stmt.executeQuery(QUERY);) {
    // 从 result set 提取数据
    while (rs.next()) {
        // 按名称提取数据
        System.out.print("ID: " + rs.getInt("id"));
        System.out.print(", Age: " + rs.getInt("age"));
        System.out.print(", First: " + rs.getString("first"));
        System.out.println(", Last: " + rs.getString("last"));
    }
} catch (SQLException e) {
    e.printStackTrace();
} 
```

#### 事务

ACID：原子性（Atomicity，一组操作要么都成功，要么都失败）、一致性（Consistency，只有一致的数据才保存到数据库，出错时回到原始状态）、隔离性（Isolation，事务之间互相隔离——隔离级别可以是：read uncommitted、read committed、repeatable read、serializable）、持久性（Durablility，完成的事务会将数据持久化）。

1. `setAutoCommit(false)` 关闭自动提交模式
2. 执行结束要手动 `commit()`
3. 发生错误或者异常，必须回滚 `rollback()`

```java
try{
   // 关闭 auot commit 模式
   conn.setAutoCommit(false);
   Statement stmt = conn.createStatement();
   
   String SQL = "正常 sql...";
   stmt.executeUpdate(SQL);  
   String eSQL = "异常 sql...";
   stmt.executeUpdate(eSQL);
   // 提交更改
   conn.commit();
}catch(SQLException se){
   // 异常回滚
   conn.rollback();
}
```

#### 批处理

批处理只能用于更新或插入

1. 关闭 auto commit 模式（不是必须的，但是推荐关闭——因为可以提高性能）
2. 获取 `Statement` / `PreparedStatement`
3. 逐记录 `addBatch()`
4. 需要执行批次时 `executeBatch()`
5. 执行过一个批次后 `commit()` 提交——如果数据量非常大，可以每达到一定数量时 `executeBatch()` 执行批次并 `commit()` 提交一次，如果提交后还有待执行的批次需要 `clearBatch()` 后再 `addBatch()`
6. 异常时 `rollback()`

#### 连接池

获取数据库连接是一个高开销的操作，为了提高获取连接的效率可以使用连接池。

常见连接池：HikariCP（性能最佳）、Druid（性能优秀，带监控）、Apache Commons DBCP、c3p0。

使用连接池的好处不只获取连接效率高，还有避免连接超时等等好处——可参考具体的连接池实现。

- 连接池是**复用**连接的，从连接池获取连接的 `close()` 是将连接归还连接池，而不是释放连接
- 批处理失败时，必须回滚，如果不回滚事务会生成**“脏”连接**，复用这个“脏”连接，可能导致行为不可预测

#### 异常

连接超时断开

The last packet successfully received from the server was 79,834,920 milliseconds ago. The last packet sent successfully to the server was 79,834,920 milliseconds ago. is longer than the server configured value of 'wait_timeout'. You should consider either expiring and/or testing connection validity before use in your application, increasing the server configured values for client timeouts, or using the Connector/J connection property 'autoReconnect=true' to avoid this problem.