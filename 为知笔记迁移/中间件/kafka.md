# kafka
*  offset保存位置  0.9及以下版本保存在zookeeper上，以上的版本保存在名为**_consumer_offsets-0**的topic中。
*  zookeeper在kafka中的作用  1、保存broker和consumer的动态加入和离开2、维护消费关系和每个partition的消费信息3、触发消费者再平衡。Kfaka的数据会存储在Zookeeper上，比如集群的broker信息（有哪些borker.id），每个broker的每个topic信息（leader是谁，ISR集合有哪些等）.注：kafka 2.8 版本已经移除了对zookeeper的依赖，使用KRaft进行管理当前处于测试阶段。
*  数据可靠性保证 0：数据提交到broker即认为成功。 1：数据提交到leader并落盘即认为成功。 -1：数据提交到leader和所有的follower落盘后才认为成功
* 消费者语义：最少一次:自动提交false，手动提交。消费者挂掉重启动时会重新消费：最多一次：自动提交true，消费者挂掉重启时会丢掉最后一条数据，
仅一次：自动提交false，自己持久化offset