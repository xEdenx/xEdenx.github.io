---
title: 使用 Jenkins 自动化部署 SpringBoot 应用
description: 使用 Jenkins 自动化构建和部署 Spring Boot 应用。
publishDate: 2018-08-23 14:55:39
tags:
  - spring boot
  - Jenkins
  - DevOps
---

## Step 1 : 参数化构建过程

```
添加字符参数
名称 ：BUILD_ID
默认值： dontKillMe
```

## Step 2 : Build

```
Root POM ：pom.xml //相对路径
Goals and options： clean package -Dmaven.test.skip=true
```

## Setp 3 : Post Steps

```
cp apigateway/target/*.jar /opt/jarbak
/opt/restart.sh 8085 apigateway/target/*.jar
```

## Step 4 : Build with Parameters

## restart.sh

```shell
#! /bin/bash
# param1 : port should be killed , param2 : jar should be started , param3 : remote debug port
# ./restart.sh 8085 /target/apigateway.jar 8086
#! /bin/bash
echo "端口号：$1";
pid=`netstat -nlp | grep -w $1 | sed -r 's#.* (.*)/.*#\1#'`
echo "Old pid $pid will be killed"
kill -9 $pid
sleep 3s
echo "启动服务：$2"

if [ "$3" -gt 0 ] 2>/dev/null;
then
  debugPid=`netstat -nlp | grep -w $3 | sed -r 's#.* (.*)/.*#\1#'`
  echo "Old debug pid $debugPid will be killed"
  kill -9 $debugPid
  sleep 3s
  echo "Remote debug port is $3 ."
  nohup java -jar -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=$3 $2 &
else
  nohup java -jar $2 &
fi

sleep 30s
```

shell permission : `chmod +x restart.sh`

restart Jenkins : `systemctl restart jenkins`
