# kafka
*  **offset保存位置**  0.9及以下版本保存在zookeeper上，以上的版本保存在名为**_consumer_offsets-0**的topic中。
*  **zookeeper在kafka中的作用**  1、保存broker和consumer的动态加入和离开2、维护消费关系和每个partition的消费信息3、触发消费者再平衡。Kfaka的数据会存储在Zookeeper上，比如集群的broker信息（有哪些borker.id），每个broker的每个topic信息（leader是谁，ISR集合有哪些等）.注：kafka 2.8 版本已经移除了对zookeeper的依赖，使用KRaft进行管理当前处于测试阶段。
*  **数据可靠性保证** 0：数据提交到broker即认为成功。 1：数据提交到leader并落盘即认为成功。 -1：数据提交到leader和所有的follower落盘后才认为成功
* **消费者语义**：最少一次:自动提交false，手动提交。消费者挂掉重启动时会重新消费：最多一次：自动提交true，消费者挂掉重启时会丢掉最后一条数据，
仅一次：自动提交false，自己持久化offset
* **kafka中协调者的作用**
 协调器在0.10版本被分成了两种，**GroupCoordinator** 和**ConsumerCoordinator**
* **GROUPCOORDINATOR**
 1. 对组员提交的请求进行处理
 2. 与消费者进行连接，并从与之连接的消费者中选出一个leader
 3. 当leader分配好分区和消费者的关系后，再把这个结果发给组协调器，组协调器再将此结果发给各消费者
 4. 管理与之连接的消费者的提交的偏移量，将这些偏移量保存在内部的consumer_offset这个主题中
 5. 通过心跳检测自己与消费者的连接
 6. 启动组协调器时启动定时任务用于清除过期的消费组元数据以及过去的消费偏移量等信息
 * **ConsumerCoordinator**
 1. 心跳 通过和组协调器发送消息来维持和群组和分区的所有权关系，只要消费者以正常的频率发送消息就被认为是活跃的，说明它正在读取分区里的消息。 **心跳一般是在提交offset和轮询获取消息时附带发送**
 2. 分区再平衡  三种情况 1. 新的消费者加入组2.消费者关闭或崩溃3.添加了新的分区，一般情况下不建议消费者数量大于分区数量，否则会频繁发生再平衡（rebalance）
 3. 获取分区策略
 4. 消费者入组过程：消费者协调器选择负载最小的消费者去寻找组协调器。找到组协调器后消费者协调器申请加入该消费组，发送 joingrouprequest请求，KafkaApis处理此请求 1. 把消费者注册到消费组中 2. 把消费者的clientid与一个uuid 生成一个memberid分配给消费者3. 构造该消费者的membermetadat信息.4. 把消费者的membermetadata信息注册到GroupMetadata中.4. 第一个加入消费组的将成为leader