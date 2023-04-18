### Try With Source

JDK 1.7 增加了 try-with-source 语法。在 try 中声明一个或者多个资源，在 try 块代码执行完成后自动关闭流，不用再写 `close()` 进行手动关闭。

1. 可以有多个 source
2. 声明的 source 变量如果没有显式的声明为 final，则会被隐式地转换为 final 类型。

```java
try (Connection conn = connPool.getConnection();
     PreparedStatement stmt = conn.prepareStatement("sql");) {
    stmt.setString(1, value.getN_essay_id());
    stmt.setString(2, value.getPage_channel());
} catch (Exception e) {
    logger.error(e);
}
```

