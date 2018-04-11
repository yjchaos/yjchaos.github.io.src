---
title: java基础加强-集合(二) 具体的集合
date: 2018-04-10 11:26:46
tags: java
categories: collection
---

<div align="left">
    <img src="/images/java基础加强-集合(二)/Classes in the collections framework.png" alt="Classes in the collections framework"/>
</div>

# Linked Lists

`ArrayList`有一个很大的缺陷：在中间删除一个元素的开销是非常大的，因为删除完该元素后，后面的元素都得往前移动，插入同理。
<div align="left">
    <img src="/images/java基础加强-集合(二)/Removing an element from an array.png" alt="Removing an element from an array"/>
</div>

`LinkedList`解决了这个问题。`ArrayList`中的元素是顺序存储的，而`LinkedList`的元素存在一个个结点中，每个结点都包含对下一个结点的引用和对上一个结点的引用。
<div align="left">
    <img src="/images/java基础加强-集合(二)/A doubly linked list.png" alt="A doubly linked list"/>
</div>

这样，删除中间某个元素的操作就只需将改元素前后两个元素连接起来即可。
<div align="left">
    <img src="/images/java基础加强-集合(二)/Removing an element from a linked list.png" alt="Removing an element from a linked list"/>
</div>

`ListIterator`作为`Iterator`的子接口，为`List`额外提供了下面一系列方法：
``` java
boolean hasPrevious()
E previous()
int nextIndex()
int previousIndex()
void set(E e)
void add(E e)
```
`AbstractList`定义了`listIterator`方法用来返回一个`ListIterator`的具体实现。

下面我们来加深一下对迭代器在`List`中添加删除元素的理解。我们用`|`来标注迭代器的位置。假设现在我们有一个`List`是`ABCD`，我们可以通过`iterator`方法或者`listIterator`方法获取到迭代器。

| 元素 | 操作 | 结果 | 
| - | :-: | -: | 
| &#124;ABCD | next | A&#124;BCD | 
| A&#124;BCD | remove | &#124;BCD | 
| &#124;BCD | next | B&#124;CD | 
| B&#124;CD | remove | &#124;CD | 
| &#124;CD | add(E) | E&#124;CD | 
| E&#124;CD | add(F) | EF&#124;CD | 
| EF&#124;CD | next | EFC&#124;D | 
| EFC&#124;D | set(G) | EFG&#124;D | 

<font color=red>综上，迭代器本身并不表示任何一个元素，它只能标识位置。`add`方法只依赖与迭代器的位置，而`remove`和`set`方法依赖迭代器的状态。所以在调用`remove`和`set`之前必须先调用`next`或者`previous`方法，因为这两个方法会返回元素。而`add`方法没有限制，它表示在迭代器位置前插入一个元素，连续插入时，元素的顺序与插入顺序一致。</font>

并发的问题：
>在`AbstractList`中定义了`modCount`字段，所有`add`和`remove`操作都会使`modCount`+1。当通过`iterator`或者`listIterator`方法获取迭代器时，迭代器中的`expectedModCount`字段会被初始化为`modCount`。所以，当别的迭代器或者集合方法改变了`modCount`值后，就会导致之前迭代器的`expectedModCount`和`modCount`值不一样，此时再使用之前迭代器就会报`ConcurrentModificationException`异常

# ArrayList

与`LinkedList`以结点存储元素不同，`ArrayList`底层使用数组来存储元素。

所以，如果你要用索引来获取元素，那么不要用`LinkedList`，因为`LinkedList`必须用迭代器来获取元素（`LinkedList`也提供了`get(int index)`方法，但这个方法时效率很低的）

`Vector`我们不经常使用，它的底层实现也是数组，它和`ArrayList`的不同之处在于它的所有方法都是同步的。

# HashSets（散列集）

散列表是一种能够快速找到元素的集合。这种数据结构会为每个元素计算一个散列码，通过散列码来定位元素在它内部数组中的位置。

如果你要向散列表中添加你自定义的类型。那么你必须实现`hashCode`方法和`equals`方法，而且要保证相等元素必定有相同的散列码。

java实现散列表的方式：
>链表的数组，每一个链表被称为一个桶(bucket)。当你向散列表中添加元素时，首先会计算元素的散列码，比如计算出来的散列码为`76268`，而此时桶的总数量为`128`，那么我们就可以计算出元素所在桶的索引为`76268 % 128 = 108`，如果此时索引为`108`的这个桶是空的，那么直接将新元素添加进链表，反之，我们称为散列冲突（hash collision），需要将新对象和桶中的所有元素一一比较来确定对象是否存在。如果散列码是合理的随机分布的，而且桶的数量足够大，那么冲突的机会会很少。

<div align="left">
    <img src="/images/java基础加强-集合(二)/A hash table.png" alt="A hash table"/>
    <br>
    <p><i><b>一个散列表</b></i></p>
</div>

如果你想更好的掌控散列表的额性能，你可以自己设置桶的初始数量，推荐值是总元素个数的75%~150%，当然你不会总是知道散列表中最终有多少个元素，此时就用默认值好了。java散列表桶的数量总是被设为2的幂，默认初始值为16，你设置的值总会被自动转为2的下一个幂。
当添加的元素过多，会导致散列冲突增加，影响性能。所以还有一个参数叫装填因子（load factor），默认值为0.75，它定义了一个散列表何时再散列 （rehashed)。比如当装填因子为0.75时，如果已经使用的桶的数量超过75%，就会触发再散列，此时会将桶的总数扩大至2倍。对于大多数应用程序来说， 装填因子为0.75 是比较合理的。

>java8中，桶满时（链表的元素大于8），链表会转为红黑树

# Tree Sets(树集)

`TreeSet`类与散列表十分相似，有一点改进就是它是一个有序集合。

你在使用`TreeSet`时，要么集合中的元素是可比较的（实现了`Comparable`接口，提供了`compareTo`方法），要么在创建`TreeSet`的时候传入一个比较器（`Comparator`的实现）。

多提一点，排序是基于树结构的（目前的实现方式是红黑树）

>从 JavaSE 6 起， TreeSet 类实现了 NavigableSet 接口。 这个接口增加了几个便于定位元素以及反向遍历的方法。

# Queues and Deques(队列与双端队列)

前面已经讨论过， 队列可以让人们有效地在尾部添加一个元素， 在头部删除一个元素。有两个端头的队列， 即双端队列， 可以让人们有效地在头部和尾部同时添加或删除元素。不支持在队列中间添加元素。在 Java SE 6 中引人了 Deque 接口， 并由 ArrayDeque 和LinkedList 类实现。这两个类都提供了双端队列，而且在必要时可以增加队列的长度。

# Priority Queues(优先级队列)

优先级队列不排序。它的底层通过二叉小顶堆实现。进行`add`和`remove`操作是都会使优先级最小的元素滑落到根节点，这样你每次在获取元素的时候，总是得到优先级最小的

>上面说的优先级是说值，习惯上值小优先级高