# HashMap详解

  

![HashMap](/images/container/HashMap.png)

## 实现步骤
1. HashMap基于哈希散列表，数组+链表/红黑树实现。
2. 通过key的hashCode()方法计算出hashCode。
3. 通过HashMap类中内部hash()方法将第2步中hashCode带入得出hash值。
4. 通过第3步中hash值和HashMap中数组长度做&(位运算)得出在数组中的位置。
5. 当第4步中位置中没有值则直接放入。
6. 当第4步中位置中有值即产生hash冲突问题，此时通过链表(拉链法)来解决hash冲突问题。
7. 如果第6步中第链表大小超过阈值（TREEIFY_THRESHOLD,8），链表转换为红黑树。
8. 在转换为红黑树时，会判断数组长度大于64才转换，否则继续采用扩容策略而不转换。

## 关键特性
1. 默认初始容量值为16，负载因子为0.75，当size>=threshold（ threshold等于“容量*负载因子”）时，会发生扩容：newsize = oldsize*2，size一定为2的n次幂
2. hash冲突默认采用单链表存储，当单链表节点个数大于8时且数组长度大于64，会转化为红黑树存储，
3. 当红黑树中节点少于6时，则转化为单链表存储。
4. 扩容针对整个Map，每次扩容时，原来数组中的元素依次重新计算存放位置，并重新插入
5. 当Map中元素总数超过Entry数组的75%，触发扩容操作，为了减少链表长度，元素分配更均匀

## HashMap在1.7和1.8之间的变化：
1. 1.7中是先扩容后插入新值的，1.8中是先插值再扩容
2. 1.7中采用数组+链表，1.8采用的是数组+链表/红黑树，即在1.7中链表长度超过一定长度后就改成红黑树存储。
3. 1.7扩容时需要重新计算哈希值和索引位置，1.8并不重新计算哈希值，巧妙地采用和扩容后容量进行&操作来计算新的索引位置。
4. 1.7是采用表头插入法插入链表，1.8采用的是尾部插入法。
5. 在1.7中采用表头插入法，在扩容时会改变链表中元素原本的顺序，以至于在并发场景下导致链表成环的问题；在1.8中采用尾部插入法，在扩容时会保持链表元素原本的顺序，就不会出现链表成环的问题了。


## 方法（JDK1.8-数组+链表/红黑树）
#### 确定哈希桶数组索引位置
![HashMpaCalPosition](/images/container/HashMpaCalPosition.png)
**第1步计算hash**

在JDK1.8的实现中，优化了高位运算的算法，通过hashCode()的高16位异或低16位实现的：(h = k.hashCode()) ^ (h >>> 16)，主要是从速度、功效、质量来考虑的。
目的都是在数组很小也能降低hash碰撞。
```java
static final int hash(Object key) {
    int h;
    // key.hashCode()：返回散列值也就是hashcode
    // ^ ：按位异或
    // >>>:无符号右移，忽略符号位，空位都以0补齐
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

**第2步计算数组位置**

通过(n - 1) & hash来得到该对象的保存位，而HashMap底层数组的长度总是2的n次方。
当**length总是2的n次方**时，h& (length-1)运算等价于对length取模，也就是h%length，但是&(位运算)比%(取模运算)具有更高的效率。

```java
(n - 1) & hash
```

#### HashMap的put方法
![HashMpaPut](/images/container/HashMpaPut.png)

```java
    public V put(K key, V value) {
        // 对key的hashCode()做hash
        return putVal(hash(key), key, value, false, true);
    }

    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                    boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // 步骤①：tab为空则创建
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        // 步骤②：计算index，并对null做处理 
        if ((p = tab[i = (n - 1) & hash]) == null) 
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            // 步骤③：节点key存在，直接覆盖value
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            // 步骤④：判断该链为红黑树
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            // 步骤⑤：该链为链表
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key,value,null);
                        //链表长度大于8转换为红黑树进行处理
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st  
                            treeifyBin(tab, hash);
                        break;
                    }
                    // key已经存在直接覆盖value
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))                                       break;
                    p = e;
                }
            }         
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }

        ++modCount;
        // 步骤⑥：超过最大容量 就扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }

    final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        //树形化还有一个要求就是数组长度必须大于等于64，否则继续采用扩容策略
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;//hd指向首节点，tl指向尾节点
            do {
                TreeNode<K,V> p = replacementTreeNode(e, null);//将链表节点转化为红黑树节点
                if (tl == null) // 如果尾节点为空，说明还没有首节点
                    hd = p;  // 当前节点作为首节点
                else { // 尾节点不为空，构造一个双向链表结构，将当前节点追加到双向链表的末尾
                    p.prev = tl; // 当前树节点的前一个节点指向尾节点
                    tl.next = p; // 尾节点的后一个节点指向当前节点
                }
                tl = p; // 把当前节点设为尾节点
            } while ((e = e.next) != null); // 继续遍历单链表
            //将原本的单链表转化为一个节点类型为TreeNode的双向链表
            if ((tab[index] = hd) != null) // 把转换后的双向链表，替换数组原来位置上的单向链表
                hd.treeify(tab); // 将当前双向链表树形化
        }
    }

    //将双向链表转化为红黑树的实现
    final void treeify(Node<K,V>[] tab) {
        TreeNode<K,V> root = null;  // 定义红黑树的根节点
        for (TreeNode<K,V> x = this, next; x != null; x = next) { // 从TreeNode双向链表的头节点开始逐个遍历
            next = (TreeNode<K,V>)x.next; // 头节点的后继节点
            x.left = x.right = null;
            if (root == null) {
                x.parent = null;
                x.red = false;
                root = x; // 头节点作为红黑树的根，设置为黑色
        }
        else { // 红黑树存在根节点
            K k = x.key; 
            int h = x.hash;
            Class<?> kc = null;
            for (TreeNode<K,V> p = root;;) { // 从根开始遍历整个红黑树
                int dir, ph;
                K pk = p.key;
                if ((ph = p.hash) > h) // 当前红黑树节点p的hash值大于双向链表节点x的哈希值
                    dir = -1;
                else if (ph < h) // 当前红黑树节点的hash值小于双向链表节点x的哈希值
                    dir = 1;
                else if ((kc == null &&
                            (kc = comparableClassFor(k)) == null) ||
                            (dir = compareComparables(kc, k, pk)) == 0) // 当前红黑树节点的hash值等于双向链表节点x的哈希值，则如果key值采用比较器一致则比较key值
                    dir = tieBreakOrder(k, pk); //如果key值也一致则比较className和identityHashCode

                TreeNode<K,V> xp = p; 
                if ((p = (dir <= 0) ? p.left : p.right) == null) { // 如果当前红黑树节点p是叶子节点，那么双向链表节点x就找到了插入的位置
                    x.parent = xp;
                    if (dir <= 0) //根据dir的值，插入到p的左孩子或者右孩子
                        xp.left = x;
                    else
                        xp.right = x;
                    root = balanceInsertion(root, x); //红黑树中插入元素，需要进行平衡调整(过程和TreeMap调整逻辑一模一样)
                    break;
                }
            }
        }
    }
    //将TreeNode双向链表转化为红黑树结构之后，由于红黑树是基于根节点进行查找，所以必须将红黑树的根节点作为数组当前位置的元素
    moveRootToFront(tab, root);
    }

    //然后将红黑树的根节点移动端数组的索引所在位置上
    static <K,V> void moveRootToFront(Node<K,V>[] tab, TreeNode<K,V> root) {
        int n;
        if (root != null && tab != null && (n = tab.length) > 0) {
            int index = (n - 1) & root.hash; //找到红黑树根节点在数组中的位置
            TreeNode<K,V> first = (TreeNode<K,V>)tab[index]; //获取当前数组中该位置的元素
            if (root != first) { //红黑树根节点不是数组当前位置的元素
                Node<K,V> rn;
                tab[index] = root;
                TreeNode<K,V> rp = root.prev;
                if ((rn = root.next) != null) //将红黑树根节点前后节点相连
                    ((TreeNode<K,V>)rn).prev = rp;
                if (rp != null)
                    rp.next = rn;
                if (first != null) //将数组当前位置的元素，作为红黑树根节点的后继节点
                    first.prev = root;
                root.next = first;
                root.prev = null;
            }
            assert checkInvariants(root);
        }
    }

#### 扩容resize 方法
进行扩容，会伴随着一次重新 hash 分配，并且会遍历 hash 表中所有的元素，是非常耗时的。在编写程序中，**要尽量避免resize**。

    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            // 超过最大值就不再扩充了，就只好随你碰撞去吧
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            // 没超过最大值，就扩充为原来的2倍
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {
            // signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        // 计算新的resize上限
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            // 把每个bucket都移动到新的buckets中
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else {
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            // 原索引
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            // 原索引+oldCap
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        // 原索引放到bucket里
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        // 原索引+oldCap放到bucket里
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
## 参考文章
1. https://www.hollischuang.com/archives/2091
2. https://yuanrengu.com/2020/ba184259.html
3. https://zhuanlan.zhihu.com/p/21673805
4. https://mp.weixin.qq.com/s?__biz=MzIwNTI2ODY5OA==&mid=2649938471&idx=1&sn=2964df2adc4feaf87c11b4915b9a018e
