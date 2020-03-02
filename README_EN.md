godfs
==========
[![Build Status](https://travis-ci.org/hetianyi/godfs.svg?branch=master)](https://travis-ci.org/hetianyi/godfs)
[![go report card](https://goreportcard.com/badge/github.com/hetianyi/godfs "go report card")](https://goreportcard.com/report/github.com/hetianyi/godfs)

[README](README_EN.md) | [中文文档](README.md)
# Description

### ```godfs``` is a simple fast, easy use distributed file system written by golang.

```godfs``` provides out-of-the-box usage and friendly support for docker，

You can pull the image on docker hub:
[https://hub.docker.com/r/hehety/godfs/](https://hub.docker.com/r/hehety/godfs/)

##### [2019-03-28 UPDATE] Latest Version 1.1.1-dev and later is NOT compatible with previous versions!

##### [2019-01-17 UPDATE] Latest Version 1.1.0+ is NOT compatible with previous versions!

##### [2018-12-05 UPDATE] Godfs is now support dashboard for monitoring godfs running state!

project is here:[https://github.com/hetianyi/godfs-dashboard](https://github.com/hetianyi/godfs-dashboard)

![architecture](/doc/20180830151005.png)

## Features

- Fast, lightweight, stable, out-of-the-box, friendly api
- Easy to expand, Stable to RUN
- Low resource overhead
- Native client api and java client api
- API for http upload and download
- Support file breakpoint download
- Support basic verification for http download and upload
- Cross-site resource protection
- Clear logs help troubleshoot errors
- Support different platforms: Linux, Windows, Mac
- Better support for docker
- File fragmentation storage
- Better data migration solution
- Support readonly node
- File synchronization in same group
- Support dashboard(beta, in development)
- Support access token

## Install

> Please install golang1.8+ first!

Take CentOS 7 as example.

### build from latest source code:
```shell
yum install golang -y
git clone https://github.com/hetianyi/godfs.git
cd godfs
./make.sh
# on windows you just need to click 'make.cmd'
```
After the build is successful, three files will be generated under the `````./bin````` directory:
```shell
./bin/client
./bin/storage
./bin/tracker
./bin/bashboard
```

Install godfs binary files to ```/usr/local/godfs```:
```shell
./install.sh /usr/local/godfs
```
You can start tracker server by:
```shell
/usr/local/godfs/bin/tracker [-c /your/tracker/config/path]
```
and start storage node by:
```shell
/usr/local/godfs/bin/storage [-c /your/storage/config/path]
```
then you can using command ```client``` directly in command line to upload and download file.
> Of course, you must first set up the tracker server.
```shell
# set up tracker servers for client
client config set "trackers=host1:port1[,host2:port2]" "log_level=debug" ...
# config secret
client config set "secret=OASAD834jA97AAQE761=="
# print client config
client config ls
```

For example, you can upload file by:
```shell
# upload a file
client upload /f/project.rar
```
![architecture](/doc/20180828095840.png)

If you want to upload file to specified group, you can add flag ```-g <groupID>``` in command line.
also, it's cool that you can upload all files in a directory by:
```shell
client upload *
```
![architecture](/doc/20180828100341.png)

if you don't has a godfs client, you can use ```curl``` to upload files by:
```shell
curl -F "file=@/your/file" "http://your.host:http_port/upload"
```
if upload success, server will return a json string like this:
```json
{
    "status":"success",
    "formData":{
        "data":[
            "G01/01/M/826d552525bceec5b8e9709efaf481ec"
        ],
        "name":[
            "mike"
        ]
    },
    "fileInfo":[
        {
            "index":0,
            "fileName":"mysql-cluster-community-7.6.7-1.sles12.x86_64.rpm-bundle.tar",
            "path":"G01/01/M/826d552525bceec5b8e9709efaf481ec"
        }
    ]
}
```

> The ```formData``` contains all parameters of your posted form, the file will be replaced by a remote path(none english characters will encode to hex string, such as '图片' --> '\\u56fe\\u7247').
If you want to upload file to specified group, you can add parameter ```?group=<groupID>``` to the request path.

```shell
# download a file as 123.zip
client download G01/10/M/2c9da7ea280c020db7f4879f8180dfd6 --name 123.zip
```



#### Usage of Token

token refers from FastDFS, it can control a file be accessable within a certain time.

you need to generate token by yourself on backend, godfs only need to calculate from request parameters and compare them, token request format is：

http://...?tk=<md5>&ts=<timestamp>

token calculatation：

md5(timestamp+filemd5+secret) ==> token



### build docker image from latest source code:

```shell
cd godfs/docker
docker build -t godfs .
```
It is highly recommended to use docker to run godfs.
You can pull the docker image from [docker hub](https://hub.docker.com/r/hehety/godfs/):
```shell
docker pull hehety/godfs
```

start tracker using docker:
```shell
docker run -d -p 1022:1022 --name tracker --restart always -v /godfs/data:/godfs/data --privileged -e log_level="info" hehety/godfs:latest tracker
```

start storage using docker:
```shell
docker run -d -p 1024:1024 -p 80:8001 --name storage -v /godfs/data:/godfs/data --privileged -e trackers=192.168.1.172:1022 -e advertise_addr=192.168.1.187 -e port=1024  -e instance_id="01" hehety/godfs storage
# you'd better add docker command '-e port=1024' on single machine.  
```
we're here using directory ```/godfs/data``` to persist data, you can use ```-e``` in docker command line to override default configuration.

client usage:
```shell
NAME:
   godfs client cli
USAGE:
   client [global options] command [command options] [arguments...]
VERSION:
   1.1.0-beta
COMMANDS:
     upload    upload local files
     download  download a file
     inspect   inspect files information by md5
     config    client cli configuration settings operation
     help, h   Shows a list of commands or help for one command
GLOBAL OPTIONS:
   --trackers value               tracker servers (default: "127.0.0.1:1022")
   --log_level value              log level (trace, debug, info, warm, error, fatal) (default: "info")
   --log_rotation_interval value  log rotation interval h(hour),d(day),m(month),y(year) (default: "d")
   --secret value                 secret of trackers (trace, debug, info, warm, error, fatal)
   --help, -h                     show help
   --version, -v                  print the version
```



## Monitoring

godfs monitoring project is now available at [HERE](https://github.com/hetianyi/godfs-dashboard)

This project is currently under development and can monitor some basic status information of godfs.

You can use it out of the box by docker([HERE](https://github.com/hetianyi/godfs-dashboard)).

```shell
# run the dashboard
docker run -d -p 8080:80 --restart always --name godfs-dashboard hehety/godfs-dashboard
```

![architecture](/doc/20181205154643.png)

![architecture](/doc/20181205154909.png)



## Tcp client based stress test on workstations (v1.1.0 and later)

[Test case is here](https://github.com/hetianyi/godfs/tree/master/example)

Machine configuration

| Node     | CPU                                      | CPU core number | Memory | Disk |
| -------- | ---------------------------------------- | :-------------: | :----: | ---- |
| tracker1 | Intel(R) Xeon(R) CPU E5-1620 0 @ 3.60GHz |        1        |  1GB   | SSD  |
| tracker2 | Intel(R) Xeon(R) CPU E5-1620 0 @ 3.60GHz |        1        |  1GB   | SSD  |
| storage1 | Intel(R) Xeon(R) CPU E5-1620 0 @ 3.60GHz |        2        |  1GB   | SSD  |
| storage2 | Intel(R) Xeon(R) CPU E5-1620 0 @ 3.60GHz |        2        |  1GB   | SSD  |

Test summary

| Key                       | Value                     |
| ------------------------- | ------------------------- |
| type                      | VM                        |
| operating System          | CentOS7                   |
| tracker number            | 2                         |
| group number              | 2（G01,G02）              |
| storage number            | 4（G01x2,G02x2）          |
| docker version            | 18.06.1-ce, build e68fc7a |
| total files               | 5,000,000                 |
| threads                   | 5                         |
| total times               | 2h 35min                  |
| System average throughput | 537 files/s               |
| Single node throughput    | 134 files/s               |
| Failure                   | 0                         |

schematic diagram

![architecture](doc/20190215153655.png)



## Simple load test on vultr(before v1.1.0)

### HTTP upload test
|Name|Value|
|---|---|
| OS        | CentOS7   |
| RAM       | 1GB       |
| CPU core  | 1         |
| DISK      | 60GB SSD  |

#### Test description
Generate 500w simple files, the file content
is just a number from 1 to 5000000.
and they were uploaded in 5 different threads by curl command(http upload).


The test took 41.26 hours with not a single error which means 33.7 files uploaded per second.

The CPU usage of the host in the test was kept at 60%-70%, and the memory consumed by the tracker and storage were both less than 30M.
>The test and one tracker server, one storage server are on the same machine.
This test shows that godfs has no problem in handling large concurrent (for file system) uploads and database writes, and the system performs very stable.

Test tool is available in release page.

### HTTP download test

storage server configuration(California)

|Name|Value|
|---|---|
| OS        | CentOS7   |
| RAM       | 512M       |
| CPU core  | 1         |
| DISK      | SSD  |

download client machine configuration(Los Angeles)

|Name|Value|
|---|---|
| OS        | CentOS7   |
| RAM       | 8GB       |
| CPU core  | 4         |
| DISK      | SSD  |


#### Test description
We are here using apache [jmeter-5.0](http://jmeter.apache.org/download_jmeter.cgi) as test tool.

In the test, we used 20 threads to download 4 files of different sizes (less than 1MB), each thread running 100000 times, a total of 800000.


| Label | # Samples | Average | Median | 90% Line | 95% Line | 99% Line | Min  | Max  | Error % | Throughput | Received KB/sec | Sent KB/sec |
| ----- | --------- | ------- | ------ | -------- | -------- | -------- | ---- | ---- | ------- | ---------- | --------------- | ----------- |
| 1     | 200000    | 78      | 72     | 116      | 135      | 195      | 5    | 8377 | 0.00%   | 79.88966   | 20079.84        | 13.65       |
| 2     | 200000    | 39      | 36     | 66       | 76       | 106      | 2    | 2715 | 0.00%   | 79.89087   | 11202.43        | 13.65       |
| 3     | 200000    | 76      | 65     | 125      | 154      | 238      | 5    | 8641 | 0.00%   | 79.89052   | 37493.62        | 13.65       |
| 4     | 200000    | 55      | 49     | 91       | 111      | 171      | 4    | 2789 | 0.00%   | 79.891     | 23045.82        | 13.65       |
| Total | 800000    | 62      | 55     | 104      | 126      | 193      | 2    | 8641 | 0.00%   | 319.55492  | 91819.69        | 54.61       |


![architecture](/doc/response-time.png)

**Test result:**

| Total       | 800000      |
| ----------- | ----------- |
| Threads     | 20          |
| Total times | 41min       |
| requests    | 319.55492/s |
| avg time    | 62ms        |
| success     | 100%        |
| error       | 0%          |

And I will do more test in the future.



## Update logs

2019/03/28
1. Refer to FastDFS to implement the token mechanism.


2019/01/17

1. Import many 3rd part opensource libs for better performance include:

   [github.com/mattn/go-sqlite3](https://github.com/mattn/go-sqlite3)

   [github.com/jinzhu/gorm](github.com/jinzhu/gorm)

   [github.com/json-iterator/go](github.com/json-iterator/go)

   [github.com/urfave/cli](https://github.com/urfave/cli)

2. Rewrite the underlying communication protocol which makes the program more scalable.

3. Code refactoring

4. Redesign client commands