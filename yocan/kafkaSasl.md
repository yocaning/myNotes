# kafka 配置SASL验证 

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

#### PLAIN配置步骤
##### Brokers
- 配置Kafka集群中的所有broker以接受来自客户端的安全连接。对代理所做的任何配置更改都需要滚动重新启动。<这也是PLAIN方案存在的问题，如果新增用户需要重启broker>
      
- JAAS【不推荐】
    - 一个方法是采用JAAS文件进行用户相关信息的设置 for example :kafka_server_jaas.conf
    - 每个KafkaServer/Broker使用JAAS文件中的KafkaServer部分为Broker提供SASL配置选项，包括任何由Broker建立的用于Broker间通信的SASL客户端连接。
      如果将多个侦听器配置为使用SASL，您可以在部分名称前加上侦听器名称的小写前缀，后跟句号(例如，sasl_ssl.KafkaServer.)。
    - 【不推荐】这种方式需要修改启动脚本，需要增加：
    -Djava.security.auth.login.config=$base_dir/../config/kafka_server_plain_jaas.conf
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
    1. 在服务器中启用SASL/PLAIN机制。每个代理的【server.properties】属性文件。
    ```
      # List of enabled mechanisms, can be more than one
      sasl.enabled.mechanisms=PLAIN
      
      # Specify one of of the SASL mechanisms
      sasl.mechanism.inter.broker.protocol=PLAIN
    ```
    2. 如果您希望为代理间通信启用SASL，请将以下内容添加到代理属性文件中(默认PLAINTEXT)。将协议设置为:
        - SASL_SSL: 如果启用了SSL加密(如果SASL机制是普通的，则应该始终使用SSL加密)
        - SASL_PLAINTEXT：如果没有启用SSL加密
        ```
       # Configure SASL_SSL if SSL encryption is enabled, otherwise configure SASL_PLAINTEXT
       security.inter.broker.protocol=SASL_SSL
       ```
    3.告诉Kafka broker监听客户端和代理间SASL连接的端口。您必须配置listeners。advertised.listeners可选，如果值与 listeners 不同。设置监听器为:
    
    ```
      # With SSL encryption
      listeners=SASL_SSL://kafka1:9092
      advertised.listeners=SASL_SSL://localhost:9092
      
      # Without SSL encryption
      listeners=SASL_PLAINTEXT://kafka1:9092
      advertised.listeners=SASL_PLAINTEXT://localhost:9092
    ```
    4. 配置SASL_SSL和PLAINTEXT端口，比如:
    ```
      # With SSL encryption
      listeners=PLAINTEXT://kafka1:9092,SASL_SSL://kafka1:9093
      advertised.listeners=PLAINTEXT://localhost:9092,SASL_SSL://localhost:9093
      
      # Without SSL encryption
      listeners=PLAINTEXT://kafka1:9092,SASL_PLAINTEXT://kafka1:9093
      advertised.listeners=PLAINTEXT://localhost:9092,SASL_PLAINTEXT://localhost:9093
    ```
    5.**如果你没有使用单独的JAAS配置文件来配置JAAS，那么为Kafka broker监听器配置JAAS如下所示**:
    ```
      # With SSL encryption
      listener.name.sasl_ssl.plain.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
         username="admin" \
         password="admin-secret" \
         user_admin="admin-secret" \
         user_kafkabroker1="kafkabroker1-secret";
      
      # Without SSL encryption
      listener.name.sasl_plaintext.plain.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
         username="admin" \
         password="admin-secret" \
         user_admin="admin-secret" \
         user_kafkabroker1="kafkabroker1-secret";
    ```
       
#### 写在最后
- 配置完成之后，启动broker就可以了
- 需要注意的是：如果采用kafka自带的producer和consumer console 工具进行测试，需要修改对应的producer.properties和consumer.properties
  增加如下配置：
  ```
    sasl.mechanism=PLAIN
    # Configure SASL_SSL if SSL encryption is enabled, otherwise configure SASL_PLAINTEXT
    security.protocol=SASL_SSL
  
    sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
    username="test" \
    password="test";
    ```
- 网上的资料乱且不规范，没有对应的版本号，各执一词，所以经常可用性无法保障，要学会看官方文档
---


>其他类型配置可以翻阅[官方文档](https://docs.confluent.io/platform/current/kafka/authentication_sasl/index.html)
