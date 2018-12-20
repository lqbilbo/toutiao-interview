## 集合
### ArrayList与LinkedList区别

|ArrayList|LinkedList|
|--------|--------|
|数组|双向链表|
|增删的时候在扩容的时候慢，通过索引查询快，通过对象查索引慢|增删快，通过索引查询慢，通过对象查索引慢|
|当数组无法容纳下此次添加的元素时进行扩容|无|
|扩容之后容量为原来的1.5倍|无

### ArrayList与Vector区别

1. Vector是线程安全的，源码中有很多的synchronized可以看出，而ArrayList不是。导致Vector效率无法和ArrayList相比；
2. ArrayList和Vector都采用线性连续存储空间，当存储空间不足的时候，ArrayList默认增加为原来的50%，Vector默认增加为原来的一倍；
3. Vector可以设置capacityIncrement，而ArrayList不可以，从字面理解就是capacity容量，Increment增加，容量增长的参数。

### HashMap
1. JDK1.8以前 HashMap 的实现是 数组+链表，即使哈希函数取得更好，也很难达到元素百分百均匀分布。当 HashMap中有大量的元素都存放在同一个桶中时，这个桶下游一条尝尝的链表，这个时候 HashMap 就相当于一个单链表（退化），加入单链表有n个元素，遍历的时间复杂度就是O(n)，完全失去了它的优势。针对这种情况，JDK1.8中引入了红黑树（查找时间复杂度为O(logn)）来优化这个问题
2. 为什么线程不安全？多线程PUT操作时可能会覆盖刚PUT进去的值；扩容操作会让链表形成环形数据结构，形成死循环
3. 容量的默认大小是16，负载因子是0.75，当HashMap的size>16*0.75时就会发生扩容。
4. 为什么容量是2的倍数？在根据hashcode查找数组中元素时，取模性能远远低于与运算性能，且和2^n-1进行与操作能保证各种不同的hashcode对应的元素也能均匀分布在数组中

### ConcurrentHashMap原理
[http://www.jasongj.com/java/concurrenthashmap/](http://www.jasongj.com/java/concurrenthashmap/)

HashTable在每次同步执行时都要锁住整个结构。ConcurrentHashMap锁的方式是稍微细粒度的。ConcurrentHashMap将hash表分为16个桶（默认值）

#### Java7
![](https://github.com/xbox1994/2018-Java-Interview/raw/master/images/j3.jpg)

ConcurrentHashMap类中包含两个静态内部类HashEntry和Segment。HashEntry用来封装映射表的键 / 值对；Segment用来充当锁的角色，每个Segment对象守护整个散列映射表的若干个桶。每个桶是由若干个HashEntry对象链接起来的链表。一个ConcurrentHashMap实例中包含由若干个Segment对象组成的数组。

#### Java8
![](https://github.com/xbox1994/2018-Java-Interview/raw/master/images/j4.jpg)

1. 为进一步提高并发性，放弃了分段锁，使用CAS + synchronized来保证
2. PUT操作：如果Key对应的数组元素为null，则通过CAS操作将其设置为当前值。如果Key对应的数组元素（也即链表表头或者树的根元素）不为null，则对该元素使用synchronized关键字申请锁，然后进行操作。如果该put操作是的当前链表长度超过一定阈值，则将链表（寻址时间复杂度为O(N)）转换为红黑树（寻址时间复杂度为O(log(N))）