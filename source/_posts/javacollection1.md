---
title: java基础之结合源码理解集合（非concurrent）
date: 2022-1-8 18:43:58
categories: java #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: [java,集合]
---
## 集合常用接口和类
java中的集合非常重要，是一种容器，也是一种引用数据类型，集合相关的接口和类非常多，不考虑concurrent中的情况下，最常见的有如下这些：
![在这里插入图片描述](https://img-blog.csdnimg.cn/3a44d042ba814faeaeff6b420fd94d79.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5raC5a6X5YuL,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)
<!--more-->
从上图也可以看到主要分为两大阵营，一个是Collection，另一个是Map，在Collection中又分为List和Set。
List是有序的，这个有序实际上指的是元素放入和取出顺序是有序的，而不是指元素的大小。
Set是无序的，这个指的是元素放入和取出顺序不能保证。但是从上边可以看到Set下列出了HashSet和TreeSet，实际上TreeSet由于底层存储是树结构，这种存储必须比较大小，所以TreeSet里边虽然元素存入和取出也是无序，但是元素本身大小是有顺序的。
除此之外，HashSet有一个子类LinkedHashSet，在原本的存储结构上加了一层链表来记录存入顺序，所以可以说也是有序的。
Map是键值对，HashMap和HashSet相似，TreeMap和TreeSet相似，不过针对的都是Key，实际上HashSet底层就是HashMap，TreeSet底层就是TreeMap。
除了上边说的这些，还有两个其实工作中不常用，但是可能面试经常出现的，一个是Vector，另一个是HashTable。之所以面试常问，主要是因为Vector拿来和ArrayList做比较，Vector是线程安全的，因为方法都加了synchronized修饰。HashTable常拿来和HashMap比较，也是因为HashTable是线程安全的，方法都加了synchronized修饰。
除此之外，他们的很多实现都相似，甚至于在jdk1.8之前基本就是一样的。另外就是，从jdk源码的注释来看，HashTable和Vector都是jdk1.0就有的，而ArrayList和HashMap则是1.2才有。
尽管Vector和HashTable是线程安全的，但是实际现在并不常用，当需要使用线程安全的集合时，更好的选择是使用jdk1.5增加的concurrent包中的相关类，例如ConcurrentHashMap、CopyOnWriteArrayList、CopyOnWriteArraySet这些。
除了上边这些概述之外，这些具体的集合类中都还有很多比较重要的技术点，如：
## ArrayList扩容原理
ArrayList底层是数组，定义为Object[]，在jdk1.8的源码中，还有一个默认的长度10，一个默认的空数组以及一个记录数组使用长度的变量size，相关的定义如下：
```
transient Object[] elementData;
private static final int DEFAULT_CAPACITY = 10;
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
private int size;
```
当使用"new ArrayList<>()"来创建一个ArrayList对象的时候，会先把这个默认的空数组赋值给elementData，当调用add方法时，会再判断elementData是否和默认的空数组对象相同，如果不同，则进行数组的扩容。
```
public ArrayList() {
	this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

public boolean add(E e) {
	ensureCapacityInternal(size + 1);  // Increments modCount!!
	elementData[size++] = e;
	return true;
}

private void ensureCapacityInternal(int minCapacity) {
	if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
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

private static int hugeCapacity(int minCapacity) {
	if (minCapacity < 0) // overflow
		throw new OutOfMemoryError();
	return (minCapacity > MAX_ARRAY_SIZE) ?
		Integer.MAX_VALUE :
		MAX_ARRAY_SIZE;
}
```
从上边的源码中可以看到，第一次往AarrayList中添加数据的时候，最终实际扩容后的数组长度就是10.
当数组长度超过10之后再扩容，新的数组长度是"oldCapacity + (oldCapacity >> 1)"，这里重要的是"oldCapacity >> 1"这一段，在位运算中，右移以为代表除以2，左移一位代表乘以2，这里这里很明显就是扩容为原数组的1.5倍。
当时需要注意的是后边还有判断，就是当这个1.5倍的数值超过MAX_ARRAY_SIZE时，则最大值是Integer.MAX_VALUE。
## LinkedList结构和特点分析
ArrayList底层的数组结构决定了它随机访问速度很快，但是如果要插入和删除，则需要移动该位置之后的所有元素，就会显得低效，因此针对插入和删除多的操作，就需要使用LinkedList来提高效率。
LinkedList底层是链表结构，所谓的链表，实际就是每个节点除了存储数据本身之外，还会记录相邻节点的信息，只有通过相邻节点才能找到另一个节点，从逻辑上看，节点之间就像是一个链子连了起来。
在jdk1.8源码中，LinkedList有几个重要属性，如下：
```
transient int size = 0;
transient Node<E> first;
transient Node<E> last;
```
这三个属性实际就是List的使用长度，首节点和尾结点，而节点Node，是LinkedList的一个内部类，定义如下：
```
private static class Node<E> {
	E item;
	Node<E> next;
	Node<E> prev;

	Node(Node<E> prev, E element, Node<E> next) {
		this.item = element;
		this.next = next;
		this.prev = prev;
	}
}
```
可以看到这个Node你除了记录数据本身外，还会记录上一个节点和下一个节点信息，这种结构实际上可以称为双向链表。还有一种是单向链表，也就是只记录下一个节点而不记录上一个节点信息。
这里边还有一个特殊的链表，叫做循环链表，也就是说最后一个节点的下一个节点会指向首节点，从而形成一个环状。
当然了，这里的LinkedList中的链表不是循环链表，这个可以从向里边添加元素的源码看出来：
```
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
```
可以看到，这里采用的是所谓的尾插法，也就是说新的元素会往最后一个节点的后边放，把新几点的上一个节点设置成之前的最后一个几点，然后把之前的最后一个节点的下一个几点设置成新节点。
之前说了，ArrayList中的底层结构是数组，因此当随机访问元素，也就是用下标来去元素的时候，实际可以直接用数组下标去除数组中的元素，但是LinkedList中的链表结构就不能这样取，只能通过遍历，直到找到需要找的那个元素。
很显然，这个查找如果是从前往后遍历，如果刚好是第一个节点的，那么只需要循环一次，但是如果刚好是最后一个，则需要遍历集合长度的次数。
因此在LinkedList中再随机访问元素的时候进行了一个优化，里边用到了二分查找算法，也就是把集合元素一分为二，当访问元素的索引位于集合前半部分时就从前往后遍历，如果访问的元素索引位于集合后半部分，则从后往前遍历，相关源码如下：
```
public E get(int index) {
	checkElementIndex(index);
	return node(index).item;
}

Node<E> node(int index) {
	// assert isElementIndex(index);

	if (index < (size >> 1)) {
		Node<E> x = first;
		for (int i = 0; i < index; i++)
			x = x.next;
		return x;
	} else {
		Node<E> x = last;
		for (int i = size - 1; i > index; i--)
			x = x.prev;
		return x;
	}
}
```
## HashSet
HashSet底层实际就是HashMap，创建HashSet对象时，底层实际就会创建HashMap对象，这个key就是要往HashSet中放的数据，Value则是一个固定的对象。往HashSet里放数据实际也是往底层的HashMap中放数据，这些从源码都可以看出来：
```
public HashSet() {
	map = new HashMap<>();
}

public boolean add(E e) {
	return map.put(e, PRESENT)==null;
}
```
针对于HashSet这种特点，一些原理性的内容实际是需要结合HashMap来说。
## LinkedHashSet
LinkedHashSet的底层是LinkedHashMap，所以原理性的内容实际也需要结合LinkedHashMap来说。
## TreeSet
TreeSet的底层是TreeMap。
## 结合源码看HashMap
Map里的数据都是键值对形式存储的，在jdk1.8中，HashMap的底层是采用的数组+链表+红黑树这样的结构存储数据。
首先，HashMap所谓的键值对，在JDK1.8中实际上是一个Node内部类，这个类继承自另一个内部类Entry（jdk1.8以前直接就是这个），大概定义如下：
```
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
...
}
```
当往HashMap中存数据的时候，实际就是生成这个Node对象，然后存入Node数组。
在存的过程中，首先就涉及到一个数组长度问题，从源码中可以看到，在创建HashMap时候并不会创建底层的数组，只是对加载因子这些变量进行了赋值，例如：
```
public HashMap() {
	this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}

public HashMap(int initialCapacity) {
	this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

public HashMap(int initialCapacity, float loadFactor) {
	if (initialCapacity < 0)
		throw new IllegalArgumentException("Illegal initial capacity: " +
										   initialCapacity);
	if (initialCapacity > MAXIMUM_CAPACITY)
		initialCapacity = MAXIMUM_CAPACITY;
	if (loadFactor <= 0 || Float.isNaN(loadFactor))
		throw new IllegalArgumentException("Illegal load factor: " +
										   loadFactor);
	this.loadFactor = loadFactor;
	this.threshold = tableSizeFor(initialCapacity);
}
```
底层数组的创建实际是在put数据的时候，put方法部分源码如下：
```
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
			   boolean evict) {
	Node<K,V>[] tab; Node<K,V> p; int n, i;
	if ((tab = table) == null || (n = tab.length) == 0)
		n = (tab = resize()).length;
	if ((p = tab[i = (n - 1) & hash]) == null)
		tab[i] = newNode(hash, key, value, null);
	else {
		Node<K,V> e; K k;
		if (p.hash == hash &&
			((k = p.key) == key || (key != null && key.equals(k))))
			e = p;
		else if (p instanceof TreeNode)
			e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
		else {
			for (int binCount = 0; ; ++binCount) {
				if ((e = p.next) == null) {
					p.next = newNode(hash, key, value, null);
					if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
						treeifyBin(tab, hash);
					break;
				}
				if (e.hash == hash &&
					((k = e.key) == key || (key != null && key.equals(k))))
					break;
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
	if (++size > threshold)
		resize();
	afterNodeInsertion(evict);
	return null;
}
```
这个方法看起来比较复杂，也是这个方法涉及了HashMap中的大部分关键知识点。
首先，在这里对底层数组table变量进行判断，是null，则会创建一个。
其次，在创建数组的时候，不仅是第一次需要创建，既然是数组，长度就固定，当装满的时候就需要进行数组扩容，而数组的扩容也就意味着要生成新的数组。
当然了，在HashMap中并不是等装满了才会扩容。
那么这个数组，不论是第一次创建，还是扩容的时候，都需要指定数组的长度，这个长度的计算就涉及到很多技术点。
在HashMap使用规范里，其实是建议指定一个初始长度的，这样后边就不用去计算，尤其是确定集合大小的时候，创建的时候指定长度就可以减少扩容的操作。
在创建底层数组的时候，有两个重要的属性，一个是默认长度，另一个是默认加载因子。默认长度在源码中的定义如下：
```
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
```
之前说过，右移一位指的是乘以2，那么右移4位，实际就是四次乘以2，即1*2*2*2*2=16，所以这个默认的长度就是16.
默认加载因子在源码中定义如下：
```
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```
这个加载因子的作用就是用来判断底层数组扩容的，例如这里是0.75，意思就是当实际数据量超过数组容量的0.75倍时就对底层数组进行扩容，扩容为原来的2倍。
在创建初始数组以及进行数组扩容的时候，有一个细节，就是数组的长度最终一定会是2的n次幂，即使创建HashMap的时候指定了不是2的n次幂，结果也会是一个比指定的数大一点的2的n次幂的数，对于这个数组长度计算的方法实际就是下边这段源码：
```
static final int tableSizeFor(int cap) {
	int n = cap - 1;
	n |= n >>> 1;
	n |= n >>> 2;
	n |= n >>> 4;
	n |= n >>> 8;
	n |= n >>> 16;
	return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```
上边的代码用了一堆的位运算，先不管具体是什么，如果自己调用一下这个方法就会验证上边的2的n次幂这个说法。另外就是，上边方法第一行的减一，其实就是为了在传入的数刚好是2的n次幂的时候不会生成一个比它大的2的n次幂的数。
为什么这个长度一定要是2的n次幂呢，结论是为了减少哈希碰撞，因为在上边putVal这个方法中可以看到，底层数组索引的取值实际是这样的代码：
```
if ((p = tab[i = (n - 1) & hash]) == null)
```
也就是说索引是通过"(n - 1) & hash"这样一个表达式来计算的，这个表达式等价于"hash % n"。
说简单点，就是HashMap底层数组的索引确定，是通过key的hash值和数组长度的取余操作来实现的，而这个操作中的n就是数组长度，当n的值是2的次幂的时候就可以降低相同的概率（需要注意这里的n是HashMap中的变量，不能和2的n次幂这个n混淆了）。
HashMap中所谓的哈希碰撞，其实就是指这个数组索引的取值相等。
当这个取值真的相等的时候，也就是说要往数组的同一个位置存数据，这时候就轮到链表发挥作用了，这部分的代码就在上边putVal中。
先判断两个key的hash值是否一样以及key的值是否一样，如果这两个都一样，则认定是同一个，如果不一样则判断是不是红黑树结构，如果是，就创建新的树节点，如果不是，则走链表的逻辑。
这里获取hash值和判断值是否一样，就涉及到对象的hashCode方法和equals方法，所以对于需要作为HashMap的key的对象，需要重写hashCode和equals方法。
在链表的逻辑中则又有关于链表转为红黑树结构的逻辑，即：
```
if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
...

final void treeifyBin(Node<K,V>[] tab, int hash) {
	int n, index; Node<K,V> e;
	if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
		resize();
	else if ((e = tab[index = (n - 1) & hash]) != null) {
		TreeNode<K,V> hd = null, tl = null;
		do {
			TreeNode<K,V> p = replacementTreeNode(e, null);
			if (tl == null)
				hd = p;
			else {
				p.prev = tl;
				tl.next = p;
			}
			tl = p;
		} while ((e = e.next) != null);
		if ((tab[index] = hd) != null)
			hd.treeify(tab);
	}
}
```
梳理上边的逻辑可以看到有这样有个关键判断，即"binCount >= TREEIFY_THRESHOLD - 1"和"(n = tab.length) < MIN_TREEIFY_CAPACITY"，这里TREEIFY_THRESHOLD是8，MIN_TREEIFY_CAPACITY是64，分析这部分代码可以知道也就是当链表长度大于8并且数组的长度大于64的时候才会把链表转换为红黑树结构，为什么是这两个值，据说是jdk研发团队经过测试发现在这样的情况下转换之后性能才更好。
红黑树具体的内容比较复杂，在这里需要记住的是它是相对均衡的二分查找树，实际也是为了提高查询性能。

## LinkedHashMap
由于HashMap的是根据hash值散列算法确定数组下标的，所以里边存入的元素就无法保证存入顺序，而HashMap有一个子类LinkedHashMap就在此基础上增加了一个链表来存储元素的存入顺序，其他内容和HashMap基本类似。
## TreeMap
TreeMap是直接基于红黑树存储键值对的，存数据的时候需要比较key的大小，这个比较借助于对象的比较器，可以是外部比较器，也可以是内部比较器，在创建TreeMap对象的时候可以选择传入外部比较器。
红黑树在存数据的过程中根节点是会变的，以尽量保持一个所谓的平衡，为什么要平衡，就是为了查找的时候更快，因为查找的时候会基于比较器和跟节点的key进行比较，如果小，则往根节点左边查找，如果大，则往根节点右边查找，尽量两边平衡的话，就可能使查询的次数变少，从而效率变高。相关源码如下：
```
final Entry<K,V> getEntry(Object key) {
	// Offload comparator-based version for sake of performance
	if (comparator != null)
		return getEntryUsingComparator(key);
	if (key == null)
		throw new NullPointerException();
	@SuppressWarnings("unchecked")
		Comparable<? super K> k = (Comparable<? super K>) key;
	Entry<K,V> p = root;
	while (p != null) {
		int cmp = k.compareTo(p.key);
		if (cmp < 0)
			p = p.left;
		else if (cmp > 0)
			p = p.right;
		else
			return p;
	}
	return null;
}
```
实际上，在存数据的时候也是这样判断key大小从而判断是往前左还是往右放，只不过不是单纯的只和根节点判断，并且最终还要做一个红黑树的平衡性和颜色的调整，相关源代码如下：
```
public V put(K key, V value) {
	Entry<K,V> t = root;
	if (t == null) {
		compare(key, key); // type (and possibly null) check

		root = new Entry<>(key, value, null);
		size = 1;
		modCount++;
		return null;
	}
	int cmp;
	Entry<K,V> parent;
	// split comparator and comparable paths
	Comparator<? super K> cpr = comparator;
	if (cpr != null) {
		do {
			parent = t;
			cmp = cpr.compare(key, t.key);
			if (cmp < 0)
				t = t.left;
			else if (cmp > 0)
				t = t.right;
			else
				return t.setValue(value);
		} while (t != null);
	}
	else {
		if (key == null)
			throw new NullPointerException();
		@SuppressWarnings("unchecked")
			Comparable<? super K> k = (Comparable<? super K>) key;
		do {
			parent = t;
			cmp = k.compareTo(t.key);
			if (cmp < 0)
				t = t.left;
			else if (cmp > 0)
				t = t.right;
			else
				return t.setValue(value);
		} while (t != null);
	}
	Entry<K,V> e = new Entry<>(key, value, parent);
	if (cmp < 0)
		parent.left = e;
	else
		parent.right = e;
	fixAfterInsertion(e);
	size++;
	modCount++;
	return null;
}
```