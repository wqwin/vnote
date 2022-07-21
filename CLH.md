## CLH原理
* 基于链表的高性能，可扩展，公平的自旋锁，申请线程只在本地变量上自旋，并不断轮训前驱的状态，如果发现前驱释放了锁就结束自旋。
  精妙之处在于 在一个CAS操作的帮助下，所有等待获取锁的线程下的节点轻松且正确的构成了全局队列。等待中的线程正如队列中的节点依次获取锁。 
1、首先有一个尾节点指针，通过这个为节点指针来构建等待线程的逻辑队列，因此能确保线程先到先服务的公平性，因此尾节点指针可以说是hi构建逻辑队列的桥梁，此外这个尾节点指针是院子引用类型，避免了多线程并发操作的线程安全问题。
2、通过等待锁的每个线程在自己的某个变量上自旋等待，这个变量酱油前一个线程写入，由于某个线程获取锁操作时总是通过尾节点指针获取到前一线程写入的变量，而尾节点指针又是原子引用类型，因此确保了这个变量获取出来总是线程安全的。

## CLH加锁的过程
* 首先获得当前线程的当前节点curNode，这里每次获取的CLHNode节点的locked状态都为false。
* 然后将当前CLHNode节点的locked状态赋值为true，表示当前线程的一种有效状态，即获取到了锁或者正在等待所的状态。
* 因为尾指针tailNode总是指向了前一个线程的CLHNode节点，因此这里利用尾指针tailNode取出前一个现成的CLHNode节点，然后赋值给当前线程的前继节点preNode,并且将尾指针重新只想最后一个节点即当前节点CLHNode节点，一遍下一个线程到来时使用。
* 根据前继节点的locked状态判断，若locked为false。则说明前一个线程释放了锁，当前线程可以获得锁，不用自旋等待，若前驱节点的locked为true，则表示前一个线程获取到了锁且正在等待或自旋等待。

```java
public void lock() {
    // 取出当前线程ThreadLocal存储的当前节点，初始化值总是一个新建的CLHNode，locked状态为false。
    CLHNode currNode = curNode.get();
    // 此时把lock状态置为true，表示一个有效状态，
    // 即获取到了锁或正在等待锁的状态
    currNode.locked = true;
    // 当一个线程到来时，总是将尾结点取出来赋值给当前线程的前继节点；
    // 然后再把当前线程的当前节点赋值给尾节点
    // 【注意】在多线程并发情况下，这里通过AtomicReference类能防止并发问题
    // 【注意】哪个线程先执行到这里就会先执行predNode.set(preNode);语句，因此构建了一条逻辑线程等待链
    // 这条链避免了线程饥饿现象发生
    CLHNode preNode = tailNode.getAndSet(currNode);
    // 将刚获取的尾结点（前一线程的当前节点）付给当前线程的前继节点ThreadLocal
    // 【思考】这句代码也可以去掉吗，如果去掉有影响吗？
    predNode.set(preNode);
    // 【1】若前继节点的locked状态为false，则表示获取到了锁，不用自旋等待；
    // 【2】若前继节点的locked状态为true，则表示前一线程获取到了锁或者正在等待，自旋等待
    while (preNode.locked) {
        try {
            Thread.sleep(1000);
        } catch (Exception e) {

        }
        System.out.println("线程" + Thread.currentThread().getName() + "没能获取到锁，进行自旋等待。。。");
    }
    // 能执行到这里，说明当前线程获取到了锁
    System.out.println("线程" + Thread.currentThread().getName() + "获取到了锁！！！");
}
```

![加锁过程](https://s2.51cto.com/images/blog/202106/25/396b1b7efafe61ef9464b273ab245001.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## CLH释放锁的过程

```java
// CLHLock.java

/**
 * 释放锁
 */
public void unLock() {
    // 获取当前线程的当前节点
    CLHNode node = curNode.get();
    // 进行解锁操作
    // 这里将locked至为false，此时执行了lock方法正在自旋等待的后继节点将会获取到锁
    // 【注意】而不是所有正在自旋等待的线程去并发竞争锁
    node.locked = false;
    System.out.println("线程" + Thread.currentThread().getName() + "释放了锁！！！");
    // 小伙伴们可以思考下，下面两句代码的作用是什么？？？
    CLHNode newCurNode = new CLHNode();
    curNode.set(newCurNode);

    // 【优化】能提高GC效率和节省内存空间，请思考：这是为什么？
    // curNode.set(predNode.get());
}
```
* 首先从当前线程的线程本地变量中取出当前的CLHNode节点，同时这个CLHNode节点被后面一个线程的preNode变量指向着。
* 将locked状态置为false释放锁（locke被volitile关键字修饰，此时后面自旋等待的线程的局部变量preNode.locked都为false，因此后面自旋等待的线程就会结束循环，获取到锁)
* 然后给当前线程的标识当前节点的线程本地变量重新赋值一个显得CLHNode。
  ![](https://s2.51cto.com/images/blog/202106/25/28ff31e9135672b7fd677e06874874a1.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)


