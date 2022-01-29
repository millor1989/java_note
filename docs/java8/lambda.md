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

员工实体类：

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

创建员工集合：

```java
protected List<Employee> employees = Arrays.asList(
	new Employee("张三", 18, 9999.99),
    new Employee("李四", 38, 5555.55),
    new Employee("王五", 60, 6666.66),
    new Employee("赵六", 16, 7777.77),
    new Employee("田七", 18, 3333.33)
);
```

##### 4.1、常规遍历集合

用 Java 方法通过遍历集合的方式来实现需求：

```java
public List<Employee> filterEmployeesByAge(List<Employee> list){
	List<Employee> employees = new ArrayList<>();
	for(Employee e : list){
		if(e.getAge() >= 30){
			employees.add(e);
		}
	}
	return employees;
}
```

如果需求变化——获取工资大于等于 5000 的员工的信息，可以用如下方法通过遍历集合的方式实现：

```java
public List<Employee> filterEmployeesBySalary(List<Employee> list){
	List<Employee> employees = new ArrayList<>();
	for(Employee e : list){
		if(e.getSalary() >= 5000){
			employees.add(e);
		}
	}
	return employees;
}
```

对比两个方法，其实只有 for 循环中条件判断不同而已！

##### 4.2、使用设计模式进行优化

定义泛型接口 `MyPredicate`：

```java
public interface MyPredicate<T> {
	/**
	* 对传递过来的 T 类型的数据进行过滤
	* 符合规则返回 true，不符合规则返回 false
	*/
	boolean filter(T t);
}
```

创建 `MyPredicate` 的实现类 `FilterEmployeeByAge` 来过滤年龄大于或者等于 30 的员工：

```java
public class FilterEmployeeByAge implements MyPredicate<Employee> {
	@Override
	public boolean filter(Employee employee) {
		return employee.getAge() >= 30;
	}
}
```

此时，可以对过滤方法进行优化：

```java
// 优化方式一
public List<Employee> filterEmployee(List<Employee> list, MyPredicate<Employee> myPredicate) {
	List<Employee> employees = new ArrayList<>();
	for(Employee e : list){
		if(myPredicate.filter(e)){
			employees.add(e);
		}
	}
	return employees;
}
```

那么，当需要过滤工资大于或者等于 5000 的员工是，只用创建一个实现 `MyPredicate` 接口的 `FilterEmployeeBySalary` 类即可：

```java
public class FilterEmployeeBySalary implements MyPredicate<Employee>{
    @Override
    public boolean filter(Employee employee) {
    	return employee.getSalary() >= 5000;
    }
}
```

此外，如果过滤条件变更，只需创建新的 `MyPredicate` 实现类即可。

##### 4.3、匿名内部类

既然 `MyPredicate` 是接口，那么，可以使用匿名内部类对代码进行简化：

```java
List<Employee> employeeList = filterEmployee(
	employees,
    new MyPredicate<Employee>() {
        @Override
        public boolean filter(Employee employee) {
        	return employee.getSalary() >= 5000;
        }
    }
);
for (Employee e : employeeList){
	System.out.println(e);
}
```

##### 4.4、Lambda 表达式

进一步使用 Lambda 表达式进行简化

```java
filterEmployee(employees, (e) -> e.getAge() >= 30).forEach(System.out::println);
```

使用 Lambda 表达式只需要一行代码就完成了员工信息的过滤和输出。

如果需求变更，只需改变 Lambda 表达式即可。

##### 4.5、Stream API

使用 Lambda 表达式结合 Stream API，可以完成对集合的各种过滤。

```java
employees.stream().filter((e) -> e.getSalary() >= 5000).forEach(System.out::println);
```

#### 5、匿名类到 Lambda 表达式

- 匿名内部类 到 Lambda 表达式

  匿名内部类代码：

  ```java
  Runnable r = new Runnable(){
      @Override
      public void run(){
      	System.out.println("Hello Lambda");
      }
  }
  ```

  Lambda 表达式表示：

  ```java
  Runnable r = () -> System.out.println("Hello Lambda"); 
  ```

- 匿名内部类作为参数传递 到 Lambda 表达式作为参数传递

  匿名内部类作为参数：

  ```java
  TreeSet<Integer> ts = new TreeSet<>(new Comparator<Integer>(){
      @Override
      public int compare(Integer o1, Integer o2){
      	return Integer.compare(o1, o2);
      }
  });
  ```

  Lambda 表达式作为参数：

  ```java
  TreeSet<Integer> ts = new TreeSet<>(
  	(o1, o2) -> Integer.compare(o1, o2);
  );
  ```

#### 6、Lambda 表达式语法

Lambda 表达式在 Java 语言中引入了 `->` 操作符，`->` 操作符称为 Lambda 表达式的操作符或者箭头操作符，它将 Lambda 表达式分为两部分：

- 左侧部分指定了 Lambda 表达式所需要的参数；

  Lambda 表达式本质上是对接口的实现，Lambda 表达式的参数列表本质上对应着接口中方法的参数列表。

- 右侧部分指定了 Lambda 体，即 Lambda 表达式要执行的功能。

##### 语法：

1. ##### 无参、无返回值

   Lambda 体只有一条语句：

   ```java
   Runnable r = () -> System.out.println("Hello Lambda");
   ```

2. ##### 一个参数、无返回值

   ```java
   Consumer<String> func = (s) -> System.out.println(s);
   ```

3. ##### 只有一个参数时，参数的小括号可以省略

   ```java
   Consumer<String> func = s -> System.out.println(s);
   ```

4. ##### 两个参数，有返回值

   ```java
   BinaryOperator<Integer> bo = (a, b) -> {
   	System.out.println("函数式接口");
   	return a + b;
   };
   ```

5. ##### Lambda 体只有一条语句时，`return` 和花括号可以省略

   ```java
   BinaryOperator<Integer> bo = (a, b) -> a + b;
   ```

6. ##### Lambda 表达式的参数列表的数据类型可以省略

   JVM 编译器能够通过上下文推断出参数的数据类型。

   ```java
   BinaryOperator<Integer> bo = (Integer a, Integer b) -> {
   	return a + b;
   };
   ```

   等同于：

   ```java
   BinaryOperator<Integer> bo = (a, b) -> {
   	return a + b;
   };
   ```

#### 7、函数式接口

Lambda 表达式需要函数式接口的支持。

只包含一个抽象方法的接口，称为**函数式接口**。

可以通过 Lambda 表达式来创建函数式接口的对象。（若 Lambda 表达式抛出一个受检异常，那么该异常需要在目标接口的抽象方法上进行声明）

可以在任意函数式接口上使用 `@FunctionalInterface` 注解，这样做可以检查它是否是一个函数式接口，同时 javadoc 也会包含一条声明，说明这个借口是一个函数式接口。

可以自定义函数式接口，并使用 Lambda 表达式来实现相应的功能。

例如，定义一个函数式接口：

```java
@FunctionalInterface
public interface MyFunc <T> {
	public T getValue(T t);
}
```

定义一个进行字符串处理的方法：

```java
public String handlerString(MyFunc<String> myFunc, String str){
	return myFunc.getValue(str);
}
```

使用 Lambda 表达式实现将字符串转换为大写的功能：

```java
handlerString((s) -> s.toUpperCase(), "lambda");
```

