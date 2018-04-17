---
title: java基础加强-集合（四）视图与包装器
date: 2018-04-14 11:41:17
tags: java
categories: collection
---

# 视图与包装器（Views and Wrappers）

看了集合的接口图和类图可能会感觉： 用如此多的接口和抽象类来实现数量并不多的具体集合类似乎没有太大必要。然而， 这两张图并没有展示出全部的情况。通过使用视图( views) 可以获得其他的实现了 `Collection` 接口和 `Map` 接口的对象。映射类的 `keySet` 方法就是一个这样的示例。初看起来， 好像这个方法创建了一个新集， 并将映射中的所有键都填进去，然后返回这个集。但是， 情况并非如此。取而代之的是： `keySet` 方法返回一个实现 `Set` 接口的类对象， 这个类的方法对原映射进行操作。这种集合称为视图。

## 轻量级集合包装器(Lightweight Collection Wrappers)

* `Arrays`的`asList`

`Arrays` 类的静态方法 `asList` 将返回一个包装了普通 Java 数组的 List 包装器。这个方法可以将数组传递给一个期望得到列表或集合参数的方法。例如：
``` java
Card[] cardOeck = new Card[52];
List<Card> cardList = Arrays.asList(cardDeck):
```
返回的对象不是 ArrayList。它是一个视图对象， 带有访问底层数组的 get 和 set方法。改变数组大小的所有方法（例如，与迭代器相关的 add 和 remove 方法）都会抛出一个`Unsupported OperationException` 异常。

`asList` 方法可以接收可变数目的参数。例如：
``` java
List<String> names = Arrays.asList("Amy", "Bob", "Carl");
```

* `Collections.nCopies`

``` java
Collections.nCopies(n, anObject)
```
将返回一个实现了 `List` 接口的不可修改的对象， 并给人一种包含n个元素， 每个元素都像是
一个 anObject 的错觉。
例 如，下面的调用将创建一个包含100个字符串的`List`, 每串都被设置为"DEFAULT"：
``` java
List<String> settings = Collections.nCopies(100, "DEFAULT");
```
存储代价很小。这是视图技术的一种巧妙应用。

* `Collections.singleton(anObject)`

将返回一个视图对象。这个对象实现了 `Set` 接口（与产生 `List` 的 `ncopies` 方法不同）。返回的对象实现了一个不可修改的单元素集。 `singletonList`方法与 `singletonMap` 方法类似。

* 空集 

类似地，对于集合框架中的每一个接口，还有一些方法可以生成空集、 列表、 映射， 等
等。特别是， 集的类型可以推导得出：
``` java
Set<String> deepThoughts = Collections.emptySet() ;
```

## 子范围(Subranges)

获得一个列表的子范围视图:
``` java
List group2 = staff.subList(10, 20)
```
第一个索引包含在内， 第二个索引则不包含在内。
可以将任何操作应用于子范围，并且能够自动地反映整个列表的情况。 例如，可以删除整个子范围：
``` java
group2.clear(); // staff reduction
```
`staff` 列表中的相应元素被清除了， 并且 `group2` 为空。

对于有序集和映射， 可以使用排序顺序而不是元素位置建立子范围。`SortedSet` 接口声明了 3 个方法：

``` java
SortedSet< E> subSet(E from, E to)
SortedSet< E> headSet(E to)
SortedSet< E> tail Set(E from)
```

这些方法将返回大于等于 `from` 且小于 `to` 的所有元素子集。有序映射也有类似的方法：
``` java
SortedMap< K, V> subMap(K from, K to)
SortedMap<K, V> headMap(K to)
SortedMap<K, V> tail Map(K from)
```

返回映射视图， 该映射包含键落在指定范围内的所有元素。
Java SE 6 引人的 `NavigableSet` 接口赋予子范围操作更多的控制能力。可以指定是否包括边界：
``` java
NavigableSet<E> subSet ( E from, boolean fromlnclusive, E to, boolean tolnclusive)
NavigableSet<E> headSet(E to, boolean tolnclusive)
Navigab1eSet<E> tail Set(E from, boolean fromlnclusive)
```

## 不可修改的视图(Unmodifable Views)

Collections 还有几个方法， 用于产生集合的不可修改视图 （ unmodifiable views)。这些视图对现有集合增加了一个运行时的检查。如果发现试图对集合进行修改， 就抛出一个异常，同时这个集合将保持未修改的状态。

可以使用下面 8 种方法获得不可修改视图：
``` java
Collections.unmodifiableCollection(Collection<? extends T> c)
Collections.unmodifiableList(List<? extends T> list)
Collections.unmodifiableSet(Set<? extends T> s)
Collections.unmodifiableSortedSet(SortedSet<T> s)
Collections.unmodifiableNavigableSet(NavigableSet<T> s)
Collections.unmodifiableMap(Map<? extends K, ? extends V> m)
Collections.unmodifiableSortedMap(SortedMap<K, ? extends V> m)
Collections.unmodifiableNavigableMap(NavigableMap<K, ? extends V> m)
```

每个方法都定义于一个接口。 例如，`Collections.unmodifiableList`的参数是`List<? extends T> list`，这样就可以与`ArrayList`、`LinkedList`或者任何实现了`List`接口的其他类一起协同工作。

例如， 假设想要查看某部分代码， 但又不触及某个集合的内容， 就可以进行下列操作：

``` java
List<String> staff = new LinkedList() ;
lookAt (Collections.unmodifiableList(staff));
```

`Collections.unmodifiableList`方法将返回一个实现 `List` 接口的类对象。

由于视图只是包装了接口而不是实际的集合对象， 所以只能访问接口中定义的方法。例如，`LinkedList`类有一些非常方便的方法，`addFirst` 和 `addLast`，它们都不是 `List` 接的方法，不能通过不可修改视图进行访问。

## 同步视图（Synchronized Views）

`Collections`类有一系列的方法来得到一个集合的同步视图：

``` java
public static <T> Collection<T> synchronizedCollection(Collection<T> c)
public static <T> List<T> synchronizedList(List<T> list)
public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m)
public static <K,V> NavigableMap<K,V> synchronizedNavigableMap(NavigableMap<K,V> m)
public static <T> NavigableSet<T> synchronizedNavigableSet(NavigableSet<T> s)
public static <T> Set<T> synchronizedSet(Set<T> s)
public static <K,V> SortedMap<K,V> synchronizedSortedMap(SortedMap<K,V> m)
public static <T> SortedSet<T> synchronizedSortedSet(SortedSet<T> s)
```

## 关于可选操作的说明
在集合和迭代器接口的 API 文档中， 许多方法描述为“可选操作”。这看起来与接口的概念有所抵触。毕竟， 接口的设计目的难道不是负责给出一个类必须实现的方法吗？ 确实，从理论的角度看，在这里给出的方法很难令人满意。一个更好的解决方案是为每个只读视图和不能改变集合大小的视图建立各自独立的两个接口。不过， 这将会使接口的数量成倍增长，这让类库设计者无法接受。

所谓“可选操作”存在的原因是因为许多视图并不支持所有操作，当尝试调用这些不支持的操作时，就会抛出`UnsupportedOperationException`异常
