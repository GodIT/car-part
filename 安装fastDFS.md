##### 什么是fastDFS

- FastDFS是由国人余庆所开发(淘宝团队中的大牛，被挖走了)，该项目淘宝也(曾经，现在或继续)使用，其项目地址：

  
    https://github.com/happyfish100


  FastDFS是一个轻量级的开源分布式文件系统，主要解决了大容量的文件存储和高并发访问的问题，文件存取时实现了负载均衡。

  FastDFS是一款类Google FS的开源分布式文件系统，它用纯C语言实现，支持Linux、FreeBSD、AIX等UNIX系统。

  FastDFS只能通过专有API对文件进行存取访问，不支持POSIX接口方式，不能mount使用。

  准确地讲，Google FS以及FastDFS、mogileFS、 HDFS、TFS等类Google FS都不是系统级的分布式文件系统，而是应用级的分布式文件存储服务。

##### FastDFS的特性

- 1》分组存储，灵活简洁、对等结构，不存在单点
  2》文件ID由FastDFS生成，作为文件访问凭证，FastDFS不需要传统的name server
  3》和流行的web server无缝衔接，FastDFS已提供apache和nginx扩展模块
  4》大、中、小文件均可以很好支持，支持海量小文件存储
  5》 支持多块磁盘，支持单盘数据恢复
  6》 支持相同文件内容只保存一份，节省存储空间
  7》 存储服务器上可以保存文件附加属性
  8》 下载文件支持多线程方式，支持断点续传

##### fastDFS架构图

![img](https://www.showdoc.cc/server/api/common/visitfile/sign/bc292c67be0d077ceb0ab2f1766dc797?showdoc=.jpg)

##### fastDFS安装部署

- 下载相关依赖(依次执行如下命令)
  
```shell
  cd ~

  wget https://github.com/happyfish100/libfastcommon/archive/V1.0.7.tar.gz -O libfastcommonV1.0.7.tar.gz

  wget http://jaist.dl.sourceforge.net/project/fastdfs/FastDFS%20Nginx%20Module%20Source%20Code/fastdfs-nginx-module_v1.16.tar.gz

  wget https://github.com/happyfish100/fastdfs/archive/V5.05.tar.gz -O FastDFS.tar.gz

  wget http://mirrors.sohu.com/nginx/nginx-1.8.0.tar.gz

  yum install -y gcc gcc-c++

  yum -y install libevent
```

  

- 下面我们开始部署

  ```shell
tar -zxvf libfastcommonV1.0.7.tar.gz -C /usr/local/
  
  ```

```
cd /usr/local/libfastcommon-1.0.7/

./make.sh && ./make.sh install

cp /usr/lib64/libfastcommon.so /usr/lib/
```


  安装tracker

```
cd ~

tar -zxvf FastDFS.tar.gz -C /usr/local/

mv /usr/local/fastdfs-5.05 /usr/local/FastDFS

cd /usr/local/FastDFS/

./make.sh && ./make.sh install

/bin/cp -rf /usr/local/FastDFS/conf/* /etc/fdfs/

cd /etc/fdfs/

cp tracker.conf.sample tracker.conf

sed -i 's/base_path=\/home\/yuqing\/fastdfs/base_path=\/home\/fastdfs/g' tracker.conf

sed -i 's/http.server_port=8080/http.server_port=80/g' tracker.conf

mkdir -p /home/fastdfs
```


  启动tracker

```
/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf restart
```


  配置和启动storage

```
cd /etc/fdfs/
```


  【*小心】：下面部分”这里填你的ip”一定要换填为当前服务器公网ip

```shell
sed -i 's/192.168.209.121:22122/这里填你的ip:22122/g' storage.conf

sed -i 's/8888/88/g' storage.conf

sed -i 's/store_path0=\/home\/yuqing\/fastdfs/store_path0=\/home\/fdfs_storage /g' storage.conf

sed -i 's/base_path=\/home\/yuqing\/fastdfs/base_path=\/home\/fastdfs/g' storage.conf

mkdir -p /home/fdfs_storage
```


  通过防火墙开启相关端口

```shell
firewall-cmd --zone=public --add-port=22122/tcp --permanent

firewall-cmd --zone=public --add-port=88/tcp --permanent

firewall-cmd --zone=public --add-port=23000/tcp --permanent

firewall-cmd --reload
```


  启动storage

```shell
/usr/bin/fdfs_storaged /etc/fdfs/storage.conf restart
```


  配置fastdfs-nginx-module

```shell
cd ~

tar -zxvf fastdfs-nginx-module_v1.16.tar.gz -C /usr/local

cd /usr/local/fastdfs-nginx-module/src/

sed -i 's/\/usr\/local/\/usr/g' config

cp mod_fastdfs.conf /etc/fdfs/

sed -i 's/base_path=\/tmp/base_path=\/home\/fastdfs/g' /etc/fdfs/mod_fastdfs.conf

sed -i 's/url_have_group_name = false/url_have_group_name = true/g' /etc/fdfs/mod_fastdfs.conf

sed -i 's/tracker_server=tracker:22122/tracker_server=你自己的ip:22122/g' /etc/fdfs/mod_fastdfs.conf

sed -i 's/store_path0=\/home\/yuqing\/fastdfs/store_path0=\/home\/fdfs_storage/g' /etc/fdfs/mod_fastdfs.conf

cp /usr/lib64/libfdfsclient.so /usr/lib/

mkdir -p /var/temp/nginx/client
```


  安装nginx

```shell
cd ~

tar -zxvf nginx-1.8.0.tar.gz -C /usr/local/

yum -y install pcre && yum -y install pcre-devel && yum -y install zlib && yum -y install zlib-devel && yum -y install openssl && yum -y install openssl-devel

cd /usr/local/nginx-1.8.0/

./configure --prefix=/usr/local/nginx --pid-path=/var/run/nginx/nginx.pid --lock-path=/var/lock/nginx.lock --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --with-http_gzip_static_module --http-client-body-temp-path=/var/temp/nginx/client --http-proxy-temp-path=/var/temp/nginx/proxy --http-fastcgi-temp-path=/var/temp/nginx/fastcgi --http-uwsgi-temp-path=/var/temp/nginx/uwsgi --http-scgi-temp-path=/var/temp/nginx/scgi --add-module=/usr/local/fastdfs-nginx-module/src

make && make install

cd /usr/local/FastDFS/conf

/bin/cp -rf http.conf mime.types /etc/fdfs/

  mkdir /usr/local/nginx/logs

  cd /usr/local/nginx/conf/

  vim nginx.conf
```


  下面是在Linux下修改文档内容，如果你不懂怎么修改内容请联系我们或者寻找会操作linux命令的技术人员

  启动nginx

```
  /usr/local/nginx/sbin/nginx
```



  nginx + fastdfs 的开机自启动
  虚拟机每次启动之后都要重新启动一下fastdfs 和 nginx服务，比较麻烦，所以增加开机自启动；

编辑 /etc/rc.d/rc.local 文件，增加启动项；

1、编辑文件

```
vim /etc/rc.d/rc.local
```

2、增加如下：

```
/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf restart
/usr/bin/fdfs_storaged /etc/fdfs/storage.conf restart
/usr/local/nginx/sbin/nginx
```

3、在centos7中, /etc/rc.d/rc.local 文件的权限被降低了，需要给rc.local 文件增加可执行的权限；

```
chmod +x /etc/rc.d/rc.local
```



