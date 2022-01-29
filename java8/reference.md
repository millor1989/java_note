### 引用

#### 1、方法引用

当要传递给 Lambda 体的操作已经有了实现的方法，可以使用方法引用。实现抽象方法的参数列表，必须与方法引用的方法的参数列表保持一致。

方法引用操作符 `::`，有三种情况：

- 对象 `::` 实例方法

  ```java
  (x) -> System.out.println(x);
  
  System.out::println
  ```

- 类 `::` 静态方法

  ```java
  BinaryOperator<Double> bo = (x, y) -> Math.pow(x, y);
  
  BinaryOperator<Double> bo = Math::pow;
  ```

- 类 `::` 实例方法

  当需要引用方法的第一个参数是方法的调用对象，并且第二个参数是需要引用方法的第二个参数（或无参数）时，`ClassName :: methodName`：

  ```java
  compare((x, y) -> x.equals(y), "lambda", "lambda")
  
  compare(String::equals, "lambda", "lambda")
  ```

#### 2、构造器引用

格式：`<ClassName> :: new`

与函数式接口相结合，自动与函数式接口中方法兼容。可以把构造器引用赋值给定义的方法，构造器参数列表要与接口中抽象方法的参数列表一致！

比如：

```java
Function<Integer, MyClass> fun = (n) -> new MyClass(n);

Function<Integer, MyClass> fun = MyClass::new;
```

#### 3、数组引用

格式：`<type>[] :: new`

```java
Function<Integer, Integer[]> fun = (n) -> new Integer[n];

Function<Integer, Integer[]> fun = Integer[]::new;
```

