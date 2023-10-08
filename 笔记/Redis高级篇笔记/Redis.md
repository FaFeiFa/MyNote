# Redis数据类型

常见的有五种：**String（字符串），Hash（哈希），List（列表），Set（集合）、Zset（有序集合）**。 **BitMap（2.2 版新增）、HyperLogLog（2.8 版新增）、GEO（3.2 版新增）、Stream（5.0 版新增）**。

## RedisObject

```c
typedef struct redisObject {
    unsigned type:4;
    unsigned encoding:4;
    unsigned lru:LRU_BITS; /* LRU time (relative to global lru_clock) or
                            * LFU data (least significant 8 bits frequency
                            * and most significant 16 bits access time). */
    int refcount;
    void *ptr;
} robj;
```

结构体RedisObject定义了5个属性：type、enconding、lru、refcount和*prt

**type**：当前值对象的数据类型

**encoding**：当前值对象底层编码类型

**lru**：储存最后一次使用此对象的时间等信息

**refcount**：记录对象引用次数

***prt**：指向真正底层数据结构的指针

---

1、**type**属性

type主要存储当前value对象的数据类型，如下：

REDIS_STRING 字符串对象

REDIS_LIST 列表对象

REDIS_HAST 哈希对象

REDIS_SET 集合对象

REDIS_ZSET 有序集合对象

2、**encoding**属性

简单说encoding存储当前值对象底层编码的实现方式。不同type对象对应不同的编码，下表可以看出上述五种对象中每种对象至少对应了两种编码。

3、**lru**属性

lru记录此对象最后一次访问的时间。

当redis内存回收算法设置为volatile-lru或者allkeys-lru时候redis会优先释放最久没有被访问的数据。

4、**refcount**属性

用于共享计数，类似于jvm的引用计数垃圾回收算法，当refcount为0时，表示没有其它对象引用，可以进行释放此对象。

5、**ptr**指针属性

ptr 指针是指向对象的底层实现数据结构。

## String

**底层实现**：int 和 SDS（简单动态字符串）

SDS 和我们认识的 C 字符串不太一样，之所以没有使用 C 语言的字符串表示，因为 SDS 相比于 C 的原生字符串：

- **SDS 不仅可以保存文本数据，还可以保存二进制数据**。因为 `SDS` 使用 `len` 属性的值而不是空字符来判断字符串是否结束，并且 SDS 的所有 API 都会以处理二进制的方式来处理 SDS 存放在 `buf[]` 数组里的数据。所以 SDS 不光能存放文本数据，而且能保存图片、音频、视频、压缩文件这样的二进制数据。
- **SDS 获取字符串长度的时间复杂度是 O(1)**。因为 C 语言的字符串并不记录自身长度，所以获取长度的复杂度为 O(n)；而 SDS 结构里用 `len` 属性记录了字符串长度，所以复杂度为 `O(1)`。
- **Redis 的 SDS API 是安全的，拼接字符串不会造成缓冲区溢出**。因为 SDS 在拼接字符串之前会检查 SDS 空间是否满足要求，如果空间不够会自动扩容，所以不会导致缓冲区溢出的问题。

**内部编码**有 3 种 ：**int、raw和 embstr**。

**int**

如果一个字符串对象保存的是整数值，并且这个整数值可以用`long`类型来表示，那么字符串对象会将整数值保存在字符串对象结构的`ptr`属性里面（将`void*`转换成 long），并将字符串对象的编码设置为`int`。

**embstr**

如果字符串对象保存的是一个字符串，并且这个字符申的长度小于等于 44字节（redis 5.+版本），那么字符串对象将使用一个简单动态字符串（SDS）来保存这个字符串，并将对象的编码设置为`embstr`， `embstr`编码是专门用于保存短字符串的一种优化编码方式。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B/embstr.png)

**raw**

如果字符串对象保存的是一个字符串，并且这个字符串的长度大于 44字节（redis 5.+版本），那么字符串对象将使用一个简单动态字符串（SDS）来保存这个字符串，并将对象的编码设置为`raw`。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B/raw.png)

`embstr`和`raw`编码都会使用`SDS`来保存值

使用**embstr**的**优点**：

- `embstr`编码将创建字符串对象所需的内存分配次数从 `raw` 编码的两次降低为一次；
- 释放 `embstr`编码的字符串对象同样只需要调用一次内存释放函数；
- 因为`embstr`编码的字符串对象的所有数据都保存在一块连续的内存里面可以更好的利用 CPU 缓存提升性能。

**缺点**：

- 如果字符串的长度增加需要重新分配内存时，整个redisObject和sds都需要重新分配空间，所以**embstr编码的字符串对象实际上是只读的**，redis没有为embstr编码的字符串对象编写任何相应的修改程序。当我们对embstr编码的字符串对象执行任何修改命令（例如append）时，程序会先将对象的编码从embstr转换成raw，然后再执行修改命令。

## List

**底层实现**：quicklist，实际还是双向链表和紧凑列表

quicklist内部存储结构:

quicklist中每一个节点都是一个quicklistNode对象，然后各个quicklistNode就构成了一个列表quicklist：



```c
typedef struct quicklist {
    quicklistNode *head;
    quicklistNode *tail;
    unsigned long count;        /* total count of all entries in all listpacks */
    unsigned long len;          /* number of quicklistNodes */
    signed int fill : QL_FILL_BITS;       /* fill factor for individual nodes */
    unsigned int compress : QL_COMP_BITS; /* depth of end nodes not to compress;0=off */
    unsigned int bookmark_count: QL_BM_BITS;
    quicklistBookmark bookmarks[];
} quicklist;
```

```c
typedef struct quicklistNode {
    struct quicklistNode *prev;          /* 指向前一个节点的指针 */
    struct quicklistNode *next;          /* 指向下一个节点的指针 */
    unsigned char *entry;               /* 存储元素数据的指针 */
    size_t sz;                          /* entry的大小（以字节为单位） */
    unsigned int count : 16;            /* listpack中的元素数量（占16位） */
    unsigned int encoding : 2;          /* 元素编码方式（RAW==1或LZF==2） */
    unsigned int container : 2;         /* 元素容器类型（PLAIN==1或PACKED==2） */
    unsigned int recompress : 1;        /* 前一个节点是否被压缩过 */
    unsigned int attempted_compress : 1; /* 是否尝试压缩这个节点，但失败了（太小） */
    unsigned int dont_compress : 1;     /* 防止压缩将来要使用的entry */
    unsigned int extra : 9;             /* 用于未来用途的额外位 */
} quicklistNode;

```

大致原理

![原理](https://img-blog.csdnimg.cn/20210507113223880.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzQ1NDA2MDky,size_16,color_FFFFFF,t_70)

**控制zipliat元素长度**

quicklist结合了双向链表和ziplist的优点，但是同样也存在一个问题，一个quicklist包含多长的ziplist合适呢？需要找到一个平衡点：

- ziplist太短，内存碎片越多。
- ziplist太长，分配大块连续内存空间的难度就越大。

如何保持ziplist的合理长度，取决于具体的应用场景。

我们可以通过`list-max-ziplist-size`参数来控制ziplist的长度，基于2种原则，即元素的长度或元素大小的总和。

语法例如：

```java
list-max-ziplist-size -2  '默认值'
```

当取正值的时候，表示按照数据项个数来限定每个quicklist节点上的ziplist长度。比如，当这个参数配置成5的时候，表示每个quicklist节点的ziplist最多包含5个数据项。

当取负值的时候，表示按照占用字节数来限定每个quicklist节点上的ziplist长度。这时，它只能取-1到-5这五个值，每个值含义如下：

```java
-5: 每个quicklist节点上的ziplist大小不能超过64 Kb。（注：1kb => 1024 bytes）
-4: 每个quicklist节点上的ziplist大小不能超过32 Kb。
-3: 每个quicklist节点上的ziplist大小不能超过16 Kb。
-2: 每个quicklist节点上的ziplist大小不能超过8 Kb。（-2是Redis给出的默认值）
-1: 每个quicklist节点上的ziplist大小不能超过4 Kb。
```

**compress属性**

list设计最容易被访问的是列表两端的数据，中间的访问频率很低，如果符合这个场景，list还有一个配置，可以对中间节点进行压缩（采用的LZF——一种无损压缩算法），进一步节省内存。配置如下：

```java
list-compress-depth 0 
```

compress属性表示压缩深度，这个参数表示一个quicklist两端不被压缩的节点个数。

可以通过参数list-compress-depth控制：

+ 0：不压缩(`默认值`)

- 1：首尾第1个元素不压缩
- 2：首位前2个元素不压缩
- 3：首尾前3个元素不压缩
- 以此类推

可用的场景例如：

1. **消息队列**：在消息队列中，新的消息通常会被添加到队列的尾部，而消费者会从队列的头部取出消息进行处理。这意味着队列的两端是最频繁访问的部分。
2. **缓存**：在缓存中，通常会使用LRU（最近最少使用）或LFU（最少使用）算法来管理缓存中的数据。这些算法通常会将最近使用的数据放在缓存的前面，而不常使用的数据可能会在列表的尾部。因此，缓存的头部通常包含热点数据。
3. **任务队列**：在任务队列中，新的任务通常会添加到队列的尾部，而工作者（消费者）会从队列的头部获取任务进行处理。
4. **历史记录**：在一些应用中，用户的历史操作记录通常会按照时间顺序存储在列表中，新的操作会添加到列表的尾部，而旧的操作可能会从列表的头部移除。















### 压缩列表和紧凑列表的区别，为什么要用紧凑列表替代压缩列表？















































# 分布式锁

SET 命令有个 NX 参数可以实现「key不存在才插入」，可以用它来实现分布式锁：

- 如果 key 不存在，则显示插入成功，可以用来表示加锁成功；
- 如果 key 存在，则会显示插入失败，可以用来表示加锁失败。

一般而言，还会对分布式锁加上过期时间，分布式锁的命令如下：

```shell
SET lock_key unique_value NX PX 10000
```

- lock_key 就是 key 键；
- unique_value 是客户端生成的唯一的标识；
- NX 代表只在 lock_key 不存在时，才对 lock_key 进行设置操作；
- PX 10000 表示设置 lock_key 的过期时间为 10s，这是为了避免客户端发生异常而无法释放锁。

而解锁的过程就是将 lock_key 键删除，但不能乱删，要保证执行操作的客户端就是加锁的客户端。所以，解锁的时候，我们要先判断锁的 unique_value 是否为加锁客户端，是的话，才将 lock_key 键删除。

可以看到，解锁是有两个操作，这时就需要 Lua 脚本来保证解锁的原子性，因为 Redis 在执行 Lua 脚本时，可以以原子性的方式执行，保证了锁释放操作的原子性。

```lua
// 释放锁时，先比较 unique_value 是否相等，避免锁的误释放
if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
else
    return 0
end
```

这样一来，就通过使用 SET 命令和 Lua 脚本在 Redis 单节点上完成了分布式锁的加锁和解锁













