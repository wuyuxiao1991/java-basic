+ 为一个消息系统，Kafka遵循了传统的方式，选择由Producer向broker push消息并由Consumer从broker pull消息
消息队列的优点：对于峰值高的应用，可以进行消峰，避免崩溃

Kalfa的优点：轻量级、分布式   
如果Partition机制设置合理，所有消息可以均匀分布到不同的Partition里，这样就实现了负载均衡。    
如果一个Topic对应一个文件，那这个文件所在的机器I/O将会成为这个Topic的性能瓶颈，而有了Partition后，不同的消息可以并行写入不同broker的不同Partition里，极大的提高了吞吐率。    
使用consumer group的概念实现单播（在一个group）和多播（各个consumer 在不同的group）    
首先创建一个Topic (名为topic1，包含3个Partition)，然后创建一个属于group1的Consumer实例，并创建三个属于group2的Consumer实例，最后通过Producer向topic1发送key分别为1，2，3的消息。结果发现属于group1的Consumer收到了所有的这三条消息，同时group2中的3个Consumer分别收到了key为1，2，3的消息。（一个partition可能会携带有很多条消息）     
超过10分钟会重复下抛一个消费者一个分组实现广播，然后默认每隔1分钟重试一次，一般会设置得越来越大得值，不会一样，重试是要在服务端接受到失败的确认，再过1分钟，重试，不然没接受到回应10分钟以后就重试，除了卡夫卡的重试机制，还有人工补偿，会有定时任务，从流水表中获取状态不对的去重新抛送卡夫卡五次，所以总共有10几次机会去重试
