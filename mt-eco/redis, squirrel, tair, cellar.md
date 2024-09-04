# 1. 点评KV存储发展
1. 客户端做一致性哈希，后端部署`memcached`. 
	1. 宕机、扩缩容丢数据：
![[Pasted image 20240903163702.png]]
2. 使用redis主从架构代替memcached
	1. 扩缩容丢数据：
![[Pasted image 20240903164208.png]]
3. tair架构：存储节点内有数据迁移和数据复制机制，解决扩容丢数据问题；存储节点定时发送心跳给中心节点。中心节点有两个配置管理节点，通过心跳监控存储节点；存储节点宕机、扩缩容，中心节点重构集群拓扑。客户端启动时从中心节点拉取路由表。
	1. 中心节点没有分布式仲裁机制，可能脑裂
	2. tair数据结构不如redis丰富
![[Pasted image 20240903164727.png]]
# 2. squirrel, cellar发展
1. redis cluster -> squirrel
2. tair -> cellar
## 2.1. squirrel
1. squirrel架构：中间就是redis-cluster模式，redis之间通过gossip协议通信维持数据一致性。右侧集群调度平台管理squirrel集群，将管理结果作为元数据存储在zookeeper。客户端订阅zookeeper上的元数据变更，实时获取到最新拓扑情况，直接在redis上进行读写操作
![[Pasted image 20240903190527.png]]
2. HA监控：HA实时监控redis集群，不管是网络抖动还是宕机，HA都会通知zookeeper更新元数据，去除该机器，客户端也能及时收到新路由
3. 数据智能迁移：
	1. redis-cluster迁移问题：要迁移的slot、迁移到哪儿不确定；migrate阻塞正常查询
	2. squirrel：
		1. “就近”原则
		2. 同一集群迁出做并发
		3. 一个migrate命令迁移一部分key，迁移速度变化类似`tcp慢开始`
		4. 大key、大value迁移时，异步migrate迁移，主线程正常处理其他请求，如果请求迁移中的key，则返回错误
4. 热点key：
	1. 主从检测热点key，收集后上报调度平台，调度平台发送迁移指令，将热点slot迁移到热点主从
	2. 热点主从：专门存放热点slot，处理能力强
	![[Pasted image 20240903193032.png]]
# 2.2 cellar
