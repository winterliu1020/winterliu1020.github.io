### 1. 首先看几个问题
- 什么时候会使用HashMap?它有什么特点？
- HashMap的工作原理是什么？
- HashMap中put和get的原理，equals()和hashCode()函数都有什么作用？
- hash如何实现？为什么要这样实现？
- 当HashMap的大小超过了负载因子(load factor)定义的容量，怎么办？

当执行下面的操作时：
```
HashMap<String,Integer> map = new HashMap<String,Integer>();
map.put("语文",1);
map.put("数学",2);
map.put("英语",3);
map.put("历史",4);
map.put("政治",5);
map.put("地理",6);
map.put("生物",7);
map.put("化学",8);
for(Entry<String, Integer> entry : map.entrySet()){
    System.out.println(entry.getKet() + ":" + entry.getValue());
}
```
执行结果：
> 政治: 5  
生物: 7  
历史: 4  
数学: 2  
化学: 8  
语文: 1  
英语: 3  
地理: 6  

为什么不是按照循序打印出来呢？看下面这张图来对HashMap结构有一个感性认识：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/HashMap/HashMap.png)

你可以看到这些entry在内存中既不是连续的、也不是按照put的循序排列的。

官方文档描述HashMap:
> Hash table based **implementation of the Map interface** . This implementation provides all of the optional map operations, and permits null values and the null key. (The HashMap class is roughly equivalent to Hashtable, except that it is **unsynchronized and permits nulls**.) This class ++makes no guarantees as to the order of the map++; in particular, it does ++not guarantee that the order will remain constant over time++.

注意点：基于Map接口实现、允许null键/值、非同步、不保证有序(比如插入的循序)、不保证序列不随时间变化。

### 2. 两个重要参数
HashMap中需要注意两个重要的参数是：容量(Capacity)和负载因子(Load factor)  
> - Initial capacity The capacity is the number of buckets in the hash table, The initial capacity is simply the capacity at the time the hash table is created.  
> - Load factor The load factor is a measure of how full the hash table is allowed to get before its capacity is automatically increased. 

Capacity其实就是buckets的数目，Load factor就是buckets填满程度的最大比例。当HashMap中元素的个数大于capacity*load factor时就需要调整buckets的数目为当前的2倍。  

### 3. put函数的实现
put函数的大致思路是：
1. 利用hashCode()函数得到key的hash值，然后再计算这个hash值的index；
2. 如果没有碰撞就直接放在bucket中；
3. 如果碰撞了就以链表的形式放在buckets的后面
4. 如果碰撞导致链表过长(大于等于TREEIFY_THRESHOLD),就把链表转换成红黑树；
5. 如果节点已经存在就替换old value(保证key的唯一性)
6. 如果bucket满了，就resize.  

具体代码如下：
```
public V put(K key, V value){
    // 对key的hashCode()做hash
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict){
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // tab为空则创建
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 计算index，并对null做处理
    if ((p = tab[i = (n -1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else{
        Node<K,V> e; K k;
        // 节点存在
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 该链为树
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 该链为链表
        else {
            for (int binCount = 0; ; ++binCount){
                if ((e = p.next) == null){
                    p.next = newNode(hash, key, value, null);
                    if(binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash); // 当链表长度超过了一定值时将链表转成红黑树
                    break;
                }
                if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                    break; // 说明节点已存在
                p = e;
            }
        }
        // 写入
        if (e != null){ // 也就是说已经存在匹配这个key的值
        V oldValue = e.value;
        if (!onlyIfAbsent || oldValue == null)
            e.value = value;
        afterNodeAccess(e);
        return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```
### 4. get函数的实现
1. bucket里面第一个节点，直接命中
2. 如果有冲突，则通过key.equals(k)去查找对应的entry，如果为树，则在树中通过key.equals(k)查找，O(logn);如果是链表，则在链表中通过key.equals(k)查找，O(n).
```
public V get(Object key){
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key){
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 && (first = tab[(n - 1) & hash]) != null){
        // 直接命中
        if (first.hash == hash && ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 未命中
        if ((e = first.next) != null){
            // 在树中get
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 在链表中get
            do {
                if(e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```
### 5. hash函数的实现
在get和put过程中，计算下标时，首先对hashCode进行hash操作获取到hash值，然后通过hash值进一步计算下标。  

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/HashMap/hash-function.png)
```
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```
计算hash值其实就是：高16bit不变，底16bit和高16bit做了一个异或。  
在设计hash函数时，因为table长度n为2的幂，而计算下标时，是使用&位操作(按位与操作)，而非%求余。
> (n-1) & hash

设计者认为这方法很容易发生碰撞，比如在n-1为15(0x1111)，其实散列真正生效的只是低4bit，因为除了低4bit其它的位都是0，通过按位与操作之后也都是0，所以当然容易碰撞了。  

因此，设计者想了一个顾全大局的方法(综合考虑了速度、作用、质量)，就是把高16bit和低16位异或了一下。设计者还解释到因为现在大多数的hashCode的分布已经很不错了，就算发生碰撞也用O(logn)的tree去做了。仅仅异或一下，既减少了系统的开销，也不会造成因为高位没有参与下标的计算(table长度比较小时)而引起的碰撞。  

如果还是产生了频繁的碰撞会发生什么问题？设计者注释说，他们使用树来处理频繁的碰撞。  
> Improve the performance of java.util.HashMap under high hash-collision conditions by using balanced trees rather than linked lists to store map entries. Implement the same improvement in the LinkedHashMap class.

回想一下get函数过程：
1. 首先根据key.hashCode()值通过hash()函数获取到hash值，然后利用hash值确定bucket的index；
2. 如果bucket的节点的key不是我们需要的，则通过keys.equals()在链中找。  

在Java 8之前是用链表来解决冲突的，所以产生冲突时，get的时间复杂度就是O(1)+O(n),所以当碰撞很厉害的时候n很大，时间复杂度比较高。  
  
所以在Java 8中，利用红黑树替换了链表，这样使得复杂度变成了O(1)+O(logn),降低了时间复杂度。  

### 6. resize的实现
在put时，如果发现目前的bucket占用程序以及超过了Load Factor所希望的比例，那么就会发生resize，resize的过程就是把bucket扩充为2倍，扩充之后需要重新计算每个节点的index，然后把节点再放到新的bucket中。resize的注释的描述：
> Initializes or doubles table size. If null, allocates in accord with initial capacity target held in field threshold. Otherwise, because we are using power-of-two expansion, the elements from each bin must either stay at same index, or move with a power of two offset in the new table.  

就是说，当超过限制的时候会resize，然而又因为我们使用的是2次幂的扩展，所以元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/HashMap/hash-resize.png)  

resize函数代码实现：

```
final Node<K,V>[] resize(){
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0){
        // 超过最大值就不再扩充，就让它碰撞
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 没超过最大值，就扩充为原来的2倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1;
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else { // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 计算新的resize上限
    if (newThr == 0){
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUN_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null){
        // 把每个bucket都移动到新的buckets中
        for (int j = 0; j < oldCap; ++j){
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>e).split(this, newTab, j, oldCap);
                else{ // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 原索引
                        if ((e.hash & oldCap) == 0){
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 原索引+oldCap
                        else{
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    }while((e = next) != null);
                    // 原索引放到bucket里
                    if (loTail != null){
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 原索引+oldCap放到bucket里
                    if (hiTail != null){
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
### 7. 总结
回到开始的几个问题： 

**1.什么时候会使用HashMap?它有什么特点？**   

HashMap是基于Map接口实现的，它存储键值对，允许存储null的键值，它是非同步的，HashMap存储着Entry(hash, key, value, next)对象。

**2.HashMap的工作原理？** 

通过hash的方法，通过put和get存储和获取对象。存储对象时，将K/V传给put方法，它调用hashCode计算hash从而得到bucket位置，HashMap还会根据当前bucket的占用情况自动调整容量。获取对象时，将K传给get,它调用hashCode计算hash从而得到bucket位置，并进一步调用equals()方法确定键值对。如果发生碰撞，会通过链表的方式将碰撞冲突的元素组织起来，Java 8中，如果碰撞冲突的元素超过某个限制，则用红黑树来替换链表，提高速度。

**3.get和put的原理？equals()和hashCode()都有什么作用？**  

通过对key的hashCode()进行hashing,并计算下标(n-1 & hash),从而获得buckets的位置。如果产生碰撞，则利用key.equals()方法去链表或树中查找对应的节点。

**4.hash怎么实现？为什么这样实现？**  

Java 8是通过hashCode()的高16bit异或低16bit实现的: ==(h = k.hashCode() ^ (h >>> 16)==,主要是从速度、功效、质量来考虑，这么做可以在bucket的n比较小的时候，也能保证考虑到高低bit都参与到hash的计算中，同时不会有太大开销。

**5.当HashMap大小超过负载因子定义的容量，怎么办？**  

如果超过了负载因子(默认0.75)，则会重新resize一个原来长度两倍的HashMap,并且重新调用hash方法。  

来自[江南白衣](http://calvin1978.blogcn.com/articles/collection.html)：
> iterator()时是顺着哈希桶数组来遍历的，看起来是个乱序。  
  
本文来自：[Java-HashMap工作原理及实现](https://yikun.github.io/2015/04/01/Java-HashMap%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/)
