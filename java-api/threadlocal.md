### [ThreadLocal](https://mp.weixin.qq.com/s/I74YzUqoyMek-BPvHMu2qQ)

**线程本地变量**，使用其能够将数据封闭在各自的线程中，线程间数据隔离。

`ThreadLocal` 能够存放一个线程级别的变量，它本身能够被多个线程共享使用，并且绝对线程安全。

`ThreadLocal` 可以用于”传递参数“（比如，trace_id、session_id），避免多次用到的参数层层传递。

##### `ThreadLocal.set()`

```java
    /**
     * Sets the current thread's copy of this thread-local variable
     * to the specified value.  Most subclasses will have no need to
     * override this method, relying solely on the {@link #initialValue}
     * method to set the values of thread-locals.
     *
     * @param value the value to be stored in the current thread's copy of
     *        this thread-local.
     */
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
```

在向 `ThreadLocal` 中存放值时，先从当前线程中获取 `ThreadLocalMap`，最后实际是把当前 `ThreadLocal` 对象作为 key、要存入的值作为 `value` 存放到 `ThreadLocalMap`。

##### `ThreadLocalMap`

`ThreadLocalMap` 是 `ThreadLocal` 的静态内部类，`ThreadLocalMap` 的作用就是管理线程中的多个 `ThreadLocal`：

```java
    /**
     * ThreadLocalMap is a customized hash map suitable only for
     * maintaining thread local values. No operations are exported
     * outside of the ThreadLocal class. The class is package private to
     * allow declaration of fields in class Thread.  To help deal with
     * very large and long-lived usages, the hash table entries use
     * WeakReferences for keys. However, since reference queues are not
     * used, stale entries are guaranteed to be removed only when
     * the table starts running out of space.
     */
    static class ThreadLocalMap {

        /**
         * The entries in this hash map extend WeakReference, using
         * its main ref field as the key (which is always a
         * ThreadLocal object).  Note that null keys (i.e. entry.get()
         * == null) mean that the key is no longer referenced, so the
         * entry can be expunged from table.  Such entries are referred to
         * as "stale entries" in the code that follows.
         */
        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }

        /**
         * The initial capacity -- MUST be a power of two.
         */
        private static final int INITIAL_CAPACITY = 16;

        /**
         * The table, resized as necessary.
         * table.length MUST always be a power of two.
         */
        private Entry[] table;

        /**
         * The number of entries in the table.
         */
        private int size = 0;

        /**
         * The next size value at which to resize.
         */
        private int threshold; // Default to 0

        /**
         * Set the resize threshold to maintain at worst a 2/3 load factor.
         */
        private void setThreshold(int len) {
            threshold = len * 2 / 3;
        }

        .
        .
        .
    }
```

##### `ThreadLocal.get()`

```java
    /**
     * Returns the value in the current thread's copy of this
     * thread-local variable.  If the variable has no value for the
     * current thread, it is first initialized to the value returned
     * by an invocation of the {@link #initialValue} method.
     *
     * @return the current thread's value of this thread-local
     */
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
```

##### `ThreadLocal.remove()`

`remove()` 方法实际调用了 `ThreadLocalMap.remove()` 方法：

```java
        /**
         * Remove the entry for key.
         */
        private void remove(ThreadLocal<?> key) {
            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);
            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                if (e.get() == key) {
                    e.clear();
                    expungeStaleEntry(i);
                    return;
                }
            }
        }
```

`int i = key.threadLocalHashCode & (len-1)` 很巧妙，从数组中找到一个元素存放位置的最简单方法就是利用该元素的 `hashcode` 对这个集合的长度取余。`ThreadLocalMap` 将容量限制为 2 的整数次幂，从而可以将取余运算转换为 `hashcode` 和 `集合长度 - 1` 的与运算，能够提高查询效率。

##### 内存泄露问题

如果垃圾回收将 ThreadLocal 对象回收，`ThreadLocalMap` 中对应 `Entry` 的 key 为变为 `null`，但是值仍然是存在的；如果线程一直未被销毁，value 就无法被回收，就会出现内存泄露。

```java
ThreadLocal<String> threadLocal = new ThreadLocal<>();
threadLocal.set("bulala");
threadLocal = null;
System.gc();
```

出现内存泄漏的原因是失去了对 ThreadLocal 对象的强引用，避免内存泄漏最简单的方法就是始终保持对ThreadLocal 对象的强引用，可以将 ThreadLocal 对象声明为一个全局常量。所有的线程共用此对象，由于此对象始终存在着一个全局的强引用，所以其不会被垃圾回收。