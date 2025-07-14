### 线程池

线程池可以实现线程的复用，节省创建线程的开销。

```java
ExecutorService executor =  ...;

executor.execute(new Runnable(){...});

// 1. 关闭线程池，拒绝新任务（不会终止正在执行的任务，也不会阻塞等待执行中的任务执行完成）
executor.shutdown();
try {
	// 2. 等待所有任务完成（到达指定时间前任务执行完毕返回 true，否则返回 false）
	if (!executor.awaitTermination(30, TimeUnit.SECONDS)) {
		// 3. 尝试强制终止未完成的任务
		executor.shutdownNow();
		// 4. 再次等待终止
		executor.awaitTermination(30, TimeUnit.SECONDS);
	}
} catch (InterruptedException e) {
	executor.shutdownNow();
	// 恢复中断状态
	Thread.currentThread().interrupt();
}
```

需要注意：

1. 不要一次性提交太多的任务到线程池，提交的任务也会占用内存，可能会导致 OOM；如果任务很多，可以分批次的提交，一批执行完成后（可以使用 `CountDownLatch` 监控是否执行完成），再提交下一批。
2. 线程池不会自动关闭，如果任务执行完毕后，进程需要结束退出，那么应该显式的关闭线程池，否则进程不会束。

