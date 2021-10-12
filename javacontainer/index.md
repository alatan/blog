# Java容器概览

  
## 概览
容器主要包括Collection和Map 两种，Collection是存储着对象的集合，而Map存储着键值对（两个对象）的映射表。

![java_container](/images/container/java_container.png)

## Collection
![collection](/images/container/collection.png)

### List
**对付顺序的好帮手： 存储的元素是有序的、可重复的。**
* ArrayList：基于动态数组实现，支持随机访问，适用于频繁的查找工作。
* Vector：和ArrayList类似，但它是线程安全的。
* LinkedList：基于双向链表实现，只能顺序访问，但是可以快速地在链表中间插入和删除元素。不仅如此，LinkedList还可以用作栈、队列和双向队列。

#### Arraylist与 LinkedList 区别?
1. 是否保证线程安全： ArrayList和LinkedList都是不同步的，也就是不保证线程安全；
2. 底层数据结构： Arraylist底层使用的是Object数组；LinkedList底层使用的是双向链表数据结构（JDK1.6 之前为循环链表，JDK1.7 取消了循环。）
3. 插入和删除是否受元素位置的影响：
    * ArrayList 采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。 比如：执行add(E e)方法的时候， ArrayList 会默认在将指定的元素追加到此列表的末尾，这种情况时间复杂度就是 O(1)。但是如果要在指定位置 i 插入和删除元素的话（add(int index, E element)）时间复杂度就为 O(n-i)。因为在进行上述操作的时候集合中第 i 和第 i 个元素之后的(n-i)个元素都要执行向后位/向前移一位的操作。
    * LinkedList 采用链表存储，所以对于add(E e)方法的插入，删除元素时间复杂度不受元素位置的影响，近似 O(1)，如果是要在指定位置 i 插入和删除元素的话（(add(int index, E element)） 时间复杂度近似为 O(n) ，因为需要先移动到指定位置再插入。
4. 是否支持快速随机访问： LinkedList 不支持高效的随机元素访问，而 ArrayList 支持。快速随机访问就是通过元素的序号快速获取元素对象(对应于get(int index)方法)。
5. 内存空间占用： ArrayList的空间浪费主要体现在在list列表的结尾会预留一定的容量空间，而 LinkedList 的空间花费则体现在它的每一个元素都需要消耗比 ArrayList 更多的空间（因为要存放直接后继和直接前驱以及数据）。

### Set
**注重独一无二的性质: 存储的元素是无序的、不可重复的。**
* HashSet：基于哈希表实现，支持快速查找，但不支持有序性操作。基于 HashMap 实现的，底层采用 HashMap 来保存元素。
* LinkedHashSet：具有 HashSet 的查找效率，并且内部使用双向链表维护元素的插入顺序。LinkedHashSet 是 HashSet 的子类，并且其内部是通过 LinkedHashMap 来实现的。
* TreeSet：基于红黑树实现（(自平衡的排序二叉树)），支持有序性操作，例如根据一个范围查找元素的操作。查找效率不如HashSet，HashSet 查找的时间复杂度为 O(1)，TreeSet 则为 O(logN)。

#### HashSet 如何检查重复
*当你把对象加入HashSet时，HashSet 会先计算对象的hashcode值来判断对象加入的位置，同时也会与其他加入的对象的 hashcode 值作比较，如果没有相符的 hashcode，HashSet 会假设对象没有重复出现。但是如果发现有相同 hashcode 值的对象，这时会调用equals()方法来检查 hashcode 相等的对象是否真的相同。如果两者相同，HashSet 就不会让加入操作成功。* 

hashCode()与 equals() 的相关规定：
1. 如果两个对象相等，则 hashcode 一定也是相同的
2. 两个对象相等,对两个 equals() 方法返回 true
3. 两个对象有相同的 hashcode 值，它们也不一定是相等的
4. 综上，equals() 方法被覆盖过，则 hashCode() 方法也必须被覆盖
5. hashCode()的默认行为是对堆上的对象产生独特值。如果没有重写 hashCode()，则该 class 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）。

### Queue
* LinkedList：可以用它来实现双向队列。
* PriorityQueue：基于堆结构实现，可以用它来实现优先队列。

## Map
![map](/images/container/map.png)
**用 Key 来搜索的专家: 使用键值对（key-value）存储，Key 是无序的、不可重复的，value 是无序的、可重复的，每个键最多映射到一个值。**
* TreeMap： 红黑树（自平衡的排序二叉树）
* HashMap： JDK1.8之前HashMap由数组+链表组成的，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）。JDK1.8 以后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间
* HashTable：和 HashMap 类似，但它是线程安全的，这意味着同一时刻多个线程同时写入 HashTable 不会导致数据不一致。它是遗留类，不应该去使用它，而是使用 ConcurrentHashMap 来支持线程安全，ConcurrentHashMap 的效率会更高。
* LinkedHashMap：LinkedHashMap 继承自 HashMap，LinkedHashMap 在上面结构的基础上，增加了一条双向链表，使得上面的结构可以保持键值对的插入顺序，顺序为插入顺序或者最近最少使用（LRU）顺序。

## 如何选择
* 主要根据集合的特点来选用，比如我们需要根据键值获取到元素值时就选用 Map 接口下的集合，需要排序时选择 TreeMap,不需要排序时就选择 HashMap,需要保证线程安全就选用 ConcurrentHashMap。

* 当我们只需要存放元素值时，就选择实现Collection 接口的集合，需要保证元素唯一时选择实现 Set 接口的集合比如 TreeSet 或 HashSet，不需要就选择实现 List 接口的比如 ArrayList 或 LinkedList，然后再根据实现这些接口的集合的特点来选用。

## 为什么要使用
* 当我们需要保存一组类型相同的数据的时候，我们应该是用一个容器来保存，这个容器就是数组，但是，使用数组存储对象具有一定的弊端，因为我们在实际开发中，存储的数据的类型是多种多样的，于是，就出现了“集合”，集合同样也是用来存储多个数据的。

* 数组的缺点是一旦声明之后，长度就不可变了；同时，声明数组时的数据类型也决定了该数组存储的数据的类型；而且，数组存储的数据是有序的、可重复的，特点单一。 但是集合提高了数据存储的灵活性，Java 集合不仅可以用来存储不同类型不同数量的对象，还可以保存具有映射关系的数据。
