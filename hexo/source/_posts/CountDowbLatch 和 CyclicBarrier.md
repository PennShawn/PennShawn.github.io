---
title: CountDownLatch adn CyclicBarrier
categories :
- 技术
tags :
- Java
---
### 1.CountDownLatch
`CountDownLatch latch = new CountDownLatch(7);`
CountDownLatch可以用来计数,之前多线程读取日志数据时,在每个线程里`  latch.countDown();`
所有线程提交后`latch.await();`
这里会等`CountDownLatch`里的计数变0才会往后执行.

### 2.CyclicBarrier
CyclicBarrier是一个同步的辅助类，允许一组线程相互之间等待，达到一个共同点，再继续执行。
```
public class CyclicBarrierCom {

    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(5, new Runnable() {

            @Override
            public void run() {
                System.out.println("all done");
            }
        });

        ExecutorService executorService = new ThreadPoolExecutor(3, 3, 0, TimeUnit.NANOSECONDS,
                new LinkedBlockingQueue<>(), new NamedThreadFactory("pool"),
                new ThreadPoolExecutor.AbortPolicy());
        for (int i = 0; i < 5; i++) {
            executorService.submit(new task(cyclicBarrier));
        }

    }

    static class task implements Runnable {

        private final CyclicBarrier cyclicBarrier;

        task(CyclicBarrier cyclicBarrier) {
            this.cyclicBarrier = cyclicBarrier;
        }

        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName());
            try {
                cyclicBarrier.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    }
}

```

`CyclicBarrier c=new CyclicBarrier( int parties,Runnable BarrierAction);`
会在cyclicBarrier.await();执行 parties 次后运行 BarrierAction()方法.
也就是一组线程(parties个线程)都达到某个状态后会执行.
#####  ！！！---但是如果线程池大小不够，那么之前的线程都不会释放资源---！！！
          
         



