### HashMap简介
HashMap继承AbstractMap，实现了Map、Cloneable、Serializable接口。特征：
* HashMap是一个散列表，存储的是键值对，key-value。内部数据结构，Jdk1.7之前是数组+链表，Jdk1.8后是数组+链表或数组+红黑树。
* 不是线程安全的。如果在多线程情况下，用ConcurrentHashMap。
* key、value都可以是null。
* 不是有序的。
* 快速失败。由所有此类的“collection 视图方法”所返回的迭代器都是快速失败的：在迭代器创建之后，如果从结构上对映射进行修改，除非通过迭代器本身的 remove 方法，其他任何时间任何方式的修改，迭代器都将抛出 ConcurrentModificationException。

影响HashMap性能的两个参数：容量、加载因子。
* 容量是哈希表中桶的数量，初始容量只是哈希表在创建时的容量。必须是2^n，为什么必须是2的幂？
* 加载因子是哈希表在其容量自动增加之前可以达到多满的一种尺度。默认是0.75。

当哈希表中的条目数超出了加载因子与当前容量的乘积时，则要对该哈希表进行 rehash 操作（即重建内部数据结构），从而哈希表将具有大约两倍的桶数。在设置初始容量时应该考虑映射中所需的条目数及其加载因子，以便最大限度地减少 rehash 操作次数。
### HashMap的容量如何设置？
HashMap默认的容量是16，加载因子是0.75，存储的键值对数量=16*0.75=12，也就是说可以存储12个key-value。
若想存储16个键值对，HashMap的容量=16 / 0.75=22.

### put操作原理
源码：
````Java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    //判断table是否为空，
    if ((tab = table) == null || (n = tab.length) == 0)
        //创建一个新的table数组，并且获取该数组的长度
        n = (tab = resize()).length;
    //根据键值key计算hash值得到插入的数组索引i，如果table[i]==null，新建节点直接添加   
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {//如果对应的节点存在
        // 若链表已经存在key，则将值赋给e
        Node<K,V> e; K k;
        //判断table[i]的首个元素是否和key一样，如果相同直接覆盖value
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        //判断table[i] 是否为treeNode，即table[i] 是否是红黑树。如果是，则直接在树中操作键值对
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 该链为链表
        else {
            //遍历table[i]，遍历过程中若发现key已经存在直接覆盖value即可；
            for (int binCount = 0; ; ++binCount) {
                // 链表不存在key
                if ((e = p.next) == null) {
                   // 添加到表尾
                    p.next = newNode(hash, key, value, null);
                    // 判断链表长度是否大于TREEIFY_THRESHOLD(默认值为8)，大于8的话把链表转换为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                // 链表已经存在key
                if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        // e不是null，说明链表中有key了
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            // 新值覆盖旧值
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }     
    }
    ++modCount;
    // 插入成功后，判断实际存在的键值对数量size是否超多了最大容量threshold，如果超过，进行扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null; 
}
````
大致思路：
1. 计算key的hash值，
2. 根据hash值计算key在table数组的下标索引，即在哪个桶中。若此桶为null，则将key-value封装成Node，直接放在数组中；否则执行步骤3.
3. 判断key是否为链表的第一个元素。若是，新值覆盖旧值，并返回旧值；若不是，执行步骤4.
4. 判断key所在数组位置是否红黑树数据结构。若是，将key-value放在树中；若不是，执行步骤5.
5. 遍历链表，查找是否已经有key。若有，覆盖旧值，并返回旧值。若没有，添加到链表尾，并判断链表的长度是否已经达到阀值，若达到阀值，将链表转化为红黑树，最后返回null。
6. modCount加1，标记操作了一次。
7. 若不存在key，当key-value添加成功后，size加1，然后size是否大于threshold。若大于，则执行resize扩容。
