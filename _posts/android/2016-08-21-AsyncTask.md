---
layout: post
title: AsyncTask
category: android
description: AsyncTask思考
---

# AsyncTask的缺陷和问题

### 关于线程池：
asynctask对应的线程池ThreadPoolExecutor都是进程范围内共享的，都是static的，所以是asynctask控制着进程范围内所有的子类实例。由于这个限制的存在，当使用默认线程池时，如果线程数超过线程池的最大容量，线程池就会爆掉(3.0后默认串行执行，不会出现这个问题)。针对这种情况，可以尝试自定义线程池，配合asynctask使用。

### 关于默认线程池：
核心线程池中最多有CPU_COUNT+1个，最多有CPU_COUNT*2+1个，线程等待队列的最大等待数为128，但是可以自定义线程池。线程池是由AsyncTask来管理的，线程池允许tasks并行运行，需要注意的是并发情况下数据的一致性问题，新数据可能会被老数据覆盖掉，类似volatile变量。所以希望tasks能够串行运行的话，使用SERIAL_EXECUTOR。


### AsyncTask在不同的SDK版本中的区别：
调用AsyncTask的excute方法不能立即执行程序的原因分析及改善方案
通过查阅官方文档发现，AsyncTask首次引入时，异步任务是在一个独立的线程中顺序的执行，也就是说一次只能执行一个任务，不能并行的执行，从1.6开始，AsyncTask引入了线程池，支持同时执行5个异步任务，也就是说同时只能有5个线程运行，超过的线程只能等待，等待前面的线程某个执行完了才被调度和运行。换句话说，如果一个进程中的AsyncTask实例个数超过5个，那么假如前5个都运行很长时间的话，那么第6个只能等待机会了。这是AsyncTask的一个限制，而且对于2.3以前的版本无法解决。如果你的应用需要大量的后台线程去执行任务，那么你只能放弃使用AsyncTask，自己创建线程池来管理Thread。不得不说，虽然AsyncTask较Thread使用起来方便，但是它最多只能同时运行5个线程，这也大大局限了它的实力，你必须要小心设计你的应用，错开使用AsyncTask的时间，尽力做到分时，或者保证数量不会大于5个，否则就会遇到上次提到的问题。可能是Google意识到了AsyncTask的局限性了，从Android3.0开始对AsyncTask的API作出了一些调整：每次只启动一个线程执行一个任务，完成之后再执行第二个任务，也就是相当于只有一个后台线程在执行所提交的任务。

### 生命周期
很多开发者会认为一个在Activity中创建的AsyncTask会随着Activity的销毁而销毁。然而事实并非如此。AsyncTask会一直执行，直到doInBackground()方法执行完毕。然后，如果cancel(boolean)被调用,那么onCancelled(Result result)方法会被执行；否则，执行onPostExecute(Result result)方法。如果我们的Activity销毁之前，没有取消AsyncTask，这有可能让我们的AsyncTask崩溃(crash)。因为它想要处理的view已经不在了。所以，我们总是必须确保在销毁活动之前取消任务。总之，我们使用AsyncTask需要确保AsyncTask正确的取消。

### 内存泄漏
如果AsyncTask被声明为Activity的非静态的内部类，那么AsyncTask会保留一个对Activity的引用。如果Activity已经被销毁，AsyncTask的后台线程还在执行，它将继续在内存里保留这个引用，导致Activity无法被回收，引起内存泄漏。

### 结果丢失
屏幕旋转或Activity在后台被系统杀掉等情况会导致Activity的重新创建，之前运行的AsyncTask会持有一个之前Activity的引用，这个引用已经无效，这时调用onPostExecute()再去更新界面将不再生效。

### 并行还是串行
在Android1.6之前的版本，AsyncTask是串行的，在1.6至2.3的版本，改成了并行的。在2.3之后的版本又做了 修改，可以支持并行和串行，当想要串行执行时，直接执行execute()方法，如果需要执行executeOnExecutor(Executor)。



	参考文献：
	[简书](http://www.jianshu.com/p/89f19d67b348)
	[源码解析](http://blog.csdn.net/singwhatiwanna/article/details/17596225)

