# 1 HashMap 基本原理  

HashMap 通过“哈希散列 + 数组 + 链表 + 红黑树”实现键值对的存储和查找  

```java
// HashMap 的核心数据结构
transient Node<K,V>[] table; // 数组 + 链表/红黑树的组合
```

Node<K, V> 是哈希表中的一个元素，包含：键、值、hash 值、next指针  

从 Java 8 开始，当链表长度大于一定阈值（默认为 8），会转化为红黑树以提升查询效率  

红黑树（Red-Black Tree）是一种自平衡的二叉搜索树（BST），它通过在插入和删除时对节点进行颜色标记和旋转操作，保持树的高度平衡，从而保证最坏情况下的查询效率为 O(log n)  

# 2 ConcurrentHashMap如何实现线程安全  

ConcurrentHashMap 通过 分段锁机制（JDK7） 或 CAS+锁分离（JDK8） 来实现线程安全，避免全表加锁带来的性能瓶颈    

```java
transient volatile Node<K,V>[] table;
```

## 读操作 

```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e; int n; K k;
    // 通过 hash 定位桶，然后链表遍历查找
}
```

table 是 volatile 修饰的，保证可见性

多线程读取不加锁，因为数据结构不会被破坏

查找通过链表或红黑树，效率高


## 写操作:CAS + synchronized  

写入流程（简化版）：
计算 hash 值

判断 table 是否初始化，懒加载

定位数组槽位（桶）

使用 CAS 尝试插入第一个节点（空桶）

如果 CAS 失败，说明已有元素：

使用 synchronized 锁定该桶的头节点

遍历链表/红黑树插入或替换节点

若链表长度超过阈值（8），转为红黑树

若当前负载超标，触发 扩容

