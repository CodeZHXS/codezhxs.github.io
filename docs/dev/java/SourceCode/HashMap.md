## 术语说明

|   术语   |                             说明                             |
| :------: | :----------------------------------------------------------: |
|   节点   |                哈希表中的一对 `<key, value>`                 |
|  哈希值  | 用 `hash(key)` 表示 `key` 的哈希值<br />令`int h = key.hashCode();` <br />则 `hash(key) = h ^ (h >>> 16)` |
|    桶    |               相同哈希值的节点放在同一个桶中。               |
|  桶数组  |            多个桶用数组表示：`Node<K,V>[] table`             |
|  桶下标  | 节点 `<key, value>` 的桶下标为： `key.hashCode() % table.length` |
|   容量   |                        桶数组的长度。                        |
| 表的大小 |                      表中存储的节点数量                      |
| 装填因子 |  固定值，用于计算扩容阈值。<br />扩容阈值 = 装填因子 * 容量  |
| 扩容阈值 |           如果表的大小超过了扩容阈值，就进行扩容。           |
|   扩容   |       扩充桶数组的长度，扩容会导致节点的位置发生改变。       |

## 省流版

-   底层数据结构：**数组 + 链表/红黑树**

-   默认装填因子：$0.75$

-   默认初始容量：$16$

-   链表组织成红黑树的条件：**容量** $\ge 64$ 且 **链表长度** $\ge 8$

-   扩容机制：$2$ 倍扩容，会将一个桶的节点分到两个桶中（证明见 [前置知识](#前置知识) ）

-   扩容条件：

    -   添加元素时表为空。

    -   添加元素后 **链表长度** $\ge 8$ 且 **容量** $< 64$
    -   添加元素后 **表大小** $>$ **装填因子**

-   红黑树退化成链表的条件：在扩容时，红黑树分成 $2$ 个子链，如果 **子链长度** $\le 6$，则退化成链表。否则保持红黑树。

## 字段说明

### 常量字段

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    =implements Map<K,V>, Cloneable, Serializable {
    static final float DEFAULT_LOAD_FACTOR = 0.75f; // 默认装填因子
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // 默认初始容量

    static final int MAXIMUM_CAPACITY = 1 << 30; // 最大容量
    static final int TREEIFY_THRESHOLD = 8; // 链表长度大于 8 时扩容或将链表组织成红黑树
    static final int MIN_TREEIFY_CAPACITY = 64; // 容量小于 64 就扩容，否则组织成红黑树
    static final int UNTREEIFY_THRESHOLD = 6; // 红黑树分裂的子链如果小于等于 6 则退化成链表
}
```

### 变量字段

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    =implements Map<K,V>, Cloneable, Serializable {
    transient Node<K,V>[] table; // 桶，用数组表示
    final float loadFactor; // 装填因子
    int threshold; // 数组未分配时 threshold 表示初始容量，数组第一次分配后，threshold 表示扩容阈值
    int size; // 表大小
}
```

## 源码分析

### 构造器

可以指定的量只有 2 个：初始容量和装载因子。

-    `initialCapacity` ：表示初始容量，如果不指定默认为 $16$
-    `loadFactor` ：表示装填因子，如果不指定默认为 $0.75$

```java
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    // 前面都是参数判断 下面进行赋值
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity); // 初始容量下一个大于等于initialCapacity的 2 的幂
}
```

### 扩容

由上一节可知，添加元素时三种情况会导致扩容：

-   添加元素时表为空。

-   添加元素后 **链表长度** $\ge 8$ 且 **容量** $< 64$
-   添加元素后 **表大小** $>$ **装填因子**

#### 前置知识

`HashMap` 的扩容是 $2$ 倍扩容，假设当前容量为 $C$，对于第 $j$ 个桶，其中的节点满足：
$$
hash(key) \equiv j \bmod C \\
$$
这表明：
$$
hash(key) = j + KC
$$
考虑 $K$ 的奇偶性，不难发现：
$$
\left\{\begin{matrix} 
hash(key) & \equiv& j & 	\bmod 2C, & K \% 2 = 0  \\  
hash(key) & \equiv& j + C & \bmod 2C, & K \% 2 = 1
\end{matrix}\right.
$$
结论：在扩容后，第 $j$ 个桶的节点要么仍然在第 $j$ 个桶，要么在第 $j + C$ 个桶。

在后面的源码中，称第 $j$ 个桶的链为 `lo`，第 $j+C$ 个桶为 `hi`

#### 源码分析

```java
final Node<K,V>[] resize() {
    // oldTab 扩容前桶数组
    // oldCap 扩容前容量
    // oldThr 扩容前装填因子(如果容量为0，则 oldThr 表示初始初始容量)
    // newCap 扩容后容量
    // newThr 扩容后装填因子
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) { // 如果扩容前桶数组非空
        if (oldCap >= MAXIMUM_CAPACITY) { 
            threshold = Integer.MAX_VALUE; // 如果容量达到上限，仅调整装填因子
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // 两倍扩容
    }
    else if (oldThr > 0) // 如果容量为0，则 oldThr 表示初始初始容量
        newCap = oldThr; // 按照初始容量扩容
    else {               // 如果 oldThr 为 0
        newCap = DEFAULT_INITIAL_CAPACITY; // 按照默认初始容量扩容
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY); // 计算装填因子
    }
    // 上面的分支会计算出 newCap 和 newThr
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor; // 计算装填因子
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        // 遍历旧的桶数组
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null; // 设置成 null，e 往后移动之后就会垃圾回收
                if (e.next == null) // 如果这条链只有 1 个元素，直接放到新桶中
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap); // 将红黑树分裂到 2 个桶中，等会展开讲
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    // 遍历这条链的所有节点
                    do {
                        next = e.next;
                        // 第 oldCap 位能检验出 K % C 是 0 还是 1
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e; // 其实就是尾插法
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

正如前置知识中所推倒的过程一样，扩容实际就是将第 $j$ 条个桶的链表拆成 $j$ 和 $j + C$ 的两个桶的链表。而如果第 $j$ 个桶是红黑树，则使用 `split(this, newTab, j, oldCap)` 进行分裂

```java
final void split(HashMap<K,V> map, Node<K,V>[] tab, int index, int bit) {
    TreeNode<K,V> b = this;
    TreeNode<K,V> loHead = null, loTail = null;
    TreeNode<K,V> hiHead = null, hiTail = null;
    int lc = 0, hc = 0; // 树分裂时还需要统计链表长度
    // 遍历红黑树的所有点，
    for (TreeNode<K,V> e = b, next; e != null; e = next) {
        next = (TreeNode<K,V>)e.next;
        e.next = null;
        if ((e.hash & bit) == 0) {
            if ((e.prev = loTail) == null)
                loHead = e;
            else
                loTail.next = e;
            loTail = e;
            ++lc; // 用尾插法分成 2 条链，同时统计长度
        }
        else {
            if ((e.prev = hiTail) == null)
                hiHead = e;
            else
                hiTail.next = e;
            hiTail = e;
            ++hc;
        }
    }

    if (loHead != null) {
        if (lc <= UNTREEIFY_THRESHOLD) // 如果子链的长度小于等于6，退化成链表
            tab[index] = loHead.untreeify(map);
        else { // 否则，这条链仍然保持成红黑树
            tab[index] = loHead;
            if (hiHead != null) // (else is already treeified)
                loHead.treeify(tab);
        }
    }
    if (hiHead != null) {
        if (hc <= UNTREEIFY_THRESHOLD)
            tab[index + bit] = hiHead.untreeify(map);
        else {
            tab[index + bit] = hiHead;
            if (loHead != null)
                hiHead.treeify(tab);
        }
    }
}
```

### 查找元素

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
```

封装在 `getNode()` 中, 代码也比较简单，无非就是找到第几个桶，然后在链表或者红黑树中查找是否有相同的 `key` 就可以了。

```java
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash &&
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

### 删除元素

```java
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}
```

封装在 `removeNode()` 中，查找元素的部分和之前一样，主要关注删除的部分。

```java
final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        // 上面就是查找元素，将待删除的元素赋值给 node
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            if (node instanceof TreeNode) // 如果在红黑树中就用红黑树的删除方法
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            // p 是 node 的前一个节点
            else if (node == p)  // 如果 node 是链表头节点
                tab[index] = node.next;
            else
                p.next = node.next; 
            ++modCount;
            --size; 
            afterNodeRemoval(node); // 回调函数
            return node;
        }
    }
    return null;
}
```

可以看到删除节点也是比较简单的，没有缩容的机制。

