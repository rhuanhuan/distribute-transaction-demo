## Seata 分布式事务 demo

#### 场景: 跨行转账
两个账户在不同的银行(张三在bank1、李四在bank2)，bank1和bank2是两个个微服务。  
交易过程是，张三 给李四转账指定金额。    

交互流程如下: 
1. 请求bank1进行转账，传入转账金额。 
2. bank1减少转账金额，调用bank2，传入转账金额, bank2添加转账金额。

#### 工具
- 数据库: MySQL 5.7。bank 1 和bank 2
- JDK 1.8 +
- 微服务框架: spring-boot, spring-cloud
- 分布式事务: seata
    - RM, TM: spring-cloud-alibaba-seata 
    - TC: seata-server, 单独部署

#### 步骤
##### 1. 创建数据库
`bank1`库 & `bank2`库 
- `account_info`表，`bank1`包含'张三'账户，`bank2`包含'李四'账户
- `undo_log`表，为seata框架使用
参考本工程`sql`目录

##### 2. 启动TC(Transaction Coordinator)
下载：https://github.com/seata/seata/releases/  
启动：`${SEATA_PATH}/bin/seata-server.sh -p 8888 -m file`。8888为服务端口号;file为启动模式，这里指seata服务将采用文件的方式存储信息。出现`Server started...`的字样则表示启动成功。

##### 3. 启动服务注册中心
基于Eureka实现注册中心。

##### 4. 业务工程配置seata
在`src/main/resource`中，新增两个文件。
- `registry.conf`
- `file.conf`  

内容可以copy上面TC server下面的`config`目录的文件，然后：  
- 在`registry.conf`中`registry.type`使用`file`
- 在`file.conf`中，
    - `service.vgroup_mapping.[springcloud服务名]-fescar-service-group = [Seata Server集群名称]`，Seata Server集群默认名称为default  
    - `service.default.grouplist =[seata服务端地址]`

##### 5. 创建代理数据源
Seata的RM通过`DataSourceProxy`才能在业务代码的事务提交时，通过这个切入点，与TC进行通信交互、记录undo_log等。  
添加`DatabaseConfiguration.java`。

```java
@Configuration
public class DatabaseConfiguration {
    private final ApplicationContext applicationContext;

    public DatabaseConfiguration(ApplicationContext applicationContext) {
        this.applicationContext = applicationContext;
    }

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.ds0")
    public DruidDataSource ds0() {
        DruidDataSource druidDataSource = new DruidDataSource();
        return druidDataSource;
    }


    @Primary
    @Bean
    public DataSource dataSource(DruidDataSource ds0)  {
        DataSourceProxy pds0 = new DataSourceProxy(ds0);
        return pds0;
    }
}
```

#### 执行流程


