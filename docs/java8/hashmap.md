### Java7 与 Java8 中的 HashMap

Java Map 家族谱系：

![img](/assets/26341ef9fe5caf66ba0b7c40bba264a5_720w.png)

**HashMap**：根据键的 hashcode 值存储元素，大多数情况下可以直接定位到它的值，具有很快的访问速度。遍历顺序不确定。HashMap 最多只允许一条键为 NULL 的记录，可以有多条值为 NULL 的记录。HashMap 非线程安全，可以使用 `Collections.synchronizedMap()` 方法，或者转向使用 ConcurrentHashMap。

**Hashtable**：Hashtable 是遗留类，很多 Map 的常用功能与 HashMap 类似，但是，它继承了 `Dictionary` 类，是线程安全的。并发性能不如 ConcurrentHashMap，因为 ConcurrentHashMap 引入了分段锁。不建议在继续使用 Hashtable，可以根据应用场景用 HashMap 或 ConcurrentHashMap 替代。

**LinkedHashMap**：HashMap 的子类，有序，Iterator 遍历时的顺序是元素的插入顺序。也可使用带参构造器，按照访问次数排序。

**TreeMap**：实现 SortedMap 接口，保存的元素按照键排序，默认按键值升序排列，可以自定义排序比较器。Iterator 遍历时的顺序是排序后的顺序。如果要使用有序 Map，建议使用 TreeMap。使用 TreeMap 时，key 必须实现 Comparable 接口或者在构造 TreeMap 时传入自定义的 Comparator，否则会抛出 `java.lang.ClassCastException`。

Java 中基本的数据结构是**数组**和**模拟指针**（引用）。

HashMap 的基本结构，数组 + 链表。key hash 值相同（元素碰撞）的元素构成一个单向链表， 数组存储的是链表的头节点。

JDK7 HashMap 结构为数组 + 链表（发生元素碰撞时，会将新元素添加到链表的开头）

JDK8 HashMap 结构为数组 + 链表 + 红黑树（发生元素碰撞时，会将新元素添加到链表末尾，当 HashMap 总容量大于等于 64，并且某个链表的大小大于等于 8，会将链表转化为红黑树）

#### 1、JDK8 HashMap 重排序

如果删除了 HashMap 中红黑树的某个元素导致元素重排序时，不需要计算贷重排的元素的 HashCode，只需要将当前元素放到（HashMap 总长度 + 当前元素在 HashMap 中的位置）的位置即可。