### 函数式接口

四大类核心函数式接口——`Consumer`、`Supplier`、`Function`、`Predicate`。

#### 1、`Consumer` 消费型接口

无返回值，对类型为 T 的对象应用操作：

```java
@FunctionalInterface
public interface Consumer<T> {

    void accept(T t);

    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```

#### 2、`Supplier` 供给型接口

无参数，返回类型为 T 的对象：

```java
@FunctionalInterface
public interface Supplier<T> {

    T get();
}
```

#### 3、`Function<T, R>` 函数式接口

对类型为 T 的对象应用操作，并返回 R 类型的值：

```java
@FunctionalInterface
public interface Function<T, R> {

	R apply(T t);

	default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
    	Objects.requireNonNull(before);
    	return (V v) -> apply(before.apply(v));
    }

	default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
    	Objects.requireNonNull(after);
    	return (T t) -> after.apply(apply(t));
    }

	static <T> Function<T, T> identity() {
    	return t -> t;
    }
}
```

#### 3、Predicate 断言型接口

确定类型为 T 的对象是否满足约束条件，并返回 boolean 类型的结果：

```java
@FunctionalInterface
public interface Predicate<T> {

    boolean test(T t);

    default Predicate<T> and(Predicate<? super T> other) {
    	Objects.requireNonNull(other);
    	return (t) -> test(t) && other.test(t);
    }

    default Predicate<T> negate() {
    	return (t) -> !test(t);
    }

    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
	}

    static <T> Predicate<T> isEqual(Object targetRef) {
    	return (null == targetRef) ? Objects::isNull : object -> targetRef.equals(object);
    }
}
```

#### 4、其它函数接口

`BitFunction(T, U, R)`、`UnaryOperator`、`BinaryOperator`、`BiConsumer`等等……