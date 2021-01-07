#  iptables 模拟丢包

```
在用iptables 进行网络相关测试时一些小tips记录
```

#### 介绍
- 查看所有的ip规则命令如下
    ```
    iptables -nvL --line-number
    ```
    - -L 查看当前表的所有规则，默认查看的是filter表，如果要查看NAT表，可以加上-t NAT参数
    - -n 不对ip地址进行查，加上这个参数显示速度会快很多
    - -v 输出详细信息，包含通过该规则的数据包数量，总字节数及相应的网络接口
    - –line-number 显示规则的序列号，这个参数在删除或修改规则时会用到

- 添加规则
    - 添加规则有两个参数：-A和-I。其中-A是添加到规则的末尾；-I可以插入到指定位置，没有指定位置的话默认插入到规则的首部
    添加一条规则到第2行
    ```
    iptables -I INPUT 2 -s 111.111.11.1 -j DROP
    ```
    - INPUT -s/OUTPUT -d

- 删除规则
    - 删除用 -D 有两种删除方式
    ```
    iptables -D $(INPUT -s 111.111.11.1 -j DROP) 精准删除,括号内写具体的规则
    iptables -D INPUT $(number) ，定点删除，number为规则id，可以通过iptables -nvL --line-number查看id
    ```
  
#### 模拟丢包命令

- 入站丢包
    - 对121这个ip进行丢包50%的处理。
    ```
    iptables -I INPUT -s 121.**.48.1 -m statistic --mode random --probability 0.5 -j DROP 
    ```
- 出站丢包
    ```
    iptables -I OUTPUT -d 9.**.102.154 -m statistic --mode random --probability 0.5 -j DROP
    ```