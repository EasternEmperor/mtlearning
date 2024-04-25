关联文档：[1、LevelDB](https://km.sankuai.com/collabpage/2203114949)

1. 解决关键问题：打破内存限制以及内存持久化
    
    1. 思考点：1024条kv为int型，hashmap会占用多大内存？
        
        1. hashmap底层结构：
            
            ```
            public class HashMap{
            	// 底层数组
              Node<K, V>[] table;
              private class Node{
              	final int hash;
                final K key;		// 底层存储为Integer对象
                V value;
                Node<K, V> next;
              }
            }
            ```
            
        2. 内存占用：[探讨HashMap占用内存大小](https://km.sankuai.com/collabpage/2227958747)
            
2. 持久化面临问题：
    
    1. 底层数据结构：LSM树，写快读慢
        ![[Pasted image 20240425162828.png]]
        
        1. 写时先写log(crash-safe)，再写入内存memtable（一般为跳表）；
            
        2. 满了切换为immutable memtable（不可更改），生成新的memtable接收新的写操作；
            
        3. 将immutable memtable刷到L0层(minor compaction)，这里不会进行合并，多个key可能重复出现
            
            1. minor compaction：即直接将​immutable memtable加入level 0的sstable末尾
                
        4. 如果Li层满了，则会将这一层所有的sstable合并（多路归并排序），会删除重复数据和被标记为删除的数据(major compaction)
            
            1. sstable结构：内容按照key有序排列，文件末尾有索引：
                ![[Pasted image 20240425162842.png]]
            2. ​删除数据时先给数据打上tombstone（带有时间戳和TTL），在合并时碰到标记的数据再物理删除
                
            3. major compaction：选择当前level的所有sstable（内部有序）进行多路归并排序，合并过程中会删除重复键（保留最新）并清理被标记为tombstone的键。合并完成后会删除所有旧的、参与合并的sstable
                
                1. level compaction：合并后sstable通常会刷到下一层，可能进行新一轮的major compaction，同时所有tombstone也一起被传递下去
                    
                2. size-tired compaction：sstable在level中按大小分组，组内合并，因此sstable合并后可能不会刷到下一层
                    
    2. 一些优化：为每个sstable设置布隆过滤器提高点查询效率；层级结构减少读磁盘次数，提高读效率...
        
3. 问题解答：
    
    1. 持久化机制：
        
        1. kv数据在sstable中按key顺序排列，并有索引以提高查询效率；
            
        2. 优化写性能：写操作直接在内存中进行，删除键只设置tombstone标记，后台线程处理compaction过程
            
        3. put成功一定能get到：先写日志，日志写成功才能返回put成功。只要有日志即使宕机memtable消失也能恢复
            
        4. 保证仅最新数据对用户可见：首先读内存，memtable中的一定是最新的数据；如果需要读sstable，按层级从低到高逐层读数据，因为在sstable中，由于compaction操作，level越低那么数据一定是越新的
            
        5. key被高频修改那么应该在memtable中，同时在level0中也可能会有其持久化过的数据
            
        6. memtable需要提升读写性能，同时对删除的数据用tombstone做标记传递
            
        7. memtable写入比sstable写入快很多可能memtbale堆积，影响读写性能：增加memtable大小以减少sstable写入频率；提升硬件品质；多线程处理sstable的刷入；
            
            1. 即：减少sstable的写入频率 + 提升sstable的写入速度
                
        8. 快照：为每个操作生成序列号，读取时只能读到序列号<=自己序列号的数据，实现一致性读；提交日志：WAL机制，实现原子性和持久性（注：levelDB不实现复杂的事务支持）
            
    2. 优化读性能：每个sstable设置布隆过滤器，level分层机制
        
        1. 布隆过滤器可以判断请求的键是否在本sstable中，从而避免无效的遍历
            
        2. key被高频修改：membale中读取；且level中越低的层保存越新的数据，从而减少读磁盘次数
            
        3. level体现在：持久化层级中对每一个level能保存的数据量限制，每一层的大小都是上一层的10倍，便于在major compaction后的刷入；读取时也是按照层级读；只有上层满了进行major compaction之后才会刷到下一层