# ✅ 一、基本使用与优劣对比
## 什么是 MMKV？它解决了什么问题？

MMKV（Memory-Mapped Key-Value） 是一个使用 内存映射文件（mmap） 实现的持久化 key-value 存储框架。它的设计目标是替代 Android 的 SharedPreferences，提供更高性能、线程安全、支持多进程访问的数据存储方案

### MMKV基本用法

MMKV支持基本类型、`String`、`byte[]`、`Set<String>`、`Parcelable`对象  

```java
MMKV kv = MMKV.defaultMMKV(); // 默认实例（单进程）
MMKV multiProcessKV = MMKV.mmkvWithID("multi_process_id", MMKV.MULTI_PROCESS_MODE); // 多进程实例
MMKV encryptedKV = MMKV.mmkvWithID("encrypted_id", "MyCryptKey"); // 加密实例

// 存数据
kv.encode("bool_key", true);
kv.encode("int_key", 100);
kv.encode("string_key", "Hello MMKV");
kv.encode("float_key", 3.14f);

List<String> list = Arrays.asList("A", "B", "C");
kv.encode("list_key", new Gson().toJson(list)); // 推荐 Gson 转换

// 读数据
boolean b = kv.decodeBool("bool_key", false); // 第二个参数为默认值
int i = kv.decodeInt("int_key");
String s = kv.decodeString("string_key");
List<String> list = new Gson().fromJson(kv.decodeString("list_key"), List.class);

// 删除
kv.removeValueForKey("key_to_delete"); // 删除单个
kv.removeValuesForKeys(new String[]{"key1", "key2"}); // 删除多个
kv.clearAll(); // 清空所有数据

// 多进程
MMKV kv = MMKV.mmkvWithID("inter_process", MMKV.MULTI_PROCESS_MODE);
kv.encode("pid", android.os.Process.myPid()); // 进程间实时同步

// 迁移旧 SP
SharedPreferences sp = getSharedPreferences("legacy", MODE_PRIVATE);
kv.importFromSharedPreferences(sp);
sp.edit().clear().apply();
```

## MMKV 和 SharedPreferences 有哪些异同？

| 维度          | SharedPreferences                          | MMKV                                      |
|---------------|-------------------------------------------|------------------------------------------|
| 实现原理      | 基于 XML 文件 + Java I/O                   | mmap 内存映射 + 序列化协议（ProtoBuf）    |
| 写操作性能    | 差（全量重写 XML，阻塞线程）               | 极快（内存追加 + 异步刷盘）               |
| ANR 风险      | 高（commit() 同步阻塞 UI 线程）            | 零 ANR（写操作 < 1ms）                    |
| 多进程支持    | 残缺（MODE_MULTI_PROCESS 不可靠）          | 原生支持（文件锁 + 进程通信）              |
| 数据安全      | 明文存储                                  | 支持 AES 加密                             |
| 崩溃恢复      | 易丢失数据（文件替换中途崩溃）             | CRC校验 + 异常恢复机制                    |
| 存储空间      | XML 冗余标签导致体积膨胀（≈实际数据 3 倍） | 二进制紧凑格式（体积小 50%+）              |

## MMKV 的使用场景有哪些？什么时候不推荐用它？

适合场景  
- 高频读写配置项：相比SharedPerferences，不容易ANR  
- 多进程数据共享：MMKV提供了多进程支撑，SP多进程存在同步问题，容易冲突  
- 敏感数据加密：MMKV自带AES加密  
- 实时状态同步：毫秒级同步，适用于全局播放、下载状态同步  

不适合场景  
- 大文件读写，>5MB文件 建议用数据库  
- 单个文件存在较多键值对，比如>10000条，首次启动容易ANR，建议分页存储，单个文件不要存太多KV  
- 单个KV较大，如value>100kb, IO性能显著下降

# ✅ 二、核心原理与实现机制  

## MMKV 为什么性能比 SharedPreferences 好？具体体现在什么方面？

1. mmap内存映射：颠覆磁盘IO模式  

SharedPreferences：Java堆内存通过FileOutputStream写磁盘，从用户空间到内核缓冲区再到磁盘，经历4次拷贝，系统调用write方法存在线程阻塞风险  

MMKV：使用虚拟内存直接映射磁盘文件，用户空间直通磁盘，利用缺页中断、操作系统自动回写脏页


2. Protobuf编码  

相比SP的XML文件，Protobuf使用Tag-Length-Value结构，编码更紧凑，文件更小，读写更快  

3. 增量更新模式  

SP读文件需要全量读取文件，写入出现崩溃容易导致文件毁坏  

MMKV定位文件尾，仅追加delta数据块，CRC校验恢复

4. 零序列化  

SP每次commit会导致全量序列化，MMKV使用了C++，可以绕开Java编码转换、绕开Java序列化，直接使用字节码操作，使用Critical模式锁定内存，避免JNI开销  

## MMKV 底层是如何存储数据的？是怎么做到持久化的？

MMKV 的持久化机制核心通过 ​​mmap 内存映射 + Protobuf 编码 + 增量追加更新​​ 三项技术协同实现  

## MMKV 是如何实现多进程读写的？是否有坑？

MMKV 通过 mmap (memory-mapped file) 技术，让多个进程可以共享同一份内存区域，从而实现多进程数据共享  

MMKV 不是完全依赖 mmap，还通过文件锁和 CRC 校验来保证多进程写入时的数据一致性和安全性  

虽然多个进程可以共享同一个MMKV实例，但如果多个进程同时写入，仍可能存在数据竞争或最后写覆盖前者的问题  

- 避免多个进程同时对同一键值对写入  
- 尽量将写操作集中在主进程（比如AIDL转发到主进程），其他进程只读  

## MMKV 的写入是线程安全的吗？底层怎么保证的？

MMKV 的写入操作是​​线程安全​​的，其底层通过 ​​C++ 原子操作 + 文件锁 + 内存屏障​​三层机制确保并发安全  

# ✅ 三、实际开发与性能问题

## MMKV 是否适合频繁写入？为什么？

## MMKV 中 putObject 是怎么实现的？它可以存储任意对象吗？

➤ 考察：是否知道需要对象实现 Parcelable，以及内部的 writeValue 与 Parcel 的机制。

## MMKV 存储文件丢失或损坏了怎么办？如何避免数据损坏？

➤ 考察：是否知道 MMKV 的 CRC 校验机制和 onMMKVCRCCheckFail 回调。

## 你在使用 MMKV 时遇到过什么问题？是怎么解决的？

➤ 考察：真实项目中的实践能力、排查与解决问题的经验。

# ✅ 四、源码与高级理解
## 你了解 MMKV 的 mmap 映射机制吗？为什么用 mmap 而不是传统 IO？

➤ 考察：是否理解 mmap 带来的性能优势（直接映射内存）、代价（内存占用、映射失败问题等）。

## MMKV 使用 Protobuf 做序列化，有什么优势？为什么不用 JSON？

➤ 考察：了解 Protobuf 的空间/性能优势、跨平台特性等。

## MMKV 的数据结构格式是什么样的？它是如何进行增量更新的？

➤ 考察：是否理解其 key-value 编码格式、读写流程。

## 有没有看过 MMKV 的源码？你觉得哪些地方设计得特别好？

➤ 考察：是否动手深入过源码，能否表达对优秀设计的欣赏与理解。

# ✅ 五、拓展性与对比思维
## 除了 MMKV，你还了解哪些轻量级本地存储方案？它们的优劣如何？

➤ 考察：对比 DataStore、Room、SharedPreferences、Jetpack Security 等的掌握程度。

## 如果 MMKV 数据突然无法读取，你会怎么排查？

➤ 考察：问题定位能力，如是否会检查 CRC 校验失败、权限、磁盘损坏等。

## MMKV 的数据是加密的吗？如何实现加密？安全吗？

➤ 考察：是否知道 cryptKey 的作用，是否理解对称加密的实现及安全性评估。