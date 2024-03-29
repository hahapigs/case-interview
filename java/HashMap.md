## HashMap

### 你能说一下 HashMap 吗？

HashMap是一个key-value形式的集合，底层结构是数组+链表。数组随机访问元素速度快，插入和删除效率低；链表访问数据需要遍历操作，插入和删除效率高。HashMap结合了数组和链表的优点，使得访问和操作元素效率都很好。

HashMap默认初始化的大小为16，负载因子为0.75，言外之意就是当数组容量达到75%后，就会进行一次扩容，扩容后的容量是原先的2倍。

HashMap常规情况下以数组形式存储，存储位置根据index = hash % n确定。当出现hash碰撞以后，就会以链表形式进行存储，也就是所谓的拉链法，前一个元素指向下一个元素的位置。当链表上的元素超过8，数组长度在64以内（不包含64本身），就会进行一次扩容进行存储；如果数组长度大于等于64，则转化为红黑树结构进行存储。

### 说一下HashMap 的 putVal() 的流程

1、先判断是否未初始化

2、根据index = hash & (n - 1)，判断存储位置的value是否为null，如果是直接进行存储

3、根据key值判断是否有相同的key，相同则覆盖，反之链表+1

4、如果链表大于8，则调用treeifyBin()进行红黑树操作

说明：
index = hash & (n - 1) 其实就是在比较hash，hash相同，则index相同，必然会发生hash碰撞
treefyBin()里有对数组大于等于64的判断

### HashMap 的负载因为什么是0.75?

假设HashMap的负载因子是1的话，那么数组必须存满之后再进行扩容。这样虽然空间的利用率上去了，但是因为可用空间有限的原因，导致hash碰撞增加，那么链表的长度就会很复杂，从而增加查询难度。

假设HashMap的独爱因子是0.1的话，那么数组有新数据进来就会马上进行扩容。这样hash碰撞就会减少，从而提高了查询效率，但是过多频率的扩容和过低的空间利用率，同样导致性能的影响。

### HashMap 的初始化值为什么是16？或者说 HashMap 为什么扩容后是原先的2倍？或者说为什么 n = 2^n ？

我们先说一下获取数据存储位置的公式：index = hash % n, 等同于 index = hash & (n - 1)。这两个公式之所以相等，是因为经过大量测算，当n为 2^n 的时候，他们惊奇的一致。在计算机二进制计算过程可定要比取余操作效率要高，所以使用 index = hash & (2^n - 1)来计算元素的存储位置。其次，还有一个非常重要的原因就是离散分布均匀的问题。假设我们要存放n个元素，他的hash不存在碰撞，集合长度为2，4，8，16，32...；hash值分别为1，2，3，4，5...n,他们执行完存储位置如下：

| 2^n  | 2^n-1 | hash | index |
| :--: | :---: | :--: | :---: |
|  2   |   1   |  1   |   1   |
|  4   |   3   |  2   |   2   |
|  8   |   7   |  3   |   4   |
|  16  |  16   |  4   |   5   |
|  32  |  31   |  5   |   5   |

由此我们可以看出index的散列排序很好，当n不是2^n的时候index的散列排序会比较差。那么至于为什么是16这个数值呢，原因很简单，太小会频繁扩容，造成性能影响；太大会导致空间利用率降低，浪费内存。

### HashMap 为什么树化的阀值是8，64？

理想情况下，随机hashcode在箱中的bin遵循泊松分布原则，当阀值为8的时候，发生hash碰撞的概率为亿分之六，比千万份之一的概率还要低。
```
* 0:    0.60653066
* 1:    0.30326533
* 2:    0.07581633
* 3:    0.01263606
* 4:    0.00157952
* 5:    0.00015795
* 6:    0.00001316
* 7:    0.00000094
* 8:    0.00000006
```
至于为什么是在大于等于64再进行树化操作呢，源码中并未直接要求该数值一定要64，原文描述为<code>至少为 4 * TREEIFY_THRESHOLD</code>，也就是32。当然设置64过多的应该是经验上的一个数值
```

/**
 * The smallest table capacity for which bins may be treeified.
 * (Otherwise the table is resized if too many nodes in a bin.)
 * Should be at least 4 * TREEIFY_THRESHOLD to avoid conflicts
 * between resizing and treeification thresholds.
 */
static final int MIN_TREEIFY_CAPACITY = 64;
```


### HashMap 为什么要用红黑树？

数组的最好情况下的时间复杂度是O(1)，最坏情况下是O(n)；链表时间复杂度是O(n)；红黑树是O(logn)。
显而易见，红黑树查询效率比链表要好。

### HashMap 为什么不直接红黑树？

空间：红黑树需要左旋，右旋操作，占用空间是链表的两倍
时间：当链表长度较小的时候，查找和遍历性能都很高，只有长链表的时候，性能会逐渐下降
综合考虑，红黑树并不能直接提高效率，结合树化的阀值，所以链表长度大于8的时候，才会转化为红黑树

延伸：
很多面试的时候，如果面对一个资深的面试官，你能提出一些不同的想法，可能会加深对你的印象

我：我认为有一部分原因可能是设计者考虑开发习惯和纠错的设计想法，我可以说说我的理解

面试官：好呀，你说说吧

我：假设我们重写 hashcode 方法，让其返回值都等于某个固定值，如果 hashCode 都等于同一个值，那么 hash 值势必都会相同，同样他们的存储位置 index 也相同。如果我们使用了开始就树化的操作，那么可定会造成空间上浪费。
