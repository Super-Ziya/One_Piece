### ThreadLocal

> ThreadLocal 对象可提供线程局部变量，每个线程拥有一份自己的副本变量，多个线程互不干扰

- 结构
  - Thread 类有个 ThreadLocal.ThreadLocalMap 的实例变量 threadLocals，每个线程有一个自己的ThreadLocalMap
  - ThreadLocalMap 的 key 为 ThreadLocal，value 为代码中放入的值（key 是 ThreadLocal 的弱引用，GC 后可能被回收造成内存泄露）
  - 每个线程往 ThreadLocal 里放值时，会往自己的 ThreadLocalMap 里存，读也是以 ThreadLocal 为引用，在自己的 map 里找对应 key，实现线程隔离
  - HashMap 是由数组+链表实现，ThreadLocalMap 没有链表结构
  - Entry 的 key 是 ThreadLocal<?> k ，继承自 WeakReference（弱引用类型）

<img src="图片.assets\ThreadLocal_数据结构.jpg" alt="ThreadLocal_数据结构" style="zoom:80%;" />

- ThreadLocal.set()

  ![ThreadLocal_set](图片.assets\ThreadLocal_set.jpg)

  - ThreadLocalMap.set()

    - hash 计算后槽位对应 Entry 数据为空，直接插入

    - 槽位数据不为空，key 值与当前 ThreadLocal 通过 hash 计算的 key 值一致，直接更新

    - 槽位数据不为空，往后遍历，找到 Entry 为 null 的槽位前，没有遇到 key 过期的 Entry，继续遍历散列数组，找到 Entry 为 null 的槽位将数据放入该槽位中，或遇到 key  值相等的数据直接更新

    - 槽位数据不为空，往后遍历，找到 Entry 为 null 的槽位前，遇到 key 过期的 Entry（key 为 null）

      > 执行 `replaceStaleEntry()`，替换过期数据，以过期 Entry 为起点开始向前迭代查找，找其他过期的数据，然后更新过期数据起始扫描下标slotToExpunge。for循环迭代，直到碰到Entry为null结束

      ![ThreadLocal_过期数据替换1](图片.assets\ThreadLocal_过期数据替换1.jpg)

      ![ThreadLocal_过期数据替换2](图片.assets\ThreadLocal_过期数据替换2.jpg)

      ![ThreadLocal_过期数据替换3](图片.assets\ThreadLocal_过期数据替换3.jpg)

      ![ThreadLocal_过期数据替换4](图片.assets\ThreadLocal_过期数据替换4.jpg)

- ThreadLocalMap Hash 算法

  ```java
  int i = key.threadLocalHashCode & (len-1);	//i是当前key在散列表中对应数组下标
  
  public class ThreadLocal<T> {
      private final int threadLocalHashCode = nextHashCode();
      private static AtomicInteger nextHashCode = new AtomicInteger();
      private static final int HASH_INCREMENT = 0x61c88647;	//斐波那契数/黄金分割数，使hash分布均匀
      private static int nextHashCode() {
          return nextHashCode.getAndAdd(HASH_INCREMENT);	//每创建一个ThreadLocal对象，就增长0x61c88647
      }
      static class ThreadLocalMap {
          ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
              table = new Entry[INITIAL_CAPACITY];
              int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
              table[i] = new Entry(firstKey, firstValue);
              size = 1;
              setThreshold(INITIAL_CAPACITY);
          }
      }
  }
  ```

- ThreadLocalMap Hash 冲突

  - 线性向后查找到 Entry 为 null 的槽才停止，将当前元素放入此槽中

  ![ThreadLocal_hash冲突](图片.assets\ThreadLocal_hash冲突.jpg)

  