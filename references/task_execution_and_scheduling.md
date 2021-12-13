# Task Execution

https://docs.spring.io/spring-framework/docs/4.2.x/spring-framework-reference/html/scheduling.html

## Introduction

> The Spring Framework provides abstractions for asynchronous execution and scheduling of tasks with the TaskExecutor and TaskScheduler interfaces, respectively. Spring also features implementations of those interfaces that support thread pools or delegation to CommonJ within an application server environment. Ultimately the use of these implementations behind the common interfaces abstracts away the differences between Java SE 5, Java SE 6 and Java EE environments.
>
> Spring also features integration classes for supporting scheduling with the Timer, part of the JDK since 1.3, and the Quartz Scheduler ( http://quartz-scheduler.org). Both of those schedulers are set up using a FactoryBean with optional references to Timer or Trigger instances, respectively. Furthermore, a convenience class for both the Quartz Scheduler and the Timer is available that allows you to invoke a method of an existing target object (analogous to the normal MethodInvokingFactoryBean operation).

- Spring Framework 는 asynchronous execution, scheduling 을 제공해주기 위해서 TaskExecutor 과 TaskScheduler 인터페이스를 제공해준다.
- 그리고 이 구현체와 이것을 사용할 수 있도록 Thread pool 을 제공해준다.

## The Spring TaskExecutor abstraction

> Spring 2.0 introduces a new abstraction for dealing with executors. Executors are the Java 5 name for the concept of thread pools. The "executor" naming is due to the fact that there is no guarantee that the underlying implementation is actually a pool; an executor may be single-threaded or even synchronous. Spring’s abstraction hides implementation details between Java SE 1.4, Java SE 5 and Java EE environments.
>
> Spring’s TaskExecutor interface is identical to the java.util.concurrent.Executor interface. In fact, its primary reason for existence was to abstract away the need for Java 5 when using thread pools. The interface has a single method execute(Runnable task) that accepts a task for execution based on the semantics and configuration of the thread pool.
>
> The TaskExecutor was originally created to give other Spring components an abstraction for thread pooling where needed. Components such as the ApplicationEventMulticaster, JMS’s AbstractMessageListenerContainer, and Quartz integration all use the TaskExecutor abstraction to pool threads. However, if your beans need thread pooling behavior, it is possible to use this abstraction for your own needs.

- spring 2.0 에서는 Executor 을 추상화한 TaskExecutor 를 제공해준다. 
- Executor 의 이름의 유래 자체는 Thread Pool 의 구현을 보장해주지 않는다. 라는 뜻이 포함되어 있다. 그래서 Executor 은 Single-thread 일 수 있다. 
- Spring 에서 제공해주는 TaskExecutor 는 Java.util.concurrent.Executor 인터페이스와 동일하다. 여기서는 execute(Runnable task) 메소드만 존재한다.

## TaskExecutor types

> here are a number of pre-built implementations of TaskExecutor included with the Spring distribution. In all likelihood, you shouldn’t ever need to implement your own.
>
> - `SimpleAsyncTaskExecutor` This implementation does not reuse any threads, rather it starts up a new thread for each invocation. However, it does support a concurrency limit which will block any invocations that are over the limit until a slot has been freed up. If you are looking for true pooling, see the discussions of SimpleThreadPoolTaskExecutor and ThreadPoolTaskExecutor below.
> - `SyncTaskExecutor` This implementation doesn’t execute invocations asynchronously. Instead, each invocation takes place in the calling thread. It is primarily used in situations where multi-threading isn’t necessary such as simple test cases.
> - `ConcurrentTaskExecutor` This implementation is an adapter for a java.util.concurrent.Executor object. There is an alternative, `ThreadPoolTaskExecutor`, that exposes the `Executor` configuration parameters as bean properties. It is rare to need to use the ConcurrentTaskExecutor, but if the `ThreadPoolTaskExecutor` isn’t flexible enough for your needs, the `ConcurrentTaskExecutor` is an alternative.
> - `SimpleThreadPoolTaskExecutor` This implementation is actually a subclass of Quartz’s SimpleThreadPool which listens to Spring’s lifecycle callbacks. This is typically used when you have a thread pool that may need to be shared by both Quartz and non-Quartz components
> - `ThreadPoolTaskExecutor` This implementation is the most commonly used one. It exposes bean properties for configuring a java.util.concurrent.ThreadPoolExecutor and wraps it in a TaskExecutor. If you need to adapt to a different kind of java.util.concurrent.Executor, it is recommended that you use a `ConcurrentTaskExecutor` instead.
> - `WorkManagerTaskExecutor` This implementation uses the CommonJ WorkManager as its backing implementation and is the central convenience class for setting up a CommonJ WorkManager reference in a Spring context. Similar to the `SimpleThreadPoolTaskExecutor`, this class implements the `WorkManager` interface and therefore can be used directly as a `WorkManager` as well.

- SimpleAsyncTaskExecutor 는 스레드를 재사용하지 않고 매번 만들어서 쓴다. Thread pool 을 이용하고 싶다면 SimpleThreadPoolTaskExecutor 나 ThreadPoolTaskExecutor 를 사용하자.
- SyncTaskExecutor 는 비동기적으로 실행하지 않는 Executor 이다. 주로 멀티스레드 환경이 필요하지 않는 테스트에서 사용하고 싶을 때 사용한다.
- ConcurrentTaskExecutor 는 ThreadPoolTaskExecutor 와 차이를 둬서 봐야하는데 ThreadPoolTaskExecutor 는 java.util.concurrent.ThreadPoolExecutor 를 래핑한 클래스이고 생성할 때 여러가지 프로퍼티를 받아서 사용한다.
  ConcurrentTaskExecutor 는 java.util.concurrent.Executor 어댑터 역할로서 사용하는 Executor 이고 Executor 를 유연하게 사용하고 싶다면 쓰자.

## Using a TaskExecutor

> Spring’s TaskExecutor implementations are used as simple JavaBeans. In the example below, we define a bean that uses the ThreadPoolTaskExecutor to asynchronously print out a set of messages.
>
> ```java
> import org.springframework.core.task.TaskExecutor;
> 
> public class TaskExecutorExample {
> 
>     private class MessagePrinterTask implements Runnable {
> 
>         private String message;
> 
>         public MessagePrinterTask(String message) {
>             this.message = message;
>         }
> 
>         public void run() {
>             System.out.println(message);
>         }
> 
>     }
> 
>     private TaskExecutor taskExecutor;
> 
>     public TaskExecutorExample(TaskExecutor taskExecutor) {
>         this.taskExecutor = taskExecutor;
>     }
> 
>     public void printMessages() {
>         for(int i = 0; i < 25; i++) {
>             taskExecutor.execute(new MessagePrinterTask("Message" + i));
>         }
>     }
> 
> }
> ```

***

## The @Async annotation

> The @Async annotation can be provided on a method so that invocation of that method will occur asynchronously. In other words, the caller will return immediately upon invocation and the actual execution of the method will occur in a task that has been submitted to a Spring TaskExecutor. In the simplest case, the annotation may be applied to a void-returning method.
>
> ````java
> void doSomething() {
>     // this will be executed asynchronously
> }
> ````
> 
> Unlike the methods annotated with the @Scheduled annotation, these methods can expect arguments, because they will be invoked in the "normal" way by callers at runtime rather than from a scheduled task being managed by the container. For example, the following is a legitimate application of the @Async annotation.
>
> ````java
> @Async
> void doSomething(String s) {
>   // this will be executed asynchronously
> }
> ````
> 
> Even methods that return a value can be invoked asynchronously. However, such methods are required to have a Future typed return value. This still provides the benefit of asynchronous execution so that the caller can perform other tasks prior to calling get() on that Future.
>
> ````java
> @Async
> Future<String> returnSomething(int i) {
>   // this will be executed asynchronously
> }
> ````
> 
> @Async methods may not only declare a regular java.util.concurrent.Future return type but also Spring’s org.springframework.util.concurrent.ListenableFuture or, as of Spring 4.2, JDK 8’s java.util.concurrent.CompletableFuture: for richer interaction with the asynchronous task and for immediate composition with further processing steps.
>
> ````java
> public class SampleBeanImpl implements SampleBean {
> 
>     @Async
>     void doSomething() {
>         // ...
>     }
> 
> }
> 
> public class SampleBeanInitializer {
> 
>     private final SampleBean bean;
> 
>     public SampleBeanInitializer(SampleBean bean) {
>         this.bean = bean;
>     }
> 
>     @PostConstruct
>     public void initialize() {
>         bean.doSomething();
>     }
> ````

- @Async 애노테이션을 통해 비동기적으로 실행할 수 있다. 물론 실행은 Spring 에서 제공해주는 TaskExecutor 를 통해서 진행된다.
- @Async 가 붙은 메소드는 @Scheduled 와는 조금 다른데, @Async 는 @Scheduled 와 달리 컨테이너에 의해서 다뤄지는 메소드는 아니라 Caller 에 의해 실행되는 메소드이므로 파라미터를 전달해줄 수 있다.
- @Async 메소드의 결과값을 가져올 수 있는데. Future 이나 CompletableFuture 타입으로 가져올 수 있다.
- @PostConstruct 와 @Async 는 같이 쓸 수 없는데 빈을 비동기적으로 처리하고 싶다면 비동기 실행하는 부분과 @PostConstruct 메소드와 분리시키면 된다.

*** 

## Executor qualification with @Async

> By default when specifying @Async on a method, the executor that will be used is the one supplied to the 'annotation-driven' element as described above. However, the value attribute of the @Async annotation can be used when needing to indicate that an executor other than the default should be used when executing a given method.
>
> ```java
> @Async("otherExecutor")
> void doSomething(String s) {
>   // this will be executed asynchronously by "otherExecutor"
> }
> ```
> 
> In this case, "otherExecutor" may be the name of any Executor bean in the Spring container, or may be the name of a qualifier associated with any Executor, e.g. as specified with the <qualifier> element or Spring’s @Qualifier annotation.

- @Async 애노테이션에 attribute 값으로 다른 Executor 값을 넘길 수 있다. 그러면 다른 Executor 가 실행한다.

## Exception management with @Async

> When an @Async method has a Future typed return value, it is easy to manage an exception that was thrown during the method execution as this exception will be thrown when calling get on the Future result. With a void return type however, the exception is uncaught and cannot be transmitted. For those cases, an AsyncUncaughtExceptionHandler can be provided to handle such exceptions.
>
> ```java
> public class MyAsyncUncaughtExceptionHandler implements AsyncUncaughtExceptionHandler {
> 
>     @Override
>     public void handleUncaughtException(Throwable ex, Method method, Object... params) {
>         // handle exception
>     }
> }
> ```
> 
> By default, the exception is simply logged. A custom AsyncUncaughtExceptionHandler can be defined via AsyncConfigurer or the task:annotation-driven XML element.

- 비동기 실행중에 예외를 잡는 방법은 Future 타입으로 리턴값을 가져오면 편하게 처리할 수 있다. 
- 하지만 리턴타입이 void 인 경우 메소드 실행 중 예외가 나면 처리하기 힘든데 이 경우 AsyncUncaughtExceptionHandler 를 이용하면 처리하는게 편하다.

***

## The 'executor' element

> The following will create a ThreadPoolTaskExecutor instance:
>
> ```xml
> <task:executor id="executor" pool-size="10"/>
> ```
> 
> As with the scheduler above, the value provided for the 'id' attribute will be used as the prefix for thread names within the pool. As far as the pool size is concerned, the 'executor' element supports more configuration options than the 'scheduler' element. For one thing, the thread pool for a ThreadPoolTaskExecutor is itself more configurable. Rather than just a single size, an executor’s thread pool may have different values for the core and the max size. If a single value is provided then the executor will have a fixed-size thread pool (the core and max sizes are the same). However, the 'executor' element’s 'pool-size' attribute also accepts a range in the form of "min-max".
>
> ````xml
> <task:executor
>        id="executorWithPoolSizeRange"
>        pool-size="5-25"
>        queue-capacity="100"/>
> ````
>
> As you can see from that configuration, a 'queue-capacity' value has also been provided. The configuration of the thread pool should also be considered in light of the executor’s queue capacity. For the full description of the relationship between pool size and queue capacity, consult the documentation for ThreadPoolExecutor. The main idea is that when a task is submitted, the executor will first try to use a free thread if the number of active threads is currently less than the core size. If the core size has been reached, then the task will be added to the queue as long as its capacity has not yet been reached. Only then, if the queue’s capacity has been reached, will the executor create a new thread beyond the core size. If the max size has also been reached, then the executor will reject the task.
>
> By default, the queue is unbounded, but this is rarely the desired configuration, because it can lead to OutOfMemoryErrors if enough tasks are added to that queue while all pool threads are busy. Furthermore, if the queue is unbounded, then the max size has no effect at all. Since the executor will always try the queue before creating a new thread beyond the core size, a queue must have a finite capacity for the thread pool to grow beyond the core size (this is why a fixed size pool is the only sensible case when using an unbounded queue).
>
> In a moment, we will review the effects of the keep-alive setting which adds yet another factor to consider when providing a pool size configuration. First, let’s consider the case, as mentioned above, when a task is rejected. By default, when a task is rejected, a thread pool executor will throw a TaskRejectedException. However, the rejection policy is actually configurable. The exception is thrown when using the default rejection policy which is the AbortPolicy implementation. For applications where some tasks can be skipped under heavy load, either the DiscardPolicy or DiscardOldestPolicy may be configured instead. Another option that works well for applications that need to throttle the submitted tasks under heavy load is the CallerRunsPolicy. Instead of throwing an exception or discarding tasks, that policy will simply force the thread that is calling the submit method to run the task itself. The idea is that such a caller will be busy while running that task and not able to submit other tasks immediately. Therefore it provides a simple way to throttle the incoming load while maintaining the limits of the thread pool and queue. Typically this allows the executor to "catch up" on the tasks it is handling and thereby frees up some capacity on the queue, in the pool, or both. Any of these options can be chosen from an enumeration of values available for the 'rejection-policy' attribute on the 'executor' element.
>
> ````xml
> <task:executor
>         id="executorWithCallerRunsPolicy"
>         pool-size="5-25"
>         queue-capacity="100"
>         rejection-policy="CALLER_RUNS"/>
> ````
>
> Finally, the keep-alive setting determines the time limit (in seconds) for which threads may remain idle before being terminated. If there are more than the core number of threads currently in the pool, after waiting this amount of time without processing a task, excess threads will get terminated. A time value of zero will cause excess threads to terminate immediately after executing a task without remaining follow-up work in the task queue.
>
> ```xml
>  <task:executor
>       id="executorWithKeepAlive"
>       pool-size="5-25"
>       keep-alive="120"/>
> ```

- ThreadPoolTaskExecutor 설정 값을 xml 을 통해서 넣을 수 있다.
- id 값은 ThreadPool 의 prefix 에 해당하는 값이다. 
- pool-size 는 thread 의 개수를 나타내며 범위를 줄 수 있다. 최소 값은 core 최대 값은 max 라고 부른다.
- pool-size 를 범위로 주면 이는 queue-capacity 와도 연관성이 있는데 메인 아이디어는 놀고 있는 스레드가 없다면 task 는 큐에 들어간다. 큐에 가득차게되면 그때부터 스레드를 하나씩 만드는 구조다.
이렇게 스레드가 만들어지는데 스레드의 개수가 max-size 에 도달하고 queue 도 가득차게 되면 이제 task 는 reject 된다.
- reject 된 task 를 처리하는 policy 는 여러가지가 있는데 기본적으로는 `TaskRejectedException` 을 내도록 하는 것이다.
- 예외를 내지않도록 할려면 policy 를 DiscardPolicy 로 바꾸면된다. 기본은 AbortPolicy 이다.
- CallerRunsPolicy 도 있는데 이는 reject 된 task 는 호출한 caller 쪽에서 직접 처리하도록 하는 정책이다.
- keep-alive 옵션은 core 사이즈 보다 많은 스레드가 곧바로 종료되지 않고 지정한 값만큼 기다렸다가 종료되는 옵션이다. 

