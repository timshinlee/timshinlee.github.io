# Executor
Executor用来抽象出任务执行的过程，将任务提交和任务处理两者相解耦。子类可以根据需求进行任务执行方式的实现，例如在新线程中执行任务，或者在线程池中执行任务，又或者在当前线程中执行。Executor的定义如下：

```java
public interface Executor {
    void execute(Runnable command);
}
```
