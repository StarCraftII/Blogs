## HashMap原理

[TOC]
## 1.前言

HashMap源码分析，基于Android SDK 29中的HashMap代码。

## 2.使用及原理

### 2.1 定义

```
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
}
```

HashMap继承了AbstractMap，实现了Cloneable、Serializable接口。

### 2.2 使用

```
HashMap<String,String> map = new HashMap<>();
map.put("name","jack");
String name = map.get("name");
```

HashMap通过key-value的方式存取数据，key值唯一且可以为null，put方法存数据，get方法取数据

### 2.3 构造方法

```
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
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

public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
```

HashMap有有参构造方法和无参构造方法，主要初始化容量和加载因子还有阙值

initialCapacity：容量（必须是2的幂 ，且小于等于 1<<30）

loadFactor：加载因子（负载因子 默认是0.75）

threshold：阙值（下一个重新调整大小的阙值 capacity * load factor）

```
/**
 * Returns a power of two size for the given target capacity.
 */
static final int tableSizeFor(int cap) {
    int n = cap - 1; //防止cap本身就是2的幂（最终结果将是实际结果的2倍）
    n |= n >>> 1;   //使最高位后的两位变为1
    n |= n >>> 2;   //使最高位后的四位变为1
    n |= n >>> 4;   //使最高位后的八位变为1
    n |= n >>> 8;   //使最高位后的十六位变为1
    n |= n >>> 16;  //使最高位后的三十二位变为1
    // if 如果cap是负数或0 n最终求出来是最小负数
    // else n最终求出来是从最高位1开始到最低位都是1的数 ,n + 1 为向上round的最小2的次方
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

计算table的size,大于等于给定值cap且为2的次幂的最小整数

### 2.4 hash方法

计算hash值 (key的hashcode 异或 (key的hashcode 无符号右移 16位))

```
/**
  * 让hashCode的高16位也参与路由运算（若不做此操作，则在路由计算时：(n-1) & hashCode，当n的二进制数小于
  * 16位时，那么h的高16位将与0求与，特征将被短路掉，无法参与路由运算。）
  *
  * 计算key.hashCode（）并将哈希的较高位（XOR）扩展为较低。 因为该表使用2的幂次掩码，所以仅在当前掩码上
  * 方的位中变化的哈希集将始终发生冲突。（众所周知的示例是在小表中包含连续整数的Float键集。）因此，我们
  * 应用了一种变换，将向下传播较高位的影响。在速度，实用性和位扩展质量之间需要权衡。由于许多常见的哈希
  * 集已经合理分布（因此无法从扩展中受益），并且由于我们使用树来处理容器中的大量冲突集，因此我们仅以最
  * 便宜的方式对一些移位后的位进行XOR，以减少系统损失， 以及合并最高位的影响，否则由于表范围的限制，这
  * 些位将永远不会在索引计算中使用。
  */
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

tab[i = (n - 1) & hash] ,绝大多数情况下table的length一般都小于2^16即小于65536,所以hash & (length-1)结果始终是hash的低16位与(length-1)进行&运算，因为&和|都会使得结果偏向0或者1 ,并不是均匀的概念,所以用^

### 2.5 put方法

```
/**
  * 将指定值与该map中的指定键相关联。 如果该map先前包含该键的map，则将替换旧值。
  */
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

### 2.6 putVal方法

```
/**
  * Implements Map.put and related methods
  *
  * @param hash hash for key	平衡后的hash值
  * @param key the key	要插入的key
  * @param value the value to put	要插入的value
  * @param onlyIfAbsent if true, don't change existing value	如果是true，表示不改变已存在的值。
  * @param evict if false, the table is in creation mode.
  * @return previous value, or null if none	返回之前的值，如果存在
  */
  
//tab：表示hashMap的散列表
//p：表示当前散列表的一个元素
//n：表示散列表数组的长度
//i：表示路由寻址结果

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
			   boolean evict) {
	Node<K,V>[] tab; Node<K,V> p; int n, i;
	// 延迟初始化逻辑。第一次调用putVal()时进行初始化hashMap中的最耗费内存的散列表
	if ((tab = table) == null || (n = tab.length) == 0)
		n = (tab = resize()).length;
	// 取出tab数组上下标为i的节点，如果为空，创建一个新节点插入当前位置
	if ((p = tab[i = (n - 1) & hash]) == null)
		tab[i] = newNode(hash, key, value, null);
	else {
		// 临时节点e，临时Key k
		Node<K,V> e; K k;
		// 当前节点的hash值如插入元素的hash相同并且key的值也相同，把当前节点赋值给e，后续会进行替换操作
		if (p.hash == hash &&
			((k = p.key) == key || (key != null && key.equals(k))))
			e = p;
		// 当前节点为一个树节点
		else if (p instanceof TreeNode)
			e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
		else {
			// 当前节点为一个链表节点
			for (int binCount = 0; ; ++binCount) {
				// 如果p的下一个节点为空
				if ((e = p.next) == null) {
					// 创建一个新的节点，并让p的next指针指向它
					p.next = newNode(hash, key, value, null);
					// 如果当前链表的长度大于等于使用树的bin计数阀值 ，那么就需要将当前链表转化为红黑树
					if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
						treeifyBin(tab, hash);
					break;
				}
				// 当找到了一个与待插入元素key相同的节点，后续会进行替换操作
				if (e.hash == hash &&
					((k = e.key) == key || (key != null && key.equals(k))))
					break;
				// p指向p的下一个节点
				p = e;
			}
		}
		
		if (e != null) { // existing mapping for key
			V oldValue = e.value;
			// 如果允许改变已存在的值或者不存在旧的值
			if (!onlyIfAbsent || oldValue == null)
				// 替换e节点值
				e.value = value;
			// Callbacks to allow LinkedHashMap post-actions
			afterNodeAccess(e);
			return oldValue;
		}
	}
	// 表示散列表结构的修改次数。增删次数
	++modCount;
	// 如果有新插入节点，size + 1，并判断是否需要扩容
	if (++size > threshold)
		resize();
	afterNodeInsertion(evict);
	return null;
}
```

### 2.7 resize方法

1.计算出扩容后的数组大小和扩容阈值
     
2.进行扩容

```
/**
  * Initializes or doubles table size.  If null, allocates in
  * accord with initial capacity target held in field threshold.
  * Otherwise, because we are using power-of-two expansion, the
  * elements from each bin must either stay at same index, or move
  * with a power of two offset in the new table.
  *
  * 初始化或加倍表的大小。如果表是空的，则按照字段阈值中保存的初始容
  * 量目标进行分配，因为我们使用的是二次幂扩张，每个 bin 中的元素必须
  * 保持相同的索引或在新表中以 2 的幂偏移量移动。
  * @return the table
  */
final Node<K,V>[] resize() {
	// 扩容前的哈希表
	Node<K,V>[] oldTab = table;
	// 扩容前的数组长度
	int oldCap = (oldTab == null) ? 0 : oldTab.length;
	// 扩容前的扩容阈值
	int oldThr = threshold;
	// 记录新的哈希表容量和扩容阈值
	int newCap, newThr = 0;
	// oldCap如果大于0说明哈希表已经初始化过了，这是一次正常扩容
	if (oldCap > 0) {
		// 扩容之前的table数组大小已经达到最大阈值后，则不扩容，且设置扩容条件为int的最大值。（隐含：再也不会扩容）
		if (oldCap >= MAXIMUM_CAPACITY) {
			threshold = Integer.MAX_VALUE;
			return oldTab;
		}
		// 左移一位容量翻倍，并且小于最大容量阈值和大于默认初始化容量值
		else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
				 oldCap >= DEFAULT_INITIAL_CAPACITY)
			newThr = oldThr << 1; // double threshold
	}
	// 初始化时有设置threshold（阈值被设置成了初始容量）
	else if (oldThr > 0) // initial capacity was placed in threshold
		newCap = oldThr;
	// 初始化时未设置threshold
	else {               // zero initial threshold signifies using defaults
		// 新的哈希表容量等于默认初始化容量值
		newCap = DEFAULT_INITIAL_CAPACITY;
		// 新的哈希表阈值等于 哈希表容量 乘以 加载因子
		newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
	}
	// 新的哈希表阈值等于0，设置一个新的哈希表阈值
	if (newThr == 0) {
		float ft = (float)newCap * loadFactor;
		newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
				  (int)ft : Integer.MAX_VALUE);
	}
	// 更新扩容阈值
	threshold = newThr;
	@SuppressWarnings({"rawtypes","unchecked"})
		Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
	// 更新扩容阈值
	table = newTab;
	// oldTab不为null，即oldTab已经指向了一个数组
	if (oldTab != null) {
		// 遍历oldTab数组
		for (int j = 0; j < oldCap; ++j) {
			// 临时节点e
			Node<K,V> e;
			// 当前tab下标存在元素节点
			if ((e = oldTab[j]) != null) {
				// 置空tab下标指针
				oldTab[j] = null;
				// 如果e没有后继节点，直接把e
				if (e.next == null)
					newTab[e.hash & (newCap - 1)] = e;
				else if (e instanceof TreeNode)
					((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
				else { // preserve order
					Node<K,V> loHead = null, loTail = null;
					Node<K,V> hiHead = null, hiTail = null;
					Node<K,V> next;
					do {
						next = e.next;
						if ((e.hash & oldCap) == 0) {
							if (loTail == null)
								loHead = e;
							else
								loTail.next = e;
							loTail = e;
						}
						else {
							if (hiTail == null)
								hiHead = e;
							else
								hiTail.next = e;
							hiTail = e;
						}
					} while ((e = next) != null);
					if (loTail != null) {
						loTail.next = null;
						newTab[j] = loHead;
					}
					if (hiTail != null) {
						hiTail.next = null;
						newTab[j + oldCap] = hiHead;
					}
				}
			}
		}
	}
	return newTab;
}
```

## 二.线程安全的对象池

HashMap：线程不安全 

HashTable：线程安全，整个put方法加锁

CurrentHashMap：线程安全，给table数组[索引]第一个元素加锁
