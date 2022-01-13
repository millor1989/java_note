### Lambda 表达式

#### 1、概念

Lambda 表达式是一个匿名函数。可以将其理解为一段可传递的代码（可以做到将代码像数据一样传递）。

#### 2、匿名内部类

使用匿名内部类比较 Integer 类型数据的大小：

```java
Comparator<Integer> com = new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
    	return Integer.compare(o1, o2);
    }
}
```

可以将其实例作为参数传递到其他方法中：

```java
TreeSet<Integer> treeSet = new TreeSet<>(com);
```

其实，整个匿名内部类中，真正有用的代码是 `Integer.compare(o1, o2)`，其它代码显得“冗余”。

#### 3、Lambda 表达式

Integer 类型数据比较的 Lambda 实现：

```java
Comparator<Integer> com = (x, y) -> Integer.compare(x, 7);
```

相比之前的写法，简洁了很多。

Lambda 表达式可以作为参数传递到 TreeSet 的构造方法中：

```java
TreeSet<Integer> treeSet = new TreeSet<>((x, y) -> Integer.compare(x, y));
```

#### 4、对比常规方法和 Lambda 表达式

需求，获取公司员工中年龄大于 30 岁的员工信息。

```java
@Data
@Builder
@ToString
@NoArgsConstructor
@AllArgsConstructor
public class Employee implements Serializable {
	private static final long serialVersionUID = -9079722457749166858L;
	private String name;
	private Integer age;
	private Double salary;
}
```

