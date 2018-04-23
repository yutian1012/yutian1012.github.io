---
title: java中HashMap源码分析（二）
tags: [java]
---

1）hashmap的初始化

```
public HashMap(int initialCapacity, float loadFactor) {
    //如果设置的初始集合容量值小于0，则抛出异常
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    //如果设置的容量大小大于最大容量值，则赋值为最大值
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    //判断装填因子是否设置合法，不合法则抛出异常
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;

    this.threshold = tableSizeFor(initialCapacity);
}

//返回最小的大于等于capacity的值，且值为2的整数次幂
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    //无符号右移，右移1位相当于除以2，
    //移位操作后使得n的二进制表示中与最高位的1紧邻的右边一位也为1
    n |= n >>> 1;
    //右移2位相当于除以2的2次幂，即除以4
    //移位操作后使得n的二进制表示中与最高位的1紧邻的右边三位也为1
    //即原来为1的位，后面紧邻的3为也为1，即有四个为1的数值，其他为不用关心
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    //n总共移动了32次，能够保证如果最高为是1，那么执行完成后
    //即原理为1的为，后面的所有值都变成1了，下面执行加1操作后，相当于找到了大于指定capacity的最大的2的整数幂次（相当于ceil操作）
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

a.理解容量和装填因子

initialCapacity指集合的容量，默认为16。loadFactor表示装填因子，默认为0.75f，即当数据量达到该值时需要对集合进行扩容操作。容量是哈希表中桶(Entry数组)的数量，初始容量只是哈希表在创建时的容量。加载因子是哈希表在其容量自动增加之前可以达到多满的一种尺度。当哈希表中的条目数超出了加载因子与当前容量的乘积时，通过调用resiez方法将容量翻倍。

HashMap默认初始容量16，加载因子0.75，也就是说最多能放16*0.75=12个元素，当put第13个时，HashMap将发生resize，resize的一系列处理比较影响性能，所以当我们需要向HashMap存放较多元素时，最好指定合适的初始容量和加载因子，否则HashMap默认只能存12个元素，将会发生多次rehash操作。

b.正确理解位操作tableSizeFor方法

该方法执行完成后返回table数组的大小，即集合的容量。容量值为最小的大于等于capacity的值，且值为2的整数次幂。

c.问题：实际数组何时初始化的呢？

hashMap的构造函数中并没有一开始就初始化table哈希表，而是等到put的时候进行的。当执行put操作时，判断table是否为null，如果为null则调用resize方法，在resize方法中兼容了集合的初始化操作。

2）hashMap的put方法

```
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

//onlyIfAbsent如果为true，表示只有元素不存在时才放入，无法实现更新
//返回值为原对象，或null
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    //Node继承了Map.Entry<K,V>对象
    //n表示数组的长度，i指向插入的数组下标位置
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    //table为Node类型的数组，即hash表，存放数据的数组结构
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    //判断插入位置处是否存放了数据，即是否发生了碰撞，没有发生碰撞则直接放入
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {//处理碰撞
        //e指向待更新的节点，hash值发生了碰撞并不一定代表key已经存在了，需要进一步使用equals方法判断
        //k指向遍历节点Node的key
        Node<K,V> e; K k;
        //p为链表的头节点，如果链表的头元素的key相等或equals方法为true，则表示头节点即为要更新的节点，将该节点赋值给e，即指向待更新的节点
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        //红黑树节点类型的判断，这里不做分析
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            //变量碰撞链
            for (int binCount = 0; ; ++binCount) {
                //如果找到链尾，也没有找到数据，则将新值连接到链的后面
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                //判断key是否在链表中，如果在则表示找到了待更新的节点，跳出循环
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        //更新节点
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    //执行到这里表示已经添加了元素，则size执行++操作，并判断集合大小是否大于阈值，如果大于阈值，需要对集合进行扩容，进行再hash计算。
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

a.hash方法，

当我们调用put存值时，HashMap首先会调用hash方法，对key进行hash计算（该方法内部会调用key的hashCode方法），获取哈希码。

b.定位数组位置

```
//i = (n - 1) & hash是用来定位位置的
if ((p = tab[i = (n - 1) & hash]) == null)
```

n是2的整数次幂，即为容器的hash表长度，减一操作后保证低位全变为1，与hash进行与操作，即从而相当于hash值对数值长度取余操作（这种方式效率更高）。通过与操作能够获取数组下标。这个下标位置可以被称之为bucketIndex。

c.冲突碰撞的解决与元素的存放

hashMap解决冲突碰撞的方式是通过链表实现的，即将冲突的数据以链表的形式进行链接。当碰撞发生时，这时会取到bucketIndex位置已存储的元素，最终通过调用key的equals来比较，equals方法就是碰撞时才会执行的方法，所以前面说HashMap很少会用到equals。HashMap通过hashCode和equals最终判断出K是否已存在，如果已存在，则使用新V值替换旧V值，并返回旧V值，如果不存在，则存放新的键值对到链表中。

3）get方法获取数据

```
//如果未找到数据则返回null，如果找到节点Node，则返回node的value值
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
//获取节点，传递计算的hash值，以及key对象
final Node<K,V> getNode(int hash, Object key) {
    //tab指向table集合，first指向集合中的bucketIndex位置处第一个节点
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        //判断hash值是否相同，并且key的equals方法也为true时，表示找到了节点
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        //循环遍历链表中的节点，直到找到并返回。
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

注：get方法相对比较简单，没有特别需要说明的。

4）remove方法

```
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}

final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    //tab指向集合                            
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            else if (node == p)
                tab[index] = node.next;
            else
                p.next = node.next;
            ++modCount;
            --size;
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
```

5）迭代器，这里以EntryIterator为例查看

迭代器提供了3种类型：KeyIterator，ValueIterator和EntryIterator

这三种迭代器都需要通过的set集合获取（迭代器被封装到了内部），如调用HashMap的entrySet方法获取EntrySet，然后在调用内部类EntrySet的iterator方法获取迭代器。

```
public Set<Map.Entry<K,V>> entrySet() {
    Set<Map.Entry<K,V>> es;
    return (es = entrySet) == null ? (entrySet = new EntrySet()) : es;
}

//该内部类继承AbstractSet，set集合中存储的对象为Map.Entry，即hashMap装填对象
final class EntrySet extends AbstractSet<Map.Entry<K,V>> {
    public final int size()  { return size; }
    public final void clear() { HashMap.this.clear(); }
    //获取迭代器
    public final Iterator<Map.Entry<K,V>> iterator() {
        return new EntryIterator();
    }
    public final boolean contains(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry<?,?>) o;
        Object key = e.getKey();
        Node<K,V> candidate = getNode(hash(key), key);
        return candidate != null && candidate.equals(e);
    }
    public final boolean remove(Object o) {
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>) o;
            Object key = e.getKey();
            Object value = e.getValue();
            return removeNode(hash(key), key, value, true, true) != null;
        }
        return false;
    }
    public final Spliterator<Map.Entry<K,V>> spliterator() {
        return new EntrySpliterator<>(HashMap.this, 0, -1, 0, 0);
    }
    //遍历集合，通过hashtable获取hash表中的元素，并逐个遍历
    public final void forEach(Consumer<? super Map.Entry<K,V>> action) {
        Node<K,V>[] tab;
        if (action == null)
            throw new NullPointerException();
        if (size > 0 && (tab = table) != null) {
            int mc = modCount;
            for (int i = 0; i < tab.length; ++i) {
                for (Node<K,V> e = tab[i]; e != null; e = e.next)
                    action.accept(e);
            }
            if (modCount != mc)
                throw new ConcurrentModificationException();
        }
    }
}

final class EntryIterator extends HashIterator 
    implements Iterator<Map.Entry<K,V>> {
    //nextNode方法定义在HashIterator中，用来获取下一个节点
    public final Map.Entry<K,V> next() { return nextNode(); }
}
```