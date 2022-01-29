### Optional

`java.util.Optional` 是一个容器类，表示一个值存在或不存在，可以避免空指针异常。

常用方法：

- `Optional.of(T t)`：创建一个 Optional 实例，`t` 不可为 `null`（否则抛出空指针异常）

- `Optional.empty()`：创建一个空的 Optional 实例

- `Optional.ofNullable(T t)`：若 t 不为 null，创建 Optional 实例，否则创建空实例

- `get()`：返回 `Optional` 对象包含的值，如果值不存在，则抛出 `NoSuchElementException` 异常

- `isPresent()`：判断 `Optional` 对象中是否有值，只有包含非空值才返回 `true`

- `orElse(T t)`：如果调用对象包含值，返回该值，否则返回 t

- `orElseGet(Supplier s)`：如果调用对象包含值，返回该值，否则返回 s 获取的值

  与 `orElse(T t)` 相比，如果 `t` 是一个表达式，那么，无论 `Optional` 对象是否包含值，`t` 表达式都会执行；而 `orElseGet(Supplier s)` 的 `s` 只有在 `Optional` 对象不包含值时才执行。

- `orElseThrow(Supplier exs)`：如果调用对象包含值，返回该值，否则抛出异常

- `filter(Predicate p)`：如果有值符合函数式接口 `p`，返回的 Optional 对象，否则返回 `Optional.empty()`

- `map(Function f)`：如果有值对其处理，返回处理后的 Optional，否则返回 `Optional.empty()`

- `flatMap(Function mapper)`：与 map 类似，返回值必须是 Optional

