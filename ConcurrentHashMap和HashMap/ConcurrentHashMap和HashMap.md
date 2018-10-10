## ConcurrentHashMap和HashMap和Hashtable以及为什么重写了equels方法一定要重写hashcode方法

[TOC]



## ConcurrentMap、hashTable与hashMap的区别

[ConcurrentMap、hashTable与hashMap的区别](https://blog.csdn.net/songfeihu0810232/article/details/70012260/)

### hashMap

1、HashMap默认不是线程安全的。 
 2、HashMap是map接口的实例，是将键映射到值的对象，其中键和值都是对象，并且不能包含重复键，但可以包含重复值。 
 3、HashMap允许null key和null value，而hashtable不允许。 
 4、因为线程安全的问题，HashMap效率比HashTable的要高。应用程序一般在更高的层面上实 现了保护机制，而不是依赖于这些底层数据结构的同步。

### hashTable

1、HashMap是非线程安全的，HashTable是线程安全的，内部的方法基本都是synchronized。 为了实现线程安全，代价很高，在高并发场景下表现很差。
 2、HashTable不允许有null值的存在。

### ConcurrentMap

1、synchronized关键字加锁的原理，其实是对对象加锁，不论你是在方法前加synchronized还是语句块前加，锁住的都是对象整体，但是ConcurrentHashMap的同步机制和这个不同，它不是加synchronized关键字，而是基于lock操作的，这样的目的是保证同步的时候，锁住的不是整个对象。一个ConcurrentHashMap由多个segment组成，每个segment包含一个Entity的数组。这里比HashMap多了一个segment类。该类继承了ReentrantLock类，所以本身是一个锁。当多线程对ConcurrentHashMap操作时，不是完全锁住map，而是锁住相应的segment。这样提高了并发效率。 
 2、不可以有null键。 
 3、缺点：当遍历ConcurrentMap中的元素时，需要获取所有的segment 的锁，使用遍历时慢。锁的增多，占用了系统的资源。使得对整个集合进行操作的一些方法（例如 size() 或 isEmpty() ）的实现更加困难，因为这些方法要求一次获得许多的锁，并且还存在返回不正确的结果的风险。所以在使用ConcurrentMap时，应当尽量避免使用 size() 或 isEmpty()  这类对整个集合进行操作的方法。

## jdk7与jdk8中HashMap实现原理及源码分析

[JDK7与JDK8中HashMap的实现](http://www.importnew.com/23164.html)

[HashMap实现原理及源码分析](https://www.cnblogs.com/chengxiao/p/6059914.html) (JDK7)

[JDK8中的HashMap实现](https://blog.csdn.net/qq_16525279/article/details/82697213)（JDK8）

### JDK7中的HashMap 

HashMap底层维护一个数组，数组中的每一项都是一个Entry 

```java
transient Entry<K,V>[] table;
```

我们向 HashMap 中所放置的对象实际上是存储在该数组当中；  

 而Map中的key，value则以Entry的形式存放在数组中 

```java
static class Entry<K,V> implements Map.Entry<K,V> {
        final K key;
        V value;
        Entry<K,V> next;
        int hash;
```

 而这个Entry应该放在数组的哪一个位置上（这个位置通常称为位桶或者hash桶，即hash值相同的Entry会放在同一位置，用链表相连），是通过key的hashCode来计算的。 

```java
final int hash(Object k) {
        int h = 0;
        h ^= k.hashCode();

        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }
```

 通过hash计算出来的值将会使用indexFor方法找到它应该所在的table下标： 

```java
static int indexFor(int h, int length) {
        return h & (length-1);
    }
```

 当两个key通过hashCode计算相同时，则发生了hash冲突(碰撞)，HashMap解决hash冲突的方式是用链表。 

当发生hash冲突时，则将存放在数组中的Entry设置为新值的next（这里要注意的是，比如A和B都hash后都映射到下标i中，之前已经有A了，当map.put(B)时，将B放到下标i中，A则为B的next，所以新值存放在数组中，旧值在新值的链表上） 

 示意图： 

![img](http://static.oschina.net/uploads/space/2016/0217/210043_4aAJ_2243330.png) 

 所以当hash冲突很多时，HashMap退化成链表。 

 总结一下map.put后的过程： 

 当向 HashMap 中 put 一对键值时，它会根据 key的 hashCode 值计算出一个位置， 该位置就是此对象准备往数组中存放的位置。  

 如果该位置没有对象存在，就将此对象直接放进数组当中；如果该位置已经有对象存在了，则顺着此存在的对象的链开始寻找(为了判断是否是否值相同，map不允许<key,value>键值对重复)， 如果此链上有对象的话，再去使用 equals方法进行比较，如果对此链上的每个对象的 equals 方法比较都为 false，则将该对象放到数组当中，然后将数组中该位置以前存在的那个对象链接到此对象的后面。  

 值得注意的是，当key为null时，都放到table[0]中 

```java
private V putForNullKey(V value) {
        for (Entry<K,V> e = table[0]; e != null; e = e.next) {
            if (e.key == null) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }
        modCount++;
        addEntry(0, null, value, 0);
        return null;
    }
```

threshold等于capacity*load factor

```java
void addEntry(int hash, K key, V value, int bucketIndex) {
        if ((size >= threshold) && (null != table[bucketIndex])) {
            resize(2 * table.length);
            hash = (null != key) ? hash(key) : 0;
            bucketIndex = indexFor(hash, table.length);
        }

        createEntry(hash, key, value, bucketIndex);
    }
```

jdk7中resize，只有当 size>=threshold并且 table中的那个槽中已经有Entry时，才会发生resize。即有可能虽然size>=threshold，但是必须等到每个槽都至少有一个Entry时，才会扩容。还有注意每次resize都会扩大一倍容量  

### JDK8中的HashMap 

 一直到JDK7为止，HashMap的结构都是这么简单，基于一个数组以及多个链表的实现，hash值冲突的时候，就将对应节点以链表的形式存储。 

 这样子的HashMap性能上就抱有一定疑问，如果说成百上千个节点在hash时发生碰撞，存储一个链表中，那么如果要查找其中一个节点，那就不可避免的花费O(N)的查找时间，这将是多么大的性能损失。这个问题终于在JDK8中得到了解决。再最坏的情况下，链表查找的时间复杂度为O(n),而红黑树一直是O(logn),这样会提高HashMap的效率。  

 JDK7中HashMap采用的是位桶+链表的方式，即我们常说的散列链表的方式，而JDK8中采用的是位桶+链表/红黑树（有关红黑树请查看[红黑树](http://my.oschina.net/hosee/blog/618828)）的方式，也是非线程安全的。当某个位桶的链表的长度达到某个阀值的时候，这个链表就将转换成红黑树。 

  

 ![img](http://static.oschina.net/uploads/space/2016/0222/184438_IA5n_2243330.jpg) 

  

 JDK8中，当同一个hash值的节点数不小于8时，将不再以单链表的形式存储了，会被调整成一颗红黑树（上图中null节点没画）。这就是JDK7与JDK8中HashMap实现的最大区别。  

 接下来，我们来看下JDK8中HashMap的源码实现。 

 JDK中Entry的名字变成了Node，原因是和红黑树的实现TreeNode相关联。 

```java
transient Node<K,V>[] table; 
```

 当冲突节点数不小于8-1时，转换成红黑树。 

```java
static final int TREEIFY_THRESHOLD = 8;
```

 以put方法在JDK8中有了很大的改变  

```java
public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
 }


final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab;
	Node<K,V> p; 
	int n, i;
	//如果当前map中无数据，执行resize方法。并且返回n
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
	 //如果要插入的键值对要存放的这个位置刚好没有元素，那么把他封装成Node对象，放在这个位置上就完事了
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
	//否则的话，说明这上面有元素
        else {
            Node<K,V> e; K k;
	    //如果这个元素的key与要插入的一样，那么就替换一下，也完事。
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
	    //1.如果当前节点是TreeNode类型的数据，执行putTreeVal方法
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
		//还是遍历这条链子上的数据，跟jdk7没什么区别
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
			//2.完成了操作后多做了一件事情，判断，并且可能执行treeifyBin方法
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null) //true || --
                    e.value = value;
		   //3.
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
	//判断阈值，决定是否扩容
        if (++size > threshold)
            resize();
	    //4.
        afterNodeInsertion(evict);
        return null;
    }
```

 treeifyBin()就是将链表转换成红黑树。 

put方法中调用的resize()方法 

```java

final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table; //当前所有元素所在的数组，称为老的元素数组
        int oldCap = (oldTab == null) ? 0 : oldTab.length; //老的元素数组长度
        int oldThr = threshold;	// 老的扩容阀值设置
        int newCap, newThr = 0;	// 新数组的容量，新数组的扩容阀值都初始化为0
        if (oldCap > 0) {	// 如果老数组长度大于0，说明已经存在元素
            // PS1
            if (oldCap >= MAXIMUM_CAPACITY) { // 如果数组元素个数大于等于限定的最大容量（2的30次方）
                // 扩容阀值设置为int最大值（2的31次方 -1 ），因为oldCap再乘2就溢出了。
                threshold = Integer.MAX_VALUE;	
                return oldTab;	// 返回老的元素数组
            }
           /*
            * 如果数组元素个数在正常范围内，那么新的数组容量为老的数组容量的2倍（左移1位相当于乘以2）
            * 如果扩容之后的新容量小于最大容量  并且  老的数组容量大于等于默认初始化容量（16），那么新数组的扩容阀值设置为老阀值的2倍。（老的数组容量大于16意味着：要么构造函数指定了一个大于16的初始化容量值，要么已经经历过了至少一次扩容）
            */
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        // PS2
        // 运行到这个else if  说明老数组没有任何元素
        // 如果老数组的扩容阀值大于0，那么设置新数组的容量为该阀值
        // 这一步也就意味着构造该map的时候，指定了初始化容量。
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            // 能运行到这里的话，说明是调用无参构造函数创建的该map，并且第一次添加元素
            newCap = DEFAULT_INITIAL_CAPACITY;	// 设置新数组容量 为 16
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY); // 设置新数组扩容阀值为 16*0.75 = 12。0.75为负载因子（当元素个数达到容量了4分之3，那么扩容）
        }
        // 如果扩容阀值为0 （PS2的情况）
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);  // 参见：PS2
        }
        threshold = newThr; // 设置map的扩容阀值为 新的阀值
        @SuppressWarnings({"rawtypes","unchecked"})
            // 创建新的数组（对于第一次添加元素，那么这个数组就是第一个数组；对于存在oldTab的时候，那么这个数组就是要需要扩容到的新数组）
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;	// 将该map的table属性指向到该新数组
        if (oldTab != null) {	// 如果老数组不为空，说明是扩容操作，那么涉及到元素的转移操作
            for (int j = 0; j < oldCap; ++j) { // 遍历老数组
                Node<K,V> e;
                if ((e = oldTab[j]) != null) { // 如果当前位置元素不为空，那么需要转移该元素到新数组
                    oldTab[j] = null; // 释放掉老数组对于要转移走的元素的引用（主要为了使得数组可被回收）
                    if (e.next == null) // 如果元素没有有下一个节点，说明该元素不存在hash冲突
                        // PS3
                        // 把元素存储到新的数组中，存储到数组的哪个位置需要根据hash值和数组长度来进行取模
                        // 【hash值  %   数组长度】   =    【  hash值   & （数组长度-1）】
                        //  这种与运算求模的方式要求  数组长度必须是2的N次方，但是可以通过构造函数随意指定初始化容量呀，如果指定了17,15这种，岂不是出问题了就？没关系，最终会通过tableSizeFor方法将用户指定的转化为大于其并且最相近的2的N次方。 15 -> 16、17-> 32
                        newTab[e.hash & (newCap - 1)] = e;
                        // 如果该元素有下一个节点，那么说明该位置上存在一个链表了（hash相同的多个元素以链表的方式存储到了老数组的这个位置上了）
                        // 例如：数组长度为16，那么hash值为1（1%16=1）的和hash值为17（17%16=1）的两个元素都是会存储在数组的第2个位置上（对应数组下标为1），当数组扩容为32（1%32=1）时，hash值为1的还应该存储在新数组的第二个位置上，但是hash值为17（17%32=17）的就应该存储在新数组的第18个位置上了。
                        // 所以，数组扩容后，所有元素都需要重新计算在新数组中的位置。
                    else if (e instanceof TreeNode)  // 如果该节点为TreeNode类型
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);  // 此处单独展开讨论
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;  // 按命名来翻译的话，应该叫低位首尾节点
                        Node<K,V> hiHead = null, hiTail = null;  // 按命名来翻译的话，应该叫高位首尾节点
                        // 以上的低位指的是新数组的 0  到 oldCap-1 、高位指定的是oldCap 到 newCap - 1
                        Node<K,V> next;
                        // 遍历链表
                        do {  
                            next = e.next;
                            // 这一步判断好狠，拿元素的hash值  和  老数组的长度  做与运算
                            // PS3里曾说到，数组的长度一定是2的N次方（例如16），如果hash值和该长度做与运算，结果为0，就说明该hash值一定小于数组长度（例如hash值为1），那么该hash值再和新数组的长度取摸的话，还是hash值本身，所该元素的在新数组的位置和在老数组的位置是相同的，所以该元素可以放置在低位链表中。
                            if ((e.hash & oldCap) == 0) {  
                                // PS4
                                if (loTail == null) // 如果没有尾，说明链表为空
                                    loHead = e; // 链表为空时，头节点指向该元素
                                else
                                    loTail.next = e; // 如果有尾，那么链表不为空，把该元素挂到链表的最后。
                                loTail = e; // 把尾节点设置为当前元素
                            }
                            // 如果与运算结果不为0，说明hash值大于老数组长度（例如hash值为17）
                            // 此时该元素应该放置到新数组的高位位置上
                            // 例：老数组长度16，那么新数组长度为32，hash为17的应该放置在数组的第17个位置上，也就是下标为16，那么下标为16已经属于高位了，低位是[0-15]，高位是[16-31]
                            else {  // 以下逻辑同PS4
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) { // 低位的元素组成的链表还是放置在原来的位置
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {  // 高位的元素组成的链表放置的位置只是在原有位置上偏移了老数组的长度个位置。
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead; // 例：hash为 17 在老数组放置在0下标，在新数组放置在16下标；    hash为 18 在老数组放置在1下标，在新数组放置在17下标；                   
                        }
                    }
                }
            }
        }
        return newTab; // 返回新数组
    }

```

在jdk8中 hashmap 的初始化 会调用 tableSizeFor 方法将用户指定的转化为大于其并且最相近的2的N次方。 15 -> 16、17-> 32  而在jdk7中使用的是 roundUpToPowerOf2 方法

## jdk7与jdk8中ConcurrentHashMap实现原理及源码分析

[ConcurrentHashMap实现原理及源码分析](https://www.cnblogs.com/chengxiao/p/6842045.html)(jdk7)

[ConcurrentHashMap在JDK7和JDK8中的不同实现原理](https://blog.csdn.net/woaiwym/article/details/80675789)

并发编程实践中，ConcurrentHashMap是一个经常被使用的数据结构，相比于Hashtable以及Collections.synchronizedMap()，ConcurrentHashMap在线程安全的基础上提供了更好的写并发能力，但同时降低了对读一致性的要求（这点好像CAP理论啊 O(∩_∩)O）。ConcurrentHashMap的设计与实现非常精巧，大量的利用了volatile，final，CAS等lock-free技术来减少锁竞争对于性能的影响，无论对于Java并发编程的学习还是Java内存模型的理解，ConcurrentHashMap的设计以及源码都值得非常仔细的阅读与揣摩。

### 1. JDK6与JDK7中的实现

#### 1.1 设计思路

ConcurrentHashMap采用了分段锁的设计，只有在同一个分段内才存在竞态关系，不同的分段锁之间没有锁竞争。相比于对整个Map加锁的设计，分段锁大大的提高了高并发环境下的处理能力。但同时，由于不是对整个Map加锁，导致一些需要扫描整个Map的方法（如size(), containsValue()）需要使用特殊的实现，另外一些方法（如clear()）甚至放弃了对一致性的要求（ConcurrentHashMap是弱一致性的，具体请查看[ConcurrentHashMap能完全替代HashTable吗？](http://my.oschina.net/hosee/blog/675423)）。

ConcurrentHashMap中的分段锁称为Segment，它即类似于HashMap（[JDK7与JDK8中HashMap的实现](http://my.oschina.net/hosee/blog/618953)）的结构，即内部拥有一个Entry数组，数组中的每个元素又是一个链表；同时又是一个ReentrantLock（Segment继承了ReentrantLock）。ConcurrentHashMap中的HashEntry相对于HashMap中的Entry有一定的差异性：HashEntry中的value以及next都被volatile修饰，这样在多线程读写过程中能够保持它们的可见性，代码如下：

```java
static final class HashEntry<K,V> {
        final int hash;
        final K key;
        volatile V value;
        volatile HashEntry<K,V> next;
```

#### 1.2 并发度（Concurrency Level）

并发度可以理解为程序运行时能够同时更新ConccurentHashMap且不产生锁竞争的最大线程数，实际上就是ConcurrentHashMap中的分段锁个数，即Segment[]的数组长度。ConcurrentHashMap默认的并发度为16，但用户也可以在构造函数中设置并发度。当用户设置并发度时，ConcurrentHashMap会使用大于等于该值的最小2幂指数作为实际并发度（假如用户设置并发度为17，实际并发度则为32）。运行时通过将key的高n位（n = 32 – segmentShift）和并发度减1（segmentMask）做位与运算定位到所在的Segment。segmentShift与segmentMask都是在构造过程中根据concurrency level被相应的计算出来。

如果并发度设置的过小，会带来严重的锁竞争问题；如果并发度设置的过大，原本位于同一个Segment内的访问会扩散到不同的Segment中，CPU cache命中率会下降，从而引起程序性能下降。（文档的说法是根据你并发的线程数量决定，太多会导性能降低）

#### 1.3 创建分段锁

和JDK6不同，JDK7中除了第一个Segment之外，剩余的Segments采用的是延迟初始化的机制：每次put之前都需要检查key对应的Segment是否为null，如果是则调用ensureSegment()以确保对应的Segment被创建。

ensureSegment可能在并发环境下被调用，但与想象中不同，ensureSegment并未使用锁来控制竞争，而是使用了Unsafe对象的getObjectVolatile()提供的原子读语义结合CAS来确保Segment创建的原子性。代码段如下：

```

if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u)) == null) { // recheck
                Segment<K,V> s = new Segment<K,V>(lf, threshold, tab);
                while ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
                       == null) {
                    if (UNSAFE.compareAndSwapObject(ss, u, null, seg = s))
                        break;
                }
}
```

#### 1.4 put/putIfAbsent/putAll

和JDK6一样，ConcurrentHashMap的put方法被代理到了对应的Segment（定位Segment的原理之前已经描述过）中。与JDK6不同的是，JDK7版本的ConcurrentHashMap在获得Segment锁的过程中，做了一定的优化 - 在真正申请锁之前，put方法会通过tryLock()方法尝试获得锁，在尝试获得锁的过程中会对对应hashcode的链表进行遍历，如果遍历完毕仍然找不到与key相同的HashEntry节点，则为后续的put操作提前创建一个HashEntry。当tryLock一定次数后仍无法获得锁，则通过lock申请锁。

需要注意的是，由于在并发环境下，其他线程的put，rehash或者remove操作可能会导致链表头结点的变化，因此在过程中需要进行检查，如果头结点发生变化则重新对表进行遍历。而如果其他线程引起了链表中的某个节点被删除，即使该变化因为是非原子写操作（删除节点后链接后续节点调用的是Unsafe.putOrderedObject()，该方法不提供原子写语义）可能导致当前线程无法观察到，但因为不影响遍历的正确性所以忽略不计。

之所以在获取锁的过程中对整个链表进行遍历，主要目的是希望遍历的链表被CPU cache所缓存，为后续实际put过程中的链表遍历操作提升性能。

在获得锁之后，Segment对链表进行遍历，如果某个HashEntry节点具有相同的key，则更新该HashEntry的value值，否则新建一个HashEntry节点，将它设置为链表的新head节点并将原头节点设为新head的下一个节点。新建过程中如果节点总数（含新建的HashEntry）超过threshold，则调用rehash()方法对Segment进行扩容，最后将新建HashEntry写入到数组中。

put方法中，链接新节点的下一个节点（HashEntry.setNext()）以及将链表写入到数组中（setEntryAt()）都是通过Unsafe的putOrderedObject()方法来实现，这里并未使用具有原子写语义的putObjectVolatile()的原因是：JMM会保证获得锁到释放锁之间所有对象的状态更新都会在锁被释放之后更新到主存，从而保证这些变更对其他线程是可见的。

#### 1.5 rehash

相对于HashMap的resize，ConcurrentHashMap的rehash原理类似，但是Doug Lea为rehash做了一定的优化，避免让所有的节点都进行复制操作：由于扩容是基于2的幂指来操作，假设扩容前某HashEntry对应到Segment中数组的index为i，数组的容量为capacity，那么扩容后该HashEntry对应到新数组中的index只可能为i或者i+capacity，因此大多数HashEntry节点在扩容前后index可以保持不变。基于此，rehash方法中会定位第一个后续所有节点在扩容后index都保持不变的节点，然后将这个节点之前的所有节点重排即可。

#### 1.6 remove

和put类似，remove在真正获得锁之前，也会对链表进行遍历以提高缓存命中率。

#### 1.7 get与containsKey

get与containsKey两个方法几乎完全一致：他们都没有使用锁，而是通过Unsafe对象的getObjectVolatile()方法提供的原子读语义，来获得Segment以及对应的链表，然后对链表遍历判断是否存在key相同的节点以及获得该节点的value。但由于遍历过程中其他线程可能对链表结构做了调整，因此get和containsKey返回的可能是过时的数据，这一点是ConcurrentHashMap在弱一致性上的体现。如果要求强一致性，那么必须使用Collections.synchronizedMap()方法。

#### 1.8 size、containsValue

这些方法都是基于整个ConcurrentHashMap来进行操作的，他们的原理也基本类似：首先不加锁循环执行以下操作：循环所有的Segment（通过Unsafe的getObjectVolatile()以保证原子读语义），获得对应的值以及所有Segment的modcount之和。如果连续两次所有Segment的modcount和相等，则过程中没有发生其他线程修改ConcurrentHashMap的情况，返回获得的值。

当循环次数超过预定义的值时，这时需要对所有的Segment依次进行加锁，获取返回值后再依次解锁。值得注意的是，加锁过程中要强制创建所有的Segment，否则容易出现其他线程创建Segment并进行put，remove等操作。代码如下：

```java
for(int j =0; j < segments.length; ++j)
ensureSegment(j).lock();// force creation
```

一般来说，应该避免在多线程环境下使用size和containsValue方法。

最后，与HashMap不同的是，`ConcurrentHashMap并不允许key或者value为null`，按照Doug Lea的说法，这么设计的原因是在ConcurrentHashMap中，一旦value出现null，则代表HashEntry的key/value没有映射完成就被其他线程所见，需要特殊处理。在JDK6中，get方法的实现中就有一段对HashEntry.value == null的防御性判断。但Doug Lea也承认实际运行过程中，这种情况似乎不可能发生

### 2. JDK8中的实现

ConcurrentHashMap在JDK8中进行了巨大改动，很需要通过源码来再次学习下Doug Lea的实现方法。

它摒弃了Segment（锁段）的概念，而是启用了一种全新的方式实现,利用CAS算法。它沿用了与它同时期的HashMap版本的思想，底层依然由“数组”+链表+红黑树的方式思想([JDK7与JDK8中HashMap的实现](http://my.oschina.net/hosee/blog/618953))，但是为了做到并发，又增加了很多辅助的类，例如TreeBin，Traverser等对象内部类。

#### 2.1 重要的属性

首先来看几个重要的属性，与HashMap相同的就不再介绍了，这里重点解释一下sizeCtl这个属性。可以说它是ConcurrentHashMap中出镜率很高的一个属性，因为它是一个控制标识符，在不同的地方有不同用途，而且它的取值不同，也代表不同的含义。

- 负数代表正在进行初始化或扩容操作

- -1代表正在初始化

- -N 表示有N-1个线程正在进行扩容操作

- 正数或0代表hash表还没有被初始化，这个数值表示初始化或下一次进行扩容的大小，这一点类似于扩容阈值的概念。还后面可以看到，它的值始终是当前ConcurrentHashMap容量的0.75倍，这与loadfactor是对应的。

#### 2.2 重要的类

Node

Node是最核心的内部类，它包装了key-value键值对，所有插入ConcurrentHashMap的数据都包装在这里面。它与HashMap中的定义很相似，但是但是有一些差别它对value和next属性设置了volatile同步锁(与JDK7的Segment相同)，它不允许调用setValue方法直接改变Node的value域，它增加了find方法辅助map.get()方法。

TreeNode

树节点类，另外一个核心的数据结构。当链表长度过长的时候，会转换为TreeNode。但是与HashMap不相同的是，它并不是直接转换为红黑树，而是把这些结点包装成TreeNode放在TreeBin对象中，由TreeBin完成对红黑树的包装。而且TreeNode在ConcurrentHashMap集成自Node类，而并非HashMap中的集成自LinkedHashMap.Entry<K,V>类，也就是说TreeNode带有next指针，这样做的目的是方便基于TreeBin的访问。

TreeBin

这个类并不负责包装用户的key、value信息，而是包装的很多TreeNode节点。它代替了TreeNode的根节点，也就是说在实际的ConcurrentHashMap“数组”中，存放的是TreeBin对象，而不是TreeNode对象，这是与HashMap的区别。另外这个类还带有了读写锁。

这里仅贴出它的构造方法。可以看到在构造TreeBin节点时，仅仅指定了它的hash值为TREEBIN常量，这也就是个标识为。同时也看到我们熟悉的红黑树构造方法

 ForwardingNode

一个用于连接两个table的节点类。它包含一个nextTable指针，用于指向下一张表。而且这个节点的key value next指针全部为null，它的hash值为-1. 这里面定义的find的方法是从nextTable里进行查询节点，而不是以自身为头节点进行查找。

#### 2.3 Unsafe与CAS

在ConcurrentHashMap中，随处可以看到U, 大量使用了U.compareAndSwapXXX的方法，这个方法是利用一个CAS算法实现无锁化的修改值的操作，他可以大大降低锁代理的性能消耗。这个算法的基本思想就是不断地去比较当前内存中的变量值与你指定的一个变量值是否相等，如果相等，则接受你指定的修改的值，否则拒绝你的操作。因为当前线程中的值已经不是最新的值，你的修改很可能会覆盖掉其他线程修改的结果。这一点与乐观锁，SVN的思想是比较类似的。

unsafe静态块

unsafe代码块控制了一些属性的修改工作，比如最常用的SIZECTL 。在这一版本的concurrentHashMap中，大量应用来的CAS方法进行变量、属性的修改工作。利用CAS进行无锁操作，可以大大提高性能。

三个核心方法

ConcurrentHashMap定义了三个原子操作，用于对指定位置的节点进行操作。正是这些原子操作保证了ConcurrentHashMap的线程安全。

```java
//获得在i位置上的Node节点
    static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
        return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
    }
		//利用CAS算法设置i位置上的Node节点。之所以能实现并发是因为他指定了原来这个节点的值是多少
		//在CAS算法中，会比较内存中的值与你指定的这个值是否相等，如果相等才接受你的修改，否则拒绝你的修改
		//因此当前线程中的值并不是最新的值，这种修改可能会覆盖掉其他线程的修改结果  有点类似于SVN
    static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                        Node<K,V> c, Node<K,V> v) {
        return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
    }
		//利用volatile方法设置节点位置的值
    static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
        U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);
    }
```

#### 2.4 初始化方法initTable

对于ConcurrentHashMap来说，调用它的构造方法仅仅是设置了一些参数而已。而整个table的初始化是在向ConcurrentHashMap中插入元素的时候发生的。如调用put、computeIfAbsent、compute、merge等方法的时候，调用时机是检查table==null。

初始化方法主要应用了关键属性sizeCtl 如果这个值〈0，表示其他线程正在进行初始化，就放弃这个操作。在这也可以看出ConcurrentHashMap的初始化只能由一个线程完成。如果获得了初始化权限，就用CAS方法将sizeCtl置为-1，防止其他线程进入。初始化数组后，将sizeCtl的值改为0.75*n。

#### 2.5 扩容方法 transfer

当ConcurrentHashMap容量不足的时候，需要对table进行扩容。这个方法的基本思想跟HashMap是很像的，但是由于它是支持并发扩容的，所以要复杂的多。原因是它支持多线程进行扩容操作，而并没有加锁。我想这样做的目的不仅仅是为了满足concurrent的要求，而是希望利用并发处理去减少扩容带来的时间影响。因为在扩容的时候，总是会涉及到从一个“数组”到另一个“数组”拷贝的操作，如果这个操作能够并发进行，那真真是极好的了。

整个扩容操作分为两个部分

-  第一部分是构建一个nextTable,它的容量是原来的两倍，这个操作是单线程完成的。这个单线程的保证是通过RESIZE_STAMP_SHIFT这个常量经过一次运算来保证的，这个地方在后面会有提到；
- 第二个部分就是将原来table中的元素复制到nextTable中，这里允许多线程进行操作。

先来看一下单线程是如何完成的：

它的大体思想就是遍历、复制的过程。首先根据运算得到需要遍历的次数i，然后利用tabAt方法获得i位置的元素：

- 如果这个位置为空，就在原table中的i位置放入forwardNode节点，这个也是触发并发扩容的关键点；

- 如果这个位置是Node节点（fh>=0），如果它是一个链表的头节点，就构造一个反序链表，把他们分别放在nextTable的i和i+n的位置上

- 如果这个位置是TreeBin节点（fh<0），也做一个反序处理，并且判断是否需要untreefi，把处理的结果分别放在nextTable的i和i+n的位置上

- 遍历过所有的节点以后就完成了复制工作，这时让nextTable作为新的table，并且更新sizeCtl为新容量的0.75倍 ，完成扩容。

再看一下多线程是如何完成的：

在代码的69行有一个判断，如果遍历到的节点是forward节点，就向后继续遍历，再加上给节点上锁的机制，就完成了多线程的控制。多线程遍历节点，处理了一个节点，就把对应点的值set为forward，另一个线程看到forward，就向后遍历。这样交叉就完成了复制工作。而且还很好的解决了线程安全的问题。 这个方法的设计实在是让我膜拜。

#### 2.6 Put方法

前面的所有的介绍其实都为这个方法做铺垫。ConcurrentHashMap最常用的就是put和get两个方法。现在来介绍put方法，这个put方法依然沿用HashMap的put方法的思想，根据hash值计算这个新插入的点在table中的位置i，如果i位置是空的，直接放进去，否则进行判断，如果i位置是树节点，按照树的方式插入新的节点，否则把i插入到链表的末尾。ConcurrentHashMap中依然沿用这个思想，有一个最重要的不同点就是ConcurrentHashMap不允许key或value为null值。另外由于涉及到多线程，put方法就要复杂一点。在多线程中可能有以下两个情况

1. 如果一个或多个线程正在对ConcurrentHashMap进行扩容操作，当前线程也要进入扩容的操作中。这个扩容的操作之所以能被检测到，是因为transfer方法中在空结点上插入forward节点，如果检测到需要插入的位置被forward节点占有，就帮助进行扩容；
2. 如果检测到要插入的节点是非空且不是forward节点，就对这个节点加锁，这样就保证了线程安全。尽管这个有一些影响效率，但是还是会比hashTable的synchronized要好得多。

整体流程就是首先定义不允许key或value为null的情况放入  对于每一个放入的值，首先利用spread方法对key的hashcode进行一次hash计算，由此来确定这个值在table中的位置。

如果这个位置是空的，那么直接放入，而且不需要加锁操作。

 如果这个位置存在结点，说明发生了hash碰撞，首先判断这个节点的类型。如果是链表节点（fh>0）,则得到的结点就是hash值相同的节点组成的链表的头节点。需要依次向后遍历确定这个新加入的值所在位置。如果遇到hash值与key值都与新加入节点是一致的情况，则只需要更新value值即可。否则依次向后遍历，直到链表尾插入这个结点。如果加入这个节点以后链表长度大于8，就把这个链表转换成红黑树。如果这个节点的类型已经是树节点的话，直接调用树节点的插入方法进行插入新的值。

`我们可以发现JDK8中的实现也是锁分离的思想，只是锁住的是一个Node，而不是JDK7中的Segment，而锁住Node之前的操作是无锁的并且也是线程安全的，建立在之前提到的3个原子操作上。`

#### 2.7 get方法

get方法比较简单，给定一个key来确定value的时候，必须满足两个条件  key相同  hash值相同，对于节点可能在链表或树上的情况，需要分别去查找。

#### 2.8 Size相关的方法

对于ConcurrentHashMap来说，这个table里到底装了多少东西其实是个不确定的数量，因为不可能在调用size()方法的时候像GC的“stop the world”一样让其他线程都停下来让你去统计，因此只能说这个数量是个估计值。对于这个估计值，ConcurrentHashMap也是大费周章才计算出来的。

mappingCount与Size方法

mappingCount与size方法的类似  从Java工程师给出的注释来看，应该使用mappingCount代替size方法 两个方法都没有直接返回basecount 而是统计一次这个值，而这个值其实也是一个大概的数值，因为可能在统计的时候有其他线程正在执行插入或删除操作。

addCount方法

在put方法结尾处调用了addCount方法，把当前ConcurrentHashMap的元素个数+1这个方法一共做了两件事,更新baseCount的值，检测是否进行扩容。

### 总结 

JDK6,7中的ConcurrentHashmap主要使用Segment来实现减小锁粒度，把HashMap分割成若干个Segment，在put的时候需要锁住Segment，get时候不加锁，使用volatile来保证可见性，当要统计全局时（比如size），首先会尝试多次计算modcount来确定，这几次尝试中，是否有其他线程进行了修改操作，如果没有，则直接返回size。如果有，则需要依次锁住所有的Segment来计算。

jdk7中ConcurrentHashmap中，当长度过长碰撞会很频繁，链表的增改删查操作都会消耗很长的时间，影响性能,所以jdk8 中完全重写了concurrentHashmap,代码量从原来的1000多行变成了 6000多 行，实现上也和原来的分段式存储有很大的区别。

主要设计上的变化有以下几点: 

1. 不采用segment而采用node，锁住node来实现减小锁粒度。
2. 设计了MOVED状态 当resize的中过程中 线程2还在put数据，线程2会帮助resize。
3. 使用3个CAS操作来确保node的一些操作的原子性，这种方式代替了锁。
4. sizeCtl的不同值来代表不同含义，起到了控制的作用。

至于为什么JDK8中使用synchronized而不是ReentrantLock，我猜是因为JDK8中对synchronized有了足够的优化吧。

## HashMap为什么线程不安全

[谈谈HashMap线程不安全的体现](http://www.importnew.com/22011.html)

­ 在多线程场景下，对hashmap进行 resize(rehash) 操作可能会产生环路，这样在执行get方法时就会产生死循环。

### resize死循环

我们都知道HashMap初始容量大小为16,一般来说，当有数据要插入时，都会检查容量有没有超过设定的thredhold，如果超过，需要增大Hash表的尺寸，但是这样一来，整个Hash表里的元素都需要被重算一遍。这叫rehash，这个成本相当的大。

大概看下transfer：

1. 对索引数组中的元素遍历
2. 对链表上的每一个节点遍历：用 next 取得要转移那个元素的下一个，将 e 转移到新 Hash 表的头部，使用头插法插入节点。
3. 循环2，直到链表节点全部转移
4. 循环1，直到所有索引数组全部转移

经过这几步，我们会发现转移的时候是逆序的。假如转移前链表顺序是1->2->3，那么转移后就会变成3->2->1。这时候就有点头绪了，死锁问题不就是因为1->2的同时2->1造成的吗？所以，HashMap 的死锁问题就出在这个`transfer()`函数上。

### 1.1 单线程 rehash 详细演示

单线程情况下，rehash 不会出现任何问题：

- 假设hash算法就是最简单的 key mod table.length（也就是数组的长度）。
- 最上面的是old hash 表，其中的Hash表的 size = 2, 所以 key = 3, 7, 5，在 mod 2以后碰撞发生在 table[1]
- 接下来的三个步骤是 Hash表 resize 到4，并将所有的 `<key,value>` 重新rehash到新 Hash 表的过程

如图所示：

[![1](http://incdn1.b0.upaiyun.com/2016/10/28c8edde3d61a0411511d3b1866f0636.jpg)](http://www.importnew.com/22011.html/1-29)

 

### 1.2 多线程 rehash 详细演示

为了思路更清晰，我们只将关键代码展示出来

1. `Entry<K,V> next = e.next;`——因为是单链表，如果要转移头指针，一定要保存下一个结点，不然转移后链表就丢了
2. `e.next = newTable[i];`——e 要插入到链表的头部，所以要先用 e.next 指向新的 Hash 表第一个元素（为什么不加到新链表最后？因为复杂度是 O（N））
3. `newTable[i] = e;`——现在新 Hash 表的头指针仍然指向 e 没转移前的第一个元素，所以需要将新 Hash 表的头指针指向 e
4. `e = next`——转移 e 的下一个结点

假设这里有两个线程同时执行了`put()`操作，并进入了`transfer()`环节

那么现在的状态为：

[![2](http://incdn1.b0.upaiyun.com/2016/10/665f644e43731ff9db3d341da5c827e1.jpg)](http://www.importnew.com/22011.html/2-24)

 

从上面的图我们可以看到，因为线程1的 e 指向了 key(3)，而 next 指向了 key(7)，在线程2 rehash 后，就指向了线程2 rehash 后的链表。

然后线程1被唤醒了：

1. 执行`e.next = newTable[i]`，于是 key(3)的 next 指向了线程1的新 Hash 表，因为新 Hash 表为空，所以`e.next = null`，
2. 执行`newTable[i] = e`，所以线程1的新 Hash 表第一个元素指向了线程2新 Hash 表的 key(3)。好了，e 处理完毕。
3. 执行`e = next`，将 e 指向 next，所以新的 e 是 key(7)

然后该执行 key(3)的 next 节点 key(7)了:

1. 现在的 e 节点是 key(7)，首先执行`Entry<K,V> next = e.next`,那么 next 就是 key(3)了
2. 执行`e.next = newTable[i]`，于是key(7) 的 next 就成了 key(3)
3. 执行`newTable[i] = e`，那么线程1的新 Hash 表第一个元素变成了 key(7)
4. 执行`e = next`，将 e 指向 next，所以新的 e 是 key(3)

这时候的状态图为：

[![3](http://incdn1.b0.upaiyun.com/2016/10/38026ed22fc1a91d92b5d2ef93540f20.jpg)](http://www.importnew.com/22011.html/3-19)

 

然后又该执行 key(7)的 next 节点 key(3)了：

1. 现在的 e 节点是 key(3)，首先执行`Entry<K,V> next = e.next`,那么 next 就是 null
2. 执行`e.next = newTable[i]`，于是key(3) 的 next 就成了 key(7)
3. 执行`newTable[i] = e`，那么线程1的新 Hash 表第一个元素变成了 key(3)
4. 执行`e = next`，将 e 指向 next，所以新的 e 是 key(7)

这时候的状态如图所示：

[![4](http://incdn1.b0.upaiyun.com/2016/10/011ecee7d295c066ae68d4396215c3d0.jpg)](http://www.importnew.com/22011.html/4-18)

 

很明显，环形链表出现了！！当然，现在还没有事情，因为下一个节点是 null，所以`transfer()`就完成了，等`put()`的其余过程搞定后，HashMap 的底层实现就是线程1的新 Hash 表了。也就是说，在多线程场景下，对hashmap进行rehash操作可能会产生环路，这样在执行get方法时就会产生死循环，导致CPU 100%。

## 重写equels方法时为什么一定要重写hashcode方法

原因就在于需要考虑HashMap，Hashtable，ConcurrentHashMap等散列表的使用。

如果只重写了equels方法而没有重写hashcode方法，意味着两个equels判断为相同的key的hashcode是不同的，那么它们在散列表中的位置也是不同的，这就意味着可以往map中插入相同的key这是不允许的。

## 为何HashMap、ConcurrentHashMap的数组长度一定是2的次幂？

为什么HashMap和ConcurrentHashMap明明有有参的构造方法，为什么无论我们初始化map大小为多少，底层的实现数组的长度都会调整为大于初始化长度的最小的2的次幂的值？



**原因在于在计算机中，位与计算比求模计算效率高。在hashmap的底层实现中，是以key的hashcode 与数组长度-1 位与计算来 代替 key的hashcode 对数组长度的求模计算的**。数组长度设为n如果不是2的次幂，那么n-1的二进制表示就会出现0，这样位与计算这一位必然为0。这样的话就代表有的位置是不会插入值的，也就是元素在数组中的分布不均匀，有的位置多，有的位置没有。



此外，key的hashcode 与数组长度-1 位与计算来 代替 key的hashcode 对数组长度的求模计算并且将数组长度设为2的次幂还有一个好处。在扩容操作后，新的长度是老的长度的两倍，假如原来长度是16，16的二进制表示为 10000，那么length-1就是15，二进制为01111，同理扩容后的数组长度为32，二进制表示为100000，length-1为31，二进制表示为011111。,我们看到这样会保证低位全为1，而扩容后只有一位差异，也就是多出了最左位的1，这样在通过 h&(length-1)的时候，只要h对应的最左边的那一个差异位为0，就能保证得到的新的数组索引和老数组索引一致。**也就是说，在扩容后和扩容前很多元素的位置是一样的**。

ConcurrentHashMap的jdk7/8的实现中，rehash扩容操作都已经对这个特性做了优化