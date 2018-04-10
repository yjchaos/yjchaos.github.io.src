---
title: java基础加强-集合(一) 集合框架
date: 2018-04-04 15:00:39
tags: java
categories: collection
---

# java集合框架

## 分离接口和实现

接口和实现分离的好处是，你在编程的时候可以用接口类型来代表一个具体集合的引用，当你改变主意时，你只需要使用不同的集合实现类即可，如下例

``` java
List<Customer> expressLane = new ArrayList<>;
```

``` java
List<Customer> expressLane = new LinkedList<>;
```

## Collection接口

Collection接口是最基本的集合接口，它提供了两个最基本的抽象方法:

``` java
public interface Collection<E>
{
    boolean add(E element);
    Iterator<E> iterator();
    . . .
}
```

## Iterators(迭代器)

`Iterator`接口有四个方法：
``` java
public interface Iterator<E>
{
    E next();
    boolean hasNext();
    void remove();
    default void forEachRemaining(Consumer<? super E> action);
}
```

使用`Iterator`遍历集合：

``` java
Collection<String> c = . . .;
Iterator<String> iter = c.iterator();
while (iter.hasNext())
{
    String element = iter.next();
    //do something with element
}
```

你也可以使用更简洁的foreach loop：
``` java 
for (String element : c)
{
    //do something with element
}
```
编译器会将foreach loop转成iterator循环遍历。凡是实现了`Iterable`接口的类都能使用foreach loop。`Iterable`接口只有一个抽象方法
``` java
public interface Iterable<E>
{
    Iterator<E> iterator();
    . . .
}
```

到了java8，你甚至不需要写循环，只需条用`Iterator`接口的`forEachRemaining`方法并传入lambda表达式，就可以对每一个元素执行lambda表达式
``` java
iterator.forEachRemaining(element -> do something with element);
```

java集合库的iterators和传统集合库的iterators有所不同。传统集合库（例如c++的标准模板库）的iterators是基于数组索引,通过索引就可以找到指定位置元素，就像数组通过索引i就可以查看数组元素a[i]一样。不需要查找元素，就可以将迭代器向前移动。但是java不是这样的，查找元素的唯一方法就是调用`next`，该方法同时会将迭代器的位置向前移动。
因此java迭代器可以认为是在两个元素之间，当调用`next`方法时，迭代器跳过下一个元素，并返回它跳过的元素
<div align="left">
    <img src="/images/java基础加强-集合(一)/Advancing an iterator.png" alt="Advancing an iterator"/>
</div>

最后一个`remove`方法会删除最后一个`next`方法返回的元素。重要的一点是在调用`remove`之前必须先调用`next`
``` java
it.remove();
it.remove(); // Error!
```

``` java
it.remove();
it.next();
it.remove(); // OK
```

## 泛型通用方法

由于`Collection`和`Iterator`都是泛型接口，这就意味着你可以给任何类型的集合写通用方法。例如你可以检测任意一个集合是否包含某个元素:
``` java
public static <E> boolean contains(Collection<E> c, Object obj)
{
    for (E element : c)
    if (element.equals(obj))
    return true;
    return false;
}
```

实际上，`Collection`接口已经定义了很多有用的通用方法,包括上面提到的`contains`方法:
``` java
int size()
boolean isEmpty()
boolean contains(Object obj)
boolean containsAll(Collection<?> c)
boolean equals(Object other)
boolean add(E e)
boolean addAll(Collection<? extends E> from)
boolean remove(Object obj)
boolean removeAll(Collection<?> c)
void clear()
boolean retainAll(Collection<?> c)
Object[] toArray()
<T> T[] toArray(T[] arrayToFill)
```

但是这样子还有一个困扰就是，如果实现`Collection`接口的类都要提供上面那么多通用方法的实现，会很麻烦，所以集合库提供了`AbstractCollection`类，它将`iterator`和`size`方法抽象化了，并提供了其他方法的实现。这样具体的类只要继承`AbstractCollection`就无需自己实现所有`Collection`方法了。

但是到了java8，上面的方法就有点过时了，更优雅的做法是使用默认方法。目前集合库依然使用的是上面的做法，不过一些新加的方法已经使用了默认方法,比如：
``` java
default boolean removeIf(Predicate<? super E> filter)
```

## 集合框架中的接口
<div align="left">
    <img src="/images/java基础加强-集合(一)/The interfaces of the collections framework.png" alt="The interfaces of the collections framework"/>
</div>

集合框架中最基础的接口是`Collection`和`Map`。当我们向`Collection`中添加元素时我们会使用：
``` java
boolean add(E element)
```
然而`Map`当中存储的是键值对，我们用`put`方法来插入元素：
``` java
V put(K key, V value)
```
`Collection`中我们用iterator来查看元素，而在`Map`中我们用`get`方法:
``` java
V get(K key)
```

`List`是一个有序的集合。其中的元素有两种获取方式：通过iterator或者通过索引值。前者成为顺序访问，而后者成为随机访问。`List`接口为随机访问增加了如下方法：
``` java
void add(int index, E element)
void remove(int index)
E get(int index)
E set(int index, E element)
```

`ListIterator`接口是`Iterator`接口的子接口，它定义了在一个在iterator位置前插入元素的方法：
``` java
void add(E element)
```

坦率地讲，集合框架的这个方面设计得很不好。实际中有两种有序集合，其性能开销有很大差异。由数组支持的有序集合`ArrayList`可以快速地随机访问，因此适合使用 List 方法并提供一个整数索引来访问。与之不同， 链表`LinkedList`尽管也是有序的， 但是随机访问很慢，所以最好使用迭代器来遍历。 如果原先提供两个接口就会容易一些了。

>为了避免对链表完成随机访问操作， Java SE 1.4 引入了一个标记接口 RandomAccess。
这个接口不包含任何方法， 不过可以用它来测试一个特定的集合是否支持高效的随机访问(`ArrayList`会返回true，而`LinkedList`会返回false)：
``` java
if (c instanceof RandomAccess)
{
    use random access algorithm
else
{
    use sequential access algorithm
}
```

`Set`接口和`Collection`接口一样，只不过`Set`接口对一些方法的定义更加严格。set的`add`方法不允许重复的元素，对于set来说要适当得定义`equals`方法：只要两个set中的元素都是相等的即可，不要求元素的顺序相同。`hashCode`方法要保证两个相等的元素必定会得到相同的散列码。

既然方法签名是一样的， 为什么还要建立一个单独的接口呢？ 从概念上讲， 并不是所有集合都是集。建立一个 Set 接口可以让程序员编写只接受集的方法。

`SortedSet` 和 `SortedMap` 接口会提供用于排序的比较器对象，这两个接口定义了可以得到集合子集视图的方法。

最后，Java SE 6 引人了接口 `NavigableSet` 和 `NavigableMap`, 其中包含一些用于搜索和遍历有序集和映射的方法。（理想情况下，这些方法本应当直接包含在 `SortedSet` 和 `SortedMap`接口中。）`TreeSet` 和 `TreeMap` 类实现了这些接口。