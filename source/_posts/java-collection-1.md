---
title: 为何 Hashtable、ConcurrentHashMap 不允许键或值为空，而 HashMap 允许
date: 2020-03-20 17:50:40
tags: [Java,Java集合]
---

### 1、底层源码实现

HashMap 在添加空键时做了特殊处理，若key为空则哈希值为0，value 则是没判空过滤操作

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```



Hashtable 在添加空值时会直接抛空指针异常，因为要调用 key.hashCode() 方法，所以 key 也不允许为空，不然一样会抛出空指针异常

```java
public synchronized V put(K key, V value) {
    // Make sure the value is not null
    if (value == null) {
        throw new NullPointerException();
    }
    // Makes sure the key is not already in the hashtable.
    Entry<?,?> tab[] = table;
    int hash = key.hashCode(); // key不能为空
    //....
}
```



ConcurrentHashMap 与 Hashable同理

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    //...
}
```



### 2、value 为不为空的限制原因

Hashtable 和 ConcurrentHashMap 都是线程安全的，使用的都是**安全失败机制（fail-safe），**这种机制会使你此次读到的数据不一定是最新的，但它们支持多线程操作，而 HashMap 则是线程不安全的，使用的是**快速失败机制（fail-fast）**，只支持单线程操作。

- fail-fast（快速失败）：在用迭代器遍历时一旦发现容器数据被修改了，立刻抛出 ConcurrentModificationException 异常，终止遍历（强一致性，不支持并发）

- fail-safe（安全失败）：在用迭代器遍历时会先复制原有集合内容，随后在拷贝的集合上进行遍历，所以即使中途数据被修改了也会继续遍历下去（弱一致性，支持并发）

所以当键值对key不为空、value为空时，这里会有一个问题：

<div align="center" style="font-size:18px;font-weight:bold">
    你如何判断 map.get(key) 取出来的值为空，是因为它值本身为空，还是因为 key 不存在，所以才返回空值的？
</div>

或许你会说，在操作前先使用 map.containsKey(key) 的方法进行判断，将它们分成两种情况即可，像下面这样

```java
if(map.containsKey(key)){
    methodA();    // 若map不存在该key，则执行methodA()
}else{
    methodB();  // 若map存在该key，则执行methodB()
}
```

**但！是！**

<img src="java-collection-1/1.gif" alt="img" style="zoom:45%;"/>

这种情况只适应于单线程，也就是说这三者中只适用于 HashMap ，对于 Hashtable 和ConcurrentHashMap 而言，在多线程环境下由于这个操作是非原子性的，所以 map.containsKey(key) 这个方法并不可靠。

举个例子，假设线程 1 执行这段代码，判断出map存在该key，于是打算执行 methodA() ，但它的线程在执行前被阻塞了，于是线程 1 挂起，轮到线程 2 在跑，但线程 2 接下来的操作却直接把这个 key 给删掉了，所以当又轮到线程 1 执行时，线程 1 会按照 map 依然存在该 key 的错误前提下继续执行代码，这就会引发线程的安全问题了，所以 Hashtable 和 ConcurrentHashMap 都不支持存储空值。

### 3、key 为不为空的限制原因

emmmm，这个问题就很有意思了，我想了半天也想不出为什么，直到我在网上找到一篇文章，写到了早在 2006 年就有人在网上发出求助邮件、里面就有提到这个问题，而回答这封邮件的人里就有 JUC 包的作者 **Doug Lea** ，以及 HashMap 的作者之一 **Josh Bloch**，两个巨佬的答复中直接给出了答案：

<div align="center"><del>他们乐意</del></div>

![image-20210228130138217](java-collection-1/image-20210228130138217.png)

Doug Lea 讨厌 null 值，他觉得允许 null 的设计本身就是不合理的，而 Josh Bloch 也快被他说服了，觉得这或许是个错误，但他自己也不是很确定。

~~Josh Bloch：或许是我错了但我就是不想改~~

俩人似乎讨论这个问题已经很久了……事实就是这么操蛋。



------

顺便贴一下这篇文章出处：https://segmentfault.com/a/1190000021105716