

# Java集合



## 集合的概念

```
由若干个确定的元素所构成的整体
```



### 集合框架

- 集合（Collection），存储元素
- 图（Map），存储键值对

<img src="https://www.pdai.tech/_images/java_collections_overview.png" alt="img" style="zoom:200%;" />

java访问集合的统一方式：迭代器（Iterator）



### 集合接口

Collection接口：最基本的集合接口，用来存储一组`不唯一`、`无序`的对象（具体性质由其实现类决定）

List接口：有序的Collection，能通过索引访问List中的元素

Set接口：存储一组唯一、无序的对象

Map接口：存储一组键值对象，提供key到value的映射



## List集合

List集合是一种有序列表，常用子类包括：

- ArrayList：基于动态数组实现，支持随机访问。
- LinkedList：基于双向链表实现，只能顺序访问，但是可以快速地在链表中间插入和删除元素。不仅如此，`LinkedList` 还可以用作栈、队列和双向队列。
- Vector：和 `ArrayList` 类似，但它是线程安全的。
  - Stack：它实现了一个标准的后进先出的栈，是 `Vector` 的子类。
- CopyOnWriteArrayList：一个线程安全的 `ArrayList`



### List的创建

除了通过new之外，在JDK9及其后的版本中，还可以通过List的of()方法创建List：

```java
List<Integer> list = List.of(1, 2, 3);
```



### List方法

| 方法名称                                | 介绍                              |
| :-------------------------------------- | :-------------------------------- |
| List<A> of(A x1, A x2, A x3, A... rest) | 返回元素为x1，x2，...，rest的列表 |
| boolean add(E e)                        | 在末尾添加一个元素                |
| boolean add(int index, E e)             | 在指定索引添加一个元素            |
| E remove(int index)                     | 删除指定索引的元素                |
| boolean remove(Object e)                | 删除某个元素                      |
| E get(int index)                        | 获取指定索引的元素                |
| int size()                              | 获取链表大小（包含元素的个数）    |

### ArrayList

#### 操作原理

ArrayList的操作等同于数组，但是ArrayList封装了数组的操作细节

当添加一个元素到指定索引位置时，ArrayList自动移动需要移动的元素

![image-20221030001820393](C:\Users\ALIENWARE\AppData\Roaming\Typora\typora-user-images\image-20221030001820393.png)

![image-20221030001842841](C:\Users\ALIENWARE\AppData\Roaming\Typora\typora-user-images\image-20221030001842841.png)

继续添加元素，但数组已满时，ArrayList将创建一个更大的数组，然后把旧数组的所有元素复制到新数组，并用新数组取代旧数组

![image-20221030001956805](C:\Users\ALIENWARE\AppData\Roaming\Typora\typora-user-images\image-20221030001956805.png)

#### 删除元素

法一：根据下标进行删除

```java
ArrayList<Integer> list = new ArrayList<>();
list.add(2);
Iterator<Integer> iterator = list.iterator();
for (int i = 0; i < list.size(); i++) {
    if (list.get(i) == 2) {
        list.remove(i);
        i--;
    }
}
```

法二：根据迭代器删除

```java
ArrayList<Integer> list = new ArrayList<>();
list.add(1);
list.add(2);
list.add(1);
Iterator<Integer> iterator = list.iterator();
while (iterator.hasNext()) {
    int i = iterator.next();
    if (i == 1) {
        iterator.remove();
    }
}
```





### LinkedList

LinkedList采用链表结构保存对象，便于向集合中插入或者删除元素

#### 常用方法

| 方法                                           | 描述                                                         |
| :--------------------------------------------- | :----------------------------------------------------------- |
| public boolean add(E e)                        | 链表末尾添加元素，返回是否成功，成功为 true，失败为 false。  |
| public void add(int index, E element)          | 向指定位置插入元素。                                         |
| public boolean addAll(Collection c)            | 将一个集合的所有元素添加到链表后面，返回是否成功，成功为 true，失败为 false。 |
| public boolean addAll(int index, Collection c) | 将一个集合的所有元素添加到链表的指定位置后面，返回是否成功，成功为 true，失败为 false。 |
| public void addFirst(E e)                      | 元素添加到头部。                                             |
| public void addLast(E e)                       | 元素添加到尾部。                                             |
| public boolean offer(E e)                      | 向链表末尾添加元素，返回是否成功，成功为 true，失败为 false。 |
| public boolean offerFirst(E e)                 | 头部插入元素，返回是否成功，成功为 true，失败为 false。      |
| public boolean offerLast(E e)                  | 尾部插入元素，返回是否成功，成功为 true，失败为 false。      |
| public void clear()                            | 清空链表。                                                   |
| public E removeFirst()                         | 删除并返回第一个元素。                                       |
| public E removeLast()                          | 删除并返回最后一个元素。                                     |
| public boolean remove(Object o)                | 删除某一元素，返回是否成功，成功为 true，失败为 false。      |
| public E remove(int index)                     | 删除指定位置的元素。                                         |
| public E poll()                                | 删除并返回第一个元素。                                       |
| public E remove()                              | 删除并返回第一个元素。                                       |
| public boolean contains(Object o)              | 判断是否含有某一元素。                                       |
| public E get(int index)                        | 返回指定位置的元素。                                         |
| public E getFirst()                            | 返回第一个元素。                                             |
| public E getLast()                             | 返回最后一个元素。                                           |
| public int indexOf(Object o)                   | 查找指定元素从前往后第一次出现的索引。                       |
| public int lastIndexOf(Object o)               | 查找指定元素最后一次出现的索引。                             |
| public E peek()                                | 返回第一个元素。                                             |
| public E element()                             | 返回第一个元素。                                             |
| public E peekFirst()                           | 返回头部元素。                                               |
| public E peekLast()                            | 返回尾部元素。                                               |
| public E set(int index, E element)             | 设置指定位置的元素。                                         |
| public Object clone()                          | 克隆该列表。                                                 |
| public Iterator descendingIterator()           | 返回倒序迭代器。                                             |
| public int size()                              | 返回链表元素个数。                                           |
| public ListIterator listIterator(int index)    | 返回从指定位置开始到末尾的迭代器。                           |
| public Object[] toArray()                      | 返回一个由链表元素组成的数组。                               |
| public T[] toArray(T[] a)                      | 返回一个由链表元素转换类型而成的数组。                       |



### Queue

队列（`Queue`）是一种经常使用的集合。`Queue` 实际上是实现了一个先进先出（FIFO：First In First Out）的有序表。它和 `List` 的区别在于，`List` 可以在任意位置添加和删除元素，而 `Queue` 只有两个操作：

- 把元素添加到队列末尾；
- 从队列头部取出元素。

超市的收银台就是一个队列：

![image.png](https://media-cn.lintcode.com/new_storage_v2/public/staff/None/8/19/dbda1af0-00be-11ec-964b-0242ac1d0002/image.png)



**队列接口继承图**
![img](https://img-blog.csdnimg.cn/20210424172014888.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3MTg0NDk3,size_16,color_FFFFFF,t_70)

**队列的分类**

有界队列：队列长度受限制

无界队列：队列长度超过界限时会进行扩容



非阻塞队列
1、ConcurrentLinkedQueue
  单向链表结构的无界并发队列, 非阻塞队列，由CAS实现线程安全，内部基于节点实现

2、ConcurrentLinkedDeque  
双向链表结构的无界并发队列, 非阻塞队列，由CAS实现线程安全    

3、PriorityQueue
内部基于数组实现，线程不安全的队列



阻塞队列
1、DelayQueue
一个支持延时获取元素的无界阻塞队列

2、LinkedTransferQueue
一个由链表结构组成的无界阻塞队列。

3、ArrayBlockingQueue
有界队列，阻塞式,初始化时必须指定队列大小，且不可改变；，底层由数组实现；

4、SynchronousQueue
最多只能存储一个元素，每一个put操作必须等待一个take操作，否则不能继续添加元素

5、PriorityBlockingQueue
一个带优先级的队列，而不是先进先出队列。元素按优先级顺序被移除，而且它也是无界的，也就是没有容量上限，虽然此队列逻辑上是无界的，但是由于资源被耗尽，所以试图执行添加操作可能会导致 OutOfMemoryError 错误；

在 Java 的标准库中，队列接口 `Queue` 定义了以下几个方法：

| 方法名称         | 描述                       |
| :--------------- | :------------------------- |
| int size()       | 获取队列长度               |
| boolean add(E)   | 添加元素到队尾             |
| boolean offer(E) | 添加元素到队尾             |
| E remove()       | 获取队首元素并从队列中删除 |
| E poll()         | 获取队首元素并从队列中删除 |
| E element()      | 获取队首元素但并不出队     |
| E peek()         | 获取队首元素但并不出队     |

对于具体的实现类，有的 `Queue` 有最大队列长度限制，有的 `Queue` 没有。注意到添加、删除和获取队列元素总是有两个方法，这是因为在添加或获取元素失败时，这两个方法的行为是不同的。我们用一个表格总结如下：

|                    | throw Exception | 返回false或null    |
| :----------------- | :-------------- | :----------------- |
| 添加元素到队尾     | add(E e)        | boolean offer(E e) |
| 取队首元素并删除   | E remove()      | E poll()           |
| 取队首元素但不删除 | E element()     | E peek()           |

举个例子，假设我们有一个队列，对它做一个添加操作，如果调用 `add()` 方法，当添加失败时（可能超过了队列的容量），它会抛出异常：



### Stack

栈是 `Vector` 的一个子类，它实现了一个标准的后进先出的栈。

堆栈只定义了默认构造函数，用来创建一个空栈。 堆栈除了包括由 `Vector` 定义的所有方法，也定义了自己的一些方法。

通过继承 `Vector` 类，`Stack` 类可以很容易的实现他本身的功能。因为大部分的功能在 `Vector` 里面已经提供支持了。

`Stack` 里面主要实现的有以下几个方法：

| 方法名称             | 描述                                             |
| :------------------- | :----------------------------------------------- |
| boolean empty()      | 测试堆栈是否为空。                               |
| E peek()             | 查看堆栈顶部的对象，但不从堆栈中移除它。         |
| E pop()              | 移除堆栈顶部的对象，并作为此函数的值返回该对象。 |
| E push(E item)       | 把项压入堆栈顶部。                               |
| int search(Object o) | 返回对象在堆栈中的位置，以 1 为基数。            |



## Map

**Map** 接口中键和值一一映射. 可以通过键来获取值。

- 给定一个键和一个值，你可以将该值存储在一个 Map 对象。之后，你可以通过键来访问对应的值。
- 当访问的值不存在的时候，方法就会抛出一个 NoSuchElementException 异常。
- 当对象的类型和 Map 里元素类型不兼容的时候，就会抛出一个 ClassCastException 异常。
- 当在不允许使用 Null 对象的 Map 中使用 Null 对象，会抛出一个 NullPointerException 异常。
- 当尝试修改一个只读的 Map 时，会抛出一个 UnsupportedOperationException 异常。

### Map方法

| 方法名称                        | 描述                                                         |
| :------------------------------ | :----------------------------------------------------------- |
| void clear( )                   | 从此映射中移除所有映射关系（可选操作）。                     |
| boolean containsKey(Object k)   | 如果此映射包含指定键的映射关系，则返回 true。                |
| boolean containsValue(Object v) | 如果此映射将一个或多个键映射到指定值，则返回 true。          |
| Set entrySet( )                 | 返回此映射中包含的映射关系的 Set 视图。                      |
| boolean equals(Object obj)      | 比较指定的对象与此映射是否相等。                             |
| Object get(Object k)            | 返回指定键所映射的值；如果此映射不包含该键的映射关系，则返回 null。 |
| int hashCode( )                 | 返回此映射的哈希码值。                                       |
| boolean isEmpty( )              | 如果此映射未包含键-值映射关系，则返回 true。                 |
| Set keySet( )                   | 返回此映射中包含的键的 Set 视图。                            |
| Object put(Object k, Object v)  | 将指定的值与此映射中的指定键关联（可选操作）。               |
| void putAll(Map m)              | 从指定映射中将所有映射关系复制到此映射中（可选操作）。       |
| Object remove(Object k)         | 如果存在一个键的映射关系，则将其从此映射中移除（可选操作）。 |
| int size( )                     | 返回此映射中的键-值映射关系数。                              |
| Collection values( )            | 返回此映射中包含的值的 Collection 视图。                     |

### HashMap

`HashMap` 实现了 Map 接口，根据键的 HashCode 值存储数据，具有很快的访问速度，最多允许一条记录的键为 `null`，不支持线程同步。

`HashMap` 是无序的，即不会记录插入的顺序。

`HashMap` 继承于 `AbstractMap`，实现了 Map、Cloneable、java.io.Serializable 接口。

![image.png](https://media-cn.lintcode.com/new_storage_v2/public/staff/None/10/4/6f32939a-24b4-11ec-9ebe-0242ac1d0002/image.png)

#### 方法



| 方法               | 描述                                                         |
| :----------------- | :----------------------------------------------------------- |
| put()              | 将指定的键/值对插入到 hashMap 中                             |
| get()              | 获取指定key对应的value                                       |
| remove()           | 删除hashMap指定key对应的键值对                               |
| clear()            | 删除 hashMap 中的所有键/值对                                 |
| clone()            | 复制一份 hashMap                                             |
| putAll()           | 将所有键/值对添加到 hashMap 中                               |
| putIfAbsent()      | 如果 hashMap 中不存在指定的键，则将指定的键/值对插入到 hashMap 中 |
| containsKey()      | 检查 hashMap 中是否存在指定的 key 对应的映射关系             |
| containsValue()    | 检查 hashMap 中是否存在指定的 value 对应的映射关系           |
| replace()          | 替换 hashMap 中是指定的 key 对应的 value                     |
| replaceAll()       | 将 hashMap 中的所有映射关系替换成给定的函数所执行的结果      |
| getOrDefault()     | 获取指定 key 对应对 value，如果找不到 key ，则返回设置的默认值 |
| forEach()          | 对 hashMap 中的每个映射执行指定的操作                        |
| entrySet()         | 返回 hashMap 中所有映射项的集合集合视图                      |
| keySet()           | 返回 hashMap 中所有 key 组成的集合视图                       |
| values()           | 返回 hashMap 中存在的所有 value 值                           |
| merge()            | 添加键值对到 hashMap 中                                      |
| compute()          | 对 hashMap 中指定 key 的值进行重新计算                       |
| computeIfAbsent()  | 对 hashMap 中指定 key 的值进行重新计算，如果不存在这个 key，则添加到 hasMap 中 |
| computeIfPresent() | 对 hashMap 中指定 key 的值进行重新计算，前提是该 key 存在于 hashMap 中 |

forEach方法示例：

```java
HashMap<Integer, Integer> map = new HashMap<>();
map.put(1, 1);
map.put(2, 2);
map.forEach((k, v) -> {
    System.out.println(k);
    System.out.println(v);
});
```

merge方法示例：

```java
HashMap<String, String> countries = new HashMap<>();
countries.put("Washington", "America");
countries.put("Canberra", "Australia");
countries.put("Madrid", "Spain");
String returnedValue = countries.merge("Washington", "USA", (oldValue, newValue) -> oldValue + "/" + newValue);
// countries：{Madrid=Spain, Canberra=Australia, Washington=America/USA}
```



### SortedMap

前面我们已经知道了，`HashMap` 是一种以空间换时间的映射表，它的实现原理决定了内部的 key 是无序的，即遍历 `HashMap` 的 key 时，其顺序是不可预测的（但每个 key 都会遍历一次且仅遍历一次）。

还有一种 `Map`，它在内部会对 key 进行排序，这种 `Map` 就是 `SortedMap`。注意到 `SortedMap` 是接口，它的实现类是 `TreeMap`。

![image.png](https://media-cn.lintcode.com/new_storage_v2/public/staff/None/8/26/6793fa86-0621-11ec-964b-0242ac1d0002/image.png)

`SortedMap` 保证遍历时以 key 的顺序来进行排序。例如，放入的 key 是 `"apple"`、`"pear"`、`"orange"`，遍历的顺序一定是 `"apple"`、`"orange"`、`"pear"`，因为 `String` 默认按字母排序：



`SortedMap` 在遍历时严格按照 key 的顺序遍历，最常用的实现类是 `TreeMap`；

作为 `SortedMap` 的Key必须实现 `Comparable` 接口，或者传入 `Comparator`；

要严格按照 `compare()` 规范实现比较逻辑，否则 `TreeMap` 将不能正常工作。





## Set

`Set` 用于存储不重复的元素集合



### HashSet

`HashSet` 基于 `HashMap` 来实现的，是一个不允许有重复元素的集合。

- `HashSet` 允许有 `null` 值。
- `HashSet` 是无序的，即不会记录插入的顺序。
- `HashSet` 不是线程安全的， 如果多个线程尝试同时修改 `HashSet`，则最终结果是不确定的。
- `HashSet` 实现了 `Set` 接口。

![img](https://www.runoob.com/wp-content/uploads/2020/07/java-hashset-hierarchy.png)





### TreeSet

前面我们已经学习了解了 `HashSet`， 它是一个无序且不含重复元素的集合。当我们要统计一个学校所有人员的年龄，且按照年龄从小排序时，那么使用 `HashSet` 就不行，这时候就需要使用 `TreeSet` 了，它是一个有序且不包含重复元素的集合。

`TreeSet` 是 `SortedSet` 接口的唯一实现类，它可以确保集合元素处于排序状态。`TreeSet` 支持两种排序方式，**自然排序**和**定制排序**，其中自然排序为默认的排序方式。

1. 自然排序：
   自然排序使用要排序元素的 `CompareTo(Object obj)` 方法来比较元素之间大小关系，然后将元素按照升序排列。
   Java 提供了一个 `Comparable` 接口，该接口里定义了一个 `compareTo(Object obj)` 方法，该方法返回一个整数值，实现了该接口的对象就可以比较大小。
   `obj1.compareTo(obj2)` 方法如果返回 `0`，则说明被比较的两个对象相等，如果**返回一个正数**，则表明 `obj1` 大于 `obj2`，如果是**负数**，则表明 `obj1` 小于 `obj2`。
   如果我们将两个对象的 `equals` 方法总是返回 `true`，则这两个对象的 `compareTo` 方法返回应该返回 `0`。
2. 定制排序
   自然排序是根据集合元素的大小，以升序排列，如果要定制排序，应该使用 `Comparator` 接口，实现 `int compare(T o1,T o2)` 方法。

`TreeSet` 的特点：

- `TreeSet` **不允许有 `null` 值。**
- `TreeSet` 是**有序**的，它会将插入的数据进行排序。
- `TreeSet` 不是线程安全的， 如果多个线程尝试同时修改 `TreeSet`，则最终结果是不确定的。
- `TreeSet` 实现了 `Set` 接口。

总结：

- `TreeSet` 是**二差树**实现的，`TreeSet` 中的数据是自动排好序的，不允许放入 `null` 值。
- `HashSet` 是哈希表实现的，`HashSet` 中的数据是无序的，可以放入 `null`，但只能放入一个 `null`，两者中的值都不能重复。



### LinkedHashSet

我们已经知道了 `HashSet` 无法保证插入数据的顺序，如果我们需要一个既能保证插入顺序，又能没有重复元素的容器呢？答案就是 `LinkedHashSet`。

`LinkedHashSet` 继承了 `HashSet` 实现了 `Set`、`Cloneable` 和 `Serializable` 接口。