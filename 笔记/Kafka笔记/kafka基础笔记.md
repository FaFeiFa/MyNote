# Kafka基础架构

## 消息队列的两种模式

![image-20231010121510192](C:\Users\Hua\AppData\Roaming\Typora\typora-user-images\image-20231010121510192.png)

![image-20231010121544549](C:\Users\Hua\AppData\Roaming\Typora\typora-user-images\image-20231010121544549.png)

![image-20231011155045760](C:\Users\Hua\AppData\Roaming\Typora\typora-user-images\image-20231011155045760.png)

![image-20231011175826269](C:\Users\Hua\AppData\Roaming\Typora\typora-user-images\image-20231011175826269.png)

分区数不能变小

# 生产者发送数据流程

![image-20231011182321816](C:\Users\Hua\AppData\Roaming\Typora\typora-user-images\image-20231011182321816.png)

# 分区策略

默认的分区策略：

1、如果指定分区，就发送到分区里面

2、如果没有指定分区，就按key的hashcode值对分区数进行取模选择分区

3、使用粘性分区策略，随机选择一个分区，并尽可能一直使用该分区，当batch（默认16K）满了，再去随机选择下一个分区

### 自定义分区器

实现Partitioner接口，重写里面的方法partition

# 生产者如何提高吞吐量

 <img src="C:\Users\Hua\AppData\Roaming\Typora\typora-user-images\image-20231012154755063.png" alt="image-20231012154755063" style="zoom:33%;" />

**灵活调整这几个参数**

# 数据可靠

![image-20231012161347930](C:\Users\Hua\AppData\Roaming\Typora\typora-user-images\image-20231012161347930.png)

![image-20231012161202838](C:\Users\Hua\AppData\Roaming\Typora\typora-user-images\image-20231012161202838.png)

![image-20231014180930308](C:\Users\Hua\AppData\Roaming\Typora\typora-user-images\image-20231014180930308.png)

![image-20231014181019015](C:\Users\Hua\AppData\Roaming\Typora\typora-user-images\image-20231014181019015.png)

![image-20231014181300256](C:\Users\Hua\AppData\Roaming\Typora\typora-user-images\image-20231014181300256.png)

![image-20231014211230310](C:\Users\Hua\AppData\Roaming\Typora\typora-user-images\image-20231014211230310.png)

![image-20231014211844398](C:\Users\Hua\AppData\Roaming\Typora\typora-user-images\image-20231014211844398.png)

如果想要保证数据全局唯一而不是只单会话唯一，还需要引入事务

![image-20231015113310665](C:\Users\Hua\AppData\Roaming\Typora\typora-user-images\image-20231015113310665.png)

![image-20231015120741447](C:\Users\Hua\AppData\Roaming\Typora\typora-user-images\image-20231015120741447.png)

![image-20231015122438584](C:\Users\Hua\AppData\Roaming\Typora\typora-user-images\image-20231015122438584.png)

![image-20231015123310539](C:\Users\Hua\AppData\Roaming\Typora\typora-user-images\image-20231015123310539.png)

## leader partition负载均衡



![image-20231018212300164](C:\Users\Hua\AppData\Roaming\Typora\typora-user-images\image-20231018212300164.png)

![image-20231018214244999](C:\Users\Hua\AppData\Roaming\Typora\typora-user-images\image-20231018214244999.png)

kafka文件清除策略

![image-20231018214932685](C:\Users\Hua\AppData\Roaming\Typora\typora-user-images\image-20231018214932685.png)

![image-20231018215258900](C:\Users\Hua\AppData\Roaming\Typora\typora-user-images\image-20231018215258900.png)

# Kafka如何做到高校读写数据

![image-20231018215547567](C:\Users\Hua\AppData\Roaming\Typora\typora-user-images\image-20231018215547567.png)

![image-20231018220032277](C:\Users\Hua\AppData\Roaming\Typora\typora-user-images\image-20231018220032277.png)

## leader选举

## Follower故障

## leader故障







## 同步发送和异步发送的区别

 