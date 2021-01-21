###  Java多线程synchronized/Lock锁
~~~~
1.sync是java关键字，属于jvm层面，底层实现是monitor；
  lock是一个类，api层面的锁；
  synchronized 可以修饰静态方法、实例方法、代码块；
  synchronized关键字加到static静态方法和synchronized(class)代码块,上都是是给Class类上锁，
  而synchronized关键字加到非static静态方法上是给对象上锁。
  Lock 实现的锁只能修饰代码块.
2.sync不需要手动释放锁，系统会自动让线程释放锁的占用；
  ReentrantLock需要手动释放锁，要不然会造成死锁；需要lock()和UnLock()方法配合try/finally语句来完成。
3.sync不可以中断，除非抛出异常或者正常运行完成。
 ReentrantLock可以中断:
   a.可以设置超时方法，tryLock(Long timeOut,TimeUnit  unit);
   b.lockInterruptibly()放到代码块中，调用interrupt（）方法中断。
4.sync是非公平锁。即不能保证等待锁的那些线程们的顺序
  renntrantLock 默认都是非公平锁(先等待的线程，先获得锁。)。可以通过构造方法传入boolean实现，true 公平锁，fasle 非公平锁。

5.ReentrantLock可以绑定多个condition，用来实现分组唤醒需要唤醒的线程，即可以精确唤醒。
  sync随机唤醒一个或者全部，notify() notifyAll();
  ~~~~

~~~~
示列出处：https://www.cnblogs.com/mengxinJ/p/13797851.html#_label2_3_3
public class lock2{
   static lock2 ll=new lock2();
   ReentrantLock lock=new ReentrantLock();
   Condition cc=lock.newCondition();

    public static void main(String[] args) {
        new Thread(new qq()).start();
        new Thread(new qq2()).start();
    }
    public static class qq implements Runnable{

        @Override
        public void run() {
            ll.aa();
        }

    }
    public static class qq2 implements Runnable{

        @Override
        public void run() {
            ll.bb();
        }

    }

    public void aa() {
        lock.lock();
        try {
            System.out.println("aa方法开始了"+Thread.currentThread().getName());
            Thread.sleep(2000);
            cc.await();
            System.out.println("aa方法结束了"+Thread.currentThread().getName());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
    public void bb() {
        lock.lock();
        try {
            System.out.println("bb方法开始了"+Thread.currentThread().getName());
            Thread.sleep(2000);
            System.out.println("bb方法结束了"+Thread.currentThread().getName());
            cc.signal();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }

}
~~~~
