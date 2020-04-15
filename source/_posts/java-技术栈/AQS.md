condition是要和lock配合使用的（它们是绑定的），lock的实现原理依赖于AQS，而ConditionObject则是AQS的内部类。
##### AQS(https://segmentfault.com/a/1190000017372067)
了解J.U.C的关键就是AQS.J.U.C，是JDK中提供的并发工具包, java.util.concurrent
AbstractQueuedSynchronizer.它提供了一个FIFO队列，是一个抽象类，定义了同步状态的获取和释放方法。
AQS的功能分为独占和共享两种
AQS的实现依赖内部的同步队列，即FIFO的双向队列（链表）。如果当前线程竞争锁失败，那么AQS会把当前线程和等待状态信息构造成一个Node加到同步队列中，同时再阻塞该进程。

###### 关于这个同步队列：
出现锁竞争和释放锁时，AQS同步队列中的节点会发生变化，添加时：
+ 新线程封装成node追加到同步队列中，设置prev，修改当前的前置的next指自己
+ 通过CAS（compare and swap)将tail指向新的尾部节点

>CAS指令执行时，当且仅当内存地址V的值与预期值A相等时，将内存地址V的值修改为B，否则就什么都不做。整个比较并替换的操作是一个原子操作。考虑ABA问题是否会有影响,下面有源码分析

释放锁移除时：
+ 修改head指向下一个获得锁的节点
+ 新的获得锁的节点，将prev指向null

###### AQS源码分析
shit的多级列表在有代码的情况下非常难用，就不弄了
1. ReentrantLock.lock()
```java
public void lock(){
    sync.lock();
}
```
这个是获取锁的入口，调用sync这个类里面的方法，sync是什么呢。
```java
abstract static class Sync extends AbstractQueuedSynchronizer
```
通过进一步分析，发现Sync这个类有两个具体的实现，分别是 NofairSync(非公平锁), FailSync(公平锁).

    公平锁 表示所有线程严格按照FIFO来获取锁
    非公平锁 表示可以存在抢占锁的功能，也就是说不管当前队列上是否存在其他线程等待，新线程都有机会抢占锁

以下以非公平锁作为主要分析逻辑
2. NonfairSync.lock()
```java
final void lock(){
    if(compareAndSetState(0,1))//通过cas操作来修改state状态，表示抢争锁的操作
        setExclusiveOwnerThread(Thread.currentThread());//设置当前获得锁状态的进程
    else
        acquire(1);//尝试去获取锁
}
```
由于这里是非公平锁，所以调用lock方法时，先去通过cas去抢占锁
如果抢占锁成功，保存获得锁成功的当前线程
抢占锁失败，调用acquire来走锁竞争逻辑

3. compareAndSetState()

```java
protected final boolean compareAndSetState(int expect,int update){
    //see below for intrinsics setup to support this
    return unsafe.compareAndSwapInt(this,stateOffset,expect,update);
}
```
如果当前内存中的state的值和预期值expect相等，则替换为update。更新成功返回true，否则返回false. 这个操作是原子的，不会出现线程安全问题，这里面涉及到Unsafe这个类的操作，以及涉及到state这个属性的意义。AQS中有一个这样的state的属性定义,这个对于重入锁的实现来说，表示一个同步状态(计数)。它有两个含义的表示

    当state=0时，表示无锁状态
    当state>0时，表示已经有线程获得了锁，也就是state=1，但是因为ReentrantLock允许重入，所以同一个线程多次获得同步锁的时候，state会递增，比如重入5次，那么state=5。 而在释放锁的时候，同样需要释放5次直到state=0其他线程才有资格获得锁
```java
private volatile int state;
```

需要注意的是：不同的AQS实现，state所表达的含义是不一样的。Unsafe类是在sun.misc包下，不属于Java标准。但是很多Java的基础类库，包括一些被广泛使用的高性能开发库都是基于Unsafe类开发的，比如Netty、Hadoop、Kafka等；Unsafe可认为是Java中留下的后门，提供了一些低层次操作，如直接内存访问、线程调度等
```java
public final native boolean compareAndSwapInt(Object var1,long var2,int var4,int var5);
```
这个是一个native方法， 第一个参数为需要改变的对象，第二个为偏移量(即之前求出来的headOffset的值)，第三个参数为期待的值，第四个为更新后的值
整个方法的作用是如果当前时刻的值等于预期值var4相等，则更新为新的期望值 var5，如果更新成功，则返回true，否则返回false；
4. acquire
acquire是AQS中的方法，如果CAS操作未能成功，说明state已经不为0，此时继续acquire(1)操作,这里大家思考一下，acquire方法中的1的参数是用来做什么呢？如果没猜中，往前面回顾一下state这个概念
我猜就是给锁++
   ```java
   public final void acquire(int arg){
       if(!tryAcquire(arg)&&
           acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
           selfInterrupt();
   }
   ```
这个方法的主要逻辑是
    通过tryAcquire尝试获取独占锁，如果成功返回true，失败返回false
    如果tryAcquire失败，则会通过addWaiter方法将当前线程封装成Node添加到AQS队列尾部
    acquireQueued，将Node作为参数，通过自旋去尝试获取锁。

5. NonfairSync.tryAcquire
这个方法的作用是尝试获取锁，如果成功返回true，不成功返回false
它是重写AQS类中的tryAcquire方法，并且大家仔细看一下AQS中tryAcquire方法的定义，并没有实现，而是抛出异常。按照一般的思维模式，既然是一个不实现的模版方法，那应该定义成abstract，让子类来实现呀？大家想想为什么
```java
protected final boolean tryAcquire(int acquires) {
    return nonfairTryAcquire(acquires);
}
```
6. nonfairTryAcquire
tryAcquire(1)在NonfairSync中的实现代码如下
```java
final boolean nonfairTryAcquire(int acquires) {
    //获得当前执行的线程
    final Thread current = Thread.currentThread();
    int c = getState(); //获得state的值
    if (c == 0) { //state=0说明当前是无锁状态
        //通过cas操作来替换state的值改为1，大家想想为什么要用cas呢？
        //理由是，在多线程环境中，直接修改state=1会存在线程安全问题，你猜到了吗？
        if (compareAndSetState(0, acquires)) {
             //保存当前获得锁的线程
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    //这段逻辑就很简单了。如果是同一个线程来获得锁，则直接增加重入次数
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires; //增加重入次数
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```
获取当前线程，判断当前的锁的状态
如果state=0表示当前是无锁状态，通过cas更新state状态的值
如果当前线程是属于重入，则增加重入次数
7. addWaiter
当tryAcquire方法获取锁失败以后，则会先调用addWaiter将当前线程封装成Node，然后添加到AQS队列
```java
private Node addWaiter(Node mode) { //mode=Node.EXCLUSIVE
        //将当前线程封装成Node，并且mode为独占锁
        Node node = new Node(Thread.currentThread(), mode); 
        // Try the fast path of enq; backup to full enq on failure
        // tail是AQS的中表示同步队列队尾的属性，刚开始为null，所以进行enq(node)方法
        Node pred = tail;
        if (pred != null) { //tail不为空的情况，说明队列中存在节点数据
            node.prev = pred;  //讲当前线程的Node的prev节点指向tail
            if (compareAndSetTail(pred, node)) {//通过cas讲node添加到AQS队列
                pred.next = node;//cas成功，把旧的tail的next指针指向新的tail
                return node;
            }
        }
        enq(node); //tail=null，将node添加到同步队列中
        return node;
    }
```
将当前线程封装成Node
判断当前链表中的tail节点是否为空，如果不为空，则通过cas操作把当前线程的node添加到AQS队列
如果为空或者cas失败，调用enq将节点添加到AQS队列
8. enq
enq就是通过自旋操作把当前节点加入到队列中
```java
private Node enq(final Node node) {
        //自旋，不做过多解释，不清楚的关注公众号[架构师修炼宝典]
        for (;;) {
            Node t = tail; //如果是第一次添加到队列，那么tail=null
            if (t == null) { // Must initialize
                //CAS的方式创建一个空的Node作为头结点
                if (compareAndSetHead(new Node()))
                   //此时队列中只一个头结点，所以tail也指向它
                    tail = head;
            } else {
//进行第二次循环时，tail不为null，进入else区域。将当前线程的Node结点的prev指向tail，然后使用CAS将tail指向Node
                node.prev = t;
                if (compareAndSetTail(t, node)) {
//t此时指向tail,所以可以CAS成功，将tail重新指向Node。此时t为更新前的tail的值，即指向空的头结点，t.next=node，就将头结点的后续结点指向Node，返回头结点
                    t.next = node;
                    return t;
                }
            }
        }
    }
```
假如有两个线程t1,t2同时进入enq方法，t==null表示队列是首次使用，需要先初始化
另外一个线程cas失败，则进入下次循环，通过cas操作将node添加到队尾

到目前为止，通过addwaiter方法构造了一个AQS队列，并且将线程添加到了队列的节点中
10. acquireQueued
将添加到队列中的Node作为参数传入acquireQueued方法，这里面会做抢占锁的操作
```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();// 获取prev节点,若为null即刻抛出NullPointException
            if (p == head && tryAcquire(arg)) {// 如果前驱为head才有资格进行锁的抢夺
                setHead(node); // 获取锁成功后就不需要再进行同步操作了,获取锁成功的线程作为新的head节点
//凡是head节点,head.thread与head.prev永远为null, 但是head.next不为null
                p.next = null; // help GC
                failed = false; //获取锁成功
                return interrupted;
            }
//如果获取锁失败，则根据节点的waitStatus决定是否需要挂起线程
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())// 若前面为true,则执行挂起,待下次唤醒的时候检测中断的标志
                interrupted = true;
        }
    } finally {
        if (failed) // 如果抛出异常则取消锁的获取,进行出队(sync queue)操作
            cancelAcquire(node);
    }
}
```
获取当前节点的prev节点
如果prev节点为head节点，那么它就有资格去争抢锁，调用tryAcquire抢占锁
抢占锁成功以后，把获得锁的节点设置为head，并且移除原来的初始化head节点
如果获得锁失败，则根据waitStatus决定是否需要挂起线程
最后，通过cancelAcquire取消获得锁的操作
前面的逻辑都很好理解，主要看一下shouldParkAfterFailedAcquire这个方法和parkAndCheckInterrupt的作用

后面不想写了，滚去看原链好了。（其实复制粘贴都要结束了）




---

condition内部维护了一个「等待队列」（单向），所有await方法的线程会加入其中，如果该线程能够从await方法返回的话一定是该线程获取了与condition相关联的lock
当前线程调用condition.await()方法后，会使得当前线程释放lock然后加入到等待队列中，直至被signal/signalAll后会使得当前线程从等待队列中移至到同步队列中去，直到获得了lock后才会从await方法返回，或者在等待时被中断会做中断处理


很显然，当线程第一次调用condition.await()方法时，会进入到这个while()循环中，然后通过LockSupport.park(this)方法使得当前线程进入等待状态，那么要想退出这个await方法第一个前提条件自然而然的是要先退出这个while循环，出口就只剩下两个地方：1. 逻辑走到break退出while循环；2. while循环中的逻辑判断为false。

再看代码出现第1种情况的条件是当前等待的线程被中断后代码会走到break退出，第二种情况是当前节点被移动到了同步队列中（即另外线程调用的condition的signal或者signalAll方法），while中逻辑判断为false后结束while循环。总结下，就是当前线程被中断或者调用condition.signal/condition.signalAll方法当前节点移动到了同步队列后 ，这是当前线程退出await方法的前提条件。

当退出while循环后就会调用acquireQueued(node, savedState)，这个方法在介绍AQS的底层实现时说过了，若感兴趣的话可以去，该方法的作用是在自旋过程中线程不断尝试获取同步状态，直至成功（线程获取到lock）。这样也说明了退出await方法必须是已经获得了condition引用（关联）的lock。到目前为止，开头的三个问题我们通过阅读源码的方式已经完全找到了答案，也对await方法的理解加深。


线程awaitThread先通过lock.lock()方法获取锁成功后调用了condition.await方法进入等待队列，而另一个线程signalThread通过lock.lock()方法获取锁成功后调用了condition.signal或者signalAll方法，使得线程awaitThread能够有机会移入到同步队列中，当其他线程释放lock后使得线程awaitThread能够有机会获取lock，从而使得线程awaitThread能够从await方法中退出执行后续操作。如果awaitThread获取lock失败会直接进入到同步队列。


---
