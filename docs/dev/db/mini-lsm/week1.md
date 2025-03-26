## Week 1

### Day 1 Memtable

-   Why doesn't the memtable provide a `delete` API? （为什么 memtable 不提供 `delete` 的 API？）

    答：因为删除实质是逻辑删除。当我们查找一个 key 的时候，首先查 memtable，如果 memtable 没有，就查 Immutable 中（持久化的）memtable，直到找到对应的 key 为止。因此删除的逻辑并不是在 memtable 中把这个 key 删掉，而是把对应的 value 置空。所以提供 delete 的 API 没有意义

-   Does it make sense for the memtable to store all write operations instead of only the latest version of a key? For example, the user puts a->1, a->2, and a->3 into the same memtable.（在 memtable 中存储所有的写操作而不是最新的 key 是否有意义？例如用户在同一个 memtable 中执行了 puts a->1, a->2, and a->3）

    答：有意义，留下所有的值就可以 MVCC 了。

-   Is it possible to use other data structures as the memtable in LSM? What are the pros/cons of using the skiplist?（可以用其他数据结构构建 LSM 的 memtable 吗？和跳表相比优点和缺点是什么？）

    可以用红黑树、哈希表等，哈希表不支持范围查询，红黑树并发性能不好。而跳表支持无锁并发，多线程性能好。不过，跳表并发读的性能不如红黑树。

-   Why do we need a combination of `state` and `state_lock`? Can we only use `state.read()` and `state.write()`? （为什么我们需要 `state` 和 `state_lock`？可以只用 `state.read()` 和 `state.write()` 吗？）

    答：不可以。关键就在于创建一个带有 WAL 的 memtable 是需要耗时的（因为这涉及到 IO）。假设我们只用 `state.read()` and `state.write()`，梳理一下此时 `state` 的 `put` 和 `delete` 操作流程：

    1.   `put` 键值，增加表大小
    2.   判断表大小是否达到容量限制，如果未达到，返回。
    3.   获取 `state.write()` 写锁
    4.   创建一个新的带有 WAL 的 空 memtable（耗时操作）
    5.   旧 memtable 写入 immutable，替换新的 memtable 
    6.   释放 `state.write()` 写锁

    可以发现在 4 和 5 执行期间其他线程是完全不能访问的，而第 4 步又是一个耗时操作，这样会让吞吐量下降。

    更好一些的方法是调换 3 和 4 的位置，如下：

    1.   `put` 键值，增加表大小
    2.   判断表大小是否达到容量限制，如果未达到，返回。
    3.   创建一个新的带有 WAL 的 空 memtable（耗时操作）
    4.   获取 `state.write()` 写锁
    5.   判断表大小是否达到容量限制，如果未达到，释放写锁并返回。
    6.   旧 memtable 写入 immutable，替换新的 memtable
    7.   释放 `state.write()` 写锁

    现在的问题是会有多个线程同时到达 3，这意味着他们会同时创建多个带有 WAL 的 memtable，但最终只有一个有用，这会存在一些问题（调度 WAL 的线程压力就上来了）。

    所以更明智的做法就是再使用一个 `state_lock`，最终流程如下：

    1.   `put` 键值，增加表大小
    2.   判断表大小是否达到容量限制，如果未达到，返回。
    3.   获取 `state_lock.lock()` 写锁
    4.   判断表大小是否达到容量限制，如果未达到，释放写锁并返回。
    5.   创建一个新的带有 WAL 的 空 memtable（耗时操作）
    6.   获取 `state.write()` 写锁
    7.   旧 memtable 写入 immutable，替换新的 memtable
    8.   释放 `state.write()` 写锁

    这种做法保证只会有 1 个线程创建带有 WAL 的 memtable，创建 memtable 期间尝试 `freeze_memtable` 的线程会因为获取不到 `state_lock.lock()` 写锁而阻塞，且创建 memtable 期间没有获取 `state` 的任何锁，不会影响跳表的读写操作。当线程 `freeze_memtable` 成功之后，在第 3 步等待的线程获取写锁成功，随后在第 4 步发现已经有其他线程执行了 `freeze_table`，就会直接返回。

-   Why does the order to store and to probe the memtables matter? If a key appears in multiple memtables, which version should you return to the user? （存储和探测 memtable 的顺序重要吗？如果一个 key 在多个 memtable 中出现，应该返回哪个版本给用户？）

    答：如果 key 在 当前内存的 memtable 中，直接返回内存中的版本即可。否则按照从新到旧的顺序，查找 immutable memtable 中的 key，找到第一个 key 就返回。

-   Is the memory layout of the memtable efficient / does it have good data locality? (Think of how `Byte` is implemented and stored in the skiplist...) What are the possible optimizations to make the memtable more efficient? （memtable 在内存中的结构高效吗/是否具有数据局部性？（考虑 `Byte` 类型是如何实现的以及如何在跳表中存储）又什么可能的优化方式让 memtable 更高效？）

    答：跳表内部的结构用到了指针， Byte 类型本身存储的也是指针，这都没有空间局部性。能想到的优化方式：因为 memtable 的容量限制是预先定好的，可以预先申请 2 倍或者 1.5 倍的空间，将跳表中所有的指针改成下标索引的方式进行，这样就能利用内存局部性了。

-   So we are using `parking_lot` locks in this course. Is its read-write lock a fair lock? What might happen to the readers trying to acquire the lock if there is one writer waiting for existing readers to stop? （本课程使用了 `parking_lot` ，这是读写公平的锁吗？当有一个线程正在等待获取写锁时，如果有其他线程尝试获取读锁，会发生什么？）

    答：查阅了 [RwLock in parking_lot - Rust](https://docs.rs/parking_lot/latest/parking_lot/type.RwLock.html)，可以知道 `parking_lot` 是读写公平的锁，所以如果先发出获取写锁的请求的话，后面即使有人获取读锁也会等待。具体的原理就是维护了一个队列，在满足条件的情况下将锁提供给队头线程。

-   After freezing the memtable, is it possible that some threads still hold the old LSM state and wrote into these immutable memtables? How does your solution prevent it from happening? （在冻结 memtable 之后，是否有可能存在一些线程仍然持有旧的 LSM 状态，并尝试 immutable 写入数据？如何避免这种情况发生）

    答：如果仅仅锁住 `state_lock.lock()`，那么确实有可能会有线程持有旧的 LSM 状态（因为 memtable 是无锁的，只要拥有 `state.read()` 就能向其中写数据），解决办法就是冻结 memtable 的时候获取 `state.write()`，这样旧可以保证冻结后其他新城看到的是新的 memtable。

-   There are several places that you might first acquire a read lock on state, then drop it and acquire a write lock (these two operations might be in different functions but they happened sequentially due to one function calls the other). How does it differ from directly upgrading the read lock to a write lock? Is it necessary to upgrade instead of acquiring and dropping and what is the cost of doing the upgrade? （有很多地方可能会先获得一个状态的读锁，然后释放掉并再后续的操作中再次获取写锁，这和直接将读锁升级成写锁有什么区别？是否有必要将其升级而不是先获取读锁然后释放读锁，升级写锁有什么开销？）

    答：还是查阅 [RwLock in parking_lot - Rust](https://docs.rs/parking_lot/latest/parking_lot/type.RwLock.html#method.upgradable_read)，可以发现这个锁是支持升级的，调用 `upgradable_read` 可以获得一个支持升级的读锁。获取 `upgradable_read` 的条件为：

    -   队列前没有写锁或其他 `upgradable_read` 锁。

    使用 `upgradable_read` 锁可以保证数据一致性，因为如果先释放读锁在获取写锁的话数据可能就变了。开销就是这种可升级的锁是互斥的。

### Day 2 Merge Iterator

-   What is the time/space complexity of using your merge iterator? （merge iterator 的时间/空间复杂度是多少？）

    答：假设有 $n$ 个迭代器，构建 merge iterator 的时间复杂度为 $O(n\log n)$， `next` 操作的时间复杂度为 $O(\log n)$，`key`，`value`，`is_valid` 时间复杂度均为 $O(n)$。空间复杂度为 $O(n)$。

-   Why do we need a self-referential structure for memtable iterator? （为什么需要在 memtable iterator 中有一个自引用的结构？）

    答：因为迭代器需要引用 memtable，但 memtable 提供迭代器的时候迭代器的生命周期并不明确，换言之，持有迭代器期间需要保证 memtable 不被 gc，解决办法就是让迭代器拥有 memtable 一样的生命周期。这里的 trick 就是在 memtable iterator 的结构中设置一个智能指针指向 memtable，然后迭代器拥有和这个智能指针一样的生命周期，这就是自引用结构。

-   If a key is removed (there is a delete tombstone), do you need to return it to the user? Where did you handle this logic? （当一个 key 被移除（有删除标记时），你会将这个值返回给用户吗？在哪里处理这个逻辑？）

    答：不能将被移除的 key 返回给用户。lsm iterator 处理了这个逻辑，会跳过 value 为空的迭代器。

-   If a key has multiple versions, will the user see all of them? Where did you handle this logic? （如果一个 key 有多个版本，用户会看到所有版本吗？在哪里处理这个逻辑？）

    答：merge iterator 维护了多个迭代器，多个迭代器可能具有相同的 key，这种情况下会返回下标最小（最新版本）的 key 和 value 给用户。在迭代时，相同的 key 的迭代器同时 `next`，这样保证下一次访问是所有相同的 key 都已经被跳过了。

-   If we want to get rid of self-referential structure and have a lifetime on the memtable iterator (i.e., `MemtableIterator<'a>`, where `'a` = memtable or `LsmStorageInner` lifetime), is it still possible to implement the `scan` functionality?（如果不使用自引用结构，而是为 memtable iterator 指定一个生命周期，（例如 `MemtableIterator<'a>`, 其中`'a` 等于 memtable 或者 `LsmStorageInner` 的生命周期），是否还能实现 `scan` 的功能？）

    答：虽然这样做可以实现 `scan` 功能，但是因为迭代器有超长的生命周期，在 `flush` 的时候我们就不能清理掉 immu_memtable 了。

-   What happens if (1) we create an iterator on the skiplist memtable (2) someone inserts new keys into the memtable (3) will the iterator see the new key? (如果（1）在 skiplist memtable 中创建一个迭代器（2）其他人向 memtable 插入了一些数据，（3）迭代器是否会看到这些新插入的数据？）

    答：会，因为我们用的 `crossbeam_skiplist` 是无锁的。如果要避免看到这些数据，就得引入 MVCC。

-   What happens if your key comparator cannot give the binary heap implementation a stable order? （如果 key 的比较器无法满足二叉堆的构建条件，会发生什么？）

    答：其实说白了就是不满足二叉堆构建的数学性质，这样的话我们既不能保证迭代的有序性，也不能保证每个 key 只出现一次

-   Why do we need to ensure the merge iterator returns data in the iterator construction order? （为什么需要保证 merge iterator 按照迭代器构造顺序返回数据？）

    答：因为下标越小的 memtable 越新，相同的 key 我们只取最新的。

-   Is it possible to implement a Rust-style iterator (i.e., `next(&self) -> (Key, Value)`) for LSM iterators? What are the pros/cons? （是否有可能实现 Rust 风格的 LSM 迭代器（例如 `next(&self) -> (Key, Value)`）？优缺点？）

    答：不可以。因为 memtable 采用的是无锁的跳表实现，值会一直变化，如果用 rust 风格的迭代器的话，很可能 `iter.is_valid()` 为 `true`，但 `next` 的时候取不出值。所以我们每次初始化迭代器就获得下一个键值对的拷贝。这样做优点就是 `next` 之后的值不会变动，所以我们利用这个性质构建了 merge iterator，但缺点就是每一次调用 `next` 就会有数据拷贝，不能做那种只移动迭代器而不拷贝的操作。

-   The scan interface is like `fn scan(&self, lower: Bound<&[u8]>, upper: Bound<&[u8]>)`. How to make this API compatible with Rust-style range (i.e., `key_a..key_b`)? If you implement this, try to pass a full range `..` to the interface and see what will happen. （scan 的接口是 `fn scan(&self, lower: Bound<&[u8]>, upper: Bound<&[u8]>)`，如何让这个接口适配 Rust 风格的 range（例如 `key_a..key_b`？如果你实现了这个适配，尝试传递一个范围 `..` 给这个接口，看看会发生什么。））

    答：没有试过，改天实现以一下。

-   The starter code provides the merge iterator interface to store `Box<I>` instead of `I`. What might be the reason behind that? （构建 merge iterator 的时候，我们传递的是 `Box<I>` 而不是 `I`，原因是什么？）

    答：因为 merge iterator 是处理泛型的，所以这里 `I` 不能是一个具体的类型，只要是任意实现了 `StorageIterator` 接口的类型都可以。所以这里用动态类型。（感谢 ppdog 解答这个问题！）

### Day 3 Block

-   What is the time complexity of seeking a key in the block? （在 block 中查找一个 key 的时间复杂度是多少？）

    答：因为 block 中的 key 也是有序的，所以直接二分查找，时间复杂度为 $O(\log n)$

-   Where does the cursor stop when you seek a non-existent key in your implementation? （在你的实现中，如果 block 中没有这个 key，光标会停在哪？）

    答：会停在第一个大于 key 的元素上。

-   So `Block` is simply a vector of raw data and a vector of offsets. Can we change them to `Byte` and `Arc<[u16]>`, and change all the iterator interfaces to return `Byte` instead of `&[u8]`? (Assume that we use `Byte::slice` to return a slice of the block without copying.) What are the pros/cons? （Block 仅仅是一些 data 和 offset 的数据，能否将其替换成 `Byte` 和 `Arc[u16]`，然后将所有返回 `Byte` 的接口改为返回 `&[u8]` （假设我们使用的是 `Byte::slice`，返回一个块的切片，并不直接复制数据））？优缺点是什么？

    答：如果用 `Byte` 和 `Arc[u16]`，同时返回 `Byte::slice` 的话，只要有用户持有这个切片的引用，那么整个 `Byte` 就不会被释放，而一个块有 4KB，但用户很可能只需要其中的一个 key，这样内存消耗就太大了（感谢迟策解答这个问题！）

-   What is the endian of the numbers written into the blocks in your implementation? （在你的实现以哪种字节序写入块中？）

    答：大端序。

-   Is your implementation prune to a maliciously-built block? Will there be invalid memory access, or OOMs, if a user deliberately construct an invalid block? （你的实现是否会对恶意构建的块进行修建？如果用户故意构造无效的块，是否会出现无效内存访问或 OOM ？）

    答：首先 `Block` 可以通过 `BlockBuilder::build(self)` 得到，为此，我们首先在 `build` 中做空块检查。接下来还需要检查`BlockBuilder::add()` 方法，对添加元素的大小进行限制，如果第一个 key 是大 `key` （长度大于 `65535`），会导致错误检查，因为第一个 `key` 一定会加入到 block 中。

    另一种构建块的方式是 `Block::decode(data: &[u8])`，这样的话就需要每一个 key 和 offset 都对应上才可以通过检查。

-   Can a block contain duplicated keys?（block 可以包含重复的 key 吗？）

    答：不可以。

-   What happens if the user adds a key larger than the target block size?

    答：第一个 key 是有可能大于 block size 的，通常我们会把 block size 设置为 4KB，这是磁盘 IO 和 操作系统虚拟内存页读取的基本单位，所以如果大于 4KB，磁盘 IO 的效率会降低。

-   Consider the case that the LSM engine is built on object store services (S3). How would you optimize/change the block format and parameters to make it suitable for such services?

    答：能想到的有增大页面大小，引入高效的数据压缩算法。

-   Do you love bubble tea? Why or why not?

    答：不太喜欢珍珠，可能是以前吃到过假珍珠形成了心理阴影。现在喝奶茶一般选择芋泥啵啵。

### Day 4 Sorted String Table (SST)

-   What is the time complexity of seeking a key in the SST? （在 SST 中查找一个 key 的时间复杂度是多少？）

    答：假设有 $n$ 个 block，平均每个 block 有 $m$ 个 key，会进行两次二分查找，先找到块，再从块中找 key，时间复杂度为 $O(\log n + \log m)$

-   Where does the cursor stop when you seek a non-existent key in your implementation? （在你的实现中，如果 sst 中没有这个 key，光标会停在哪？）

    答：会停在第一个大于 key 的位置。

-   Is it possible (or necessary) to do in-place updates of SST files? （是否有可能（或有必要）在 SST 文件上进行原地修改？）

    答：感觉没有必要，因为不管怎样都是一堆磁盘数据 IO 到内存里。


-   An SST is usually large (i.e., 256MB). In this case, the cost of copying/expanding the `Vec` would be significant. Does your implementation allocate enough space for your SST builder in advance? How did you implement it?（SST 表通常是很大的（例如 256MB），这种情况下，复制/扩容 `Vec` 的开销是巨大的，你的实现是否提前为 SST builder 分配了足够的空间？如何实现？）

    答：在我们调用 `SsTableBuilder.build()`  的时候，我们已经收集了所有的 block 和 meta 的数据，然后进行编码。编码的时候首先复用 block 数据（因为这一块比较大），向其中预先分配好所有的 meta（meta 仅仅包含 block_offset、first_key 和 last_key，其长度是可以精确计算出来的） 和 meta_offset（固定 4 字节） 的空间，这样 `Vec` 扩容导致的扩容至多触发一次。

-   Looking at the `moka` block cache, why does it return `Arc<Error>` instead of the original `Error`?

    答：如果用普通的 `Error` 的话，`Error` 本身可能很大，在多个线程之间复制就会增大开销，而返回 `Arc<Error>` 就可以避免拷贝 `Error` 的开销。

-   Does the usage of a block cache guarantee that there will be at most a fixed number of blocks in memory? For example, if you have a `moka` block cache of 4GB and block size of 4KB, will there be more than 4GB/4KB number of blocks in memory at the same time? （使用块缓存是否可以保证内存中最多有固定数量的块？例如，如果您有一个 4GB 且块大小为 4KB 的 `moka` 块缓存，内存中是否会同时有超过 4GB/4KB 的块数？）

    答：可以保证。`moka` 相当于缓冲池，如果内存满了就会找一个不常用的块写回磁盘。顺便看了一下 `moka` 用的是 LFU。

-   Is it possible to store columnar data (i.e., a table of 100 integer columns) in an LSM engine? Is the current SST format still a good choice? （是否可以在 LSM 引擎中存储列式数据（例如包含 100 个整数列的表）？当前的 SST 格式仍然是一个不错的选择吗？）

    答：当前的用法类似于 `key` 是主键，`value` 是整行数据。如果是列存储，或者列很多的情况，可以针对某一列或某几列用一个单独的 SST，这样仍然是适用。当然针对列存储的一些需求，还可以再单独抽出一个以数字为 key 的 SST，在事务上保证操作的原子性即可。

-   Consider the case that the LSM engine is built on object store services (i.e., S3). How would you optimize/change the SST format/parameters and the block cache to make it suitable for such services? （假设 LSM 引擎建立在对象存储服务（即 S3）上。如何优化/更改 SST 格式/参数和块缓存以使其适合此类服务？）

    答：能想到的就是增大缓存空间、增大 SST 大小、memtable 大小等，然后引入高效的压缩和传输算法等。

-   For now, we load the index of all SSTs into the memory. Assume you have a 16GB memory reserved for the indexes, can you estimate the maximum size of the database your LSM system can support? (That's why you need an index cache!) （目前，我们将 SST 的索引存储在内存中，假设你为 SST 索引准备了 16GB 大小，估算一下这个数据库系统最多能存储多少数据）

    答：假设每个块大小是 4KB，每个块的 meta 是 32B，那么一共能存 $\frac{\text{16GB}}{\text{32B}} \times \text{4KB} = \text{2TB}$ 数据

### Day 5 Read Path

-   Consider the case that a user has an iterator that iterates the whole storage engine, and the storage engine is 1TB large, so that it takes ~1 hour to scan all the data. What would be the problems if the user does so? (This is a good question and we will ask it several times at different points of the course...) （假设用户有一个迭代器，可以迭代整个存储引擎，并且存储引擎大小为 1TB，因此扫描所有数据需要大约 1 小时。如果用户这样做，会出现什么问题？（这是一个好问题，我们会在课程的不同阶段多次提出这个问题……））

    答：在用户迭代期间，我们不能压缩将 memtable 写入到 SST（或者说，即使写入了，由于用户还持有 `Arc<MemTable>`，所以这些内存是不会被释放的），也不能将 SST 进一步压缩成更高维的 SST。

-   Another popular interface provided by some LSM-tree storage engines is multi-get (or vectored get). The user can pass a list of keys that they want to retrieve. The interface returns the value of each of the key. For example, `multi_get(vec!["a", "b", "c", "d"]) -> a=1,b=2,c=3,d=4`. Obviously, an easy implementation is to simply doing a single get for each of the key. How will you implement the multi-get interface, and what optimizations you can do to make it more efficient? (Hint: some operations during the get process will only need to be done once for all keys, and besides that, you can think of an improved disk I/O interface to better support this multi-get interface). （一些 LSM-tree 存储引擎提供的多重获取的接口（或向量获取）。用户可以传递他们想要检索的 key 列表。接口返回每个键的值。例如，`multi_get(vec!["a", "b", "c", "d"]) -> a=1,b=2,c=3,d=4`。显然，一个简单的实现方法是简单地对每个键执行一次获取。您将如何实现多获取接口，以及您可以进行哪些优化以使其更高效？（提示：获取过程中的一些操作只需要对所有键执行一次，除此之外，您可以考虑改进磁盘 I/O 接口以更好地支持此多获取接口）。）

    答：要支持这个接口的话，我们可以支持在迭代器上进一步二分的操作，将输入的 key 排序，获得初始的最大值和最小值作为上下界，然后从最小的 key 开始，每一次二分一个当前最小的 key，每一次二分出结果后范围会进一步缩小，这样平均的时间复杂度会降低。

### Day 6 Write Path

-   What happens if a user requests to delete a key twice? （如果用户请求一个被删除的 key 两次会发生什么？）

    答：删除只是单纯的把 value 设置成空，所以不会发生什么。

-   How much memory (or number of blocks) will be loaded into memory at the same time when the iterator is initialized? （当迭代器初始化的时候，有多少内存（块数量）会被加载？）

    答：假设一次 `scan` 涉及到 $n$ 个迭代器迭代器，初始化的时候会加载 $n$ 个块进内存。仅当一个块遍历结束之后才会加载下一个块。

-   Some crazy users want to *fork* their LSM tree. They want to start the engine to ingest some data, and then fork it, so that they get two identical dataset and then operate on them separately. An easy but not efficient way to implement is to simply copy all SSTs and the in-memory structures to a new directory and start the engine. However, note that we never modify the on-disk files, and we can actually reuse the SST files from the parent engine. How do you think you can implement this fork functionality efficiently without copying data? (Check out [Neon Branching](https://neon.tech/docs/introduction/branching)). （有些疯狂的用户想要复制他们的 LSM 树。他们希望启动引擎来获取一些数据，然后分叉它，这样他们就可以获得两个相同的数据集，然后分别对它们进行操作。一种简单但不高效的实现方法是简单地将所有 SST 和内存结构复制到新目录并启动引擎。但是，请注意，我们永远不会修改磁盘上的文件，我们实际上可以重用父引擎中的 SST 文件。您认为如何在不复制数据的情况下有效地实现此功能？（查看 [Neon Branching](https://neon.tech/docs/introduction/branching)）。）

    答：因为（目前） LSM 树的实现不会写文件，所以我们直接加载同一个 LSM 树引擎的文件作为 SST 表的底层文件，这样就没有复制的开销了。如果实在有写的需求，写时复制即可。在复制之后，需要保证两个引擎对 SST 的命名方式不同。

-   Imagine you are building a multi-tenant LSM system where you host 10k databases on a single 128GB memory machine. The memtable size limit is set to 256MB. How much memory for memtable do you need for this setup?

    -   Obviously, you don't have enough memory for all these memtables. Assume each user still has their own memtable, how can you design the memtable flush policy to make it work? Does it make sense to make all these users share the same memtable (i.e., by encoding a tenant ID as the key prefix)?

    假设您正在构建一个多租户 LSM 系统，其中您在一台 128GB 内存的机器上托管 10k 个数据库。memtable 大小限制设置为 256MB。此设置需要多少 memtable 内存？显然，您没有足够的内存来存储所有这些内存表。假设每个用户仍然有自己的内存表，如何设计 memtable 刷新策略以使其工作？让所有这些用户共享同一个 memtable（即通过将租户 ID 编码为键前缀）是否有意义？

    答：有意义，我们按租户的 ID 为键前缀，因为我们内存多，将 memtable 对所有租户公用，让 immutable 的容量调大一些，每一次回刷的时候再合并一次，这样同一个用户的 key 会尽可能进入到同一个 sst，后续的查找就会节省很多时间。

### Day 7 Snack Time: SST Optimizations

-   How does the bloom filter help with the SST filtering process? What kind of information can it tell you about a key? (may not exist/may exist/must exist/must not exist) （布隆过滤器如何帮助 SST 过滤？它能告诉你关于 key的哪些信息？（可能不存在/可能存在/必须存在/一定不存在））

    答：对于单点查询，如果布隆过滤器对应的值全为 1，这个 key 可能存在。如果有一个地方为 0，这个 key 一定不存在。

-   Consider the case that we need a backward iterator. Does our key compression affect backward iterators? （假设我们需要一个后向迭代器。key 压缩会影响后向迭代器吗？）

    答：不会影响，因为迭代的时候块是整块进入内存的。

-   Can you use bloom filters on scan? （可以将布隆过滤器用在 scan 上吗？）

    答：不可以，只能在单点查询的时候用。

-   What might be the pros/cons of doing key-prefix encoding over adjacent keys instead of with the first key in the block? （对相邻 key 进行前缀编码而不是对块中的第一个 key 进行编码的优点/缺点是什么？）

    答：如果选择相邻 key 编码，那么当 key 的前缀变化较大时会更加节约空间，适合动态的数据。缺点是如果所有 key 都和第 1 个 key 都有相似的前缀的话，其压缩效果不如第 1 个 key 好，例如可能某一个 key 本身就很短 `a`，那么下一个 key 能存储的剩余部分就会很长。另外，相邻编码在编码和解码的时候要更加复杂一些，数据改动时还需要改动下一个 key 的前缀长度。

