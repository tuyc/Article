
[TOC]

### Android 常用数据结构解析

本着共同学习的原则，来一波常用数据结构源码走读。包括ArrayList、LinkedList、HashMap。对这三种常用到的数据结构进行源码分析，本次列车解析主要包含各数据集合增、删、改、查。

#### ArrayList

- ArrayList是List接口的可变数组的实现。实现了所有可选列表操作，并允许包括 null 在内的所有元素。

数据结构：

![](https://github.com/tuyc/tuyc.github.io/blob/master/images/WX20170718-171815@2x%E7%BA%BF%E6%80%A7%E5%AD%98%E5%82%A8.png?raw=true)

线性表的顺序存储结构具有两个基本特点：

- 线性表中所有元素所占的存储空间是连续的；
- 线性表中各数据元素在存储空间中是按逻辑顺序依次存放的。

##### 源码

构造函数：

```java
private static final Object[] EMPTY_ELEMENTDATA = {};
private static final int DEFAULT_CAPACITY = 10;

public ArrayList(int initialCapacity) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity];
    }

    public ArrayList() {
        super();
        this.elementData = EMPTY_ELEMENTDATA;
    }    
```

ArrayList有两个构造函数，一种是给定初始容量，一种是默认构造。使用默认构造函数是不会分配多余的空间的，由源码`private static final Object[] EMPTY_ELEMENTDATA = {};`可以看出。而对于已分配的空间数量不够情况会怎么办呢？在元素增加（执行add方法）的时候会调用`ensureCapacityInternal()`方法进行容量检查并扩充容量。下面就来Look下add、remove、set、get方法源码：

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

public void add(int index, E element) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1, size - index);
        elementData[index] = element;
        size++;
}

public E remove(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
        modCount++;
        E oldValue = (E) elementData[index];

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index, numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
}

public E get(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));

        return (E) elementData[index];
 }
 
 public E set(int index, E element) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));

        E oldValue = (E) elementData[index];
        elementData[index] = element;
        return oldValue;
 }
```

ArrayList的数据操作都是简单的数组操作，这里主要介绍下两个方法：`ensureCapacityInternal`和`System.arraycopy`。System.arraycopy是数组操作的核心方法，理解了它你就理解了数组世界。

###### ensureCapacityInternal

```java
public void ensureCapacity(int minCapacity) {
        int minExpand = (elementData != EMPTY_ELEMENTDATA)
            // any size if real element table
            ? 0
            // larger than default for empty table. It's already supposed to be
            // at default size.
            : DEFAULT_CAPACITY;

        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }

    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
```

你阅读代码后发现，居然都是容量是否足够上的逻辑判断，容量不够的情况最后走到的是`grow()`方法。

数组容量增长：

```java
private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
}
```

这里主要介绍下`Arrays.copyOf`方法，跟踪Arrays.copyOf代码可以看到核心还是`System.arraycopy`方法。

###### System.arraycopy：

```java
/*
 * @param      src      源数组
 * @param      srcPos   源数组复制的起始位置
 * @param      dest     目的数组
 * @param      destPos  目的数组放置的起始位置
 * @param      length   复制的长度
 */
public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
```

该方法是native的，不对底层的源码进行研究，知道方法用法和产生的结果就OK了。根据参数的解释很好理解可以达到的效果，有兴趣的朋友可以直接调这个函数进行试验。

##### 小结

ArrayList实例都有一个容量，它总是至少等于列表的大小。随着向ArrayList中不断添加元素，其容量也自动增长。自动增长会带来数据向新数组的重新拷贝`Arrays.copyOf(elementData, newCapacity)`，因此，如果可预知数据量的多少，可在构造ArrayList时指定其容量。在添加大量元素前，应用程序也可以使用ensureCapacity操作来增加ArrayList实例的容量，这可以减少递增式再分配的数量。 

#### LinkedList

- LinkedList 是一个继承于AbstractSequentialList的双向链表。它也可以被当作堆栈、队列或双端队列进行操作。
- LinkedList 实现 List 接口，能对它进行队列操作。
- LinkedList 实现 Deque 接口，即能将LinkedList当作双端队列使用。

数据结构：

![双向链表](https://github.com/tuyc/tuyc.github.io/blob/master/images/WX20170719-003336@2x.png?raw=true)

链表存储结构具有两个基本特点：

- 链表是一种物理存储单元上非连续、非顺序的存储结构。
- 数据元素的逻辑顺序是通过链表中的指针链接次序实现的。

##### 源码

构造函数：

```java
	public LinkedList() {
    }
    
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }
```

LinkedList的构造函数有两个：一种是默认构造和一种是给初始集合数据。使用有参构造函数会把Collection数据依次取出放到链表的尾部。这里先来看一下构造函数中使用到的`addAll()`方法：

```java
	public boolean addAll(Collection<? extends E> c) {
        return addAll(size, c);
    }

    public boolean addAll(int index, Collection<? extends E> c) {
        checkPositionIndex(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        if (numNew == 0)
            return false;

        Node<E> pred, succ;
        if (index == size) {
            succ = null;
            pred = last;
        } else {
            succ = node(index);
            pred = succ.prev;
        }

        for (Object o : a) {
            @SuppressWarnings("unchecked") E e = (E) o;
            Node<E> newNode = new Node<>(pred, e, null);
            if (pred == null)
                first = newNode;
            else
                pred.next = newNode;
            pred = newNode;
        }

        if (succ == null) {
            last = pred;
        } else {
            pred.next = succ;
            succ.prev = pred;
        }

        size += numNew;
        modCount++;
        return true;
    }
```

这个方法的功能就是将Collection集合的全部数据拿出来放到index索引开始的链表上，可能是表头、表中、表尾，根据index参数决定。

下面来Look下add、remove、set、get，以及队列和栈常用方法源码：

```java
public boolean add(E e) {
        linkLast(e);
        return true;
}

void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
} 

public E remove(int index) {
        checkElementIndex(index);
        return unlink(node(index));
}

 public E set(int index, E element) {
        checkElementIndex(index);
        Node<E> x = node(index);
        E oldVal = x.item;
        x.item = element;
        return oldVal;
 }
 
 public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
 }
 
 /*******************队列*****************/
 public E poll() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
  }
  
  public boolean offer(E e) {
        return add(e);
  }
  
/****************栈******************/
public void push(E e) {
    addFirst(e);
}

public E pop() {
    return removeFirst();
}
```

LinkedList的数据操作都是简单的链表操作，前提是熟悉链表操作，想当初的C语言学习链表时也是经历了懵懂的代价，成熟了就好。

add方法会把元素加到表尾而不是表头；LinkedList可以当做栈和队列使用是因为实现了Deque接口，有队列poll、offer方法和栈push、pop方法，数据结构是基于链表来实现的；

##### 小结

相比ArrayList而言，LinkedList数据的增加，删除，不会有增容和数组拷贝，效率更高效。对于查询和修改，LinkedList需要根据指针依次检查，ArrayList是随存随取，ArrayList会更高效。有一点需要注意，很多时候都会谈到性能问题，但是对于数据量小的情况影响真的不大，哪一种数据结构方便就使用哪种就好了，不用去纠结和比较。



#### HashMap

+ HashMap 是一个采用哈希表（数组+链表）实现的键值对集合，继承自 AbstractMap，实现了 Map 接口 。 
+ 两个关键因子：初始容量、加载因子
+ 插入、获取的时间复杂度基本是 O(1)

数据结构：

![img](https://github.com/tuyc/tuyc.github.io/blob/master/images/WX20170719-001915@2x.png?raw=true)

##### 源码

构造函数：

```java
static final int DEFAULT_INITIAL_CAPACITY = 4; 
static final float DEFAULT_LOAD_FACTOR = 0.75f;

public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY) {
            initialCapacity = MAXIMUM_CAPACITY;
        } else if (initialCapacity < DEFAULT_INITIAL_CAPACITY) {
            initialCapacity = DEFAULT_INITIAL_CAPACITY;
        }

        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
        threshold = initialCapacity;
        init();
    }

    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

    public HashMap() {
        this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
    }

    public HashMap(Map<? extends K, ? extends V> m) {
        this(Math.max((int) (m.size() / DEFAULT_LOAD_FACTOR) + 1,
                      DEFAULT_INITIAL_CAPACITY), DEFAULT_LOAD_FACTOR);
        inflateTable(threshold);
        putAllForCreate(m);
    }
```

HashMap的构造函数有四个，而最终都会执行`HashMap(int initialCapacity, float loadFactor)`构造函数，无论是默认构造、初始容量构造，还是初始集合构造。主要有2点：

+ DEFAULT_INITIAL_CAPACITY默认初始容量，DEFAULT_LOAD_FACTOR装载因子，这是个经验值。
+ 可以指定初始容量，以及装载因子。一般情况都会使用默认的装载因子。

下面来Look下put（新增和修改）、remove、get源码：

```java
public V put(K key, V value) {
    if (table == EMPTY_TABLE) {
        inflateTable(threshold);
    }
    if (key == null)
        return putForNullKey(value);
    int hash = sun.misc.Hashing.singleWordWangJenkinsHash(key);
    int i = indexFor(hash, table.length);
    for (HashMapEntry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }

    modCount++;
    addEntry(hash, key, value, i);
    return null;
}

static int indexFor(int h, int length) {
       return h & (length-1);
}

void addEntry(int hash, K key, V value, int bucketIndex) {
        if ((size >= threshold) && (null != table[bucketIndex])) {
            resize(2 * table.length);
            hash = (null != key) ? sun.misc.Hashing.singleWordWangJenkinsHash(key) : 0;
            bucketIndex = indexFor(hash, table.length);
        }
        createEntry(hash, key, value, bucketIndex);
    }

void createEntry(int hash, K key, V value, int bucketIndex) {
        HashMapEntry<K,V> e = table[bucketIndex];
        table[bucketIndex] = new HashMapEntry<>(hash, key, value, e);
        size++;
}

public V remove(Object key) {
        Entry<K,V> e = removeEntryForKey(key);
        return (e == null ? null : e.getValue());
}

public V get(Object key) {
        if (key == null)
            return getForNullKey();
        Entry<K,V> entry = getEntry(key);

        return null == entry ? null : entry.getValue();
}
```

+ put函数

  ```java
  if (key == null)
          return putForNullKey(value);
  ```

  由此可知key可以为空。

  ```java
  int hash = sun.misc.Hashing.singleWordWangJenkinsHash(key);
  int i = indexFor(hash, table.length);
  ```

  hash的的算法不用care，只要理解根据key得到hash值即可；再根据hash值，`indexFor(int h, int length)`计算桶（数组）的下标；这一步任务就是定位桶的下标。

  ```java
  for (HashMapEntry<K,V> e = table[i]; e != null; e = e.next) {
          Object k;
          if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
              V oldValue = e.value;
              e.value = value;
              e.recordAccess(this);
              return oldValue;
          }
      }
  ```

  这里可以发现for循环初始值HashMapEntry<K,V> e=table[i]（i是上一步计算出的下标），HashMapEntry是一个链表结点，并拥有next结点；滑动链表结点，依次比较，当`e.hash == hash && ((k = e.key) == key || key.equals(k))`条件满足就找到了相同的key，进行修改操作，否则就进行增加操作，如下：

  ```java
  addEntry(hash, key, value, i);
  ```

  最后会执行到：

  ```java
  void createEntry(int hash, K key, V value, int bucketIndex) {
          HashMapEntry<K,V> e = table[bucketIndex];
          table[bucketIndex] = new HashMapEntry<>(hash, key, value, e);
          size++;
  }
  ```

  由代码可以发现，新增的元素都是放在对应桶的链表的表头。由此可以得出不发生hash碰撞情况，插入操作的时间复杂度是O(1)的。

+ remove函数

  ```java
  final Entry<K,V> removeEntryForKey(Object key) {
          if (size == 0) {
              return null;
          }
          int hash = (key == null) ? 0 : sun.misc.Hashing.singleWordWangJenkinsHash(key);
          int i = indexFor(hash, table.length);
          HashMapEntry<K,V> prev = table[i];
          HashMapEntry<K,V> e = prev;

          while (e != null) {
              HashMapEntry<K,V> next = e.next;
              Object k;
              if (e.hash == hash &&
                  ((k = e.key) == key || (key != null && key.equals(k)))) {
                  modCount++;
                  size--;
                  if (prev == e)
                      table[i] = next;
                  else
                      prev.next = next;
                  e.recordRemoval(this);
                  return e;
              }
              prev = e;
              e = next;
          }

          return e;
      }
  ```

  可以发现主要的过程也是由key得到hash值，再定位桶的下标，继续遍历链表，寻找与之对应的key，之后都是链表操作就不再描述了。

+ get函数

  核心函数是：

  ```java
  final Entry<K,V> getEntry(Object key) {
          if (size == 0) {
              return null;
          }

          int hash = (key == null) ? 0 : sun.misc.Hashing.singleWordWangJenkinsHash(key);
          for (HashMapEntry<K,V> e = table[indexFor(hash, table.length)];
               e != null;
               e = e.next) {
              Object k;
              if (e.hash == hash &&
                  ((k = e.key) == key || (key != null && key.equals(k))))
                  return e;
          }
          return null;
      }
  ```

  你会发现主要的过程还是根据key得到hash值，根据hash值定位桶的下标，继续遍历链表，寻找与之对应的key，找到了就返回对应的value。由此可以得出：在不发生hash碰撞的情况下，查找的时间复杂度为O(1)。因为不发生Hash碰撞，所要找的元素就是对应桶里的唯一元素，而桶的下标是根据key的hash值计算直接得到的。

##### 小结

数组存储的特点：区间是连续的，占用内存严重，故空间复杂的很大。寻址容易，插入和删除困难。

链表存储特点：区间离散，占用内存比较宽松，故空间复杂度很小，但时间复杂度很大，达O（N）。寻址困难，插入和删除容易。

哈希表就是结合了两者特性，做出一种寻址容易，插入删除也容易的数据结构。

