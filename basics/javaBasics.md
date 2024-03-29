<h1>java基础</h1>




<h2>集合</h2>

### Arraylist与 LinkedList 异同点？

+ 是否保证线程安全： ArrayList 和 LinkedList 都是不同步的，也就是不保证线程安全；
+ 底层数据结构： Arraylist 底层使用的是Object数组；LinkedList 底层使用的是双向循环链表数据结构；
+ 插入和删除是否受元素位置的影响： ArrayList 采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。 比如：执行add(E e)方法的时候， ArrayList 会默认在将指定的元素追加到此列表的末尾，这种情况时间复杂度就是O(1)。但是如果要在指定位置 i 插入和删除元素的话（add(int index, E element)）时间复杂度就为 O(n-i)。因为在进行上述操作的时候集合中第 i 和第 i 个元素之后的(n-i)个元素都要执行向后位/向前移一位的操作。 LinkedList 采用链表存储，所以插入，删除元素时间复杂度不受元素位置的影响，都是近似 O（1）而数组为近似 O（n）。
+ 是否支持快速随机访问： LinkedList 不支持高效的随机元素访问，而ArrayList 实现了RandmoAccess 接口，所以有随机访问功能。快速随机访问就是通过元素的序号快速获取元素对象(对应于get(int index)方法)。
+ 内存空间占用： ArrayList的空 间浪费主要体现在在list列表的结尾会预留一定的容量空间，而LinkedList的空间花费则体现在它的每一个元素都需要消耗比ArrayList更多的空间（因为要存放直接后继和直接前驱以及数据）。

### ArrayList 的扩容机制？

+ ArrayList扩容的本质就是计算出新的扩容数组的size后实例化，并将原有数组内容复制到新数组中去。默认情况下，新的容量会是原容量的1.5倍。

### hashmap的底层结构，jdk1.7和1.8的区别

+ hashmap底层是哈希表（数组+链表）的数据结构，数组中每个元素是一个单向链表 
，每个元素存放的是entry类型，entry(Entry包含四个属性：key, value, hash 值和用于单向链表的 next)，
为了解决hash冲突，采用拉链式，（链表结构）。
 
* 区别:
  + jdk8在数组长度大于64（不大于64直接扩容），且链表长度大于8时，会转换成红黑树
  + 链表数据的插入，jdk7是通过头插法插入的，而jdk8是通过尾插法插入的。
  （多线程并发扩容情况下，头插法会形成死循环）
 
### jdk1.8为什么链表需要转换成红黑树

+ 当链表过长，则会严重影响 HashMap 的性能，红黑树搜索时间复杂度是 O(logn)，而链表是糟糕的 O(n)。因此，JDK1.8 对数据结构做了进一步的优化，引入了红黑树，链表和红黑树在达到一定条件会进行转换：

### hashmap为什么线程不安全的

+ 任意时刻可以有多个线程同时写入hashmap,导致数据不一致，数据会覆盖。


### hashmap的put过程  jdk 1.8

+ 1.首先根据 key 的值计算 hash 值，找到该元素在数组中存储的下标；
+ 2.如果数组是空的，则调用 resize 进行初始化；
+ 3.如果没有哈希冲突直接放在对应的数组下标里；
+ 4.如果冲突了，且 key 已经存在，就覆盖掉 value；
+ 5.如果冲突后，发现该节点是红黑树，就将这个节点挂在树上；
+ 6.如果冲突后是链表，判断该链表是否大于 8 ，如果大于 8 并且数组容量小于 64，就进行扩容；如果链表节点大于 8 并且数组的容量大于 64，则将这个结构转换为红黑树；否则，链表插入键值对，若 key 存在，就覆盖掉 value。

### hashmap的get过程 jdk 1.8 

+ 首先将 key hash 之后取得所定位的桶。
+ 如果桶为空则直接返回 null 。
+ 否则判断桶的第一个位置(有可能是链表、红黑树)的 key 是否为查询的 key，是就直接返回 value。
+ 如果第一个不匹配，则判断它的下一个位置是红黑树还是链表。
+ 红黑树就按照树的查找方式返回值。不然就按照链表的方式遍历匹配返回值。


### hashmap的扩容过程

+ 扩容是调用的resize(int newCapacity)方法，
+ 扩容条件：
   + 1.hashmap默认大小是16. 负载因子是0.75.当hashmap的数组的长度达到16*0.75 =12时进行扩容，
   + 2.存放新值的时候当前存在数据发生hash碰撞(当前key计算的hash值换算出来的数组下标位置已经存在值)
+ 扩容使用一个容量更大的数组来代替已有的容量小的数组,容量为原来的2倍，扩容后，元素要不在原数组index上，要不在index+old_cap。
根据 e.hash & (newCap - 1)来计算出新数组的位置。

ps:
（1）hashmap在存值的时候（默认大小为16，负载因子0.75，阈值12），可能达到最后存满16个值的时候，再存入第17个值才会发生扩容现象，因为前16个值，每个值在底层数组中分别占据一个位置，并没有发生hash碰撞。
（2）当然也有可能存储更多值（超多16个值，最多可以存26个值）都还没有扩容。原理：前11个值全部hash碰撞，存到数组的同一个位置（这时元素个数小于阈值12，不会扩容），后面所有存入的15个值全部分散到数组剩下的15个位置（这时元素个数大于等于阈值，但是每次存入的元素并没有发生hash碰撞，所以不会扩容），前面11+15=26，所以在存入第27个值的时候才同时满足上面两个条件，这时候才会发生扩容现象。

### hashmap的优势

+ 在于hashmap底层使用的是哈希表（数组+链表）的数据结构，对于查询，插入和删除效率都高

### 解决hash冲突有哪些方法

+ 解决Hash冲突方法有:开放定址法、再哈希法、链地址法（拉链法）、建立公共溢出区。HashMap中采用的是 链地址法 。
+ 开放定址法也称为再散列法，基本思想就是，如果p=H(key)出现冲突时，则以p为基础，再次hash，p1=H(p),如果p1再次出现冲突，则以p1为基础，以此类推，直到找到一个不冲突的哈希地址pi。 因此开放定址法所需要的hash表的长度要大于等于所需要存放的元素，而且因为存在再次hash，所以只能在删除的节点上做标记，而不能真正删除节点。
+ 再哈希法(双重散列，多重散列)，提供多个不同的hash函数，当R1=H1(key1)发生冲突时，再计算R2=H2(key1)，直到没有冲突为止。 这样做虽然不易产生堆集，但增加了计算的时间。
+ 链地址法(拉链法)，将哈希值相同的元素构成一个同义词的单链表,并将单链表的头指针存放在哈希表的第i个单元中，查找、插入和删除主要在同义词链表中进行。链表法适用于经常进行插入和删除的情况。
建立公共溢出区，将哈希表分为公共表和溢出表，当溢出发生时，将所有溢出数据统一放到溢出区。

### ConcurrentHashMap底层原理，为啥是线程安全的，jdk1.7和1.8的区别</h5>

+ jdk1.7中ConcurrentHashMap 是  一个Segment数组,Segment 通过继承ReentrantLock 来进行加锁，（segment是一个内部类型，HashMap 非常类似，唯一的区别就是其中的核心数据如 value ，以及链表都是 volatile 修饰的，保证了获取时的可见性。）
  每次需要加锁的操作锁住的是一个 segment，这样只要保证每个 Segment 是线程安全的，也就实现了全局的线程安全
+ jdk1.8中舍弃了segment概念，采用了cas和synchronized保证并发安全性。优先使用CAS,CAS失败后使用内置的synchronized

### HashMap 在JDK1.8的优化

+ 扩容过程 resize之后，元素的位置在原来的位置，或者原来的位置 +oldCap (原来哈希表的长度）。不需要像 JDK1.7 的实现那样重新计算hash ，只需要看看原来的 hash 值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成“原索引 + oldCap ”。这个设计非常的巧妙，省去了重新计算 hash 值的时间。
+ JDK1.7 中 rehash 的时候，旧链表迁移新链表的时候，如果在新表的数组索引位置相同，则链表元素会倒置（头插法）。JDK1.8 不会倒置，使用尾插法。

### ConcurrentHashMap的put()过程

+ jdk 1.7
  + 1.将当前 Segment 中的 table 通过 key 的 hashcode 定位到 HashEntry。
  + 2.遍历该 HashEntry，如果不为空则判断传入的 key 和当前遍历的 key 是否相等，相等则覆盖旧的 value。
  + 3.不为空则需要新建一个 HashEntry 并加入到 Segment 中，同时会先判断是否需要扩容。
  + 4.最后会解除在 1 中所获取当前 Segment 的锁。
+ jdk 1.8
  + 1.根据 key 计算出 hashcode 。
  + 2.判断是否需要进行初始化。
  + 3.当前 key 定位出的 Node，如果为空表示当前位置可以写入数据，利用 CAS 尝试写入，失败则自旋保证成功。
  + 4.如果当前位置的 hashcode == MOVED == -1,则需要进行扩容。
  + 5.如果都不满足，则利用 synchronized 锁写入数据。
  + 6.如果数量大于 TREEIFY_THRESHOLD(8) 则要转换为红黑树

### ConcurrentHashMap的get()过程

+ 1.根据计算出来的 hashcode 寻址，如果就在桶上那么直接返回值。
+ 2.如果是红黑树那就按照树的方式获取值。
+ 3.都不满足那就按照链表的方式遍历获取



### ConcurrentHashMap 不支持 key 或者 value 为 null 的原因？

+ 我们先来说value 为什么不能为 null ，因为ConcurrentHashMap 是用于多线程的 ，如果map.get(key)得到了 null ，无法判断，是映射的value是 null ，还是没有找到对应的key而为 null ，这就有了二义性。
  而用于单线程状态的HashMap却可以用containsKey(key) 去判断到底是否包含了这个 null 。

### JDK1.7与JDK1.8 中ConcurrentHashMap 的区别？

+ 数据结构：取消了Segment分段锁的数据结构，取而代之的是数组+链表+红黑树的结构。
+ 保证线程安全机制：JDK1.7采用Segment的分段锁机制实现线程安全，其中segment继承自ReentrantLock。JDK1.8 采用CAS+Synchronized保证线程安全。
+ 锁的粒度：原来是对需要进行数据操作的Segment加锁，现调整为对每个数组元素加锁（Node）。
+ 链表转化为红黑树:定位结点的hash算法简化会带来弊端,Hash冲突加剧,因此在链表节点数量大于8时，会将链表转化为红黑树进行存储。
+ 查询时间复杂度：从原来的遍历链表O(n)，变成遍历红黑树O(logN)。



### 
### 
### 
### 
### 
### 
### 


<hr>

   