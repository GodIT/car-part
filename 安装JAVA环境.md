##### 版本要求

* `JAVA版本要求1.8+`

##### 开始部署java环境

* 上传完成，我们开始解压压缩包  
    依次执行如下linux命令：  
    
```shell
    tar -zxvf jdk-8u251-linux-x64.tar.gz

    mkdir /usr/local/java

    cp -R ./jdk1.8.0_251/* /usr/local/java/
    
    echo -e "export JAVA_HOME=/usr/local/java\n\
    export JRE_HOME=${JAVA_HOME}/jre\n\
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib\n\
    export PATH=${JAVA_HOME}/bin:$PATH\n" >> /etc/profile

    source /etc/profile
    ```
    
    
    
* 输入命令

    
    
    ```
    java -version
    ```
    
    