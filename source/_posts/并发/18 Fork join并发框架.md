---
title: fork join
date: 2021-2-28
cover:
top_img:
categories: java并发编程
tags: 
mathjax: true
katex: true
---
# 18.1 Fork/join

- Java并发的进程
    + Runnable
    + Executor和ExecutorService接口
- 执行器框架(Executor Framework)将任务的创建和执行进行了分离，通过这个框架，只需要实现Runnable接口的对象和使用Executor对象，然后将Runnable对象发送给执行器。执行器再负责运行这些任务所需要的的线程，包括线程的创建，线程的管理以及线程的结束。
- Java7，提出了Fork/join框架，有时分解/合并框架，包括了ExecutorService接口的另一种实现，用来解决特殊类型的问题。

1. Fork/Join框架和执行器框架
- ForkJoinPool类看作一个特殊的Executor执行器类型，这个框架有两种操作
    + 分解（Fork）: 当需要将一个任务拆分成更小的多个任务时，在框架中执行这些任务。
    + 合并(Join): 当一个主任务等待其创建的多个子任务的完成执行

Fork/Join框架执行框架在于工作窃取算法。与执行器框架不同，使用join操作让一个主任务等待它所创建的子任务的完成，执行这个任务的线程称之为工作者线程。

为了达到这个目标，通过Fork/Join框架执行的任务有以下限制。
- 任务只能使用fork()和join()操作当做同步机制。如果使用其他的同步机制，工作者线程就不能执行其他任务。如果在Fork/Join框架中将一个任务休眠，正在执行这个任务的工作者线程在休眠期内不能执行另一个任务。
- 任务不能执行I/O操作，比如文件数据的读取与写入。
- 任务不能抛出非运行时异常，必须在代码中处理掉这些异常。
2. Fork/Join框架的核心是由下列两个类组成
- ForkJoinPool: 这个类实现了ExecutorService接口和工作窃取算法。它管理工作者线程，并提供任务的状态信息，以及任务的执行信息。
- ForkJoinTask: 这个类是一个将在ForkJoinPool中执行的任务的基类。
3. Fork/Join框架提供了在一个任务里fork()和join()操作的机制和控制任务状态的方法。通常，为了实现Fork/Join任务，需要实现一个以下两个类之一的子类。
- RecursiveAction: 用于任务没有返回结果的场景
- RecursiveTask: 用于任务有返回结果的场景
4. 工作窃取算法
工作窃取算法优点是充分利用线程进行并行计算，并减少了线程间的竞争，其缺点是在某些情况下还是存在竞争。<br>窃取的基本思路就是：当worker自己的任务队列里面没有任务时，就去scan别的线程的队列，把别人的任务拿过来执行
5. Fork/Join框架基础类
- ForkJoinPool:用来执行Task,或生成新的ForkJoinWorkerThread,执行ForkJoinWorkerThread间的work-stealing逻辑。ForkJoinPool不是为替代ExecutorServie,而是它的补充，在某些应用场景下性能比ExecutorService更好。
- ForkJoinTask: 执行具体的分支逻辑，声明以同步/异步方式进行执行。
- ForkJoinWorkerThread: 是ForkJoinPool内的worker thread，执行ForkJoinTask，内部有ForkJoinPool.WorkQueue来保存要执行的ForkJoinTask
- ForkJoinPool.WorkQueue:保存要执行的ForkJoinTask
6. 基本思想
- ForkJoinPool的每个工作线程都维护着一个工作队列(WorkQueue),这是一个双端队列(Deque)，里面存放的对象是任务(ForkJoinTask).
- 每个工作线程在运行中产生新的任务，(通常是调用了fork()),会放入工作队列的队尾，并且工作线程在处理自己的工作队列时，使用得是LIFO方式，也就是说每次从队尾取出任务来执行
- 每个工作线程在处理自己的工作队列的同时，会尝试窃取一个任务(或是来自于刚刚提交到pool的任务，或是来自于其他工作线程的工作队列)，窃取的任务位于其他线程的工作队列的队首，也就是说工作线程在窃取其他工作线程的任务时，使用得是FIFO方式。
- 在遇到join()时，如果需要join的任务尚未完成，则会先处理其他任务，并等待其完成。
- 在既没有自己的任务，也没有可以窃取的任务时，进入休眠。
7. 代码演示
```
@Slf4j
public class ForkJoinTaskExample extends RecursiveTask<Integer> {

    public static final int threshold = 2;
    private int start;
    private int end;

    public ForkJoinTaskExample(int start, int end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Integer compute() {
        int sum = 0;

        //如果任务足够小就计算任务
        boolean canCompute = (end - start) <= threshold;
        if (canCompute) {
            for (int i = start; i <= end; i++) {
                sum += i;
            }
        } else {
            // 如果任务大于阈值，就分裂成两个子任务计算
            int middle = (start + end) / 2;
            ForkJoinTaskExample leftTask = new ForkJoinTaskExample(start, middle);
            ForkJoinTaskExample rightTask = new ForkJoinTaskExample(middle + 1, end);

            // 执行子任务
            leftTask.fork();
            rightTask.fork();

            // 等待任务执行结束合并其结果
            int leftResult = leftTask.join();
            int rightResult = rightTask.join();

            // 合并子任务
            sum = leftResult + rightResult;
        }
        return sum;
    }

    public static void main(String[] args) {
        ForkJoinPool forkjoinPool = new ForkJoinPool();

        //生成一个计算任务，计算1+2+3+4
        ForkJoinTaskExample task = new ForkJoinTaskExample(1, 100);

        //执行一个任务
        Future<Integer> result = forkjoinPool.submit(task);

        try {
            log.info("result:{}", result.get());
        } catch (Exception e) {
            log.error("exception", e);
        }
    }
}
```
8. 注意
- ForkJoinPool使用submit或invoke提交的区别：invoke是同步执行，调用之后需要等待任务完成，才能执行后面的代码，submit是异步执行，只有在Future调用get的时候会阻塞。
- 可以继承RecursiveTask，也可以继承RecursiveAction。前者适用于有返回值的场景，而后者适合于没有返回值得场景
- invokeAll会比fork好一些, 原因：对于Fork/Join模式，假如Pool里面线程数量是固定的，那么调用子任务的fork方法相当于A先分工给B，然后A当监工不干活，B去完成A交代的任务。所以上面的模式相当于浪费了一个线程。那么如果使用invokeAll相当于A分工给B后，A和B都去完成工作。这样可以更好利用线程池，缩短执行的时间。
```
leftTask.fork();  
rightTask.fork();

替换为

invokeAll(leftTask, rightTask);
```
9. 实用举例
```
public class FibonacciTest {

    class Fibonacci extends RecursiveTask<Integer> {

        int n;

        public Fibonacci(int n) {
            this.n = n;
        }

        // 主要的实现逻辑都在compute()里
        @Override
        protected Integer compute() {
            // 这里先假设 n >= 0
            if (n <= 1) {
                return n;
            } else {
                // f(n-1)
                Fibonacci f1 = new Fibonacci(n - 1);
                f1.fork();
                // f(n-2)
                Fibonacci f2 = new Fibonacci(n - 2);
                f2.fork();
                // f(n) = f(n-1) + f(n-2)
                return f1.join() + f2.join();
            }
        }
    }

    @Test
    public void testFib() throws ExecutionException, InterruptedException {
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        System.out.println("CPU核数：" + Runtime.getRuntime().availableProcessors());
        long start = System.currentTimeMillis();
        Fibonacci fibonacci = new Fibonacci(40);
        Future<Integer> future = forkJoinPool.submit(fibonacci);
        System.out.println(future.get());
        long end = System.currentTimeMillis();
        System.out.println(String.format("耗时：%d millis", end - start));
    }


}
```
10. Java stream parall
11. ScheduledThreadPoolExecutor