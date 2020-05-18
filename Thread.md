### 线程

1. 中断线程

   + 中断一个线程非常简单，只需要在其他线程中对目标线程调用`interrupt()`方法，目标线程需要反复检测自身状态是否是interrupted状态，如果是，就立刻结束运行。

   ```java
   public class Main {
       public static void main(String[] args) throws InterruptedException {
           Thread t = new MyThread();
           t.start();
           Thread.sleep(1); // 暂停1毫秒
           t.interrupt(); // 中断t线程
           t.join(); // 等待t线程结束
           System.out.println("end");
       }
   }
   
   class MyThread extends Thread {
       public void run() {
           int n = 0;
           while (! isInterrupted()) {
               n ++;
               System.out.println(n + " hello!");
           }
       }
   }
   ```

   - 另一个常用的中断线程的方法是设置标志位。我们通常会用一个`running`标志位来标识线程是否应该继续运行，在外部线程中，通过把`HelloThread.running`置为`false`，就可以让线程结束：

     ```java
     public class Main {
         public static void main(String[] args)  throws InterruptedException {
             HelloThread t = new HelloThread();
             t.start();
             Thread.sleep(1);
             t.running = false; // 标志位置为false
         }
     }
     
     class HelloThread extends Thread {
         //volatile关键字解决的是可见性问题：当一个线程修改了某个共享变量的值，其他线程能够立刻看到修改后的值。
         public volatile boolean running = true;
         public void run() {
             int n = 0;
             while (running) {
                 n ++;
                 System.out.println(n + " hello!");
             }
             System.out.println("end!");
         }
     }
     ```

2. 守护线程

   - 有一种线程的目的就是无限循环，例如，一个定时触发任务的线程，守护线程是指为其他线程服务的线程。在JVM中，所有非守护线程都执行完毕后，无论有没有守护线程，虚拟机都会自动退出。如何创建守护线程呢？方法和普通线程一样，只是在调用`start()`方法前，调用`setDaemon(true)`把该线程标记为守护线程：

     ```java
     Thread t = new MyThread();
     t.setDaemon(true);
     t.start();
     ```

3. 线程同步

   ```java
   synchronized(lock) {
       n = n + 1;
   }
   ```

   - 我们来概括一下如何使用`synchronized`：

     1. 找出修改共享变量的线程代码块；
     2. 选择一个共享实例作为锁；
     3. 使用`synchronized(lockObject) { ... }`。
     4. 在使用`synchronized`的时候，不必担心抛出异常。因为无论是否有异常，都会在`synchronized`结束处正确释放锁

   - 不需要synchronized的操作

     JVM规范定义了几种原子操作：

     - 基本类型（`long`和`double`除外）赋值，例如：`int n = m`；
     - 引用类型赋值，例如：`List list = anotherList`。

4. 同步方法

   - 更好的方法是把`synchronized`逻辑封装起来

     ```java
     public class Counter {
         private int count = 0;
     
         public void add(int n) {
             synchronized(this) {
                 count += n;
             }
         }
     }
     ```

   - 当我们锁住的是`this`实例时，实际上可以用`synchronized`修饰这个方法。下面两种写法是等价的：

     ```java
     public void add(int n) {
         synchronized(this) { // 锁住this
             count += n;
         } // 解锁
     }
     public synchronized void add(int n) { // 锁住this
         count += n;
     } // 解锁
     ```

   - 如果对一个静态方法添加`synchronized`修饰符，它锁住的是哪个对象？

     ```java
     public synchronized static void test(int n) {
         ...
     }
     ```

   - 对于`static`方法，是没有`this`实例的，因为`static`方法是针对类而不是实例。但是我们注意到任何一个类都有一个由JVM自动创建的`Class`实例，因此，对`static`方法添加`synchronized`，锁住的是该类的`Class`实例。上述`synchronized static`方法实际上相当于：

     ```java
     public class Counter {
         public static void test(int n) {
             synchronized(Counter.class) {
                 ...
             }
         }
     }
     ```

5. 死锁

   - 可重入锁
     - JVM允许同一个线程重复获取同一个锁，这种能被同一个线程反复获取的锁，就叫做可重入锁，所以，获取锁的时候，不但要判断是否是第一次获取，还要记录这是第几次获取。每获取一次锁，记录+1，每退出`synchronized`块，记录-1，减到0的时候，才会真正释放锁。
     - 两个线程各自持有不同的锁，然后各自试图获取对方手里的锁，造成了双方无限等待下去

6. 

