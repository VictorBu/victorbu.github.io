---
title: MinIO
date: 2020-06-18 10:00:00
updated: 2020-06-18 10:00:00
categories: [IT]
tags: [MinIO, OSS, Java]
---

> MinIO 是一个非常轻量的基于 Apache License v2.0 开源协议的对象存储服务。它兼容亚马逊 S3 云存储服务接口，非常适合于存储大容量非结构化的数据，例如图片、视频、日志文件、备份数据和容器/虚拟机镜像等，而一个对象文件可以是任意大小，从几 kb 到最大 5T 不等。

# 一、MinIO 服务部署

## 1.1 下载安装包

本文使用 CentOS 7，安装包地址(中国镜像)：http://dl.minio.org.cn/server/minio/release/linux-amd64/minio


## 1.2 启动服务

```
chmod +x minio
./minio server /project/miniodata/
```

其中 /project/miniodata/ 是文件存储位置，默认端口是 9000，AccessKey 和 SecretKey 均为 minioadmin

也可以手动指定端口号：

```
./minio server --address :9001 /project/miniodata/
```

也可以手动修改 AccessKey 和 SecretKey：

```
vi /etc/profile

#在最后添加内容
export MINIO_ACCESS_KEY=admin
export MINIO_SECRET_KEY=password

# 然后执行命令
source /etc/profile
```

## 二、Java SDK 使用

## 2.1 新建项目添加配置

```
minio:
  url: http://192.168.137.101:9000
  accessKey: minioadmin
  secretKey: minioadmin

spring:
  servlet:
    multipart:
      max-file-size: 100MB
      max-request-size: 100MB
```

## 2.2 上传、下载、文件分享代码

```
@RestController
public class MinioController {

    @Autowired
    private MinioProperties minioProperties;

    @PostMapping("/upload")
    public String upload(UploadModel uploadModel) {

        if(StringUtils.isEmpty(uploadModel.getFileName())) {
            uploadModel.setFileName(uploadModel.getFile().getOriginalFilename());
        }

        try {
            // 使用 MinIO 服务的 IP:PORT，Access key 和 Secret key 创建一个 MinioClient 对象
            MinioClient minioClient = new MinioClient(
                    minioProperties.getUrl(), minioProperties.getAccessKey(), minioProperties.getSecretKey());

            // 检查存储桶是否已经存在
            boolean isBucketExist = minioClient.bucketExists(uploadModel.getBucket());
            if(!isBucketExist) {
                minioClient.makeBucket(uploadModel.getBucket());
            }

            PutObjectOptions putObjectOptions = new PutObjectOptions(uploadModel.getFile().getSize()
                    , uploadModel.getFile().getSize() >= 5242880L ? uploadModel.getFile().getSize() : 0L);

            minioClient.putObject(uploadModel.getBucket(), uploadModel.getFileName(),
                    uploadModel.getFile().getInputStream(), putObjectOptions);

        } catch (Exception e) {
            return e.getMessage();
        }
        return "success";
    }

    @RequestMapping("/download")
    public ResponseEntity<Resource> download(DownloadModel downloadModel) {
        try {
            MinioClient minioClient = new MinioClient(
                    minioProperties.getUrl(), minioProperties.getAccessKey(), minioProperties.getSecretKey());

            InputStream inputStream = minioClient.getObject(downloadModel.getBucket(), downloadModel.getFileName());
            Resource resource = new InputStreamResource(inputStream);
            return ResponseEntity.ok()
                    .header(HttpHeaders.CONTENT_DISPOSITION,
                            "attachment;filename=\"" +
                                    downloadModel.getFileName().replace('/', '-') + "\"")
                    .body(resource);
        } catch (Exception e) {
            return null;
        }
    }

    @RequestMapping("/share")
    public String share(DownloadModel downloadModel) {
        try {
            MinioClient minioClient = new MinioClient(
                    minioProperties.getUrl(), minioProperties.getAccessKey(), minioProperties.getSecretKey());

            String url = minioClient.presignedGetObject(downloadModel.getBucket(), downloadModel.getFileName(), 120);
            return url;
        } catch (Exception e) {
            return e.getMessage();
        }
    }
}
```

新建一个 Bucket 就是在文件路径新建一个目录，默认上传的文件都在该 Bucket 的根目录，如果要存储多级目录，可以指定在文件名中指定目录结构，如：img/avatar.jpg

注意校正 MinIO 服务器时间，否则可能会报错：

```
Caused by: io.minio.errors.ErrorResponseException: The difference between the request time and the server's time is too large.
```

完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/minio-demo)

参考：[MinIO 中文文档](http://docs.minio.org.cn/docs/)