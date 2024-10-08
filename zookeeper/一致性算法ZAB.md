1. ZAB(Zookeeper Atomic Broadcast)是一种类似于paxos的共识算法，设计得更适于主从模式
# 主要步骤
## 1. 领导者选举
1. ZK中，总有一个节点被选为领导者，其他节点作为追随者
2. 选举过程中：
	1. ZAB使用最新事务ID(XID)和任期ID(Epoch ID)作为投票依据：XID -> Epoch ID -> 节点ID
	2. 每个节点始终投票给最新XID、最大Epoch ID、最大节点ID的节点
	3. 每个节点投票后，将选票（包含投票对象的XID, Epoch ID, 节点ID）发送给所有其他节点
	4. 每个节点收到其他节点的投票后，根据三个ID，如果比自己投票对象更合适，则修改自己的投票，投给该节点
3. 最终收敛后，每个节点都统计选票，获得多数（超过一半）的票数的节点当选为领导者
# 2. 发现
1. 领导者在发现阶段，收集所有其他节点的状态：最新事务日志、最新XID、最新Epoch ID等
2. 领导者收集完毕后，确定新Epoch本集群的一致性状态
# 3. 同步
1. 领导者将确定的一致性状态发送给所有追随者，如最新事务日志、XID、Epoch ID等，确保所有追随者在一致的状态下处理新的事务
# 4. 广播
1. 领导者开始接收客户端的请求，并将这些请求广播给所有追随者
2. 每个追随者收到事务后，先写入本地的事务日志，写入成功后发送ACK给领导者
3. 领导者收到多数（超过一半）节点返回的ACK后，提交该事务，同时通知所有追随者提交该事务