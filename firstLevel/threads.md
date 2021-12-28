统一进程中多个执行路径同时执行

优点：
● 解决了一个进程里可以同时运行多个任务
● 提高了资源利用率，而不是提高效率
缺点：
● 降低了一个进程里面的线程执行频率
● 对线程进行管理要求额外的cpu开销，给cpu增加额外负担
● 公有变量同时读写，导致线程安全问题
线程的死锁，较长时间的等待或资源竞争以及死锁等多线程症状


多线程的实现方式
● 继承Thread类，重新run()方法
  ○ 创建线程类对象，调用start方法开启线程
● 实现Runable接口，实现run()方法
  ○ 调用start方法开启线程
● 实现Callable接口
● 使用线程池

实现Runable接口更好，因为一个类继承thread类就不能继承其他类，可能增加项目的逻辑复杂度；java可以实现多接口，不影响其他逻辑

Thread run 和 start
run(); 当做普通方法的方式调用。程序还是要顺序执行，要run方法结束后才往下走，程序依然只有一个主线程
start();真正实现了多线程

Runable
多线程的实现方式二

```java
callable
Callable<List<ProjectInfo>> projectInfoList = new Callable<List<ProjectInfo>>() {
				@Override
				public List<ProjectInfo> call() throws Exception {
					logger.error(new Date()+"11111111   --------");
					return list;
				}
			};
          
  FutureTask<List<ProjectInfo>> projectInfoFuture 
          = new FutureTask<List<ProjectInfo>>(projectInfoList);
	new Thread(projectInfoFuture).start();
          
   //获取多线程的返回值
	List<ProjectInfo> list = projectInfoFuture.get();     
```
          
什么是线程安全问题，如何解决
多线程：一个进程可以包含若干个线程，同时创建多个线程来完成某项任务，就是多线程
如果有全局变量被多个线程同时使用，就会出现线程安全问题
解决办法：
1.通过加锁（Synchronized）来避免线程安全问题。此时虽然还是单例，但是对于多线程的访问，每次只有一个请求进入方法体内执行，只有执行完毕后，其他线程才能访问，降低了吞吐量
2.避免使用全局变量，使用局部变量可以避免线程安全问题，最好