# hadoop Migration Tool

## Language
- [简体中文](README.md)
- [English](README.en.md)

## Background
Currently, the command-line synchronization tool supported by wcs only supports local file synchronization to wcs, but this tool cannot read data from the Hadoop file system, thus taking advantage of Hadoop's distributed features. If customers use Hadoop clusters, they need to download from the Hadoop cluster to the local and then upload from the local to wcs, which is time-consuming and labor-intensive. The wcs development team has developed a synchronization tool wcs-hadoop-migrate-tool that can directly migrate data from Hadoop clusters to wcs. By executing scripts through shell commands, you can easily migrate files from the Hadoop cluster to wcs.

## Usage Instructions
### 1. Prerequisites
Ensure that the current machine can normally access your Hadoop cluster, that is, you can access HDFS with Hadoop commands, for example, you can execute the following command:
```
hadoop fs -ls /
```

### 2. Download wcs-hadoop-migrate-tool installation package
[wcs-hadoop-migrate-tool](https://wcsd.chinanetcenter.com/tool/wcs-hadoop-migrate-tool.zip)
The currently supported hadoop version is 2.6.x

### 3. Unzip the installation package
Unzip the tool to the local directory:
```
unzip wcs-hadoop-migrate-tool.zip
```

### 4. Execute synchronization command
Synchronize HDFS data to wcs:
```
sh hdfs2wcs.sh  hdfspath wcspath wcsMgrDomain [-Dmapreduce.job.queuename=qname] [-overwrite] [-skipetagcheck] 
```

Parameter description is as follows:

| Parameter name | Required | Parameter description |
| ------------ | ------------ | ------------ |
| hdfspath | Yes | Specify the hadoop path to be synchronized, the path format is: hdfs://host:port/path |
| wcspath | Yes | Specify the path to synchronize to wcs, the path format is: wcs://AccessKeyId:AccessKeySecret@bucket-name.uploadDomain/path/on/wcs, where AccessKeyId, AccessKeySecret, uploadDomain (upload domain name) can be queried from the web console |
| wcsMgrDomain | Yes | Specify the user management domain name, which can be queried on the web console |
| Dmapreduce.job.queuename | No | Specify queueName, if not specified, queueName defaults to default |
| overwrite | No | Specify whether to upload and overwrite. If -overwrite is written, it will overwrite and upload, that is, files with the same name will be overwritten during synchronization. If not specified, the default is not to overwrite upload, that is, files with the same name during synchronization and etag are equal to skip directly. <br/>Note: When overwrite and skipetagcheck are specified at the same time, the skipetagcheck parameter will be invalid, and the two parameters cannot be specified at the same time |
| skipetagcheck | No | If -skipetagcheck is filled in, the existing files will not be compared with etag during synchronization. If not specified, the file etag will be compared regardless of whether the file already exists during synchronization |

### 5. Usage scenario description
Scenario 1: Overwrite migration
No matter whether the target file already exists in wcs, it will be overwritten and uploaded to wcs, and it will not check whether wcs exists or whether the etag value is equal when the file exists. The usage example is as follows:
```
sh hdfs2wcs.sh  hdfs://10.8.204.209:9000/input wcs://e78f1ffbee3fe25c117d6cccb4aff1916af0516a:e8d63cd8b3ddf9d40ef3e3855bec511eaa683cdf@bucket-cache.apitestuser2.up8.v3.wcsapi.com:9090/test apitestuser2.mgr8.v3.wcsapi.com:9090  -overwrite 
```

Scenario 2: Compare file etag migration
If the target file already exists in wcs, the file etag will be compared, and if the etag is equal, this file will be skipped;
If the etag is not equal, the file will be migrated again. By default, etag comparison will be done during migration. The usage example is as follows:
```
sh hdfs2wcs.sh  hdfs://10.8.204.209:9000/input wcs://e78f1ffbee3fe25c117d6cccb4aff1916af0516a:e8d63cd8b3ddf9d40ef3e3855bec511eaa683cdf@bucket-cache.apitestuser2.up8.v3.wcsapi.com:9090/test apitestuser2.mgr8.v3.wcsapi.com:9090
```

Scenario 3: No overwrite migration
If the target file already exists in wcs, this file will be skipped directly without comparing the file etag. The usage example is as follows:
```
sh hdfs2wcs.sh  hdfs://10.8.204.209:9000/input wcs://e78f1ffbee3fe25c117d6cccb4aff1916af0516a:e8d63cd8b3ddf9d40ef3e3855bec511eaa683cdf@bucket-cache.apitestuser2.up8.v3.wcsapi.com:9090/test apitestuser2.mgr8.v3.wcsapi.com:9090 -skipetagcheck
```