---

title: Map
date: 2021-06-17 16:31:24
tags: Map
categories: 集合
---

##### 一、HashMap

基于数组+链表+红黑树构成（jdk1.8），非线程安全，支持序列化，可以被克隆，允许 key 和 value 都为 null，key为null的键值对永远都放在以table[0]为头结点的链表中

```java
//HashMap 继承于AbstractMap ，实现Cloneable, Serializable
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable
```

<!-- more -->

存储数组

~~~java
//HashMap主干数组，Entry是HashMap组成单位，初始值是空数组
transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;
~~~

其他重要值

~~~java
/**实际存储的key-value键值对的个数*/
transient int size;

/**阈值，当table == {}时，该值为初始容量（初始容量默认为16）；当table被填充了，也就是为table分配内存空间后，
threshold一般为 capacity*loadFactory。HashMap在进行扩容时需要参考threshold，后面会详细谈到*/
int threshold;

/**负载因子，代表了table的填充度有多少，默认是0.75
加载因子存在的原因，还是因为减缓哈希冲突，如果初始桶为16，等到满16个元素才扩容，某些桶里可能就有不止一个元素了。
所以加载因子默认为0.75，也就是说大小为16的HashMap，到了第13个元素，就会扩容成32。
*/
final float loadFactor;

/**HashMap被改变的次数，由于HashMap非线程安全，在对HashMap进行迭代时，
如果期间其他线程的参与导致HashMap的结构发生变化了（比如put，remove等操作），
需要抛出异常ConcurrentModificationException*/
transient int modCount;
~~~

构造方法

~~~java
/** HashMap有4个构造器，其他构造器如果用户没有传入initialCapacity 和loadFactor这两个参数，会使用默认值
initialCapacity默认为16，loadFactory默认为0.75 */
public HashMap(int initialCapacity, float loadFactor) {
    //此处对传入的初始容量进行校验，最大不能超过MAXIMUM_CAPACITY = 1<<30(2的30次方)
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    /** 进行数组填充（为table分配实际内存空间），入参为threshold，此时threshold为initialCapacity 默认是1<<4(2的4次方=16) */
    this.threshold = tableSizeFor(initialCapacity);
}
~~~

静态内部类 Entry ，存储 key-value 值

~~~java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;//哈希值
    final K key;
    V value;
    Node<K,V> next;//对象引用

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
~~~

Put 操作

![img](https://img-blog.csdn.net/20161021144506448?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

~~~java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    /** 初始化或数组的大小。如果为空，按照字段阈值中持有的初始容量目标分配。否则，因为我们使用的是2的幂展开，所以每个bin中的元素要么保持在相同的索引位置，要么在新表中以2的幂移动 */
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
	//计算index，并对null做处理
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        // 节点key存在，直接覆盖value
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        //判断该链为红黑树
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        //该链为链表
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                     //链表长度大于8转换为红黑树进行处理
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
         //如果该对应数据已存在，执行覆盖操作。用新value替换旧value，并返回旧value
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    //超过最大容量，就扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
~~~

Hash算法

```java
//key的hashcode 与 hashcode右移16位 做异或(^)操作，得出hash值
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

Get 操作

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

/**
 * Implements Map.get and related methods
 *
 * @param hash hash for key
 * @param key the key
 * @return the node, or null if none
 */
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    //计算index值
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        //永远会判断第一个节点的hash值和value是否匹配
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        //判断下一节点是否为空
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

扩容

```java
final Node<K,V>[] resize() {
    // 当前table保存
    Node<K,V>[] oldTab = table;
    // 保存table大小
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    // 保存当前阈值 
    int oldThr = threshold;
    int newCap, newThr = 0;
    // 之前table大小大于0
    if (oldCap > 0) {
        // 之前table大于最大容量
        if (oldCap >= MAXIMUM_CAPACITY) {
            // 阈值为最大整形
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 容量翻倍，使用左移，效率更高
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
            oldCap >= DEFAULT_INITIAL_CAPACITY)
            // 阈值翻倍
            newThr = oldThr << 1; // double threshold
    }
    // 之前阈值大于0
    else if (oldThr > 0)
        newCap = oldThr;
    // oldCap = 0并且oldThr = 0，使用缺省值（如使用HashMap()构造函数，之后再插入一个元素会调用resize函数，会进入这一步）
    else {           
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 新阈值为0
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    // 初始化table
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    // 之前的table已经初始化过
    if (oldTab != null) {
        // 复制元素，重新进行hash
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    // 将同一桶中的元素根据(e.hash & oldCap)是否为0进行分割，分成两个不同的链表，完成rehash
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

红黑树处理

```java
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        TreeNode<K,V> hd = null, tl = null;
        do {
            TreeNode<K,V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
```

##### 二、HashMap 与 HashTable 区别

**相同点：**hashmap 和 Hashtable 都实现了 map、Cloneable（可克隆）、Serializable（可序列化）这三个接口

**不同点：**1. 底层数据结构不同:jdk1.7底层都是数组+链表,但jdk1.8 HashMap加入了红黑树

				2. Hashtable 是不允许键或值为 null 的，HashMap 的键值则都可以为 null
   				3. 添加key-value的hash值算法不同：HashMap添加元素时，是使用自定义的哈希算法，而HashTable是直接采用key的hashCode()
                        				4. 实现方式不同：Hashtable 继承的是 Dictionary类，而 HashMap 继承的是 AbstractMap 类
                  				5. 初始化容量不同：HashMap 的初始容量为：16，Hashtable 初始容量为：11，两者的负载因子默认都是：0.75
                                				6. 扩容机制不同：当已用容量>总容量 * 负载因子时，HashMap 扩容规则为当前容量翻倍，Hashtable 扩容规则为当前容量翻倍 +1
                              				7. 支持的遍历种类不同：HashMap只支持Iterator遍历,而HashTable支持Iterator和Enumeration两种方式遍历
                                           				8. 部分API不同：HashMap不支持contains(Object value)方法，没有重写toString()方法,而HashTable支持contains(Object value)方法，而且重写了toString()方法
                                          				9. 同步性不同: Hashtable是同步(synchronized)的，适用于多线程环境，而hashmap不是同步的，适用于单线程环境。多个线程可以共享一个Hashtable；而如果没有正确的同步的话，多个线程是不能共享HashMap的

##### 三、HashMap 和 TreeMap 的区别

1. HashMap是通过hashcode()对其内容进行快速查找的，HashMap中的元素是没有顺序的；

   TreeMap中所有的元素都是有某一固定顺序的，如果需要得到一个有序的结果，就应该使用TreeMap

2. HashMap和TreeMap都不是线程安全的；

3. HashMap继承AbstractMap类；覆盖了hashcode() 和equals() 方法，以确保两个相等的映射返回相同的哈希值；

   TreeMap继承SortedMap类；他保持键的有序顺序；

4. HashMap：适用于Map插入，删除，定位元素；

   TreeMap：适用于按自然顺序或自定义顺序遍历键（key）；

##### 四、ConcurrentHashMap

###### 为什么要使用ConcurrentHashMap？

首先我们先讨论一下为什么要使用ConcureentHashMap，为了线程安全？确实是为了线程安全，但不知这个原因，因为HashMap在多线程下不安全，但是HashTable是线程安全的，完全可以使用HashTable，那为什么不呢？这是因为HashTable相比HashMap在所有方法上都添加了synchronized关键字来修饰，即：HashTable是对整个table数组进行了锁定，即在一个线程访问HashTable的同步方法时，另外一个线程访问HashTable的同步方法时便会进入阻塞或轮询状态。这就造成了HashTable在多线程竞争激烈的情况下的效率低下。例如：线程1访问HashTable的put方法，线程2的任何同步方法都不能使用，put、get等方法都不能使用。

而ConcurrentHashMap解决了这两个问题。既然给整个table表加一把锁效率低下，那么把一把锁分为好多把锁，把table数组分段，让每一把锁负责一部分数据，这样如果是访问不同锁的时候就不会产生竞争，提高了效率。这就ConcurrentHashMap的锁分段技术的思想。在jdk1.7中就是采用锁分段技术，ConcurrentHashMap由多个Segment组(Segment下包含很多Node，也就是我们的键值对了)，每个Segment都有把锁来实现线程安全，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。

###### JDK1.7 与 JDK1.8 ConcurrentHashMap 实现区别

1. 在1.8中ConcurrentHashMap仍然保留了segment，源码中注释中写道只是为了兼容以前版本的序列化而申明的类
2. 在1.8中ConcurrentHashMap由数组(Node)+链表+红黑树实现，与HashMap不同的是红黑树对象不是TreeNode，而是用TreeBin进行了封装。而在1.7中，ConcurrentHashMap采用锁分段技术，数组(Segment)+链表的数据结构。下图为1.7的ConcurrentHashMap的实现结构图

![img](https://img-blog.csdnimg.cn/2019042720043475.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0hvbGxha2U=,size_16,color_FFFFFF,t_70)

3. 1.7的分段锁Segment继承于ReentrantLock，所以带有锁功能，保证线程安全

	1.8中采用数组+链表+红黑树的数据结构来实现，这和1.8中HashMap的实现很相似，不同之处就是ConcurrentHashMap采用synchronized保证了线程安全以及使用了CAS来保证原子性操作。同样是对put举例，1.8中一次hash定位出数组的索引值table[i]，接着使用synchronized来对table[i]进行锁定，保证线程安全，接着判断这个位置是否有元素，如果有，证明这个元素key以及存在，替换并返回旧值，如果为null，直接插入到table[i]这个位置，如果不为null，判断是链表或者红黑树，然后进行插入，这个过程基本和HashMap一致，不过这里的查找插入很多都是采用CAS的原子性操作
###### 源码

**Node**

​	Node就是一个链表，可以指向下一个值，只允许查，不允许setValue()，实现了Map的静态内部接口Entry

~~~java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        volatile V val;
        volatile Node<K,V> next;

        Node(int hash, K key, V val, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.val = val;
            this.next = next;
        }

        public final K getKey()       { return key; }
        public final V getValue()     { return val; }
        public final int hashCode()   { return key.hashCode() ^ val.hashCode(); }
        public final String toString(){ return key + "=" + val; }
        public final V setValue(V value) {
            throw new UnsupportedOperationException();
        }

        public final boolean equals(Object o) {
            Object k, v, u; Map.Entry<?,?> e;
            return ((o instanceof Map.Entry) &&
                    (k = (e = (Map.Entry<?,?>)o).getKey()) != null &&
                    (v = e.getValue()) != null &&
                    (k == key || k.equals(key)) &&
                    (v == (u = val) || v.equals(u)));
        }

        /**
         * Virtualized support for map.get(); overridden in subclasses.
         */
        Node<K,V> find(int h, Object k) {
            Node<K,V> e = this;
            if (k != null) {
                do {
                    K ek;
                    if (e.hash == h &&
                        ((ek = e.key) == k || (ek != null && k.equals(ek))))
                        return e;
                } while ((e = e.next) != null);
            }
            return null;
        }
    }
~~~

**TreeNode**

​	TreeNode继承于Node，这是个红黑树的数据结构，在大于8时，Node链表便会转换为红黑树

```java
static final class TreeNode<K,V> extends Node<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;

    TreeNode(int hash, K key, V val, Node<K,V> next,
             TreeNode<K,V> parent) {
        super(hash, key, val, next);
        this.parent = parent;
    }

    Node<K,V> find(int h, Object k) {
        return findTreeNode(h, k, null);
    }

    /**
     * Returns the TreeNode (or null if not found) for the given key
     * starting at given root.
     */
    final TreeNode<K,V> findTreeNode(int h, Object k, Class<?> kc) {
        if (k != null) {
            TreeNode<K,V> p = this;
            do  {
                int ph, dir; K pk; TreeNode<K,V> q;
                TreeNode<K,V> pl = p.left, pr = p.right;
                if ((ph = p.hash) > h)
                    p = pl;
                else if (ph < h)
                    p = pr;
                else if ((pk = p.key) == k || (pk != null && k.equals(pk)))
                    return p;
                else if (pl == null)
                    p = pr;
                else if (pr == null)
                    p = pl;
                else if ((kc != null ||
                          (kc = comparableClassFor(k)) != null) &&
                         (dir = compareComparables(kc, k, pk)) != 0)
                    p = (dir < 0) ? pl : pr;
                else if ((q = pr.findTreeNode(h, k, kc)) != null)
                    return q;
                else
                    p = pl;
            } while (p != null);
        }
        return null;
    }
}
```

**TreeBin**

​	TreeBin是将TreeNode红黑树进行了封装，增加了一些其他方法

**Put操作**

```java
public V put(K key, V value) {
    return putVal(key, value, false);
}

/** Implementation for put and putIfAbsent */
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    //计算hash值
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        //判断是在正在扩容
        else if ((fh = f.hash) == MOVED)
            //帮助扩容
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            //锁定头节点
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        binCount = 1;
                        //遍历链表
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            //比较key是否相等
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            //没有匹配到key，则在最后节点新增
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```

**initTable**

```java
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    // 当table为null时才进行初始化
    while ((tab = table) == null || tab.length == 0) {
        // 当sizeCtl小于0的时候表明正在初始化，sizeCtl初始默认状态为0
        if ((sc = sizeCtl) < 0)
            //线程从运行状态转到可运行状态进行等待
            Thread.yield(); // lost initialization race; just spin
        //否则将sizeCTL采用CAS操作设置为-1，表明正在初始化
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

**保证操作的原子性方法**

```java
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
    return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
}

static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                    Node<K,V> c, Node<K,V> v) {
    return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
}

static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
    U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);
}
```