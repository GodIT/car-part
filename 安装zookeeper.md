##### 下载压缩包

* 执行如下linux命令  
    cd ~  
    wget https://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz

* 解压压缩包  
  
    ```shell
    tar -zxvf zookeeper-3.4.14.tar.gz
    ```
    
    

##### 开始部署zookeeper(下面我均简称zk)

* 创建zk目录  
  
```shell
    mkdir /usr/local/zookeeper/
```


​    
* 复制zk到指定目录  
  
    ```shell
cd ~  
    cp -R zookeeper-3.4.14/* /usr/local/zookeeper/

    cd /usr/local/zookeeper/conf/

    cp zoo_sample.cfg zoo.cfg
    ```
    
    
    
* 启动zk  
  
    ```shell
    sh /usr/local/zookeeper/bin/zkServer.sh start
    ```
    
    