---
layout:     post
title:      FastDFS docker安装教程
subtitle:   分布式文件管理服务器
date:       2018-12-10
author:     pengfeng
header-img: img/post-bg-coffee.jpeg
catalog: true
tags:
    - docker
    - FastDFS
---

>总结整理

  *`docker`*属于基础设施，这里不再赘述，不会安装的小伙伴请参考[docker安装教程](http://www.runoob.com/docker/docker-tutorial.html)。
  
  --
  FastDFS分为tracker，storage两个服务。tracker:追踪服务,跟踪文件储存位置，负责存取。storage: 存储服务，用来储存文件。
  
  [](http://47.100.206.217/group1/M00/00/00/rBAYR1wOQwCAXtuNAAn3sLm0lac142.jpg)
  
### 1.拉取镜像
    docker pull morunchang/fastdfs

### 2.查看镜像
    docker images
    
### 3.运行tracker
    创建tracker容器并运行
    docker run -d --name tracker --net=host morunchang/fastdfs sh tracker.sh

### 4.运行storage
创建storage容器并运行
    
    docker run -d --name storage --net=host -e TRACKER_IP=47.100.206.217:22122 -e GROUP_NAME=group1 morunchang/fastdfs sh storage.sh
    -e:表示添加容器环境变量
    TRACKER_IP:47.100.206.217:22122
    GROUP_NAME=group1
    docker ps 查看容器运行状况
   [](http://47.100.206.217/group1/M00/00/00/rBAYR1wOQrmAHFCyAAJVXAhyrYU891.jpg)
    
### 5.修改storage容器内部的nginx配置
例:docker exec -it 容器名 /bin/bash
storage:容器名

    1.进入容器
    docker exec -it storage  /bin/bash
    
    2.进入目录
    cd /data/nginx/conf
    
    3.vi /data/nginx/conf/nginx.conf 修改配置添加规则
    
    location /group1/M00 {
       proxy_next_upstream http_502 http_504 error timeout invalid_header;
         proxy_cache http-cache;
         proxy_cache_valid  200 304 12h;
         proxy_cache_key $uri$is_args$args;
         proxy_pass http://fdfs_group1;
         expires 30d;
     }
     
     4.退出容器，并重启容器
     docker restart storage
     

### 代码部分
    
POM文件:

    <dependency>
       <groupId>net.oschina.zcx7878</groupId>
       <artifactId>fastdfs-client-java</artifactId>
       <version>1.27.0.0</version>
    </dependency>

配置:fdfs_client.conf
    
    connect_timeout = 60
    #网络超时时间
    network_timeout = 60
    #字符集
    charset = UTF-8
    #跟踪服务器的端口
    http.tracker_http_port = 8080
    http.anti_steal_token = no
    http.secret_key = 123456
    #跟踪服务器地址 。跟踪服务器主要是起到负载均衡的作用
    tracker_server = 47.100.206.217:22122
    
Demo:

    private static String upload(String local_path, String file_ext_name) throws IOException, MyException {
            // 1、使用 StorageClient 对象上传图片,扩展名不带“.”
            String[] strings = common().upload_file(local_path, file_ext_name, null);
            List<String> list = Arrays.asList(strings);
            return "http://47.100.206.217/" + list.get(0) + "/" + list.get(1);
        }
    
        private static String download(String local_path, String group, String path) throws IOException, MyException {
            byte[] file_bytes = common().download_file(group, path);
            File file = new File(local_path);
            FileOutputStream fileOutputStream = new FileOutputStream(file);
            fileOutputStream.write(file_bytes);
            fileOutputStream.flush();
            fileOutputStream.close();
            return local_path;
        }
    
        private static StorageClient common() throws IOException, MyException {
            String filePath = new ClassPathResource("fdfs_client.conf").getFile().getAbsolutePath();
            // 1、加载配置文件，配置文件中的内容就是 tracker 服务的地址。
            ClientGlobal.init(filePath);
            // 2、创建一个 TrackerClient 对象。直接 new 一个。
            TrackerClient trackerClient = new TrackerClient();
            // 3、使用 TrackerClient 对象创建连接，获得一个 TrackerServer 对象。
            TrackerServer trackerServer = trackerClient.getConnection();
            // 5、创建一个 StorageClient 对象，需要两个参数 TrackerServer 对象、StorageServer 的引用
            StorageServer storageServer = null;
            StorageClient storageClient = new StorageClient(trackerServer, storageServer);
            return storageClient;
        }
    
        public static void main(String[] args) throws IOException, MyException {
    //        String upload = upload("/Users/pengfeng/Downloads/abc.jpg", "jpg");
    //        System.out.println(upload);
            String download = download("/Users/pengfeng/Desktop/a.jpg", "group1", "M00/00/00/rBAYR1wOFNiAJCeDABLRWgUDUSc289.jpg");
            System.out.println("download = " + download);
        }    

[测试连接](http://47.100.206.217/group1/M00/00/00/rBAYR1wOFNiAJCeDABLRWgUDUSc289.jpg)




















    