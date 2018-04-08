##Java集合类知识点总结

Java的集合类都放到了java.util包下，层次关系如下图：

![图片 10](https://ws1.sinaimg.cn/large/006tKfTcgy1fq5fe82ipaj30o10lptbs.jpg)

1、Collection接口是集合类的根接口，Collection接口又继承Iterable类，都实现了Iterator接口，这是一个用于遍历集合中元素的接口，主要包含以下三种方法：

hasNext()是否还有下一个元素。

next()返回下一个元素。

remove()删除当前元素。

Collection接口没有直接的实现类，被Set、List、Queue继承，其中Set和List的区别：List元素有放入顺序，元素可重复，Set元素无放入顺序，元素不可重复，但是元素在set中的位置是由该元素的HashCode决定，其位置是固定。其中HashSet的底层实现是HashMap。

**ArrayList实现原理：**

 每个ArrayList实例都有一个容量（ArrayList默认容量是10），该容量是指用来存储列表元素的数组的大小。它总是至少等于列表的大小。随着向ArrayList中不断添加元素，其容量也自动增长。由下图可看出自动增长会带来数据向新数组的重新拷贝。每次数组容量增长大约是其容量的1.5倍，因此这种操作的代价很高，我们应尽量避免数组容量的扩充，当可预知时最好提前指定数据大小。ArrayList的最大容量是Java的MAX_VALUE -8(当前容量没超过该值是)/MAX_VALUE(当前容量已超过MAX_VALUE -8时),此实现是不同步的。

![图片 11](https://ws3.sinaimg.cn/large/006tKfTcgy1fq5fe69ei5j30o1090abq.jpg)

**LinkedList实现原理：**

LinkedList是一个继承于AbstractSequentiaList的双向链表，它可以被当做堆栈、队列、双端队列进行操作。为什么要继承自AbstractSequentialList ?

1.   AbstractSequentialList 实现了get(int index)、set(intindex, E element)、add(int index, E element) 和remove(int index)这些骨干性函数。降低了List接口的复杂度。LinkedList底层的数据结构是基于双向循环链表的，且头结点中不存放数据。它的查找是分半查找，先判断index是在链表的哪一半，然后再去对应区域查找，这样最多只要遍历链表的一半节点即可找到。LinkedList是非同步的。

2、Map是Java.util包中的另一个接口，它和Collection接口没有关系，是相互独立的，但都属于集合类的一部分。Map包含了key-value对，Map不能包含重复的key，但是可以包含相同的的value，其中HashMap允许存在一个为null的key，多个为null的value，而hashtable的key和value都不允许为null。

**HashMap的实现原理：**

HashMap是基于哈希表的Map接口的非同步实现，此类不保证映射的顺序。HashMap实际上是一个“链表散列”的数据结构，即数组和链表的结合体，HashMap的默认容量是16。HashMap的存储实现：先根据key的hashCode重新计算hash值，根据hash值得到这个元素在数组中的位置（即下标），如果数组该位置上已经存放有其他元素了，那么在这个位置上的元素将以链表的形式存放，新加入的放在链头，最先加入的放在链尾。如果数组该位置上没有元素，就直接将该元素放到此数组中的该位置上。 当HashMap中的元素越来越多的时候，hash冲突的几率也就越来越高，因为数组的长度是固定的。所以为了提高查询的效率，就要对HashMap的数组进行扩容，数组扩容这个操作也会出现在ArrayList中，这是一个常用的操作，而在HashMap数组扩容之后，最消耗性能的点就出现了：原数组中的数据必须重新计算其在新数组中的位置，并放进去，这就是resize。那么HashMap什么时候进行扩容呢？当HashMap中的元素个数超过数组大小*loadFactor时，就会进行数组扩容，loadFactor的默认值为0.75，这是一个折中的取值。负载因子衡量的是一个散列表的空间的使用程度，负载因子越大表示散列表的装填程度越高，反之愈小。因此如果负载因子越大，对空间的利用更充分，然而后果是查找效率的降低；如果负载因子太小，那么散列表的数据将过于稀疏，对空间造成严重浪费。jdk1.8之前的hashmap都采用上图的结构，都是基于一个数组和多个单链表，hash值冲突的时候，就将对应节点以链表的形式存储。如果在一个链表中查找其中一个节点时，将会花费O（n）的查找时间，会有很大的性能损失。到了jdk1.8，当同一个hash值的节点数小于8时，采用单链表形式存储，大于8时采用红黑树，当已采用红黑树时，删减到6个时又继续采用单链表形式存储。如下图所示：

![图片 13](https://ws4.sinaimg.cn/large/006tKfTcgy1fq5fe4bt13j30ld0ojabf.jpg)

采用了Fail-Fast机制，通过一个modCount值记录修改次数，对HashMap内容的修改都将增加这个值。迭代器初始化过程中会将这个值赋给迭代器的expectedModCount，在迭代过程中，判断modCount跟expectedModCount是否相等，如果不相等就表示已经有其他线程修改了Map，马上抛出异常。

(扩展知识点：红黑树

R-B Tree，全称是Red-Black Tree，又称为“红黑树”，它一种特殊的二叉查找树，它的时间复杂度是O(lgn)。红黑树相对是接近平衡的二叉树，红黑树的特性**:**
（1）每个节点或者是黑色，或者是红色。
（2）根节点是黑色。
（3）每个叶子节点（NIL）是黑色。 [注意：这里叶子节点，是指为空(NIL或NULL)的叶子节点！]
（4）如果一个节点是红色的，则它的子节点必须是黑色的。
（5）从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点。

![图片 12](https://ws2.sinaimg.cn/large/006tKfTcgy1fq5fe8ybugj30o10bpwfv.jpg)

**Hashtable的实现原理：**

Hashtable是基于哈希表的Map接口的同步实现的，这意味着它是线程安全的，它的key、value都不可以为null，Hashtable中的映射不是有序的。Hashtable的初始容量默认为11，负载因子为0.75。Hashtable在底层将key-value当成一个整体进行处理，这个整体就是一个Entry对象。Hashtable底层采用一个Entry[]数组来保存所有的key-value对，当需要存储一个Entry对象时，会根据key的hash算法来决定其在数组中的存储位置，在根据equals方法决定其在该数组位置上的链表中的存储位置；当需要取出一个Entry时，也会根据key的hash算法找到其在数组中的存储位置，再根据equals方法从该位置上的链表中取出该Entry。synchronized是针对整张Hash表的，即每次锁住整张表让线程独占。

**ConcurrentHashMap的实现原理：**

 ConcurrentHashMap最重要的一个概念就是分段锁Segment，同HashMap一样，Segment包含一个HashEntry数组，数组中的每一个HashEntry既是一个键值对，也是一个链表的头节点。在ConcurrentHashMap集合中有2的N次方个Segment，共同保存在一个名为segments的数组中，整个ConcurrentHashMap结构如下：

![屏幕快照 2018-04-02 下午4.57.22](https://ws3.sinaimg.cn/large/006tKfTcgy1fq5fe74e27j31480nsgoe.jpg)

可以说，ConcurrentHashMap是一个二级哈希表。在一个总的哈希表下面有若干个子哈希表。这样的优势就是每一个Segment就好比一个自治区，读写操作高度自治，Segment之间互不影响。不同的Segment的写入是可以并发执行的，同一个Segment的读写是可以并发执行的，Segment的写入是需要上锁的，因此对同一个Segment的并发写入会被阻塞。由此可见，ConcurrentHashMap当中每个Segment各自持有一把锁，在保证线程安全的同时降低锁的粒度，让并发操作效率更高。

**Get方法：**

1.为输入的Key做Hash运算，得到hash值。

2.通过hash值，定位到对应的Segment对象

3.再次通过hash值，定位到Segment当中数组的具体位置。

**Put方法：**

1.为输入的Key做Hash运算，得到hash值。

2.通过hash值，定位到对应的Segment对象

3.获取可重入锁

4.再次通过hash值，定位到Segment当中数组的具体位置。

5.插入或覆盖HashEntry对象。

6.释放锁。

**Size方法：**

Size方法的目的是统计ConcurrentHashMap的总元素数量， 自然需要把各个Segment内部的元素数量汇总起来。

1.遍历所有的Segment。

2.把Segment的元素数量累加起来。

3.把Segment的修改次数累加起来。

4.判断所有Segment的总修改次数是否大于上一次的总修改次数。如果大于，说明统计过程中有修改，重新统计，尝试次数+1；如果不是。说明没有修改，统计结束。

5.如果尝试次数超过阈值，则对每一个Segment加锁，再重新统计。

6.再次判断所有Segment的总修改次数是否大于上一次的总修改次数。由于已经加锁，次数一定和上次相等。

7.释放锁，统计结束。



**LinkedHashMap的实现原理：**

LinkedHashMap继承与HashMap，底层使用哈希表与双向链表来保存所有元素，采用的hash算法和HashMap相同，但是它重新定义了数组中保存的元素Entry，该Entry除了保存当前对象的引用外，还保存了其上一个元素before和下一个元素after的引用，从而在哈希表的基础上构成了双向链接列表。

**LinkedHashSet的实现原理：**

对于LinkedHashSet而言，它继承与HashSet、又基于LinkedHashMap来实现的。LinkedHashSet底层使用LinkedHashMap来保存所有元素，它继承与HashSet，其所有的方法操作上又与HashSet相同。

**TreeMap的实现原理：**

TreeMap是一个二叉树的数据结构，不允许出现相同的键，从下图代码可看出，有一个泛型对象实体Entry，Entry里面维持一个左、右子树跟父树的对象属性。如下图所示：

![图片 14](https://ws1.sinaimg.cn/large/006tKfTcgy1fq5fe59gr2j30mh0t6gog.jpg)



**迭代器相关知识总结：**

因为Java容器的内部结构不同，很多时候可能不知道该怎样去遍历一个容器中的元素。所以为了使容器内元素的操作更为简单，Java引入迭代器模式，把访问逻辑从不同类型的集合类中抽取出来，从而避免向外部暴露集合的内部结构。在使用Iterator的时候禁止对所遍历的容器进行改变其大小结构的操作，因为在迭代之前，迭代器已经通过list.iterator()创建出来了，如果在迭代的过程中，又对list进行了改变其容器大小的操作，那么Java就会给出异常，因为此时Iterator对象已经无法主动同步list做出的改变，Java会认为你做出这样的操作是线程不安全的，抛出ConcurrentModificationException异常。关于快速失败迭代和安全失败迭代的区别：

| 快速失败迭代器                                      | 安全失败迭代器                   |
| --------------------------------------------------- | -------------------------------- |
| 在迭代时不允许修改集合                              | 在迭代时允许修改集合             |
| 迭代时被修改抛出ConcurrentModificationException异常 | 迭代时集合被修改不抛出异常       |
| 使用原集合遍历集合元素                              | 使用原集合的副本遍历集合元素     |
| 迭代器不要求额外的内存                              | 迭代器需要额外的内存克隆集合对象 |
| 示例：ArrayList、Vector、HashMap                    | 示例：ConcurrentHashMap          |

 

 