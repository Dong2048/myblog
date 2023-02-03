---
title: 分布式文件系统MinIO
date: 2022-04-18 14:24:38
tags: java
cover: https://s1.ax1x.com/2022/04/24/L4O3ZV.jpg
category: 笔记
top_img: https://s1.ax1x.com/2022/04/24/L4O3ZV.jpg
---
#### MinIO基础介绍
MinIO是一个基于Apache License V2.0开源协议的对象存储服务。它兼容亚马逊S3云存储服务接口，非常适合于存储大容量非结构化的数据，例如：图片、视频、日志文件、备份数据和容器/虚拟机镜像等。
而一个对象文件可以是任意大小，从几K到最大5T不等。
MinIO是一个非常轻量的服务，可以很简单的和其他应用的结合，类似NodeJS，Redis或者MySQL  
英文API：https://min.io  
中文API：http://www.minio.org.cn
#### MinIO基础概念
- **Object** 存储到MinIO的基本对象。如文件、字节流等，举例：test。jpg、test.pdf等。
- **Bucket** 用来存储Object的逻辑空间。每个Bucket之间的数据是相互隔离的。对于客户端而言，就相当于一个存放文件的顶层文件夹。
- **Drive** 即存储数据的磁盘，在MinIO启动时，以参数的方式传入。MinIO中所有的对象数据都会存储在Drive里。
- **Set** 即一组Drive的合集，分布式部署1根据集群规模自动划分一个或多个Set、每个Set中的Drive分布在不同的位置，一个对象存储在一个Set上。  
  一个对象存储在一个Set上。  
  一个集群分为多个Set。  
  一个Set包含的Drive数量是固定的，默认由系统根据集群规模自动计算得出。  
  一个Set中的Drive尽可能分布在不同的节点上。  
#### MinIO的EC码和文件存储结构
MinIO使用纠删码机制来保证高可靠性，使用highwayhsah来处理数据损坏（Bit Rot Protection）关于纠删码，简单来说就是可以通过数学计算，
把丢失的数据进行还原，它可以将N份原始数据，增加M份数据，并能通过N+M份中的任意N份数据，还原为原始数据。即如果有任意小于等于M份的数据失效、
仍然能通过剩下的数据还原出来。
#### 基于centos7和docker单机部署
##### 二进制部署
下载minIO：wget https://dl.min.io/server/minio/release/darwin-amd64/minio  
给下载好的二进制文件权限：chmod +x minio  
执行启动minio：./minio server /data  
/data是存储数据的驱动器或目录的路径。  
单机版本无法使用，版本控制、对象锁定和存储桶复制等功能。如果需要使用，是需要部署 带有纠删码的分布式 MinIO。  
##### docker部署
docker run \  
-p 9000:9000 \  
-p 9001:9001 \  
-e "MINIO_ROOT_USER=AKIAIOSFODNN7EXAMPLE" \
-e "MINIO_ROOT_PASSWORD=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY" \
quay.io/minio/minio server /data --console-address ":9001"
MINIO_ROOT_USER为登陆用户名，MINIO_ROOT_PASSWORD为密码，9000及90001为docker向外暴露的端口号。9000为web控制台，9001为API端口。
#### MinIO纠删码模式部署
##### 二进制部署
minio server /data{1...12}
定义12个目录或磁盘
##### docker部署
podman run \
-p 9000:9000 \
-p 9001:9001 \
--name minio \
-v /mnt/data1:/data1 \
-v /mnt/data2:/data2 \
-v /mnt/data3:/data3 \
-v /mnt/data4:/data4 \
-v /mnt/data5:/data5 \
-v /mnt/data6:/data6 \
-v /mnt/data7:/data7 \
-v /mnt/data8:/data8 \
quay.io/minio/minio server /data{1...8} --console-address ":9001"
测试：随机拔出驱动器并继续在系统上执行 I/O。或者随机删除目录下文件执行I/O。
#### MinIO分布式集群部署
#### GNU/Linux 和 macOS
export MINIO_ROOT_USER=<ACCESS_KEY>
export MINIO_ROOT_PASSWORD=<SECRET_KEY>
minio server http://host{1...n}/export{1...m}
示例： minio server http://192.168.1.11/export1 http://192.168.1.12/export2 \
http://192.168.1.13/export3 http://192.168.1.14/export4 \
http://192.168.1.15/export5 http://192.168.1.16/export6 \
http://192.168.1.17/export7 http://192.168.1.18/export8
#### MinIO客户端MC使用
MinIO Client (mc) 为 UNIX 命令（如 ls、cat、cp、mirror、diff、find 等）提供了现代替代方案。它支持文件系统和兼容 Amazon S3 的云存储服务（AWS Signature v2 和 v4）。  
下载：wget https://dl.min.io/client/mc/release/linux-amd64/mc
赋予权限：chmod +x mc
查看帮助命令：./mc --help
#### java整合MinIO文件上传
文件上传示例，更多请参考API:https://docs.min.io/docs/java-client-api-reference
> import io.minio.BucketExistsArgs;
import io.minio.MakeBucketArgs;
import io.minio.MinioClient;
import io.minio.UploadObjectArgs;
import io.minio.errors.MinioException;
import java.io.IOException;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
public class FileUploader {
public static void main(String[] args)
throws IOException, NoSuchAlgorithmException, InvalidKeyException {
try {
// Create a minioClient with the MinIO server playground, its access key and secret key.
MinioClient minioClient =
MinioClient.builder()
.endpoint("https://play.min.io")
.credentials("Q3AM3UQ867SPQQA43P2F", "zuf+tfteSlswRu7BJ86wekitnifILbZam1KYY3TG")
.build();
      // Make 'asiatrip' bucket if not exist.
      boolean found =
          minioClient.bucketExists(BucketExistsArgs.builder().bucket("asiatrip").build());
      if (!found) {
        // Make a new bucket called 'asiatrip'.
        minioClient.makeBucket(MakeBucketArgs.builder().bucket("asiatrip").build());
      } else {
        System.out.println("Bucket 'asiatrip' already exists.");
      }
      // Upload '/home/user/Photos/asiaphotos.zip' as object name 'asiaphotos-2015.zip' to bucket
      // 'asiatrip'.
      minioClient.uploadObject(
          UploadObjectArgs.builder()
              .bucket("asiatrip")
              .object("asiaphotos-2015.zip")
              .filename("/home/user/Photos/asiaphotos.zip")
              .build());
      System.out.println(
          "'/home/user/Photos/asiaphotos.zip' is successfully uploaded as "
              + "object 'asiaphotos-2015.zip' to bucket 'asiatrip'.");
    } catch (MinioException e) {
      System.out.println("Error occurred: " + e);
      System.out.println("HTTP trace: " + e.httpTrace());
    }
}
}
