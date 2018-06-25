# hadoop迁移工具

## 背景
目前wcs支持的命令行同步工具只支持本地文件同步到wcs，但这个工具无法读取Hadoop文件系统的数据，从而发挥Hadoop分布式的特点。如果客户使用Hadoop集群，需要从hadoop集群上下载到本地然后再从本地上传到wcs，整个过程费时又费力。wcs开发团队开发了同步工具wcs-hadoop-migrate-tool能从Hadoop集群直接迁移数据到wcs上，通过shell命令执行脚本就可以轻松将hadoop集群上的文件迁移到wcs上。
## 使用说明
### 1. 前提条件
确保当前机器可以正常访问您的Hadoop集群，即能够用Hadoop命令访问HDFS，比如可执行如下命令：
```
hadoop fs -ls /
```
### 2. 下载wcs-hadoop-migrate-tool安装包
[wcs-hadoop-migrate-tool](https://wcsd.chinanetcenter.com/tool/wcs-hadoop-migrate-tool.zip)
当前支持的hadoop版本为2.6.x
### 3. 解压安装包
解压工具到本地目录：
```
unzip wcs-hadoop-migrate-tool.zip
```
### 4. 执行同步命令
同步HDFS数据到wcs上：
```
sh hdfs2wcs.sh  hdfspath wcspath wcsMgrDomain [-Dmapreduce.job.queuename=qname] [-overwrite] [-skipetagcheck] 
```
参数说明如下表：

| 参数名  |是否必填   |参数说明   |   
| ------------ | ------------ | ------------ | 
| hdfspath  | 是  | 指定需要同步的hadoop路径，路径格式为：hdfs://host:port/path |  
|wcspath   | 是 |指定同步到wcs的路径，路径格式为： wcs://AccessKeyId:AccessKeySecret@bucket-name.uploadDomain/path/on/wcs，其中AccessKeyId，AccessKeySecret，uploadDomain（上传域名）可以从web控制台上查询 | 
| wcsMgrDomain  | 是  |指定用户管理域名，可在web控制台上查询   |
|Dmapreduce.job.queuename|否|指定queueName，不指定时queueName默认为default|
|overwrite|否|指定是否上传覆盖，若写了-overwrite，则覆盖上传，即同步时有同名文件会覆盖，若不指定则默认不覆盖上传，即同步时有同名文件且etag相等直接跳过。<br/>注：overwrite与skipetagcheck同时指定时，skipetagcheck参数会无效，两个参数不可同时指定|
|skipetagcheck|否|若填写-skipetagcheck,则同步时已经存在的文件不会对比etag，若不指定，则同步时不管文件是否已经存在都会对比文件etag|
### 5. 使用场景说明
场景一： 覆盖迁移   
无论wcs是否已经存在目标文件，都覆盖上传到wcs，不校验wcs是否存在、文件存在时etag值是否相等，使用例子如下：
```
sh hdfs2wcs.sh  hdfs://10.8.204.209:9000/input wcs://e78f1ffbee3fe25c117d6cccb4aff1916af0516a:e8d63cd8b3ddf9d40ef3e3855bec511eaa683cdf@bucket-cache.apitestuser2.up8.v3.wcsapi.com:9090/test apitestuser2.mgr8.v3.wcsapi.com:9090  -overwrite 
```
场景二：对比文件etag迁移
wcs已经存在目标文件，则会对比文件etag，etag相等则跳过此文件；
etag不相等会重新迁移文件。默认迁移时都会做etag对比，使用例子如下：
```
sh hdfs2wcs.sh  hdfs://10.8.204.209:9000/input wcs://e78f1ffbee3fe25c117d6cccb4aff1916af0516a:e8d63cd8b3ddf9d40ef3e3855bec511eaa683cdf@bucket-cache.apitestuser2.up8.v3.wcsapi.com:9090/test apitestuser2.mgr8.v3.wcsapi.com:9090
```
场景三：不覆盖迁移
若wcs已经存在目标文件，则直接跳过此文件，不对比文件etag。使用例子如下：
```
sh hdfs2wcs.sh  hdfs://10.8.204.209:9000/input wcs://e78f1ffbee3fe25c117d6cccb4aff1916af0516a:e8d63cd8b3ddf9d40ef3e3855bec511eaa683cdf@bucket-cache.apitestuser2.up8.v3.wcsapi.com:9090/test apitestuser2.mgr8.v3.wcsapi.com:9090 -skipetagcheck
```


