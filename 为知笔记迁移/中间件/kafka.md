# kafka
*  offset保存位置  0.9及以下版本保存在zookeeper上，以上的版本保存在名为**_consumer_offsets-0**的topic中。
*  zookeeper在kafka中的作用  1、保存broker和consumer的动态加入和离开2、维护消费关系和每个partition的消费信息3、触发消费者再平衡。Kfaka的数据会存储在Zookeeper上，比如集群的broker信息（有哪些borker.id），每个broker的每个topic信息（leader是谁，ISR集合有哪些等）.注：kafka 2.8 版本已经移除了对zookeeper的依赖，使用KRaft进行管理当前处于测试阶段。
*  