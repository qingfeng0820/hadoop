# java.lang.ThreadLocal
* Each Thread keeps a ThreadLocalMap (key <ThreadLocal>, value).

* ThreadLocalMap relies on ThreadLocal.threadLocalHashCode to hash place the entries.
threadLocalHashCode由HASH_INCREMENT确定
HASH_INCREMENT是一个常量
private static final int HASH_INCREMENT = 0x61c88647;
总之呢，第一个threadLocalHashCode值求出来就是0，然后第二个是0+HASH_INCREMENT，第三个0+HASH_INCREMENT+HASH_INCREMENT…
这个数(斐波那契散列)很有魔力，这个数可以使得每次求得的hash值放入entry数组中分布均匀。

* How ThreadLocalMap solve hash conflict: linear-probe (线性探测法/开放地址法).
If hash conflict for ThreadLocal.threadLocalHashCode, loop the entry from ThreadLocal.threadLocalHashCode and find one free slot to save this ThreadLocal.

* The Entry in ThreadLocalMap is a WeakReference of ThreadLocal, if ThreadLocal is no strong referent, it will be gc for next turn.


# io.netty.util.concurrent.FastThreadLocal
* FastThreadLocal is stored in InternalThreadLocalMap.
* io.netty.util.concurrent.FastThreadLocalThread keeps a InternalThreadLocalMap (key <FastThreadLocal>, value) to store FastThreadLocal.
* For Thread,  FastThreadLocal will create a ThreadLocal<InternalThreadLocalMap> -> UnpaddedInternalThreadLocalMap.slowThreadLocalMap to store the FastThreadLocal.

* InternalThreadLocalMap stores entries in an indexed variable array sequentially.
使用数组中的下标来代替用hash方法查找元素，对比hash方法有略微的优势，适用于经常访问的情况

* InternalThreadLocalMap expanding:
在指定下标index处存放数据，如果原先已经有数据，返回 FALSE，否则说明是第一次存放，返回 TRUE。 当index越界时，以大于index的最小 2 的 N 次幂扩容。

```
因为初始容量是 32，实际上就是翻倍扩容。
由于index的全局唯一性，导致在同一线程中的FastThreadLocal实例 ID 不一定连续，因此index越界不代表数组就没有空间了，只是这些空间不能被使用。
// round up power of 2 （最小 2 的 n 次幂扩容，并在 index 处存放数据）
private void expandIndexedVariableTableAndSet(int index, Object value) {
    Object[] oldArray = indexedVariables;
    final int oldCapacity = oldArray.length;
    // 计算最小 2 的 n 次幂
    int newCapacity = index;
    newCapacity |= newCapacity >>>  1;
    newCapacity |= newCapacity >>>  2;
    newCapacity |= newCapacity >>>  4;
    newCapacity |= newCapacity >>>  8;
    newCapacity |= newCapacity >>> 16;
    newCapacity ++;
    // 扩容并拷贝原数据
    Object[] newArray = Arrays.copyOf(oldArray, newCapacity);
    // 无数据部分填充占位符 UNSET
    Arrays.fill(newArray, oldCapacity, newArray.length, UNSET);
    // 存放本次数据
    newArray[index] = value;
    // 设置新数组
    indexedVariables = newArray;
}
```
* 当新建了FastThreadLocal实例并赋值 （用了一个新的index），会把这个FastThreadLocal实例记录在InternalThreadLocalMap的variablesToRemoveIndex （一般是0）位置上的remove variables set里面
* 当FastThreadLocal remove的时候，也会从remove variables set里面删除它