# 多线程

## 线程创建

java中只有用Thread类的start()方法才能创建一个真正的操作系统线程。

## FutureTask

![temp](E:\图片\temp.png)

- FutureTask类实际为一个装饰者模式的实现，实现了Runnable接口，对真正的业务处理Runnable对象进行包装。

- 在使用时需要业务处理类实现Callable接口，然后在call方法中将想要返回给当前线程的调用线程的信息。

- FutureTask对象因为实现了Runnable方法可以作为线程任务去执行，run()方法中调用了Callable对象的cal()方法,并将结果保存在属性中，如下图所示：

  ```java
  public void run() {
      if (state != NEW ||
          !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                       null, Thread.currentThread()))
          return;
      try {
          Callable<V> c = callable;
          if (c != null && state == NEW) {
              V result;
              boolean ran;
              try {
                  result = c.call();
                  ran = true;
              } catch (Throwable ex) {
                  result = null;
                  ran = false;
                  setException(ex);
              }
              if (ran)
                  set(result);
          }
      } finally {
          // runner must be non-null until state is settled to
          // prevent concurrent calls to run()
          runner = null;
          // state must be re-read after nulling runner to prevent
          // leaked interrupts
          int s = state;
          if (s >= INTERRUPTING)
              handlePossibleCancellationInterrupt(s);
      }
  }
  ```

- 返回信息的获取

  ```java
  public V get() throws InterruptedException, ExecutionException {
      int s = state;
      if (s <= COMPLETING)
          s = awaitDone(false, 0L);
      return report(s);
  }
  ```



## 线程池

![线程池接口](E:\图片\线程池接口.png)

- 线程池工具类为Executors可以创建多中现成的线程池，自定义线程池参数使用ThreadPoolExecutor

  ### execute和submit的区别

  Executor接口中只声明了execute()方法,submit方法为ExecutorService接口增加的，各个实现类中通过将Runnable对象通过Future对象包装后接收创建的线程中的返回值（只有传入submit()中参数为callable对象时才能有返回值，否则返回值永远为null）。

  



