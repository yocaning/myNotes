# kafka 配置SASL验证【Share about GC】

```
做一个推送kafka服务，需要支持kafka免密和SASL验证
记录一下关于kafka 配置SASL 验证的步骤
以PLAIN 为例
```

#### 关于SASL/PLAIN 
- PLAIN或SASL/PLAIN是一种简单的用户名/密码身份验证机制，通常与TLS一起用于加密以实现安全身份验证。Apache Kafka支持SASL/PLAIN的默认实现，
可以扩展到生产环境中使用。
- SASL/PLAIN将密码作为明文传输(可以通过二次开发自行进行补充)，应该仅与SSL作为传输层一起使用，以确保在没有加密的情况下不会在网络上传输清晰的密码。
- Kafka中SASL/PLAIN的默认实现在JAAS配置文件中指定用户名和密码。您可以通过配置您自己的回调处理程序，
使用配置选项sasl.server.callback.handler.class和sasl.client.callback.handler.class来从外部源获取用户名和密码，从而避免在磁盘上存储干净的密码。
  - 

#### 配置步骤 <以PLAIN 为例>
##### Brokers
- 配置Kafka集群中的所有broker以接受来自客户端的安全连接。对代理所做的任何配置更改都需要滚动重新启动。<这也是PLAIN方案存在的问题，如果新增用户需要重启broker>
      
- JAAS【不推荐】
    - 一个方法是采用JAAS文件进行用户相关信息的设置 for example :kafka_server_jaas.conf
    - 每个KafkaServer/Broker使用JAAS文件中的KafkaServer部分为Broker提供SASL配置选项，包括任何由Broker建立的用于Broker间通信的SASL客户端连接。
      如果将多个侦听器配置为使用SASL，您可以在部分名称前加上侦听器名称的小写前缀，后跟句号(例如，sasl_ssl.KafkaServer.)。
    - 【不推荐】这种方式还需要修改启动脚本，需要增加：-Djava.security.auth.login.config=$base_dir/../config/kafka_server_plain_jaas.conf
```
       KafkaClient {
       org.apache.kafka.common.security.plain.PlainLoginModule required
       username="admin"
       password="test";
       user_admin ="test"
       user_test ="test"
       };
       Client {
       org.apache.zookeeper.server.auth.DigestLoginModule required
       username="admin"
       password="test";
       };
```

- 开始配置






```markdown
其他类型配置可以翻阅[官方文档](https://docs.confluent.io/platform/current/kafka/authentication_sasl/index.html)
```
