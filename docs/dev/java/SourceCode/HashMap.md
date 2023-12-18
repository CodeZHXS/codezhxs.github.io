## 字段说明

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    =implements Map<K,V>, Cloneable, Serializable {
    transient Node<K,V>[] table; // 桶，用数组表示
    final float loadFactor; // 装填因子
    int threshold; // 初始的桶数组容量。数组第一次分配后，这个变量表示下一次扩容的阈值
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
    this.threshold = tableSizeFor(initialCapacity); // 初始容量下一个大于等于initialCapacity的2的幂
}
```

其中装填因子是直接赋值，初始容量会经过 `tableSizeFor()` 方法修改为下一个 $2$ 的幂的值，具体来说：

-   如果 `initialCapacity < 0` ，那么初始容量设置成 $1$
-   如果 `initialCapacity >= MAXIMUM_CAPACITY` ，其中`MAXIMUM_CAPACITY` 是一个常数 $2^{30}$, 那么初始容量设置成 $2 ^ {30}$
-   其他情况，初始容量设置为下一个大于等于 `initialCapacity` 的2的幂

### 扩容

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
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

