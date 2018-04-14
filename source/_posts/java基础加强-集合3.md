---
title: java基础加强-集合（三）映射
date: 2018-04-12 10:01:01
tags: java
categories: collection
---

# 映射

## 基本映射操作

Java 类库为映射提供了两个通用的实现： HashMap 和 TreeMap 。这两个类都实现了 Map 接口。

散列映射（hash map）对键进行散列， 树映射（tree map）用键值对元素进行排序， 并将其组织成搜索树。散列或比较函数只能作用于键。与键关联的值不能进行散列或比较。

键和值都允许为null。当使用`get(Object key)`方法获取元素时，若`key`不存在，则返回null。你也可以使用`getOrDefault(Object key, V defaultValue)`方法指定一个当`key`不存在时的默认值，一点要注意的是若`key`存在，而对应的`value`为null，此时返回null，不会返回你指定的默认值。

映射中的`key`必须唯一，如果`key`值相同，那么后`put`的`value`将会取代先前的`value`。

要迭代处理映射的键和值， 最容易的方法是使用 forEach 方法。
``` java
scores.forEach((k, v) ->
    System.out.println("key=" + k + ", value=" + v));
```

## 更新映射项

更新元素最简单的方法自然是`V put(K key, V value)`，但是如果我们是要对其中的元素值进行一些操作的话，可能还有更好的选择。

* merge

``` java
default V merge(K key, V value, BiFunction<? super V,? super V,? extends V> remappingFunction)
```
`merge`方法的作用是：如果`get(key)`不为null，那么会将`get(key)`和`value`作为`remappingFunction`方法的参数，`remappingFunction`的返回值作为结果重新和`key`绑定；反之如果如果`get(key)`为null，那么会直接`put(key,value)`。如果`remappingFunction`的返回值为null，则会删除该元素。例如：
``` java
//统计单词出现的次数
counts.merge(word,1,Integer::sum);
```

``` java
default V compute(K key, BiFunction<? super K,? super V,? extends V> remappingFunction)
```

* compute

``` java
default V compute(K key, BiFunction<? super K,? super V,? extends V> remappingFunction)
```
`compute`方法的作用是将`key`和`get(key)`作为参数传给`remappingFunction`方法，`remappingFunction`方法的返回值作为结果和`key`绑定，若`get(key)`为null，则会新创建一个元素`put`到map里。如果`remappingFunction`的返回值为null，则会删除该元素。

* computeIfPresent

``` java
default V computeIfPresent(K key, BiFunction<? super K,? super V,? extends V> remappingFunction)
```
`computeIfPresent`方法与`compute`方法的不同点在于：`computeIfPresent`方法只有在`get(key)`不为null的情况下，才会去做`remappingFunction`

* computeIfAbsent

``` java
default V computeIfAbsent(K key, Function<? super K,? extends V> mappingFunction) 
```
`computeIfAbsent`方法的作用是如果`get(key)`为null，则将`key`做为参数传给`mappingFunction`,`mappingFunction`方法的返回值作为结果和`key`绑定，若`mappingFunction`方法的返回值为null，则什么也不做。

* replaceAll

``` java
default void replaceAll(BiFunction<? super K,? super V,? extends V> function)
```
用`function`函数的结果替换每个`entry`

## 映射视图（Map Views）

有3种视图：键集、值集合（不是一个集）以及键 / 值对集。
``` java
Set<K> keySet()
Collection<V> values()
Set<Map.Entry<K, V>> entrySet()
```

`keySet()`返回的并不是一个`HashSet`或者`TreeSet`,而是实现了 `Set` 接口的另外某个类的对象。 `Set` 接口扩展了 `Collection` 接口。因此， 可以像使用集合一样使用 `keySet`。例如：
``` java
Set<String> keys = map.keySet();
for (String key : keys)
{
    //do something with key
}
```

如果想同时查看键和值， 可以通过枚举条目来避免查找值。使用以下代码：
``` java
for (Map.Entry<String, Employee> entry : staff.entrySet())
{
    String k = entry.getKey();
    Employee v = entry.getValue();
    //do something with k, v
}
```
原先上面的方法是访问所有映射条目的最高效的方法。如今，只需要使用 forEach 方法：
``` java
counts.forEach((k， v) -> {
    //dosomethingwith k, v
});
```

如果在键集视图上调用迭代器的 remove 方法， 实际上会从映射中删除这个键和与它关联的值。不过，不能向键集视图增加元素。另外， 如果增加一个键而没有同时增加值也是没有意义的。 如果试图调用 add 方法， 它会抛出一个 UnsupportedOperationException。 条目集视图有同样的限制。

## 弱散列映射（Weak Hash Maps）

设计 `WeakHashMap` 类是为了解决一个有趣的问题。 一个值，对应的键已经不再使用了，为了使它能被垃圾回收器回收，我们可以用`WeakHashMap` 类。

## 链接散列集与映射（Linked Hash Sets and Maps）

`LinkedHashSet` 和 `LinkedHashMap` 类用来记住插人元素项的顺序。当条目插入到表中时，就会并人到双向链表中。
<div align="left">
    <img src="/images/java基础加强-集合(三)/A linked hash table.png" alt="A linked hash table"/>
</div>

默认的情况下，遍历键集、值集合、键 / 值对集的时候是按插入顺序来的。但是如果你是用下面的方法创建的`LinkedHashMap`
``` java
LinkedHashMap<K, V>(initialCapacity, loadFactor, true)
```
`true`表示使用access-order，此时对元素进行`get`和`put`操作，都会使元素被放到链表尾部。它的用处在于可以用来实现高速缓存，我们可以构造一`LinkedHashMap`的子类，然后覆盖下面这个方法：
``` java
protected boolean removeEldestEntry(Map.Entry<K, V> eldest)
```
当函数返回`true`时会自动删除eldest元素。

## 枚举集与映射（Enumeration Sets and Maps）

`EmumSet` 是一个枚举类型元素集的高效实现。 由于枚举类型只有有限个实例， 所以`EnumSet` 内部用位序列实现。如果对应的值在集中， 则相应的位被置为 1。

`EnumSet` 类没有公共的构造器。可以使用静态工厂方法构造这个集：
``` java
enum Weekday { MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY };
EnumSet<Weekday> always = EnumSet.allOf(Weekday.class);
EnumSet<Weekday> never = EnumSet.noneOf(Weekday.class);
EnumSet<Weekday> workday = EnumSet.range(Weekday.MONDAY, Weekday.FRIDAY);
EnumSet<Weekday> mwf = EnumSet.of(Weekday.MONDAY, Weekday.WEDNESDAY, Weekday.FRIDAY);
```

EnumMap 是一个键类型为枚举类型的映射。它可以直接且高效地用一个值数组实现。在使用时， 需要在构造器中指定键类型：
``` java
EnumMap<Weekday, Employee> personInCharge = new EnumMap<>(Weekday.class);
```

## 标识散列映射(Identity Hash Maps)

类 `IdentityHashMap` 有特殊的作用。在这个类中， 键的散列值不是用 `hashCode` 函数计算
的， 而是用 `System.identityHashCode` 方法计算的。 这是 `Object.hashCode` 方法根据对象的内存地址来计算散列码时所使用的方式。 而且， 在对两个对象进行比较时，`IdentityHashMap` 类使用 ==, 而不使用 equals。也就是说， 不同的键对象， 即使内容相同， 也被视为是不同的对象。 