# dcocker 一些小tips记录

```
在使用dockerfile的过程中，遇到的问题进行整理
```

#### docker登录
- 私有镜像仓库是需要登陆的，命令如下
    ```
    docker login -u username -p password dockerhub
    ```


#### dockerfile配置记录

- 每一行RUN命令都会额外构造一个镜像层，为了减少镜像大小，可以合并多个RUN命令
- 遇到docker pull http/https resp不兼容问题
    ```
    vim  /etc/docker/daemon.json
    在文件中加入需要修改的dockerhub，然后重启
    systemctl restart docker
    ```
- 删除docker容器/镜像
    ```
    docker ps -a 可以看到因为异常down掉的容器
    docker rm -f  <容器ID>
    docker rmi    <镜像ID>
    ```
  
- 调试docker 启动时
    - 可以在docker run 启动参数上加上 -v /test(宿主机目录):/root(镜像目录)方便查看相关日志等
    - 直接查看docker启动日志可以用 
    ```
    docker  logs <容器id>
    ```