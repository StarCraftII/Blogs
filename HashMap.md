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
			// 当前tab下标存在数据（可能是单个节点、链表、红黑树）
			if ((e = oldTab[j]) != null) {
				// 置空tab下标指针
				oldTab[j] = null;
				// 如果e没有后继节点，直接把e放到新tab的对应下标位置
				if (e.next == null)
					newTab[e.hash & (newCap - 1)] = e;
				// 如果e后面是一个树
				else if (e instanceof TreeNode)
					((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
				// 如果e后面是一个链表
				else { // preserve order	
					// 由于是扩容了一倍的大小，因此向左多计算一位hash
					// 且e后面的元素所在新tab上的位置只能是 [j] 或者 [j + oldCap]
					// 假如oldCap大小是16 j是在13这个位置，
                    // hash-> ... 1 1101
                    // hash-> ... 0 1101
					// 低位链表头和低位链表尾
					Node<K,V> loHead = null, loTail = null;
					// 高位链表头和高位链表尾
					Node<K,V> hiHead = null, hiTail = null;
					// 下一个节点
					Node<K,V> next;
					do {
						next = e.next;
						// 如果高位是0，插入低位链表
						if ((e.hash & oldCap) == 0) {
							// 如果低位链表尾是空的，直接把e插入低位链表头
							if (loTail == null)
								loHead = e;
							else
								// 把低位链表尾的下一个节点指向e
								loTail.next = e;
							// 更新e为新的低位链表尾
							loTail = e;
						}
						// 如果高位是1，插入高位链表
						else {
							// 如果高位链表尾是空的，直接把e插入高位链表头
							if (hiTail == null)
								hiHead = e;
							else
								// 把高位链表尾的下一个节点指向e
								hiTail.next = e;
							// 更新e为新的高位链表尾
							hiTail = e;
						}
					} while ((e = next) != null);
					// 如果低位链表不为空
					if (loTail != null) {
						loTail.next = null;
						// 把低位链表插入到新tab的[j]位置处
						newTab[j] = loHead;
					}
					if (hiTail != null) {
						hiTail.next = null;
						// 把高位链表插入到新tab的[j + oldCap]位置处
						newTab[j + oldCap] = hiHead;
					}
				}
			}
		}
	}
	// 返回新的tab表
	return newTab;
}
```

### 2.8 get方法

```
/**
  * Returns the value to which the specified key is mapped,
  * or {@code null} if this map contains no mapping for the key.
  *
  * 返回指定key映射的值或者这个map不包含这个key的映射则返回null
  *
  * <p>More formally, if this map contains a mapping from a key
  * {@code k} to a value {@code v} such that {@code (key==null ? k==null :
  * key.equals(k))}, then this method returns {@code v}; otherwise
  * it returns {@code null}.  (There can be at most one such mapping.)
  *
  * 更正的说，如果此map包含从键{@code k}到值{@code v}的映射,使得
  * {@code（key == null？k == null：key.equals（k） ）}，则此方法返回
  * {@code v}; 否则返回{@code null}。 （最多可以有一个这样的映射。）
  *
  * <p>A return value of {@code null} does not <i>necessarily</i>
  * indicate that the map contains no mapping for the key; it's also
  * possible that the map explicitly maps the key to {@code null}.
  * The {@link #containsKey containsKey} operation may be used to
  * distinguish these two cases.
  *
  * 返回值 {@code null} 并不必然表示该map不包含该键的映射； 这也是该映
  * 射可能将键显式映射到 {@code null}。{@link #containsKey containsKey} 
  * 操作可用于区分这两种情况。
  *
  * @see #put(Object, Object)
  */
public V get(Object key) {
	Node<K,V> e;
	return (e = getNode(hash(key), key)) == null ? null : e.value;
}
```

### 2.9 getNode方法

```
/**
  * Implements Map.get and related methods
  *
  * 实现 Map.get 和相关方法
  *
  * @param hash hash for key key的hash值（平衡后的hash值）
  * @param key the key 要查找的key
  * @return the node, or null if none 返回节点或null
  */
final Node<K,V> getNode(int hash, Object key) {
	// 当前tab
	// first：tab某索引处的头结点
	// e：临时节点
	// n：tab的长度
	// k：临时key
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
	// 如果tab不为空并且tab的长度大于0，hash命中索引处的首元素不为空
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
		// 首元素的hash值与要查找的hash值一样并且它们的key值要么地址相同要么值相同
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
			// 返回首元素
            return first;
		// 如果首元素有后继节点，把后继节点赋值给e
        if ((e = first.next) != null) {
			// 首元素是树节点
            if (first instanceof TreeNode)
				// 返回树查找返回的节点
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
			// 首元素是链表节点，不断向后查找
            do {
				// e的hash值与要查找的hash值一样并且它们的key值要么地址相同要么值相同，返回e
				// e未匹配上，如果e有后继节点，把e的后继节赋值给e
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

### 2.10 remove方法

删除指定key对应的节点

```
/**
  * Removes the mapping for the specified key from this map if present.
  *
  * 删除此map中指定key的映射（如果存在）。
  *
  * @param  key key whose mapping is to be removed from the map
  *
  *	@param  key 要从map中删除其映射的key
  *
  * @return the previous value associated with <tt>key</tt>, or
  *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
  *         (A <tt>null</tt> return can also indicate that the map
  *         previously associated <tt>null</tt> with <tt>key</tt>.)
  *
  * @return 返回与key关联的前一个值，或者null(没有关于key的映射),
  *         返回null也可能是这个key先前显示关联了null
  *
  */
public V remove(Object key) {
    Node<K,V> e;
	// 未找到要删除的节点返回null，找到要删除的节点返回节点的value值
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}

/**
  * 当key与value都满足条件才能删除
  */
@Override
public boolean remove(Object key, Object value) {
    return removeNode(hash(key), key, value, true, true) != null;
}
```

### 2.11 removeNode方法

```
/**
  * 删除指定节点
  *
  * @param hash hash for key 
  * Implements Map.remove and related methods
  *
  * 实现 Map.remove 和相关方法
  *
  * @param key the key
  * @param value the value to match if matchValue, else ignored
  * @param value 如果设置matchValue就匹配value值，否则忽略
  * @param matchValue if true only remove if value is equal
  * @param matchValue 如果为true只删除value值相等的
  * @param movable if false do not move other nodes while removing
  * @param movable 如果为false在删除过程中不能移动其他节点
  * @return the node, or null if none
  * @return 返回删除节点或null
  */
final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
	// 当前tab
	// p：当前元素节点
	// n：tab的长度
	// index：命中的tab下标索引
    Node<K,V>[] tab; Node<K,V> p; int n, index;
	// 如果tab不为空并且tab的长度大于0，hash命中索引处的首元素不为空
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
		// node为查找到的结果，e为临时节点
		// k表示key,v表示value
        Node<K,V> node = null, e; K k; V v;
		// 首元素的hash值与要查找的hash值一样并且它们的key值要么地址相同要么值相同
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
			// 把首元素赋值给node
            node = p;
		// 如果首元素有后继节点，把后继节点赋值给e
        else if ((e = p.next) != null) {
			// 首元素是树节点
            if (p instanceof TreeNode)
				// 把树查找返回的节点赋值给node
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
				// 首元素是链表节点，不断向后查找
                do {
					// e的hash值与要查找的hash值一样并且它们的key值要么地址相同要么值相同，把e赋值给node
					// e未匹配上，如果e有后继节点，把e的后继节赋值给e
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
					// 把当前节点e赋值给p
                    p = e;
				// 把e的下一个节点赋值给e
                } while ((e = e.next) != null);
            }
        }
		
		// 如果node不为空，说明根据key已经匹配到合适的Node，再根据matchValue判断是否需要根据value去校验
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
			// 在树结构里删除node节点
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
			// 如果node是tab命中索引处的头节点，那么让tab命中索引指向node的后继节点
            else if (node == p)
                tab[index] = node.next;
            else
				// 如果node不是tab命中索引处的头节点，让p的后继节点指向node的后继节点，把node从链表中逻辑删除
                p.next = node.next;
			// 表示散列表结构的修改次数。增删次数
            ++modCount;
			// 节点总数减一
            --size;
			// Callbacks to allow LinkedHashMap post-actions
			// 允许LinkedHashMap事后操作的回调
            afterNodeRemoval(node);
			// 返回node节点
            return node;
        }
    }
    return null;
}
```

### 2.12 replace方法

```
@Override
public boolean replace(K key, V oldValue, V newValue) {
	// 临时节点e，临时Value v
    Node<K,V> e; V v;
	// 根据key找到对应节点赋值给e，如果e节点的value与oldValue地址相同或者值相等
    if ((e = getNode(hash(key), key)) != null &&
        ((v = e.value) == oldValue || (v != null && v.equals(oldValue)))) {
		// 把e的Value设置为newValue
        e.value = newValue;
		// Callbacks to allow LinkedHashMap post-actions
		// 允许LinkedHashMap事后操作的回调
        afterNodeAccess(e);
		// 返回替换成功
        return true;
    }
	// 返回替换失败
    return false;
}

@Override
public V replace(K key, V value) {
	// 临时节点e
    Node<K,V> e;
	// 根据key找到对应节点赋值给e，如果e不为空
    if ((e = getNode(hash(key), key)) != null) {
		// 先取出e节点旧的value
        V oldValue = e.value;
		// 把e节点的Value设置为新的value
        e.value = value;
		// Callbacks to allow LinkedHashMap post-actions
		// 允许LinkedHashMap事后操作的回调
        afterNodeAccess(e);
		// 返回旧的value值
        return oldValue;
    }
	// 返回null
    return null;
}
```

### 2.13 clear方法

```
/**
  * Removes all of the mappings from this map.
  *
  * 删除这个map里所有的映射关系
  *
  * The map will be empty after this call returns.
  * 
  * 此调用返回后map将为空
  */
public void clear() {
	// 散列表
    Node<K,V>[] tab;
	// 表示散列表结构的修改次数。增删次数
    modCount++;
	// 如果tab不为空并且size大于0
    if ((tab = table) != null && size > 0) {
		// size设置为0
        size = 0;
		// 遍历tab数组，置空每一项
        for (int i = 0; i < tab.length; ++i)
            tab[i] = null;
    }
}
```

## 二.线程安全的对象池

HashMap：线程不安全 

HashTable：线程安全，整个put方法加锁

CurrentHashMap：线程安全，给table数组[索引]第一个元素加锁
