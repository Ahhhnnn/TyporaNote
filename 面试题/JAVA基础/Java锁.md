参考：https://juejin.cn/post/6844904084911161357

## 偏向锁/轻量级锁/重量级锁

### 偏向锁获取

```
1、获取对象头的Mark Word；
2、判断mark是否为可偏向状态，即mark的偏向锁标志位为 1，锁标志位为 01；
3、判断mark中JavaThread的状态：如果为空，则进入步骤（4）；如果指向当前线程，
则执行同步代码块；如果指向其它线程，进入步骤（5）；
4、通过CAS原子指令设置mark中JavaThread为当前线程ID，
如果执行CAS成功，则执行同步代码块，否则进入步骤（5）；
5、如果执行CAS失败，表示当前存在多个线程竞争锁，当达到全局安全点（safepoint），
获得偏向锁的线程被挂起，撤销偏向锁，并升级为轻量级，升级完成后被阻塞在安全点的线程继续执行同步代码块；
```

在大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获得，因此为了减少同一线程获取锁(会涉及到一些CAS操作,耗时)的代价而引入偏向锁。偏向锁的核心思想是，如果一个线程获得了锁，那么锁就进入偏向模式，此时Mark Word 的结构也变为偏向锁结构，当这个线程再次请求锁时，无需再做任何同步操作，即获取锁的过程，这样就省去了大量有关锁申请的操作，从而也就提供程序的性能。所以，对于没有锁竞争的场合，偏向锁有很好的优化效果，毕竟极有可能连续多次是同一个线程申请相同的锁



注意 JVM 提供了关闭偏向锁的机制， JVM 启动命令指定如下参数即可

```shell
-XX:-UseBiasedLocking
```





### 重量锁

重量级锁通过对象内部的监视器（monitor）实现，其中monitor的本质是依赖于底层操作系统的Mutex Lock实现，操作系统实现线程之间的切换需要从用户态到内核态的切换，切换成本非常高。

其本质就是通过CAS设置monitor的_owner字段为当前线程，如果CAS成功，则表示该线程获取了锁，跳出自旋操作，执行同步代码，否则继续被挂起；

Monitor 释放:

当某个持有锁的线程执行完同步代码块时，会进行锁的释放，给其它线程机会执行同步代码，**在HotSpot中，通过退出monitor的方式实现锁的释放，并通知被阻塞的线程.**



## AQS AbstractQueuedSynchronizer

AQS 即队列同步器。它是构建锁或者其他同步组件的基础框架（如 ReentrantLock、ReentrantReadWriteLock、Semaphore 等），J.U.C 并发包的作者（Doug Lea）期望它能够成为实现大部分同步需求的基础。

```java
	  //同步队列头节点
    private transient volatile Node head;

    //同步队列尾节点
    private transient volatile Node tail;

    //同步状态
    private volatile int state;

```



AQS 使用一个 int 类型的成员变量 state 来表示同步状态：

- 当 `state > 0` 时，表示已经获取了锁。
- 当 `state = 0` 时，表示释放了锁。

Node构成FIFO的同步队列来完成线程获取锁的排队工作

- 如果当前线程获取同步状态失败（锁）时，AQS 则会将当前线程以及等待状态等信息构造成一个节点（Node）并将其加入同步队列，同时会阻塞当前线程
- 当同步状态释放时，则会把节点中的线程唤醒，使其再次尝试获取同步状态



### 实现原理

AQS提供的方法 可以分为多层

![img](assets/82077ccf14127a87b77cefd1ccf562d3253591.png)

当有自定义同步器接入时，只需重写第一层所需要的部分方法即可，不需要关注底层具体的实现流程。当自定义同步器进行加锁或者解锁操作时，先经过第一层的API进入AQS内部方法，然后经过第二层进行锁的获取，接着对于获取锁失败的流程，进入第三层和第四层的等待队列处理，而这些处理方式均依赖于第五层的基础数据提供层。



比如ReentrantLock 实现了公平锁和非公平锁

内部的Sync类继承了AbstractQueuedSynchronizer

NonfairSync  和 FairSync 继承了Sync类



lock方法就是分别调用了公平锁和非公平是的得方法

**非公平锁**

直接通过cas设置state的值，如果成功，设置独占线程为自己，如果失败则调用aqs的acquire方法

```java
final void lock() {
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
        }
```

**acquire 方法中需要调用 tryAcquire方法，这个方法需要同步起自己实现**

非公平锁的实现

同样是使用cas实现，如果失败，**那么会加入到阻塞队列中，由acquireQueued方法来控制获取锁**

```java
public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }

protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }

final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```

公平锁

公平锁的lock方法直接调用 aqs的 acquire方法获取锁

**不同于非公平的，会通过hasQueuedPredecessors方法判断是否有其他线程在等待锁，如果有那么获取失败，加入阻塞队列获取锁**

```java
public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }

protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```



 