## Seata 分布式事务 demo

#### 场景: 跨行转账
两个账户在不同的银行(张三在bank1、李四在bank2)，bank1和bank2是两个个微服务。  
交易过程是，张三 给李四转账指定金额。    

交互流程如下: 
1. 请求bank1进行转账，传入转账金额。 
2. bank1减少转账金额，调用bank2，传入转账金额, bank2添加转账金额。

#### 组成
- 数据库: MySQL 5.7。bank 1 和bank 2
- JDK 1.8 +
- 微服务框架: spring-boot, spring-cloud
- 分布式事务: seata
    - RM, TM: spring-cloud-alibaba-seata 
    - TC: seata-server


